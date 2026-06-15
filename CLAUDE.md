# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Comandos

```bash
# Build completo (roda checkstyle na fase validate + testes)
./mvnw clean package          # Linux/Mac
mvnw.cmd clean package        # Windows

# Somente testes
./mvnw test

# Teste único
./mvnw test -Dtest=ClientServiceTest

# Build sem testes
./mvnw clean package -DskipTests

# Somente checkstyle
./mvnw checkstyle:check

# Relatório JaCoCo (gerado em target/site/jacoco/)
./mvnw verify
```

### Ambiente local

```bash
make local-env-create   # Sobe PostgreSQL via docker-compose (stack.yaml) e executa DDL
make local-env-destroy  # Derruba o ambiente
```

O PostgreSQL sobe na porta `5432` com usuário `admin`, senha `senha123`, banco `postgres`. O schema está em `data/ddl.sql` (tabelas `CLIENT` e `ADDRESS`).

## Arquitetura

Projeto Spring Boot 3 com camadas planas (sem Hexagonal):

```
controller/          → REST endpoints + request/response DTOs
service/             → Lógica de negócio (ClientService)
model/               → Domain records Java (Client, Address) com validação no construtor
repository/          → Spring Data JPA + entity classes (ClientEntity, AddressEntity)
apiclient/           → Feign client para ViaCep + DTO de resposta
mapper/              → Interfaces MapStruct (geradas em compile-time)
exception/           → Exceções de negócio
exceptionhandler/    → @ControllerAdvice com ProblemDetail (RFC 7807)
utils/               → ValidationCustom (dispara Bean Validation manualmente no record)
```

### Fluxo de criação de cliente

`POST /v1.0/clients` → `ClientController` → `ClientService.create()`:
1. Converte `ClientRequest` → `Client` (record de domínio) via `ClientMapper`; o construtor do record formata o CPF e dispara Bean Validation.
2. Consulta CEP na API ViaCep via `ViaCepApiClient` (OpenFeign); lança `AddressNotFoundException` se `cep` retornado for `null`.
3. Atualiza os campos de endereço no record via `Client.updateAddressFrom()`.
4. Converte `Client` → `ClientEntity` via `ClientEntityMapper` e persiste; `DataIntegrityViolationException` é capturada e relançada como `CPFAlreadyExistException`.
5. Retorna `ClientResponse` via `ClientResponseMapper`.

### Convenções importantes

- **Records como modelo de domínio**: `Client` e `Address` são Java records imutáveis; mutação se dá via `toBuilder()` do Lombok.
- **MapStruct com Spring**: todos os mappers usam `componentModel = "spring"` e são injetados como beans.
- **Sem migrations automáticas**: `generate-ddl: false` no JPA; o schema é gerenciado manualmente pelo `data/ddl.sql`.
- **Undertow no lugar de Tomcat**: o starter Tomcat está excluído; o servidor embarcado é o Undertow.

## Checkstyle

O `checkstyle.xml` (Google Style Guide modificado) roda na fase `validate` e bloqueia o build. Regras mais restritivas:
- Indentação: **4 espaços** (tab proibido)
- Linha máxima: **150 caracteres**
- Star imports proibidos
- Magic numbers proibidos (exceto `0`, `0.5`, `1`)
- Complexidade ciclomática máxima: **10** por método

## Testes

Testes unitários em `src/test/java/com/app/client/service/` usando JUnit 5 + Mockito. Os mappers MapStruct são usados com `@Spy` instanciando as implementações geradas (ex: `new ClientMapperImpl()`), enquanto repositório e Feign client são `@Mock`. Não há testes de integração.

## Microsserviços relacionados

Este serviço é o ponto de entrada do ecossistema — os outros dois dependem dos clientes aqui cadastrados.

| Serviço | Porta | Função |
|---------|-------|--------|
| `client-api` | 8080 | **Este serviço** — cadastro de clientes |
| `credit-analysis-api` | 9001 | Análise e aprovação de crédito |
| `card-holder-api` | 9002 | Portadores e cartões de crédito |
