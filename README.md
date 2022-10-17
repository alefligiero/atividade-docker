# Subir uma aplicação WordPress + MYSQL com o Docker Compose

Inicialmente, crie um diretório aonde irá ficar o arquivo de configuração do Docker Compose.

Em seguida crie o arquivo de configuração nomeando com `docker-compose.yaml` 

LEMBRE-SE, a identação deste arquivo é de suma importância, pois caso não esteja sendo cumprida de forma exata, o arquivo não irá executar ou retornará um erro.

Abra o arquivo de configuração para definir os dados que farão parte da criação do Docker Compose.

* Defina primeiro a versão que será usada do Docker Compose. Neste caso a versão usada será a 3.9.  
  ```
  version: '3.9'
  ```

* No campo seguinte, iremos definir os serviços que serão utilizados para a construção da aplicação.  
1. Primeiramente, iremos criar o serviço do mysql que será responsável pela criação do container e configurações do mesmo.
   ```
   mysql_db:
    container_name: mysql-container
    image: mysql:8.0.31
    restart: always
    environment:
      MYSQL_DATABASE: wordpressdb
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql
      ```
    * `mysql_db:` -> Nome do serviço do MySql.
    * `container_name: mysql-container` -> Nome do container que será criado para o MySql.
    * `image: mysql:8.0.31` -> A imagem que será baixada do repositório do MySql no dockerhub. Neste caso foi escolhida a versão 8.0.31 por ser a última versão estável.
    * `restart: always` -> Para que, em casos de erro na criação do container, o mesmo será reiniciado até que ela seja consertada.
    * `environment:` -> Lista de variáveis de ambiente que são setadas previamente para a criação do container:
      * `MYSQL_DATABASE: wordpressdb` -> Nome da database que será criada.
      * `MYSQL_USER: wordpress` -> Nome do usuário que terá acesso a Database.
      * `MYSQL_PASSWORD: wordpress` -> Senha do usuário.
      * `MYSQL_RANDOM_ROOT_PASSWORD: '1'` -> Geração de uma senha aleatória para o usuário root. OBS:. Para a criação de um container MySql, é necessário definir uma senha para o usuário root.
    * `volumes: ` -> Lista de pastas que serão replicadas em volumes que serão criados localmente.
      *  `- db:/var/lib/mysql` -> Volume e caminho da pasta em que será salva.

2. Segundamente, criaremos agora, o serviço relacionado para a criação do container do WordPress e suas configurações.
    ```
    wordpress:
      container_name: wordpress-container
      image: wordpress:6.0.2
      restart: always
      ports:
        - 8080:80
      environment:
        WORDPRESS_DB_HOST: mysql_db
        WORDPRESS_DB_USER: wordpress
        WORDPRESS_DB_PASSWORD: wordpress
        WORDPRESS_DB_NAME: wordpressdb
      volumes:
        - wordpress:/var/www/html
      depends_on:
        - mysql_db
    ```
    * `wordpress:` -> Nome do serviço do WordPress.
    * `container_name: wordpress-container` -> Nome do container que será criado para o WordPress
    * `image: wordpress:6.0.2` -> A imagem que será baixada do repositório do WordPress no dockerhub. Neste caso foi escolhida a versão 6.0.2 por ser a última versão estável.
    * `restart: always` -> Para que, em casos de erro na criação do container, o mesmo será reiniciado até que ela seja consertada.
    * `ports:` -> Lista de portas de acesso da máquina local para o container criado.
      * `- 8080:80` -> A porta da máquina local será a 8080 e a porta da máquina do container será 80(padrão WordPress).
    * `environment:` -> Lista de variáveis de ambiente que são setadas previamente para a criação do container:
      * `WORDPRESS_DB_HOST: mysql_db` -> Nome do Host criado para acesso ao MySql.
      * `WORDPRESS_DB_USER: wordpress` -> Nome do usuário criado para acessar a database no MySql.
      * `WORDPRESS_DB_PASSWORD: wordpress` -> Senha do usuário criada para acessar a database no MySql.
      * `WORDPRESS_DB_NAME: wordpressdb` -> Nome da database criada no MySql.
    * `volumes:` -> Lista de pastas que serão replicadas em volumes que serão criados localmente.
      * `- wordpress:/var/www/html` -> Volume e caminho da pasta em que será salva.
    * `depends_on:` -> Define dependência entre o serviço do WordPress em relação ao MySql.
      * `- mysql_db` -> Nome do serviço dado ao MySql.

3. Logo em seguida, iremos definir a network que fará a conexão entre os dois serviços que estão sendo criados anteriormente.
    ```
    networks:
      wordpress-network:
        driver: bridge
    ```
    * `networks:` -> Lista de networks que serão criadas.
      * `wordpress-network:` -> Nome da network que será criada.
        * `driver: bridge` -> Driver que será usado na network.
4. E para finalizar, criaremos os volumes relacionado aos seus respectivos serviços.
    ```
    volumes:
      wordpress:
      db:
    ```
    * `volumes:` -> Lista de volumes que serão criados.
      * `wordpress:` -> Nome do volume do WordPress.
      * `db:` -> Nome do volume do MySql.

O resultado final do arquivo será este:
```
version: '3.9'

services:

  mysql_db:
    container_name: mysql-container
    image: mysql:8.0.31
    restart: always
    environment:
      MYSQL_DATABASE: wordpressdb
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

  wordpress:
    container_name: wordpress-container
    image: wordpress:6.0.2
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql_db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpressdb
    volumes:
      - wordpress:/var/www/html
    depends_on:
      - mysql_db

networks:
  wordpress-network:
    driver: bridge

volumes:
  wordpress:
  db:
```

Para subir a aplicação, abra o terminal na pasta em que se encontra o arquivo de configuração do Docker Compose e digite o comando `docker compose up` para subi-la. O terminal irá gerar os logs de inicialização da aplicação e deverá manter o terminal aberto e travado. 

Caso queira inicializar a aplicação sem que trave o uso do terminal, basta acrescentar ao comando de inicialização a opção `-d`. Ficará dessa forma:
`docker compose up -d` onde o mesmo inicia os serviços de forma detachada.

E enfim os serviços deverão estar em funcionamento..

Para acessar a página da aplicação do WordPress, visite `http://localhost:8080` e será redirecionado para página de configuração inicial do WordPress escolhendo o idioma a ser utilizado

![img1](https://user-images.githubusercontent.com/108817932/195676993-0f8f5e72-3e6d-433f-9947-1877ce8f1135.png)

Logo em seguida, preencha as informações para poder instalar o WordPress..

![img2](https://user-images.githubusercontent.com/108817932/195677188-9e01c83d-a971-43f5-aa77-248ad0949186.png)

E Pronto! Sua aplicação foi instalada com sucesso! Basta logar no site para ter acesso ao dashboard.

![img3](https://user-images.githubusercontent.com/108817932/195677288-30af25b5-9c12-43fa-ae07-893e778a91e1.png)
