# client-api

Microsserviço responsável pelo cadastro e consulta de clientes no ecossistema CardApp.

## Visão Geral

Gerencia o cadastro de **clientes** com enriquecimento automático de endereço via **ViaCep**. O CPF é validado; o endereço (rua, bairro, cidade, estado) é preenchido automaticamente a partir do CEP informado. Este é o ponto de entrada do ecossistema — `credit-analysis-api` e `card-holder-api` dependem dos clientes aqui cadastrados.

## Stack

- **Java 17** + **Spring Boot 3.0.6**
- **Spring Cloud OpenFeign** — integração com ViaCep
- **Spring Data JPA** + **PostgreSQL** + **Undertow**
- **MapStruct** — mapeamento request → domain → entity
- **Lombok** | **JUnit 5** + **Mockito**

## Rodando

### Opção 1 — Docker Compose (recomendado)

Sobe todos os serviços do ecossistema de uma vez. Ver [README raiz](../README.md).

### Opção 2 — Local

**Pré-requisitos:** Java 17+, Docker, Maven 3.8+

```bash
# 1. Subir o banco PostgreSQL (porta 5432)
docker-compose -f stack.yaml up -d

# 2. Criar as tabelas
docker cp data/ddl.sql postgres:/tmp/ddl.sql
docker exec postgres psql -U admin -d postgres -f /tmp/ddl.sql

# 3. Iniciar a aplicação
mvn spring-boot:run
# ou, se preferir o wrapper:
./mvnw spring-boot:run      # Linux/Mac
mvnw.cmd spring-boot:run    # Windows

# Parar o banco
docker-compose -f stack.yaml down
```

A API fica disponível em `http://localhost:8080`.

## Endpoints

| Método | Endpoint | Status | Descrição |
|--------|----------|--------|-----------|
| `POST` | `/v1.0/clients` | 201 | Criar cliente |
| `GET` | `/v1.0/clients/{id}` | 200 | Buscar por ID |
| `GET` | `/v1.0/clients?cpf=xxx` | 200 | Buscar por CPF ou listar todos |

### Exemplo — Criar cliente

```http
POST /v1.0/clients
Content-Type: application/json

{
  "name": "João Silva",
  "cpf": "12345678900",
  "birthdate": "1990-05-15",
  "address": {
    "cep": "01310-100",
    "houseNumber": 1000,
    "complement": "Apto 42"
  }
}
```

## Banco de dados

Schema em `data/ddl.sql`. PostgreSQL na porta `5432`, usuário `admin`, senha `senha123`, banco `postgres`.

## Testes

```bash
mvn test              # Todos os testes
mvn verify            # Testes + relatório JaCoCo (target/site/jacoco/)
mvn checkstyle:check  # Verificar estilo
```

## Microsserviços relacionados

| Serviço | Porta | Função |
|---------|-------|--------|
| `client-api` | 8080 | **Este serviço** |
| `credit-analysis-api` | 9001 | Análise de crédito |
| `card-holder-api` | 9002 | Portadores e cartões |
