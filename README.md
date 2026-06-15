# client-api

Microsserviço responsável pelo cadastro e consulta de clientes no ecossistema JazzTech.

## Visão Geral

A API gerencia o cadastro de **clientes** com enriquecimento automático de endereço via integração com a API pública **ViaCep**. O CPF é validado e formatado automaticamente; o endereço é preenchido a partir do CEP informado. Este serviço é o ponto de entrada do ecossistema — tanto o `credit-analysis-api` quanto o `card-holder-api` dependem dos clientes aqui cadastrados.

## Stack

- **Java 17** + **Spring Boot 3.0.6**
- **Spring Cloud OpenFeign** — integração com ViaCep
- **Spring Data JPA** + **PostgreSQL** + **Undertow** (no lugar de Tomcat)
- **MapStruct** — mapeamento de objetos em 3 camadas (request → domain → entity)
- **Lombok** — redução de boilerplate
- **JUnit 5** + **Mockito** — testes unitários

## Pré-requisitos

- Java 17+
- Docker + Docker Compose

## Rodando localmente

```bash
# Subir banco de dados PostgreSQL
make local-env-create

# Iniciar a aplicação
./mvnw spring-boot:run          # Linux/Mac
mvnw.cmd spring-boot:run        # Windows
```

A API fica disponível em `http://localhost:8080`.

## Endpoints

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `POST` | `/v1.0/clients` | Criar cliente (enriquece endereço via ViaCep) |
| `GET` | `/v1.0/clients/{id}` | Buscar cliente por ID |
| `GET` | `/v1.0/clients?cpf=xxx` | Buscar por CPF ou listar todos |

## Banco de dados

O schema está em `data/ddl.sql`. PostgreSQL sobe via Docker na porta `5432`.

```bash
make local-env-create   # Sobe o ambiente
make local-env-destroy  # Derruba o ambiente
```

## Testes

```bash
./mvnw test           # Roda todos os testes
./mvnw verify         # Testes + relatório JaCoCo (target/site/jacoco/)
```

## Microsserviços relacionados

| Serviço | Porta | Função |
|---------|-------|--------|
| `client-api` | 8080 | **Este serviço** — cadastro de clientes |
| `credit-analysis-api` | 9001 | Análise e aprovação de crédito |
| `card-holder-api` | 9002 | Portadores e cartões de crédito |