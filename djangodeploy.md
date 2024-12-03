add an user for better deploy

```sudo apt update
sudo apt upgrade
apt install nginx curl wget certbot python3-certbot-nginx python3-venv
mkdir django
cd django/
wget https://exampledomain.com/projecttar.tar.gz
tar xzvf projecttar.tar.gz 
cd projectdir/
nano settings.py
```
```python
#DEBUG = True # hata çıktısını ekrana basar. static dosyaları sunar.
DEBUG = False # hata çıktısı basmaz, static dosyaları sunmak için "runserver 0.0.0.0:8080 --insecure" insecure flagı gerekir. aksi taktirde static dosyalar nginx ile sunulur.
.
.
.
ALLOWED_HOSTS = ["localhost","domainexample.com","serveripforbeforedeploytest","127.0.0.1"]
.
.
.
import os # static dosyalarda dizin hatası almamak için os ile join yapıyoruz.
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles/') # static dosyalarda dizin hatası almamak için os ile join yapıyoruz. STATIC_ROOT değişkeni static dosyaların collect olup toplanacağı dizini ifade eder.
# STATIC_ROOT = BASE_DIR / 'staticfiles' # eski hali
STATIC_URL = '/static/' # staticlerin tarayıcıda görünecek adresi
STATICFILES_DIRS = [ # proje içerisindeki toplanması gereken tüm static dosyaların olduğu dizinleri tanımlıyoruz.
    BASE_DIR / 'static',
    BASE_DIR / 'app1' / 'static'
]
  ```
  ```bash
cd ..
python3 -m venv venv
source venv/bin/activate
pip3 install -r projectdir/requirements.txt # django ve diğer bağımlılıkları yükler
python3 projectdir/manage.py runserver 0.0.0.0:8000 --insecure #debug false ayarlandığı için insecure ile static dosyaların serve olması sağlanır.
python3 projectdir/manage.py collectstatic #static dosyaların hepsi settings.py'de belirtilen dizinde toplanır.
nano /etc/nginx/sites-enabled/exampledomain.conf
  ```
  ```bash
server {
    server_name exampledomain.org;
    listen 80 ;
    listen [::]:80 ;
    return 301 https://$host$request_uri;
}
  ```
  ```bash
ln -s /etc/nginx/sites-available/exampledomain.com.conf /etc/nginx/sites-enabled/exampledomain.com.conf
service nginx restart
sudo certbot --nginx -d exampledomain.org -d www.exampledomain.org
rm -rf /etc/nginx/sites-enabled/exampledomain.conf
nano /etc/nginx/sites-enabled/exampledomain.conf
  ```
```bash
server {
    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
        server_name exampledomain.org;
#       root /var/www/html;
#       index index.html index.htm index.nginx-debian.html;
        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
                root /var/www/html/;            
#               try_files $uri $uri/ =404;
        }
    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }

    ssl_certificate /etc/letsencrypt/live/freelancertoplulugu.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/freelancertoplulugu.org/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
server {
    server_name exampledomain.org;
    listen 80 ;
    listen [::]:80 ;
    return 301 https://$host$request_uri;
}
```
```bash
pip3 install gunicorn
gunicorn --bind 0.0.0.0:8000 projectdir.wsgi # test amaçlı gunicorn ile bir test sunucusu başlatılır. sunucuip:8000 ile dışardan erişilebilir. static dosyalar serve edilmez. static dosyaları nginx serve edecek.
sudo nano /etc/systemd/system/gunicorn.socket
```   
```bash
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target

```
```bash
sudo nano /etc/systemd/system/gunicorn.service
```
```bash
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/root/projectdir/projectdir
ExecStart=/usr/local/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          projectdir.wsgi:application

[Install]
WantedBy=multi-user.target

```
```bash
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
file /run/gunicorn.sock
sudo systemctl status gunicorn
curl --unix-socket /run/gunicorn.sock localhost
chmod 750 static
chown www-data:www-data static/
service nginx restart
service gunicorn restart
```
