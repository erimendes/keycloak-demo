# Servidores
## Arquivo de hosts
Acrescentar as linhas abaixo:
```
127.0.0.1   meusite.local
127.0.0.1   admin.meusite.local
127.0.0.1   auth.meusite.local
127.0.0.1   green.meusite.local
127.0.0.1   blue.meusite.local
127.0.0.1   red.meusite.local
127.0.0.1   nodejs.meusite.local
```

### Windows
Editar o arquivo "C:\Windows\System32\drivers\etc\hosts".

### Linux
Editar o arquivo "/etc/hosts"

## Iniciar os containers com o commando
docker compose up -d

## O que possui em cada container

### nginx proxy manager
Proxy reverso - O unico que precisa de porta no host
Ele recebe todas as requisições e redireciona os protegidos para o keycloak

#### Primeiro login: - meusite.local:81 (Gerenciador do proxy manager)
- user: admin@example.com
- pass: changeme

Troque o email e senha.

#### Acrescentar um proxys para cada um dos hosts/containers:
- Exemplo
    Domain Name: exemplo.meusite.local
    Schema: http
    Forward Hostname/IP: <container name>
    Forward Port: <Sevice Port> (Cada aplicação tem sua porta, não precia esta no host, não precisa ta no compose, é especifico da aplicação)

- KeyCloak (admin)

```
1. Details
        Domain Name: admin.meusite.local
        Schema: http
        Forward Hostname/IP: keycloak
        Forward Port: 8080
```

```
2- Custom locations
        Define location: =/
        Schema: http
        Forward Hostname/IP: keycloak
        Forward Port: 8080
        Settings:
            - return 301 /admin;
```

```
3- Advanced
        proxy_buffer_size   128k;
        proxy_buffers   4 256k;
        proxy_busy_buffers_size   256k;
```

- Green
```
1- Details
        Domain Name: green.meusite.local
        Schema: http
        Forward Hostname/IP: green
        Forward Port: 80
```

- Blue
```
1- Details
        Domain Name: blue.meusite.local
        Schema: http
        Forward Hostname/IP: blue
        Forward Port: 80
```

```
2- Custom locations
        a- Location 1
            Define location: /
            Schema: http
            Forward Hostname/IP: blue
            Forward Port: 80
            Settings:
                - auth_request     /oauth2/auth;
                - error_page 401 = @error401;
        b- Location 2
            Define location: /oauth2/auth
            Schema: http
            Forward Hostname/IP: keycloak
            Forward Port: 4180
            Settings:
                - auth_request     /oauth2/auth;
                - error_page 401 = @error401;
```

```
3- Advanced
        location @error401 {
            return 302 https://auth.meusite.local/oauth2/start?rd=https://$host$uri;
        }
```


- Red
```
    Domain Name: red.meusite.local
```

- NodeJS
```
    Domain Name: nodejs.meusite.local
```


### KeyCloak
Servidor de Autenticação
Se o usuário tiver acesso, é redirecionado para o serviço que pretende

1- Acessar o painel de Admin admin.meusite.local
2- Criar um ClientID

5- Entrar no ClientID
6- Na aba de Settings:
    - Alterar o Valid redirect URIs para https://auth.meusite.local/oauth2/callback
    - Valid post logout redirect URIs para https://auth.example.local/oauth2/sign_out
7- Na abap Credentials copiar o clientID e atualizar na variavel de ambiente do keycloak, no docker compose, OAUTH2_PROXY_CLIENT_SECRET

### nginx Webservice - green
Não precisa de autenticação
Qualquer um PODE entrar aqui, SEM login

### nginx Webservice - blue
Precisa de autenticação
NÃO PODE conseguir entrar aqui, SEM login

### nginx Webservice - red
Precisa de autenticação
NÃO PODE conseguir entrar aqui, SEM login

### nginx Webservice - NodeJS (Precisa de autenticação)
O código NodeJS precisa identificar quem está autenticado
NÃO PODE conseguir entrar aqui, SEM login