---
title: Como instalar o Zabbix 7.2 no Debian 12
category: tutorial
---

Primeiramente acessamos o modo de superuser:<br>
`su -`

Acessamos a pasta temporaria:<br>
`cd /tmp/`

Baixamos o pacote do Zabbix 7.2 para Debian 12:<br>
`wget https://repo.zabbix.com/zabbix/7.2/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.2+debian12_all.deb`

Realize a instalação do pacote .deb que acabamos de baixar:<br>
`dpkg -i zabbix-release_latest_7.2+debian12_all.deb`

Rodamos um update:<br>
`apt update`

Realizamos a instalacão de pacotes necessarios para o Zabbix:<br>
`apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent mariadb-server`

Acesse o MySQL com o root:<br>
`mysql -uroot -p`

Crie uma senha de root: (Obs: Nesse caso irei utilizar pass@root@123)<br>
![assets/img/install-zabbix/senha-root.png](assets/img/install-zabbix/senha-root.png)

Crie um banco de dados chamado zabbix:<br>
`create database zabbix character set utf8mb4 collate utf8mb4_bin;`

Crie o usuário zabbix e defina uma senha: (Obs: Nesse caso irei utilizar a senha pass@zabbix@123)<br>
`create user zabbix@localhost identified by 'pass@zabbix@123';`

Conceda permissões ao usuário Zabbix:<br>
`grant all privileges on zabbix.* to zabbix@localhost;`

Ative uma configuração no MySQL que permite a criação de funções armazenadas:<br>
`set global log_bin_trust_function_creators = 1;`

Saia do MySQL:<br>
`quit;`

Importe o schema e os dados iniciais do Zabbix no banco de dados MySQL:<br>
`zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix`

(Obs: Será solicitado a senha do user zabbix criada anteriormente, no meu caso pass@zabbix@123)<br>
![assets/img/install-zabbix/schema.png](assets/img/install-zabbix/schema.png)

Esse processo costuma demorar um pouco, aguarde a linha de comando aparecer novamente:<br>
![assets/img/install-zabbix/senha-root-2.png](assets/img/install-zabbix/senha-root-2.png)

Acesse o MySQL com o root novamente:<br>
`mysql -uroot -p`

Desative a configuração no MySQL que permite a criação de funções armazenadas:<br>
`set global log_bin_trust_function_creators = 0;`

Saia do MySQL:<br>
`quit;`

Precisamos adicionar a senha do DB no arquivo zabbix_server.conf, insira o comando abaixo:<br>
`nano /etc/zabbix/zabbix_server.conf`

Localize a linha # DBPassword: (Utilize Ctrl + W para localizar)<br>

Descomente a linha e insira a senha do user zabbix criada anteriormente, no meu caso é pass@zabbix@123:<br>
`DBPassword=pass@zabbix@123`<br>
![assets/img/install-zabbix/db-password.png](assets/img/install-zabbix/db-password.png)

Reinicie os serviços do Zabbix Server, Zabbix Agent e Apache2:<br>
`systemctl restart zabbix-server zabbix-agent apache2`

Habilite os serviços do Zabbix Server, Zabbix Agent e Apache2:<br>
`systemctl enable zabbix-server zabbix-agent apache2`

Pronto, agora acesse o seu Zabbix com o http://IP_DO_SEU_HOST/zabbix, exemplo: http://192.168.0.2/zabbix.<br>

Voce verá a pagina abaixo, clique em Próximo Passo:<br>
![assets/img/install-zabbix/bem-vindo.png](assets/img/install-zabbix/bem-vindo.png)

Clique em Próximo Passo novamente:<br>
![assets/img/install-zabbix/pre-requisitos.png](assets/img/install-zabbix/pre-requisitos.png)

Preencha o campo Senha com a senha do user zabbix, no meu caso é pass@zabbix@123. Clique em Próximo Passo:<br>
![assets/img/install-zabbix/conexao-db.png](assets/img/install-zabbix/conexao-db.png)

Ajuste conforme necessário e clique em Próximo Passo:<br>
![assets/img/install-zabbix/configuracoes.png](assets/img/install-zabbix/configuracoes.png)

Verifique o sumário, se estiver correto, clique em Próximo Passo:<br>
![assets/img/install-zabbix/sumario.png](assets/img/install-zabbix/sumario.png)

Você recebará a informacão que a instalação foi concluída, clique em Fim:<br>
![assets/img/install-zabbix/instalacao-ok.png](assets/img/install-zabbix/instalacao-ok.png)

Na pagina de login, insira os dados de acesso:<br>
User: Admin<br>
Password: zabbix <br>
![assets/img/install-zabbix/login.png](assets/img/install-zabbix/login.png)

Instalacão concluída:<br>
![assets/img/install-zabbix/pagina-inicial.png](assets/img/install-zabbix/pagina-inicial.png)
