# Full Cycle 3.0

## Docker

### Integrando Docker Desktop com WSL 2

![image](https://github.com/tmansur/fullcycle/assets/18071398/903f09eb-11c1-4237-8d63-686daf853fad)

### Comandos

https://docs.docker.com/reference/cli/docker/

#### Listando containers

`docker ps`

Parâmetros:

- a --> lista todos containers (os execução e os parados)

#### Iniciando container

`docker run <imagem>`

Parâmetros:
-it --> modo iterativo
-p --> porta a ser publicada para acesso externo [porta-máquina-local:porta-container-docker]
-d --> desvincula o terminal do processo que será executado (o bash não fica travado na execução do serviço)
--name --> nome do container
-v <diretorio-local>:<diretório-container> --> faz o mapeamento de um diretório local para dentro do container

#### Removendo container

`docker rm <nome-container>`

Parâmetros:
-f --> força a execução do comando

#### Executando comandos no container

`docker exec <comando>`

Exemplo:

- Acessando o bash do container: `docker exec -it <nome-container> bash`

### Bind mounts

Bind mounts é uma forma de mapearmos diretórios locais detro de containers em execução.

Esse tipo de mapeamento serve para trabalharmos com os fontes localmente e utilizando a infra nos containers para execução.

#### Mapear diretório local para container no docker

Podemos fazer esse mapeamento utilizando volumes ou mount.

`docker run -d --name <nome-container> -p <portas> -v <diretório-origem>:<diretório-destino> <imagem>`

`docker run -d --name <nome-container> -p <portas> --mount type=bind,source=<diretorio-origem>,target=<diretorio-destino> <imagem>`

> [!TIP]
>
> - O atalho `"$(pwd)"` (que retorna o caminho dá pasta corrente) pode ser utilizado para simplificar o comando.
> - O comando deve ser executado num bash no Windows.
> - Exemplo: `docker run -d --name meunginx -p 8080:80 --mount type=bind,source="$(pwd)"/html,target=/usr/share/nginx/html nginx`

> [!NOTE]
> Caso o diretório de origem não exista, ela será criada quando fizemos o mapeamento utilizando volumes, já com `--mount` será exibido um erro.

### Volumes

Comandos: `docker volume --helps`

Podemos criar volumes para serem utilizados dentro de containers do docker.

#### Criação de volume

docker volume create <nome-volume>

#### Detalhes do volume criado

docker volume inspect <nome-volume>

#### Mapeando volume em um container

docker run --name <nome-container> -d --mount type=**volume**,source=<nome-volume>,target=<diretório-container> <imagem>

> [!NOTE]
> O volume será compartilhado entre todos os containers que forem criados apontando para ele.

### Imagens

docker images <nome-imagem>

docker rmi <nome-imagem>

#### Criação de imagens

Utilizamos o arquivo Dockerfile para criação de imagens docker.

Lista de atribuitos do Dockerfile: https://docs.docker.com/reference/dockerfile/

FROM <nome-imagem>

USER <usuário-linux> --> sem esse atributo, o usuário utilizado é o root

WORKDIR <diretorio-dentro-container>

RUN <comandos-a-serem-executados-na-imagem>

> [!TIP]
>
> - Padrão quando temos vários comandos seguidos é utilizar quebra de linha para melhorar a visibilidade.
> - Exemplo:
>   ```
>   RUN apt-get update && \
>       apt-get install vim -y
>   ```

COPY <diretorio-origem>/ <diretorio-destino>/ --> copia o conteúdo do diretório local para um diretório no container

ENTRYPOINT ["<comando>", "<parametro>"] --> comando fixo

CMD ["<comando>", "<parametro>"] --> comando variável, pode ser substituído ao executar `docker run <imagem> <comando> <parâmetro>`. Também pode entrar como parâmetro do ENTRYPOINT quando é executado depois.

##### Gerando uma imagem

`docker build -t tmansur/<nome-imagem>:<versao> <dockerfile>

> [!TIP]
>
> - Se o arquivo Dockerfile estiver na pasta corrente, basta adicionar um . no final do comando.

#### Publicanod imagem no Docker Hub

`docker push tmansur/<nome-imagem>:<versao>`

> [!TIP]
>
> - Para fazer a publicação da imagem temos que fazer a autenticação no Docker Hub atráves do comando: `docker login`

### Networks

Tipos de networks:

- bridge --> Comunicação entre containers (tipo mais comum de ser utilizado).
- host --> Mescla a rede do docker com a rede do host do docker (máquina onde o docker está executando).
- overlay --> Comunicação entre dockers instalados em diferentes máquinas.
- maclan --> Setar um mac address de uma máquina em um container permitindo a comunicação entre eles.
- none --> Container roda de forma isolado.

#### Bridge

Comandos: docker network --help

Listando redes: docker network ls

Detalhes da rede: docker network inspect bridge --> detalhes sobre as redes bridges

Criando uma rede: docker network create --driver bridge <nome-da-rede>

Crinado um container e utilizando a rede criada: docker run -d -it --name <nome-container> --network <nome-da-rede> <imagem>

Exemplo: docker run -d -it --name ubuntu1 --network minharede bash

Conectar um container já em execução a uma rede: docker network connnect <nome-da-rede> <container>

### Criando aplicação Node.js sem o Node

Criando o container com imagem do node e compartilhando diretório local onde ficará os arquivos: `docker run --name meunode --rm -it -p 3000:3000 --mount type=bind,source="$(pwd)"/,target=/usr/src/app node bash`

Gerar projeto inicial no diretório do container:

- cd /usr/src/app
- `npm init` --> Geração do projeto inicial
- `npm install express --save`--> Instalação do Express
- Criando arquivo index.js

  ```JavaScript
  const express = require('express')
  const app = express()
  const port = 3000

  app.get("/", (req, res) => {
      res.send('<h1>Full Cycle</h1>')
  })

  app.listen(port, () => {
      console.log("Rodando na porta " + port)
  })
  ~~~
  ```

- Executando o serviço: `node index.js`
- Acessar o serviço via navegador: localhost:3000

### Multistage Building

Forma de gerar imagens otimizadas.

Dividido em duas ou mais etapas:

- Estágio inicial de geração da imagem
- Estágios de otimização da imagem.

#### Exemplo de imagem otimizada

```YAML
# Dockerfile

# Estágio 1
FROM php:7.4-cli AS builder

WORKDIR /var/www

RUN apt-get update && \
    apt-get install libzip-dev -y && \
    docker-php-ext-install zip

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');"

RUN php composer.phar create-project --prefer-dist laravel/laravel laravel

# Estágio 2
FROM php:7.4-fpm-alpine

WORKDIR /var/www

RUN rm -rf /var/www/html

# Copia o conteúdo da pasta /var/www/laravel da imagem anterior para o WORKDIR da imagem atual
COPY --from=builder /var/www/laravel .

RUN chown -R www-data:www-data /var/www

EXPOSE 9000

CMD [ "php-fpm" ]
```

Geração da imagem: docker build -t tmansur/laravel:prod <caminho-dockerfile>

### Nginx como proxy reverso

...

### Docker-compose

Ferramenta complementar ao docker que baseado em um arquivo .yaml permite gerenciarmos vários containers simultaneamente.

Subir containers: docker-compose up

Parâmetros:

- -d --> desbloqueia o terminal para uso
- --build --> faz rebuild das imagens

Remover containers: docker-compose down

Listar os containers criados pelo docker compose: docker-compose ps

Exemplo de arquivo docker-compose.yaml:

```YAML
version: '3'

services: # Configuraçaõ dos serviços por container
  meularavel:
    build:
      context: ./laravel # Nome da pasta onde está o Dockerfile
      dockerfile: Dockerfile.prod # Nome do arquivo Dockerfile
    image: tmansur/laravel:v1 # Nome da imagem a ser gerada
    container_name: laravel
    networks:
      - laranet

  meunginx:
    build:
      context: ./nginx # Nome da pasta onde está o Dockerfile
      dockerfile: Dockerfile.prod # Nome do arquivo Dockerfile
    image: tmansur/nginx:v1 # Nome da imagem a ser gerada
    container_name: nginx
    networks:
      - laranet
    ports:
      - "8080:80"

networks:
  laranet: # Nome da rede
    driver: bridge
```
