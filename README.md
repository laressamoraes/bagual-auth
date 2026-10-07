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
3. Copiar o Client Secret (aba Credentials do client) e colocá-lo em um arquivo `.env` na raiz do `transaction`, no formato `KEYCLOAK_CLIENT_SECRET=<secret>`;
4. Criar um usuário de teste, preenchendo e-mail, nome e sobrenome (campos obrigatórios) e definir uma senha não temporária.

## Como obter um token
O Keycloak não possui tela de login para a API: o token é pedido por requisição HTTP, e o Postman é a forma mais simples de se fazer isso.

**1. Criar a requisição no Postman**
* Método: `POST`
* URL: `http://localhost:8180/realms/bagual-bank/protocol/openid-connect/token`
* Aba **Body** -> opção `x-www-form-urlencoded`, com os campos abaixo:

| Key | Value |
| --- | --- |
|`grant_type`| `password`|
|`client_id`| `bagual-client`|
|`client_secret`| Client secret do `bagual-client`|
|`username`| Usuário criado no Keycloak |
|`password`| Senha do usuário |

**2. Enviar e copiar o token**
Clique em **Send**. A resposta é um JSON, e o valor de `access_token` é o JWT a ser usado nas chamadas.

**3. Usar o token na API**
Na requisição para qualquer serviço (por exemplo: `GET http://localhost:8081/accounts`), abra a aba **Authorization**, escolha **Bearer Token** e cole o `access_token`.

O token expira em cerca de 5 minutos. Depois disso, basta repetir a requisição do passo 1 para gerar outro.

**Alternativa pelo terminal**

No PowerShell use `curl.exe`, porque `curl` é um alias de outro comando:

```bash
curl.exe -X POST http://localhost:8180/realms/bagual-bank/protocol/openid-connect/token -d "grant_type=password" -d "client_id=bagual-client" -d "client_secret=<CLIENT_SECRET>" -d "username=<USUARIO>" -d "password=<SENHA>"
```

## Problemas comuns
* `401` ao chamar a API: o token expirou ou não foi enviado. Abrir a URL direto no navegador também retorna `401`, porque o navegador não envia token. Isso é esperado e mostra que o serviço está protegido;
* `unauthorized_client` ao pedir o token: o `client_secret` está errado. Confira se copiou o Client secret da aba Credentials, e não um `access_token`. Se o Keycloak foi recriado, o secret mudou;
* `Account is not fully set up`: preencha e-mail, nome e sobrenome do usuário no Keycloak;
* `invalid_grant` com usuário e senha certos: confirme que a senha foi definida com **Temporary** desligado.
