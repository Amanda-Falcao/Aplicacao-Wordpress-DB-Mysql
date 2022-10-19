# Atividade Aplicação Wordpress + DB Mysql
Esta documentação irá descrever e detalhar o processo de inicializar uma aplicação WordPress com um banco de dados MySQL usando o Docker Compose.

É necessário verificar se a máquina possui Docker instalado, e após isso verificar também o Docker Compose; caso não, realize a instalação de ambos. Neste caso, foi utilizado o Docker Desktop, o qual já possui o Docker Compose.  
Para realizar a instalação do Docker, siga a [documentação oficial](https://docs.docker.com/get-docker/).
Para a realização da instalação do Docker Compose, basta seguir [as instruções](https://github.com/docker/compose/) do respositório no GitHub.


## Ambiente
Crie uma pasta específica para a aplicação e nela crie um arquivo chamado **docker-compose.yml**

Dentro deste arquivo serão definidos os paramêtros da aplicação que queremos subir.

### Arquivo
 É valido salientar que a indentação de arquivos YAML é feita só com espaços, e a indentação correta é fundamental para a execução da aplicação.

 Primeiramente dentro do aquivo será definida a versão do docker-compose.yml.
 Após isso é adicionada a propriedade services, na qual são definidos os serviços que a nossa aplicação terá.
~~~
version: '3.6'

services:
~~~

 Os serviços em questão serão Wordpress e DB.

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
Primeiramente é determinada qual imagem (image) será utilizada do banco de dados Mysql. Foi escolhida a versão 8.0.31 da imagem, a mais estável até o presente dia. Logo após isso, é definido também o restart: always, que diz ao Docker para reiniciar o contêiner caso ele pare.

Logo após, o volume - ./db:/var/lib/mysql  é definido, com o intuito de persistir os dados do banco de dados mesmo após o contêiner ser parado. O command serve para inicializar o MySQL no modo de autenticação nativo.

A próxima propriedade definida é o environment, e nela são definidas os valores das variáveis de ambiente dentro do contêiner. A variável TZ é referente ao timezone de onde o contêiner está sendo executado;  a variável MYSQL_ROOT_PASSWORD irá definir a senha do usuário root; a variável MYSQL_USER indicará o usuário, o MYSQL_PASSWORD indicará a senha do usuário fornecido anteriormente; e a variável MYSQL_DATABASE indica o nome do banco de dados que será criado quando o MySQL for inicializado. Esse banco de dados será utilizado pelo Wordpress. Definimos a porta utilizada pelo contêiner: 3308 no host, 3306 no contêiner. E por fim indicamos que o contêiner utilizará a rede network-docker, que será definida no final do arquivo.

O segundo apresentado será o wordpress:

~~~
    wordpress:
        image: wordpress:6.0.2
        restart: always
        volumes:
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
Primeiro é especificada qual a imagem do Wordpress será utilizada. Nesse caso foi a 6.0.2, a mais estável até o momento da realização desta atividade. Após isso é definido restart: always, que diz ao Docker para reiniciar o contêiner caso ele pare. 
Logo após iremos especificar os volumes utilizados para persistir os dados do contêiner, definido ./wordpress:/var/www/html. Esse diretório será utilizado para que o contêiner guarde e use dados que estão neste diretório.

Em seguida as portas são definidas, que no caso será a 80 no contêiner e 8080 no host. Outra propriedade é adicionada, a depends_on, que indicará ao docker compose que o serviço do wordpress depende do serviço do db, e com isso, o db deverá iniciar antes.

A próxima propriedade definida é o environment, e nela são definidas os valores das variáveis de ambiente dentro do contêiner. A variável TZ é referente ao timezone de onde o contêiner está sendo executado; a variável WORDPRESS_DB_HOST indicará o banco de dados hospedeiro da aplicação wordpress; a WORDPRESS_DB_NAME indica o nome do banco de dados que será utilizado pelo Wordpress. Ele já deve ter sido criado pelo MySQL; a variável
WORDPRESS_DB_USER indicará o usuário do banco de dados, e WORDPRESS_DB_PASSWORD indicara a senha deste usuário. 
E por fim indicaremos que o Wordpress utilizará a rede network-docker.

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
Quando a aplicação terminar de subir, é possível acessar o Wordpress através do endereço "localhost:8080".
