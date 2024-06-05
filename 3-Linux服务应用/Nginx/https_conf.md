```
cat https.conf 
server {
        listen       443 ssl;
        server_name  yfxz.bjsizy.gov.cn;

        ssl_certificate      /etc/certificate/2023/bjsizy.gov.cn_cert_chain.pem;
        ssl_certificate_key  /etc/certificate/2023/bjsizy.gov.cn_key.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;


        location ^~ / {
            alias /usr/share/nginx/html/;
            index index.html;
        }
}
```

