#### Cria container
docker run <nome_da_imagem>

#### Verificar containers que estão sendo executados (ativos)
docker ps
docker ps -q (retorna apenas ID)

#### Verificar todos os containers (incluindo os parados)
docker ps -a

#### Rodar comando dentro do container
docker run <nome_da_imagem> echo "Ola Mundo"

#### Abrir terminal do container
docker run -it <nome_da_imagem>

#### Roda container já criado
docker start <id_do_container>
docker stop <id_do_container>
docker stop $(docker ps -q)

#### Roda container já criado integrando terminal
docker start -a -i <id_do_container>

#### Remover container
docker rm <id_do_container>
docker container prune (remover todos os stopped)
docker rm -f <id_do_container> (forca stop e remove)

#### Mostrar imagens Docker
docker images
docker rmi <nome_da_imagem> (remove imagem)

#### Não atrelar docker run ao terminal (segundo plano)
docker run -d <nome_da_imagem>

#### Encaminhamento de porta
docker run -d -P <nome_da_imagem>
docker run -d -p 12345:80 <nome_da_imagem>
docker port <id_do_container> (Saber qual porta está encaminhando)

#### Dar nome ao container
docker run -d -P --name <nome_que_quiser> <nome_da_imagem>

#### Setar variável de ambiente
-e NOMEVAR="conteudo"

#### Uso de volumes externos (-v = volume) (docker host)
docker run -v "/var/www" <nome_da_imagem>
docker inspect <id_do_container> (inspeciona qual pasta o diretorio foi criado)
docker run -it -v "C:\Users\Alura\Desktop:/var/www" <nome_da_imagem>

#### Working directory pra iniciar comando em container
docker run -p 8080:3000 -v "C:\Users\Alura\Desktop\volume-exemplo:/var/www" -w "/var/www" node npm start (executa comando "npm start" dentro de -w "/var/www")

#### Usar comandos dentro de comando docker
$(pwd)

# Mount Dockerfile
```
FROM node:latest (imagem base)
MAINTAINER ACME! (mantedor)
ENV VAR=123 (variavel de ambiente)
COPY . /home/ (pasta pra copiar o código)
WORKDIR /home/ (pasta que rodará comandos)
RUN npm install (rodar comando durante instalacao)
ENTRYPOINT npm start (comando executado ao iniciar container)
EXPOSE 3000 (porta que ficara exposto)
```
### Build
```
docker build -f nomearquivo.Dockerfile -t viniciosbarretos/nomeprojeto .
```

# Create network
```
docker network create --driver bridge <name_network>
docker network ls
docker run -it --name <name_container> --network <name_network> <nome_da_imagem>
```


# Docker Compose
docker-compose.yml (descreve como aplicação vai subir)
```
version: '3' (versao do docker compose)
services:
  nginx:
    build:
      dockerfile: ./docker/nginx.dockerfile
      context: .
    image: viniciosbarretos/nginx
    container_name: nginx
    ports:
      - "80:80"
    networks: production-network
    depends_on:
      - mongo
      - node1
  mongodb:
    image: mongo
    networks: production-network

networks:
  production-network:
    driver: bridge
```
```
sudo docker-compose build
docker-compose up
docker-compose up -d
docker-compose ps (lista servicos rodando)
docker-compose down (stopa os containers e os remove)
docker-compose restart
docker exec -it <id_container> bash
```

# Docker Swarm
Orquestrar containers em cluster distintos
