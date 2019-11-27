


Podemos acessar usando

ssh -i chavelucas.pem centos@3.228.201.57

### Na instancia EC2

Vamos alterar para root.

sudo su

Vamos atualizar o sistema

yum update

Vamos instalar o ****epel-release****.

yum -y install epel-release

Vamos instalar o nginx

yum -y install nginx

Vamos iniciar ele usando

systemctl start nginx

Em seguida vamos adicionar o nginx

systemctl enable nginx

Instalando o vim

yum install vim 

Criando a estrutura dos arquivos

[centos@ip-172-31-24-40 html]$ pwd
/var/www/html

Vamos criar um diretorio chamado ****site****.

mkdir site

> ele vai ficar no endereço ****/var/www/html/site****.

Vamos agora criar um site html simples, da seguinte forma:

echo "<center><h1>Consegui chegar</h1><center>" > /var/www/html/index.html

Agora vamos criar um arquivo de configuração para o ****nginx****.

Vamos até o diretorio

cd /etc/nginx/conf.d/

Vamos criar agora o arquivo chamado ****site.conf****.

server {
 listen  80;
  server_name site-lucaszu.lucaszu-labs.tk;
 error_log /var/log/nginx/blog/error.log;
 location / {
 root  /var/www/html/site/;
 index  index.html index.htm;
 }
 error_page  500 502 503 504  /50x.html;
}

Podemos testar ver se o arquivo está ok usando

nginx -t

Vamos agora resetar o serviço

service nginx restart

Vamos agora adicionar na ****route 53**** o apontamento.

>   

Na configuração do Route 53 meu dominio é o lucaszu-labs.tk

crio uma entrada do tipo A – com o subdominio – site-lucaszu.meudominio

Tomcat

> [https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-centos-7](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-centos-7)

Inicialmente vamos instalar o ****OpenJDK****.

sudo yum install java-1.7.0-openjdk-devel

Vamos criar um usuario para o tomcat

sudo groupadd tomcat

Vamos precisar instalar o ****wget****.

sudo yum install wget

Vamos realizar o download do tomcat usando o wget no seguinte link

wget https://www.apache.org/dist/tomcat/tomcat-8/v8.5.49/bin/apache-tomcat-8.5.49.tar.gz /tmp

Agora vamos descompactar.

cd /tmp
tar xf apache-tomcat-8.5.49.tar.gz
mv apache-tomcat-8.5.49 /opt/tomcat

Agora vamos alterar a permissão do arquivo

sudo useradd -M -s /bin/nologin -g tomcat -d /opt/tomcat tomcat

Agora vamos conceder ao grupo do tomcat a propriedade de todo o diretorio da instalação.

sudo chgrp -R tomcat /opt/tomcat

Em seguida vamos fornecer ao grupo tomcat acesso de leitura ao diretório conf e todo o seu conteúdo e execute o acesso ao próprio diretório:

sudo chmod -R g+r conf
sudo chmod g+x conf

Em seguida, vamos tornar o usuário do tomcat o proprietário dos diretórios webapps, work, temp e logs:

sudo chown -R tomcat webapps/ work/ temp/ logs/

Vamos criar um arquivo de inicialização.

sudo vi /etc/systemd/system/tomcat.service

O valor do arquivo é o seguinte

# Systemd unit file for tomcat
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/bin/kill -15 $MAINPID

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

Não podemos esquecer de salvar e sair.

Vamos realizar um reload do systemd

sudo systemctl daemon-reload

Vamos dar um start no serviço tomcat

sudo systemctl start tomcat

Vamos ver o status do tomcat

sudo systemctl status tomcat

Se você deseja ativar o serviço Tomcat, para que ele seja iniciado na inicialização do servidor, execute este comando:

sudo systemctl enable tomcat

> Agora nosso tomcat já vai estar funcionando na porta ****8080****, para testar na amazon vamos precisar realizar a adição da porta 8080 no Security groups.

Nesse caso vamos criar um proxy reverso no diretorio ****/etc/nginx/conf.d/site.conf****. O valor do arquivo é o seguinte.

server {
 listen 80;
 server_name tomcat-lucaszu.lucaszu-labs.tk;
 access_log /var/log/nginx/tomcat/access.log;
 error_log /var/log/nginx/tomcat/error.log;

 location / {
 proxy_set_header X-Real-IP $remote_addr;
 proxy_set_header Host $host;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

 proxy_pass http://127.0.0.1:8080;
 }
}

> Não esqueça de salvar

