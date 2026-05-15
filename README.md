sudo chown -R shopapi:www-data /opt/llumiquinga-noche-shoapi/staticfiles
sudo chmod -R 755 /opt/llumiquinga-noche-shoapi/staticfiles
sudo chmod -R 755 /opt/llumiquinga-noche-shoapi

####
ls /opt/llumiquinga-noche-shoapi/staticfiles/admin/

####
[Unit]
Description=Gunicorn daemon for ShopAPI
After=network.target postgresql.service

[Service]
User=shopapi
Group=www-data
WorkingDirectory=/opt/llumiquinga-noche-shoapi
Environment="PATH=/opt/llumiquinga-noche-shoapi/.venv/bin"
EnvironmentFile=/opt/llumiquinga-noche-shoapi/.env
ExecStart=/opt/llumiquinga-noche-shoapi/.venv/bin/gunicorn \
          --workers 3 \
          --bind unix:/run/gunicorn-shopapi.sock \
          --access-logfile /var/log/gunicorn-shopapi-access.log \
          --error-logfile /var/log/gunicorn-shopapi-error.log \
          config.wsgi:application
ExecReload=/bin/kill -s HUP $MAINPID
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target

####
server {
    listen 80;
    server_name 158.23.59.142;

    # Logs
    access_log /var/log/nginx/llumiquinga-noche-shopapi-access.log;
    error_log  /var/log/nginx/llumiquinga-noche-shopapi-error.log;

    # Archivos estáticos (incluye CSS del admin de Django)
    location /static/ {
        alias /opt/llumiquinga-noche-shopapi/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Peticiones a la API via Gunicorn
    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn-shopapi.sock;
        proxy_read_timeout 90;
        proxy_connect_timeout 90;
    }
}