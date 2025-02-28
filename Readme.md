# PGROK

## 1. 服务端安装
配置 Google OAuth2 认证
https://console.cloud.google.com/auth/clients?project=supple-alpha-444509-n2

生成 Nginx 配置文件
```sh
cat << EOF > ./etc/nginx/nginx.conf
server {

        listen       80;
        server_name *.pgrok.com;

        location / {
            proxy_pass http://pgrokd:3000;
        }

}

server {

    listen       80;
    server_name pgrok.com;

    location / {
        proxy_pass http://pgrokd:3320;
    }
}
EOF
```

生成 Pgrokd 配置文件
```sh
cat << EOF > ./etc/pgrokd/pgrokd.yml
external_url: "http://pgrok.com:88"
web:
  port: 3320
proxy:
  port: 3000
  scheme: "http"
  domain: "pgrok.com"
sshd:
  port: 2222

database:
  # Use "host.docker.internal" if your PostgreSQL is running locally on the same host.
  host: "postgres"
  port: 5432
  user: "REDACTED"
  password: "REDACTED"
  database: "pgrokd"

identity_provider:
  type: "oidc"
  display_name: "Google"
  issuer: "https://accounts.google.com"
  client_id: "这里需要修改"
  client_secret: "这里需要修改"
  field_mapping:
    identifier: "email"
    display_name: "name"
    email: "email"
# # The required domain name, "field_mapping.email" is required to set for this to work.
#  required_domain: "pgrok.com"
EOF
```

启动服务端(On Docker)
``` sh
docker-compose up -d
```

访问服务端
http://pgrok.com:88

## 2. 客户端安装

安装客户端
https://github.com/pgrok/pgrok/releases/

端口映射
``` sh
export PGROK_TOKEN=xxx

pgrok init --remote-addr pgrok.com:2222 --forward-addr http://localhost:80 --token ${PGROK_TOKEN}
```