Em seguida vamos dar um checar o nginx e dar um restart no serviço.

sudo nginx -t
sudo service nginx restart

Podemos realizar o restart no tomcat.

sudo service tomcat restart

#### Mysql

Vamos adicionar o mysql no repositorio do centos

sudo yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm

Vamos atualizar o sistema

sudo yum update

Instalando o mysql

sudo yum install mysql-community-server

Vamos adicionar o mysql para iniciar junto com o sistema

sudo systemctl enable mysqld

Vamos iniciar o mysql

sudo systemctl start mysqld

Agora vamos analisar qual é a senha de instalação do mysql.

sudo grep 'temporary password' /var/log/mysqld.log

Vamos ter uma mensagem semelhante a

A temporary password is generated for root@localhost: q&0)V!?fjksL

É recomendado usar as configurações para uma instalação segura.

sudo mysql_secure_installation

#### Wordpress

##### Criando banco de dados

Vamos criar um banco chamado ****wordpress****.

CREATE DATABASE wordpress;

Vamos criar um usuario para o banco com o nome ****wordpressuser**** e a senha ****senhatemporaria****.

CREATE USER wordpressuser@localhost IDENTIFIED BY 'senhatemporaria';

Vamos conceder privilegio para o usuario que criamos anteriomente

GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'senhatemporaria';

Vamos realizar as modificações usando

FLUSH PRIVILEGES;

Em seguida podemos sair usando

exit

##### Instalando wordpress

Vamos ate o diretorio

cd /var/www

Vamos criar um diretorio chamado blog

mkdir html

Vamos iniciar instalando o ****php-gd****.

sudo yum install php-gd

Vamos até o diretorio /tmp e la realizar o download do wordpress.

cd /tmp

Vamos realizar o download dele

wget http://wordpress.org/latest.tar.gz

Vamos descompactar e mandar ele para /var/www/html

tar xf latest.tar.gz

Vamos agora mover o diretorio para outro diretorio

mv wordpress /var/www/html/blog

Vamos criar o diretorio uploads

mkdir /var/www/html/blog/wp-content/uploads

Vamos alterar a permissão do diretorio do blog

sudo chown -R nginx:nginx /var/www/html/blog/*

Vamos ir até o diretorio do blog e fazer uma copia do arquivo de configuração chamado ****wp-config.php****.

cp wp-config-sample.php wp-config.php

Vamos alterar o arquivo ****wp-config.php****.

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');

> Nesse passo vamos setar as senhas do nosso banco.

Vamos agora criar um arquivo de configuração em ****/etc/nginx/conf.d/blog.conf**** com o seguinte valor.

server {
server_name blog-lucaszu.lucaszu-labs.tk;
 root /var/www/html/blog/;
 index index.html index.htm index.php;
 #access_log /var/log/nginx/blog/access.log;
 #error_log /var/log/nginx/blog/error.log;
 location = /favicon.ico {
 log_not_found off;
 access_log off;
 }
 location = /robots.txt {
 allow all;
 log_not_found off;
 access_log off;
 }
 location / {
 try_files $uri $uri/ /index.php?$args;
 }
 location ~ \.php$ {
 try_files $uri =404;
 fastcgi_pass unix:/run/php/php-fpm.sock;
 fastcgi_index   index.php;
 fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
 include fastcgi_params;
 }
 location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
 expires max;
 log_not_found off;
 }
}

Vamos realizar uma configuração no dirtorio /var/www/html/blog para conceder permissão ao sistema.

sudo setsebool -P httpd_can_network_connect on
getenforce
chcon -Rt httpd_sys_content_t /var/www/html/blog

Vamos agora ver se o nosso arquivo de configuração está OK.

sudo nginx -t

Agora vamos realizar um restart no serviço nginx.

sudo service nginx restart

  

sudo yum update

  

```
yum -y install epel-release
```

  

```
yum -y install nginx
```

  

```
systemctl start nginx
```

  

```
yum -y install net-tools
```

  

  

  

  

`Remova o comentário da linha cgi.fix_pathinfo 762 e altere o valor para 0.`

  

`cgi.fix_pathinfo=0`

  

`-------------`

  

`Configure o limite de memória, o tempo máximo de execução e ative a compactação de saída zlib, certifique-se de definir os valores conforme mostrado abaixo.`

  

```
memory_limit = 512M
```

  

`Alterar no arquivo vim /etc/php-fpm.d/www.conf`

  

`O PHP-FPM7 será executado no usuário e grupo 'nginx', altere o valor para 'nginx' para as linhas de usuário e grupo.`

  

