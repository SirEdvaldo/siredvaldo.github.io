---
title: Como instalar o Daloradius no Debian 12
category: tutorial
---

Primeiramente acessamos o modo de superuser:<br>
`su -`

Realizamos a instalacão de pacotes necessarios para o MariaDB:<br>
`apt install mariadb-server mariadb-client`

Acesse o MySQL com o root:<br>
`mariadb -u root -p`

Crie uma senha de root: (Obs: Nesse caso irei utilizar pass@root@123)<br>
![assets/img/daloradius/password-root.png](assets/img/daloradius/password-root.png)

Crie um banco de dados chamado radius:<br>
`CREATE DATABASE radius;`

Crie o usuário e senha radius e conceda a ele todas as permissões no banco de dados radius:<br>
`GRANT ALL ON radius.* TO radius@localhost IDENTIFIED BY "pass@daloradius@123";`

Recarregue as permissões dos usuários:<br>
`FLUSH PRIVILEGES;`

Saia do MySQL:<br>
`quit;`

Acesse o MySQL com o agora com o user radius:<br>
`mariadb -u radius -p`

Verifique se o banco de dados radius foi criado:<br>
`SHOW DATABASES;`

![assets/img/daloradius/show-databases.png](assets/img/daloradius/show-databases.png)

Saia do MySQL:<br>
`quit;`

Realize a instalação do apache2, php e outros pacotes necessários:<br>
`apt install apache2 php libapache2-mod-php php-{gd,common,mail,mail-mime,mysql,pear,mbstring,xml,curl}`

Verique a versão do PHP:<br>
`php -v`

![assets/img/daloradius/php-v.png](assets/img/daloradius/php-v.png)

Verifique o status do apache2:<br>
`systemctl status apache2`
![assets/img/daloradius/status-apache2.png](assets/img/daloradius/status-apache2.png)

Realize a instalação do freeradius e seus componentes:<br>
`apt install freeradius freeradius-mysql freeradius-utils`

Habilite e inicie o freeradius:<br>
`systemctl enable --now freeradius.service`

Verifique o status do freeradius:<br>
`systemctl status freeradius`
![assets/img/daloradius/status-freeradius.png](assets/img/daloradius/status-freeradius.png)

Importe o schema e os dados iniciais do FreeRadius no banco de dados MySQL: (Obs: Será solicitado a senha de root)<br>
`mariadb -u root -p radius < /etc/freeradius/*/mods-config/sql/main/mysql/schema.sql`

Crie um link simbólico sql: <br>
`ln -s /etc/freeradius/*/mods-available/sql /etc/freeradius/*/mods-enabled/`

Vamos editar o arquivo de configuração sql:<br>
`nano /etc/freeradius/*/mods-enabled/sql`

Em SQL, altere os seguintes campos:<br>
```
       dialect = "mysql"`
       driver = "rlm_sql_mysql"`
```
Em MySQL, comente todo o bloco de tls:<br>
```
                # If any of the files below are set, TLS encryption is enabled
#               tls {
#                       ca_file = "/etc/ssl/certs/my_ca.crt"
#                       ca_path = "/etc/ssl/certs/"
#                       certificate_file = "/etc/ssl/certs/private/client.crt"
#                       private_key_file = "/etc/ssl/certs/private/client.key"
#                       cipher = "DHE-RSA-AES256-SHA:AES128-SHA"
#
#                       tls_required = no
#                       tls_check_cert = no
#                       tls_check_cert_cn = no
#               }
```

Ainda no mesmo arquivo, descomente e altere os seguintes campos: (Obs: Em password, insira a senha criada para o user daloradius)<br>
```
server = "localhost"
port = 3306
login = "radius"
password = "pass@daloradius@123"
...
`read_clients = yes`
```

Ajuste as permissões:<br>
```
chgrp -h freerad /etc/freeradius/*/mods-available/sql
chown -R freerad:freerad /etc/freeradius/*/mods-enabled/sql
```

Reinicie o serviço freeradius:<br>
`systemctl restart freeradius`

Instale o git para clonar o daloradius:<br>
`apt install git`

Acesse o diretório temporário:<br>
`cd /tmp/`

Clone o daloradius:<br>
`git clone https://github.com/lirantal/daloradius.git`

