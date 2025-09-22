

#!/data/data/com.termux/files/usr/bin/bash
# Termux VPS-10TB-Server Setup Script

echo "Updating Termux packages..."
pkg update -y && pkg upgrade -y

echo "Installing basic server tools..."
pkg install -y openssh nginx php python git htop

echo "Setting up storage access..."
termux-setup-storage

echo "Starting nginx web server..."
nginx

echo "Starting SSH server..."
sshd

echo "Creating VPS-10TB-Server directory..."
mkdir -p ~/VPS-10TB-Server/www
echo "<?php phpinfo(); ?>" > ~/VPS-10TB-Server/www/index.php

echo "Configuring nginx to serve ~/VPS-10TB-Server/www"
cat > ~/../usr/etc/nginx/conf.d/10tb-server.conf <<EOF
server {
    listen 8080;
    server_name localhost;

    root /data/data/com.termux/files/home/VPS-10TB-Server/www;
    index index.php index.html index.htm;

    location / {
        try_files \$uri \$uri/ =404;
    }

    location ~ \.php\$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        include fastcgi_params;
    }
}
EOF

echo "Restarting nginx to apply new config..."
pkill nginx
nginx

echo "VPS-10TB-Server is running!"
echo "Access it via: http://localhost:8080/"
