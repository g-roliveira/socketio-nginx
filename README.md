# Configuração do Nginx para o Socket.IO com SSL

Este documento descreve uma configuração básica do Nginx para uso com o Socket.IO e SSL.

## Pré-requisitos

1. Servidor Nginx instalado.
2. Servidor Socket.IO rodando na porta 3000 (ou a porta que você preferir).
3. Certificados SSL (no nosso exemplo, vamos usar os certificados do Let's Encrypt).

## Configuração

Abra o arquivo de configuração do Nginx. Dependendo da sua instalação, este arquivo estará em um local como `/etc/nginx/nginx.conf` ou `/etc/nginx/sites-available/default`.

A seguir, está a configuração recomendada. Lembre-se de substituir `seu_dominio.com` pelo seu domínio real e de ajustar a localização dos seus certificados.

```nginx
server {
    listen 80;
    server_name seu_dominio.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name seu_dominio.com;

    ssl_certificate /etc/letsencrypt/live/seu_dominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/seu_dominio.com/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:3000;
    }
}
```

## Explicação da Configuração
- `listen 80`; e `return 301 https://$host$request_uri;`: Essas duas linhas configuram o servidor para ouvir na porta 80 (HTTP) e redirecionar todas as solicitações HTTP para HTTPS.

- `listen 443 ssl;`: Esta linha faz com que o servidor ouça na porta 443 (HTTPS) e habilite a SSL.

- `ssl_certificate` e `ssl_certificate_key`: Estas linhas especificam os caminhos para os arquivos de certificado e chave SSL.

- As linhas `proxy_set_header` e `proxy_pass` configuram o proxy para o seu servidor Socket.IO.

Depois de fazer essas alterações, salve o arquivo de configuração e reinicie o Nginx para aplicar as alterações.

```bash
sudo systemctl restart nginx
```