Importe a estrutura do banco de dados do daloRADIUS dentro do banco radius: (Obs: Será solicitado a senha de root)<br>
`mariadb -u root -p radius < daloradius/contrib/db/fr3-mariadb-freeradius.sql`<br>
`mariadb -u root -p radius < daloradius/contrib/db/mariadb-daloradius.sql`

Mova o diretório daloradius para dentro do diretório da raiz de hospedagem da web:<br>
`mv daloradius /var/www/`

Acesse o seguinte diretório:<br>
`cd /var/www/daloradius/app/common/includes/`

Crie uma copia do arquivo daloradius.conf.php.sample:<br>
`cp daloradius.conf.php.sample daloradius.conf.php`

Altere o dono e grupo do arquivo:<br>
`chown www-data:www-data daloradius.conf.php`

Edite o arquivo o seguinte arquivo:<br>
`nano daloradius.conf.php`

Altere os seguintes campos:<br>
```
$configValues['CONFIG_DB_HOST'] = 'localhost';
$configValues['CONFIG_DB_PORT'] = '3306';
$configValues['CONFIG_DB_USER'] = 'radius';
$configValues['CONFIG_DB_PASS'] = 'pass@daloradius@123';
$configValues['CONFIG_DB_NAME'] = 'radius';
```
![assets/img/daloradius/edit-sql.png](assets/img/daloradius/edit-sql.png)

Agora acesse o seguinte diretório:<br>
`cd /var/www/daloradius/`

Crie as seguintes pastas:<br>
`mkdir -p var/{log,backup}`

Altere o dono e grupo do seguinte diretório:<br>
`chown -R www-data:www-data var`

Aplique a seguinte configuração a ports.conf: (Obs: Aplique todo o bloco de comando de uma vez)<br>
```
tee /etc/apache2/ports.conf<<EOF
Listen 80
Listen 8000

<IfModule ssl_module>
    Listen 443
</IfModule>

<IfModule mod_gnutls.c>
    Listen 443
</IfModule>
EOF
```

Aplique a seguinte configuração a operators.conf: (Obs: Aplique todo o bloco de comando de uma vez)<br>
```
tee /etc/apache2/sites-available/operators.conf<<EOF
<VirtualHost *:8000>
    ServerAdmin operators@localhost
    DocumentRoot /var/www/daloradius/app/operators

    <Directory /var/www/daloradius/app/operators>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    <Directory /var/www/daloradius>
        Require all denied
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/daloradius/operators/error.log
    CustomLog \${APACHE_LOG_DIR}/daloradius/operators/access.log combined
</VirtualHost>
EOF
```

Aplique a seguinte configuração a users.conf: (Obs: Aplique todo o bloco de comando de uma vez)<br>
```
tee /etc/apache2/sites-available/users.conf<<EOF
<VirtualHost *:80>
    ServerAdmin users@localhost
    DocumentRoot /var/www/daloradius/app/users

    <Directory /var/www/daloradius/app/users>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    <Directory /var/www/daloradius>
        Require all denied
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/daloradius/users/error.log
    CustomLog \${APACHE_LOG_DIR}/daloradius/users/access.log combined
</VirtualHost>
EOF
```
![assets/img/daloradius/tee.png](assets/img/daloradius/tee.png)

Aplique as configurações alteradas:<br>
`a2ensite users.conf operators.conf`

Crie os seguintes diretorios:<br>
`mkdir -p /var/log/apache2/daloradius/{operators,users}`

Desabilite a configuração do site padrao do apache:<br>
`a2dissite 000-default.conf`

Reinicei os serviços do apache e do freeradius:<br>
`systemctl restart apache2 freeradius`

Verifique o status dos serviços:<br>
`systemctl status apache2 freeradius`

Realize a instalação dos seguintes pacotes:<br>
```
pear install DB
pear install MDB2
```

Pronto, agora acesse o seu Daloradius com o http://IP_DO_SEU_HOST:8000, exemplo: http://192.168.0.3:8000.

Na pagina de login, insira os dados de acesso:<br>
Username: administrator<br>
Password: radius
![assets/img/daloradius/dalo-login.png](assets/img/daloradius/dalo-login.png)

Instalacão concluída:<br>
![assets/img/daloradius/dalo-front.png](assets/img/daloradius/dalo-front.png)
