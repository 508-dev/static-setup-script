# static-setup-script
Script to run when creating new static deploy environment for new static project

# Steps
1. Ensure Domain $NAME.508.dev A record is created on our domain manager.

2. Create new user account of name $NAME
```bash
sudo adduser --disabled-password --gecos "" $NAME
```
3. Create SSH keys
// need to update this because you need to generate the key somewhere and then get it in the authorized_keys file somehow
```bash
sudo mkdir /home/$NAME/.ssh
sudo ssh-keygen -t ed25519 -C "$NAME@508.dev" -f /home/$NAME/.ssh/id_ed -N ""
sudo cp /home/$NAME/.ssh/id_ed authorized_keys
sudo chown -R $NAME:$NAME /home/$NAME/.ssh/
```
3. Add user to group `www-data`
```bash
sudo usermod -a -G www-data $NAME
sudo usermod -a -G ssh $NAME
sudo usermod -g www-data $NAME
```
4. Create directory for html, css, js output in `/home/www-data/`

```bash
sudo mkdir /home/www-data/$NAME
sudo chown $NAME:www-data /home/www-data/$NAME
```
5. Create file of `/etc/nginx/sites-available/$NAME.508.dev`
```bash
 server {
   listen 80;
   listen [::]:80;

   server_name $NAME.508.dev www.$NAME.508.dev;
   return 301 https://$server_name$request_uri;
 }

 server {
   server_name $NAME.508.dev;

   gzip on;

   location / {
	autoindex on;
	root /home/www-data/$NAME/;
	error_page 404 = /index.html;
   }


   listen 443 ssl;
   listen [::]:443;

   ssl_certificate /etc/letsencrypt/live/$NAME.508.dev/fullchain.pem; # CHANGE ME
   ssl_certificate_key /etc/letsencrypt/live/$NAME.508.dev/privkey.pem; # CHANGE ME
   include /etc/letsencrypt/options-ssl-nginx.conf;
}

EOF
```
6. Softlink into `/etc/nginx/sites-enabled/`

```bash
    sudo ln -s /etc/nginx/sites-available/$NAME.508.dev /etc/nginx/sites-enabled/
```

7. Temporarily take down nginx

```bash
sudo service nginx stop
```

8. Create https certificate

```bash
sudo certbot certonly --standalone -d $NAME.508.dev
```

9. Bring up nginx again

```bash
sudo service nginx start
```
