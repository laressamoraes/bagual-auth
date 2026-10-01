# auth
Serviço responsável pela autenticação e autorização do Bagual Bank.

## Sobre o serviço
Fornece autenticação OAuth2/JWT para os microsserviços `account`, `transaction` e `notification`, que atuam como Resource Servers, validando os tokens emitidos.

Dois fluxos são utilizados:

* **Password grant:** para usuários reais, obtendo um token via usuário e senha (utilizado em testes manuais via Postman);
* **Client credentials grant:** para comunicação entre serviços: o `transaction` obtém seu próprio token para se autenticar ao chamar o `account`, sem envolver um usuário.

## Tecnologias
- Keycloak 25;
- PostgreSQL;
- Docker e Docker Compose.

## Decisões técnicas
* **Keycloak como Authorization Server:** open source, padrão de mercado, com interface administrativa completa para gerenciar Realms, Clients e usuários sem a necessidade de escrever código de autenticação do zero;
* **Postgres dedicado:** substitui o banco H2 em memória padrão, garantindo que a configuração (Realm, Client, usuários) persista entre reinícios do container.

## Como executar
Pré-requisito: Docker Desktop instalado e em execução.

```bash
docker compose up -d
```
O Admin Console fica disponível em: http://localhost:8180

Login padrão: `admin` / `admin`

## Configuração necessária

Após subir o Keycloak pela primeira vez é necessário configurar manualmente:

1. Criar o Realm `bagual-bank`;
2. Criar o Client `bagual-client`, com o Client authentication ativo e os fluxos Standard flow, Direct access grants e Service accounts roles habilitados;
3. Copiar o Client Secret (aba Credentials do client) e configurá-lo como variável de ambiente `KEYCLOAK_CLIENT_SECRET` no `transaction`;
4. Criar um usuário de teste, preenchendo e-mail, nome e sobrenome (campos obrigatórios) e definir uma senha não temporária.
