# User Guide Docker 

Rodando o docker de maneira simples

$ docker search "nome da tecnologia ou web server"

ex.:

$ docker search apache

NAME                        DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED

tomcat                      Apache Tomcat is an open source implementa...   1268                 [OK]       

eboraas/apache-php          PHP5 on Apache (with SSL support), built o...   124                  [OK]

eboraas/apache              Apache (with SSL support), built on Debian      70                   [OK]

webdevops/php-apache        Apache with PHP-FPM (based on webdevops/php)    30                   [OK]

nimmis/apache-php5          This is docker images of Ubuntu 14.04 LTS ...   27                   [OK]


execute

$ docker run -d -p 8080:80 -v /home/user:/var/www/html --name eboraas/apache-php 

sempre o primeiro atributo é sobre o servidor local e ou segundo atributo é para o container

-p = mapear porta 

-d = executar em background

-v = mapear um diretorio local dentro do container

--name = nomear o container



### Criando uma imagem Docker personalizada ###

Para criar uma imagem personalizada, primeiro precisamos conhecer o Dockerfile, depois voltamos a criação da imagem própriamente dita. ;)
Dockerfile

Podemos utilizar a linha de comando para inicializar um Docker container com um parâmetro que é o comando a ser executado direto no container para inicializar algum serviço, trazer alguma informação sobre o container em execução, etc.

Ex.:

Executando um container do NGINX com o comando para startar o Web Server:

docker run -d -p <porta_host>:<porta_container> --name <nome_container> <nome_imagem>

ficaria assim:

$ docker run -d -p 8080:80 nginx /usr/sbin/nginx -g "daemon off;"

Mapeando uma pasta:

$ docker run -d -p 8080:80 -v /home/diogo/projeto/html:/usr/sbin/nginx --name diogoab nginx -g "daemon off;"

Com esse comando vamos criar o container com a imagem do nginx passando os parâmetros para inicializar o servidor e listar a porta 80 do container na 8080 do nosso host.

Agora, imagine utilizar essa linha inteira para esse serviço e/ou outros mais que possam ser executados como um MySQL, Redis, MongoDB, etc.

Isso pode e é bem pouco produtivo.

A maneira de automatizar essas tarefas é utilizando um Dockerfile para passar os parâmetros e configurações para a imagem.
Dockerfile é o arquivo de configuração do Docker, nós só precisamos do Dockerfile para criar a imagem, fazer o build e, depois de gerada com as configurações passadas, basta usar a imagem. Não vamos mais precisar do Dockerfile no mesmo diretório da nossa aplicação.

Os comandos no Dockerfile são iguais utilizar Shell Script.
Exemplo de Dockerfile e configuração de um Web Server no Ubuntu

Imagine o seguinte cenário:

Você vai subir um container com a imagem do Ubuntu e instalar e configurar o Servidor Web NGINX.

### Nosso Dockerfile pode ficar assim: ###

FROM ubuntu

MAINTAINER Diogo Alves Barbosa <diogo.alves.barbosa@gmail.com>

RUN apt-get update

RUN apt-get install -y nginx && apt-get clean

ADD ./configs/nginx.conf /etc/nginx/sites-enabled/default

RUN ln -sf /dev/stdout /var/log/nginx/access.log

RUN ln -sf /dev/stderr /var/log/nginx/error.log

RUN echo "daemon off;" >> /etc/nginx/nginx.conf

EXPOSE 8080

ENTRYPOINT [“/usr/sbin/nginx”]

CMD [“start”, “-g”]

Onde:
O parâmetro FROM é a imagem que você vai usar como base para a criação de sua nova imagem.

### Você pode usar o nome da imagem, como no exemplo:###

FROM ubuntu

### Ou com com a tag específica: ###

FROM ubuntu:latest

### Mantenedor da imagem ###

É a pessoa, empresa ou org que mantém essa imagem.

Quando você criar sua própria imagem para subir no Docker Hub, vai precisar deixar isso como seu contato para o mundo.