```
user = nginx
```

  

  

  

  

Em seguida, crie um novo diretório para o caminho da sessão e o local do arquivo php sock. Em seguida, altere o proprietário para o usuário e grupo 'nginx'.

Crie um novo diretório para o caminho da sessão.

```
mkdir -p /var/lib/php/session/
```

Crie um novo diretório para o local do arquivo de soquete php-fpm.

```
mkdir -p /run/php/
```

A configuração do PHP-FPM7 está concluída, inicie o daemon agora e ative-o no momento da inicialização com o comando systemctl abaixo.

```
systemctl start php-fpm
```

Quando não há erros, você pode usar o comando abaixo para verificar se o php-fpm está em execução no arquivo de soquete.

```
netstat -pl | grep php-fpm.sock
```

  

  

  

Magento 2.1 requer uma versão atual do MySQL, você pode usar o MySQL 5.6 ou 5.7 para a instalação. Neste tutorial, usaremos o MySQL 5.7, que pode ser instalado no repositório oficial do MySQL. Então, precisamos adicionar o novo repositório MySQL primeiro.

  
  

Faça o download e adicione o novo repositório MySQL para instalação do MySQL 5.7.

  
  

```
yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
```

Agora instale o MySQL 5.7 a partir do repositório MySQL com o comando yum abaixo.

```
yum install mysql-community-server
```

Quando a instalação estiver concluída, inicie o mysql e adicione-o para iniciar no momento da inicialização.

```
systemctl start mysqld
```

  

O MySQL 5.7 foi instalado com uma senha root padrão, é armazenado no arquivo mysqld.log. Use o comando grep para ver a senha padrão do MySQL 5.7. Execute o comando abaixo.

```
grep 'temporary' /var/log/mysqld.log
```

  

  

SENHA MYSQL - Zd=oq3qqFVcx

  

Alterar senha root

  

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'PSwTQ$ek5';
```

  

  

  

Criaremos um novo banco de dados chamado 'magentodb' e um novo usuário 'magentouser' com a senha ' Magento123 @ '. Em seguida, conceda todos os privilégios do banco de dados ao novo usuário. Execute a consulta mysql abaixo.

```
create database magentodb;
```

## `Instalação e configuração` `magento`

Nesta etapa, começaremos a instalar e configurar o Magento. Para o diretório raiz da web, usaremos o diretório '/ var / www / magento2'. Precisamos do compositor PHP para a instalação do pacote Magento.

**Instale o PHP Composer**

Usaremos o Composer para instalação de dependências de pacotes PHP. Instale o compositor com o comando curl abaixo.

```
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer
```

Quando a instalação estiver concluída, você pode verificar a versão do compositor, como mostrado abaixo.

```
composer -v
```

  
  

O PHP Composer foi instalado.

**Faça o download e extraia o Magento**

Vá para o diretório '/ var / www' e faça o download do código Magento com o comando wget.

```
cd /var/www/
```

  
  

Instale descompacte com yum.

```
yum -y install unzip
```

Extraia o código Magento e renomeie o diretório para o diretório 'magento2'.

```
unzip 2.1.zip
```

**Instalar dependências do PHP**

Vá para o diretório magento2 e instale todas as dependências do Magento com o comando composer.

```
cd magento2
```

  

  

  

================================//=================

  

Instalação TOMCAT

  

  

Instalar java

  

`sudo yum install java-1.8.0-openjdk.x86_64`

  

java -version

  

  

```
bin/magento setup:install --backend-frontname="adminlogin" \
```

  
  

Vamos agora realizar a modificação de permissão

chmod 700 /var/www/magento2/app/etc
chown -R nginx:nginx /var/www/magento2

> [https://stackoverflow.com/questions/23948527/13-permission-denied-while-connecting-to-upstreamnginx](https://stackoverflow.com/questions/23948527/13-permission-denied-while-connecting-to-upstreamnginx)

O arquivo de confoguração do nginx é o seguinte e ele precisa estar em ****/etc/nginx/conf.d****.

Vamos criar o arquivo ****magento.conf****.

upstream fastcgi_backend {
 server  unix:/run/php/php-fpm.sock;
}

server {

 listen 80;
 server_name loja-lucaszu.lucaszu-labs.tk;
 set $MAGE_ROOT /var/www/html/magento2;
 set $MAGE_MODE developer;
 include /var/www/html/magento2/nginx.conf.sample;
}

Vamos ver se está tudo OK com o arquivo de configuração

nginx -t
systemctl restart nginx
