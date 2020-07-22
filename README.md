ARA Records Ansible
===================

# How to Deploy

```
sudo apt install python3 python3-pip
pip3 install -r requirements.txt

# Modify your setting such as database
cp settings/settings.yaml.example settings/settings

# Test gunicorn
gunicorn --bind 0.0.0.0:8000 ara.server.wsgi

# Create Gunicorn service
sudo vi /etc/systemd/system/gunicorn.service
paste config below
### START OF SERVICE ###
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/ara
ExecStart=/home/ubuntu/.pyenv/shims/gunicorn --access-logfile - --workers 3 --bind unix:/home/ubuntu/gunicorn.sock ara.server.wsgi:application

[Install]
WantedBy=multi-user.target
### END OF SERVICE ###

# Enable it so that it starts at boot

sudo systemctl start gunicorn
sudo systemctl enable gunicorn

# Check for the Gunicorn Socket File
sudo systemctl status gunicorn

# Reload Gunicorn setting if needed
sudo journalctl -u gunicorn
sudo systemctl daemon-reload
sudo systemctl restart gunicorn

# Generate static files
python3 manage.py collectstatic

# Configure Nginx to Proxy Pass to Gunicorn
sudo vi /etc/nginx/sites-available/ara

### PASTE CONF ###
server {
    listen 80;
    server_name ara.apollo.me;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/ara/www;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/gunicorn.sock;
    }
}
### END OF CONF ###
sudo ln -s /etc/nginx/sites-available/ara /etc/nginx/sites-enabled
sudo nginx -t
sudo nginx -s reload

```