MAINTAINER Diogo Alves Barbosa <diogo.alves.barbosa@gmail.com>
<<<<<<< HEAD

Executando comandos no container

Agora as coisas legais.
=======
>>>>>>> 

O comando # RUN serve para executar comandos dentro do seu container, assim que você criar ele.

No exemplo, estou atualizando o sistema, instalando o NGINX e depois removendo os pacotes que o sistema não vai precisar (para deixar a imagem mais limpa).

RUN apt-get update
RUN apt-get install -y nginx && apt-get clean

Outra forma de executar comandos via Dockerfile é vista mais abaixo, o ENTRYPOINT junto com o CMD

ENTRYPOINT [“/usr/sbin/nginx”]
CMD [“start”, “-g”]

Aqui estamos inicializando nosso Web Server.

O ENTRYPOINT é quem vai receber os comandos passados pelo CMD. Caso não use o ENTRYPOINT, pode-se passar os comandos somente com o CMD:

CMD service nginx start -g

Adicionando arquivos de configuração no container

Para melhorar a organização, podemos deixar arquivos de configuração dos serviços (Web Server, Banco de Dados, Interpretador, etc) em locais separados.

No exemplo eu usei o arquivo configs/nginx.conf, que será copiado para dentro do container no diretório /etc/nginx/sites-enabled/, com o nome de default.

ADD ./configs/nginx.conf /etc/nginx/sites-enabled/default

O conteúdo do arquivo nginx.conf é o seguinte:

server {
    listen 8080 default_server;
    servlser_name localhost;
    root /usr/share/nginx/html;
    index index.html index.htm;
}

Só uma configuração basica para o NGINX.

Algo interessante que podemos fazer com o Docker é monitorar os logs de uma maneira mais prática. O Docker pega os logs em stdout e stderr. Podemos capturar esses logs e ver de maneira mais rápida (do que entrar pelo container) pelo comando docker logs.

E para deixar isso automático na nossa imagem, deixamos as seguintes linhas no Dockerfile:

RUN ln -sf /dev/stdout /var/log/nginx/access.log
RUN ln -sf /dev/stderr /var/log/nginx/error.log

Nesse caso, estamos capturando os logs do NGINX.

Depois que o container estiver ativo, podemos usar o comando docker logs id_ou_apelido_do_container.
Expondo portas do container

Para expor a porta que o nosso Web Server vai utilizar, usamos o parâmetro EXPOSE com o número da porta:

EXPOSE 8080

Aqui é bem simples, ele vai expor a porta 8080.

Para saber qual porta o host está utilizando como ponte para acesso a porta do container, podemos procurar com o comando:

docker port id_ou_apelido_do_container

Ou usar um:

docker inspect id_ou_apelido_do_container

Na documentação oficial do Docker você encontrará outras informações legais sobre os comandos que podem ser executados no Dockerfile.

# Como gerar a nova imagem

Com nosso arquivo Dockfile configurado, podemos gerar nossa nova imagem.

Para tal, execute o comando build:

docker build -t diogoab/nginx .

O comando build, como o nome diz, é responsável por executar o build da imagem. Com isso vamos criar uma nova imagem com as configurações do nosso Dockerfile.

O -t é um parâmetro para informar que a imagem pertence ao meu usuário.

Em seguida vem o nome da imagem. Aqui coloquei como diogoab/nginx para ficar fácil de localizar no Docker Hub.

Depois vem o “.“” que significa o diretório corrente, pois eu executei o comando build dentro da pasta onde se encontra meu Dockerfile (para facilitar a vida).

Basta aguardar o processo de build, que pode demorar dependendo de sua conexão, quantos programas você colocou para instalar, etc.

Finalizado o processo, você pode ver sua nova imagem com o comando docker images.

# Exportando a imagem para .tar

Você pode querer, agora, carregar essa imagem em um Pen Drive.

Para isso, utilize o comando save:

docker save diogoab/nginx > /tmp/meu_web_server.tar

Você deve passar o ID da imagem ou o nome.

O Docker vai exportar a imagem e compactar em formato .tar.

