# getssl-doc

The getssl-doc help you to deploy Let’s Encrypt free certificate.

# Overview

> **Let’s Encrypt** is a free, automated, and open Certificate Authority. Visit to https://letsencrypt.org/

> **getssl** obtain SSL certificates from the letsencrypt.org ACME server. Suitable for automating the process on remote servers. Visit to https://github.com/srvrco/getssl

#Installation

Install getssl:

    curl --silent https://raw.githubusercontent.com/srvrco/getssl/master/getssl > /usr/bin/getssl; chmod 700 /usr/bin/getssl

or

    git clone https://github.com/srvrco/getssl.git

# Getting started

Create domain configuration

    /usr/bin/getssl -c www.yourdomain.com

vi ~/.getssl/www.yourdomain.com/getssl.cfg

    CA="https://acme-v01.api.letsencrypt.org"
    ACL=('/atmd/www/.well-known/acme-challenge')
    DOMAIN_CERT_LOCATION="/etc/ssl/www.yourdomain.com.crt"
    DOMAIN_KEY_LOCATION="/etc/ssl/www.yourdomain.com.key"
    CA_CERT_LOCATION="/etc/ssl/chain.crt"

Apply ssl Certificate

    /usr/bin/getssl www.yourdomain.com

# Automating updates

add crontab:

    23 5 1 * * /usr/bin/getssl -u -a -q

# Configure Apache

vi /etc/https/conf.d/ssl.conf

    DocumentRoot "/var/www"
    ServerName www.yourdomain.com:443

    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1

    SSLCipherSuite      ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256

    SSLCertificateFile /etc/ssl/www.yourdomain.com.crt
    SSLCertificateKeyFile /etc/ssl/www.yourdomain.com.key
    SSLCertificateChainFile /etc/ssl/chain.crt

    RewriteEngine on
    RewriteCond %{REQUEST_METHOD} ^(TRACE|TRACK)
    RewriteRule .* - [F]

vi /etc/httpd/conf/extra/httpd-vhosts-www.conf

    <VirtualHost *:80>
        ServerName www.yourdomain.com
        DocumentRoot /var/www
        ErrorLog "|/usr/sbin/rotatelogs /etc/httpd/logs/www-error_log.%Y-%m-%d 86400 +480"
        CustomLog "|/usr/sbin/rotatelogs /etc/httpd/logs/www-access_log.%Y-%m-%d 86400 +480" combined
         <Directory "/var/www">
            DirectoryIndex index.htm index.html index.php
            Options MultiViews FollowSymLinks
            Order deny,allow
            Allow from all
         </Directory>

        RewriteEngine on
        RewriteCond %{REQUEST_METHOD} ^(TRACE|TRACK)
        RewriteRule .* - [F]

        RewriteCond %{SERVER_PORT} !^443$
        RewriteRule ^(.*)?$ https://%{SERVER_NAME}$1 [L,R]

    </VirtualHost>

