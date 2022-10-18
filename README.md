# Atividade Aplicação Wordpress + DB Mysql
Esta documentação irá descrever e detalhar o processo de inicializar uma aplicação WordPress com um banco de dados MySQL usando o Docker Compose.

É necessário verificar se a máquina possui Docker instalado, e após isso verificar também o Docker Compose; caso não, realize a instalação de ambos. Neste caso, foi utilizado o Docker Desktop, o qual já possui o Docker Compose.

## Ambiente
Crie uma pasta específica para a aplicação e nela crie um arquivo chamado **docker-compose.yml**

Dentro deste arquivo serão definidos os paramêtros, ele será do tipo YAML.

### Arquivo
 É valido salientar que a indentação de arquivos YAML é feita só com espaços, e a indentação correta é fundamental para a execução da aplicação.

 Primeiramente dentro do aquivo será definida a versão do docker-compose.yml.
 Após isso é adicionada a propriedade services, na qual são definidos os serviços que a nossa aplicação terá.
~~~
version: '3.6'

services:
~~~

 Os serviços em questão serão Wordpress e DB, que basicamente funcionarão como contêiners.

 O primeiro apresentado será o db:
~~~
db:
     image: mysql:8.0.31
     restart: always
     volumes:
      - ./db:/var/lib/mysql   
     command: mysqld --default_authentication_plugin=mysql_native_password
     environment:
      TZ: America/Sao_Paulo
      MYSQL_ROOT_PASSWORD: $MYSQL_ROOT_PASSWORD 
      MYSQL_USER: $MYSQL_USER
      MYSQL_PASSWORD: $MYSQL_PASSWORD
      MYSQL_DATABASE: $MYSQL_DATABASE
     ports:
      - 3308:3306
     networks:
      - network-docker
~~~
Primeiramente é determinada qual imagem, docker image, será utilizada do banco de dados Mysql, logo após isso, é definido também o restart: always, que diz ao Docker para reiniciar o contêiner db em caso de falha.

Logo após, o volume - ./db:/var/lib/mysql  é definido, com o intuito de não perdermos as mudanças feitas no banco de dados. O command serve para inicializar o MySQL no modo de autenticação nativo.

A próxima propriedade definida é o environment, e nela são definidas os valores das variáveis de ambiente dentro do contêiner. A variável TZ é referente ao timezone de onde o contêiner está sendo executado;  a variável MYSQL_ROOT_PASSWORD irá definir a senha do usuário root; a variável MYSQL_USER indicará o usuário, o MYSQL_PASSWORD indicará a senha do usuário fornecido anteriormente; e a variável MYSQL_DATABASE indicará a aplicação. E por fim indicaremos a rede network-docker que será definida ao final.

O segundo apresentado será o wordpress:

~~~
    wordpress:
        image: wordpress:6.0.2
        restart: always
        volumes:
         - ./config/php.conf.uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
         - ./wp-app:/var/www/html
        ports:
         - 8080:80
        depends_on:
         - db
        environment:
         TZ: America/Sao_Paulo
         WORDPRESS_DB_HOST: $WORDPRESS_DB_HOST
         WORDPRESS_DB_NAME: $WORDPRESS_DB_NAME
         WORDPRESS_DB_USER: $WORDPRESS_DB_USER
         WORDPRESS_DB_PASSWORD: $WORDPRESS_DB_PASSWORD
        networks:
         - network-docker
~~~
Primeiro é especificada qual a imagem, Docker image, do wordpress, e nesse caso foi a 6.0.2, a última mais estável até o momento. Após isso é definido restart: always, que diz ao Docker para reiniciar o contêiner wordpress em caso de falha. 
Logo após iremos especificar os volumes utilizados em questão, definido ./wordpress:/var/www/html é passado ao Docker que, qualquer alteração na nossa pasta criada, a mudança também deve ser realizada na pasta /var/www/html do contêiner.

Em seguida as portas são definidas, que no caso será a 80 no contêiner e 8080 no host. Outra propriedade é adicionada, a depends_on, que indicará ao docker compose que o serviço do wordpress depende do serviço do db, e com isso, o db deverá iniciar antes.

A próxima propriedade definida é o environment, e nela são definidas os valores das variáveis de ambiente dentro do contêiner. A variável TZ é referente ao timezone de onde o contêiner está sendo executado; a variável WORDPRESS_DB_HOST indicará o banco de dados hospedeiro da aplicação wordpress; a WORDPRESS_DB_NAME indicará o nome desejado ao banco de dados; a variável
WORDPRESS_DB_USER indicará o usuário, o WORDPRESS_DB_PASSWORD indicaráa senha do usuário fornecido anteriormente. 
E por fim indicaremos a rede network-docker que será definida ao final.

Por último é adicionada a propriedade networks, o nome da rede, como já foi visto, será network-docker e ela será inicializada em modo bridge.
 ~~~
 networks:
     network-docker:
      driver: bridge   
 ~~~

Ao final da edição, o arquivo ficará assim:

~~~
version: '3.6'

services: 

   db:
     image: mysql:8.0.31
     restart: always
     volumes:
      - ./db:/var/lib/mysql   
     command: mysqld --default_authentication_plugin=mysql_native_password
     environment:
      TZ: America/Sao_Paulo
      MYSQL_ROOT_PASSWORD: $MYSQL_ROOT_PASSWORD 
      MYSQL_USER: $MYSQL_USER
      MYSQL_PASSWORD: $MYSQL_PASSWORD
      MYSQL_DATABASE: $MYSQL_DATABASE
     ports:
      - 3308:3306
     networks:
      - network-docker

   wordpress:
     image: wordpress:6.0.2
     restart: always
     volumes:
      - ./config/php.conf.uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
      - ./wp-app:/var/www/html
     ports:
      - 8080:80
     depends_on:
      - db
     environment:
      TZ: America/Sao_Paulo
      WORDPRESS_DB_HOST: $WORDPRESS_DB_HOST
      WORDPRESS_DB_NAME: $WORDPRESS_DB_NAME
      WORDPRESS_DB_USER: $WORDPRESS_DB_USER
      WORDPRESS_DB_PASSWORD: $WORDPRESS_DB_PASSWORD
     networks:
      - network-docker
   
networks:
     network-docker:
      driver: bridge    
~~~

Após isso, o terminal de comando é aberto e direcionado a pasta criada. Com isso digite o comando:
~~~
docker compose up
~~~
Por fim, o navegador é aberto e é possível o acesso à aplicação, através do endereço "localhost:8080".