Agora você pode carregar por Pen Drive, subir em algum lugar para um amigo baixar (ou você mesmo), etc.

# Importando a imagem de um .tar

Como importar essa imagem gerada via comando save?

Usando o comando load.

docker load < /tmp/meu_web_server.tar

O Docker vai importar sua imagem com o nome, tag, etc, que a imagem já possuia antes, não com o nome do arquivo gerado.

Para ver se deu tudo certo, basta rodar o docker images também.
Subindo a imagem para o Docker Hub

Para subir uma imagem para o Docker Hub, primeiro precisará de uma conta no site. Caso não possua, não precisa entrar no site agora (calma apressado(a)), durante o Login, é criado sua conta via Terminal mesmo.

Faça login com sua conta no Terminal usando o comando docker login:

docker login

Para subir a imagem, basta fazer um push, parecido com o nosso querido Git.

docker push nome_da_imagem

Ex.:

docker push diogoab/nginx

Essa imagem, de exemplo, você pode ver nesse link do Docker Hub e os arquivos de configuração do exemplo (Dockerfile e nginx.conf) nesse repositório.


# Troubleshooting

Como remover todos os containers parados:

docker rm $(docker ps -a -q)

Sincronizar o relógio do host com o container:

Para Centos (Na hora de criar o container mapeia o diretório do host com o do container)

docker run -v /etc/localtime:/etc/localtime:ro centos date

Para Ubuntu (Na hora da criar o container passa como variável o timezone(tz)

docker run -e "TZ=America/Salvador" ubuntu date

Acessar o Bash do container

docker exec -it laughing_bardeen bash 



### Exemplo 2 ###

Instalar um template teste

execute: docker run --name diogoab hello-world

este comando vai instalar uma template com nome de hello-world

 $ docker run hello-world
 Unable to find image 'hello-world:latest' locally
 latest: Pulling from library/hello-world
 535020c3e8ad: Pull complete
 af340544ed62: Pull complete
 Digest: sha256:a68868bfe696c00866942e8f5ca39e3e31b79c1e50feaee4ce5e28df2f051d5c
 Status: Downloaded newer image for hello-world:latest

 Hello from Docker.
 This message shows that your installation appears to be working correctly.

 To generate this message, Docker took the following steps:
 1. The Docker Engine CLI client contacted the Docker Engine daemon.
 2. The Docker Engine daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker Engine daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker Engine daemon streamed that output to the Docker Engine CLI client, which sent it
    to your terminal.

 To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

 Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

 For more examples and ideas, visit:
 https://docs.docker.com/userguide/

Vamos rodar o comando docker ps -a para listar todos os container ativos no sistema

 $ docker ps -a

 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
 592376ff3eb8        hello-world         "/hello"            25 seconds ago      Exited (0) 24 seconds ago                       diogoab

 
# Exemplo de como subir o ambiente WordPress

Vamos testar com um WordPress:

    Criar um diretório de trabalho (mkdir /my-app);
    Faça download do WordPress: cd /my-app; curl https://wordpress.org/latest.tar.gz | tar -xvzf –
    Criar um arquivo Dockerfile com o seguinte código:

    docker pull webdevops/php-nginx

    

    obs.: explore templates em https://hub.docker.com/explore/

    Criar um arquivo my-app.yml com o código:

    FROM orchardup/php5
    ADD . /app

    web:
      build: .
      command: php -S 0.0.0.0:8000 -t /my-app
      ports:
        - "80:8000"
      links:
        - db
      volumes:
        - .:/my-app
    db:
      image: orchardup/mysql
      environment:
        MYSQL_DATABASE: wordpress

    Certifique-se de que seu wp-config esteja parecido com este:

    <?php
    define('DB_NAME', 'wordpress');
    define('DB_USER', 'root');
    define('DB_PASSWORD', '');
    define('DB_HOST', "db:3306");
    define('DB_CHARSET', 'utf8');
    define('DB_COLLATE', '');

    define('AUTH_KEY',         'put your unique phrase here');
    define('SECURE_AUTH_KEY',  'put your unique phrase here');
    define('LOGGED_IN_KEY',    'put your unique phrase here');
    define('NONCE_KEY',        'put your unique phrase here');
    define('AUTH_SALT',        'put your unique phrase here');
    define('SECURE_AUTH_SALT', 'put your unique phrase here');
    define('LOGGED_IN_SALT',   'put your unique phrase here');
    define('NONCE_SALT',       'put your unique phrase here');

    $table_prefix  = 'wp_';
    define('WPLANG', '');
    define('WP_DEBUG', false);

    if ( !defined('ABSPATH') )
        define('ABSPATH', dirname(__FILE__) . '/');

    require_once(ABSPATH . 'wp-settings.php');


O que falta? execute docker-compose up no diretório onde você criou o Dockerfile my-app.yml.

Agora é só acessar: http://ipdamaquina e prosseguir com a instalação do Woprdress normalmente.

O que o Compose fez então? nós criamos um arquivo Dockerfile contendo a descrição da imagem que queremos criar e que será utilizada como base para o nosso ambiente, depois definimos qual era esse ambiente, veja que definimos 1 container para atender requisições web e 1 banco de dados, quando executamos o docker-compose up ele criará a imagem baseado no Dockerfile e criar os containers de serviços que definimos no my-app.yml. 


# Comandos Docker compose:

docker-compose up = subir ambiente

docker-compose up -d = Se quiser que o ambiente fique pronto e funcionando em segundo plano.

docker-compose scale web=5 = Se precisar escalar o ambiente, para ter mais containers web, o compose criará e iniciará 5 containers do serviço web que definimos no exemplo my-app.yml. obs.: melhor utilização quando não existe nenhuma mapeamento de portas


# Comandos Docker

docker attach  – Acessar dentro do container e trabalhar a partir dele.

docker build   – A partir de instruções de um arquivo Dockerfile eu possa criar uma imagem.

docker commit  – Cria uma imagem a partir de um container.

docker cp      – Copia arquivos ou diretórios do container para o host.

docker create  – Cria um novo container.

docker diff    – Exibe as alterações feitas no filesystem do container.

docker events  – Exibe os eventos do container em tempo real.

docker exec    – Executa uma instrução dentro do container que está rodando sem precisar atachar nele.

docker export  – Exporta um container para um arquivo .tar.

docker history – Exibe o histórico de comandos que foram executados dentro do container.

docker images  – Lista as imagens disponíveis no host.

docker import  – Importa uma imagem .tar para o host.

docker info    – Exibe as informações sobre o host.

docker inspect – Exibe r o json com todas as configurações do container.

docker kill    – Da Poweroff no container.

docker load    – Carrega a imagem de um arquivo .tar.

docker login   – Registra ou faz o login em um servidor de registry.

docker logout  – Faz o logout de um servidor de registry.

docker logs    – Exibe os logs de um container.

docker port    – Abre uma porta do host e do container.

docker pause   – Pausa o container.

docker ps      – Lista todos os containers.

docker pull    – Faz o pull de uma imagem a partir de um servidor de registry.

docker push    – Faz o push de uma imagem a partir de um servidor de registry.

docker rename  – Renomeia um container existente.

docker restart – Restarta um container que está rodando ou parado.

docker rm      – Remove um ou mais containeres.

docker rmi     – Remove uma ou mais imagens.

docker run     – Executa um comando em um novo container.

docker save    – Salva a imagem em um arquivo .tar.

docker search  – Procura por uma imagem no Docker Hub.

docker start   – Inicia um container que esteja parado.

docker stats   – Exibe informações de uso de CPU, memória e rede.

docker stop    – Para um container que esteja rodando.

docker tag        – Coloca tag em uma imagem para o repositorio.

docker top        – Exibe os processos rodando em um container.

docker unpause – Inicia um container que está em pause.

docker version – Exibe as versões de API, Client e Server do host.

docker wait   – Aguarda o retorno da execução de um container para iniciar esse container.



