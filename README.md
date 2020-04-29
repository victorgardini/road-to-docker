# Road to Docker
Este Readme é um material de consulta para usuários de docker.

#### Criar um novo container
``` docker container run <nome_da_imagem> --name <nome_do_container> ```

Concatenação de 4 comandos:
-docker container pull: baixa a imagem 
-docker container create: cria o container
-docker container start: inicia a execução do container
-docker container exec: executa o container em modo interativo


### Criar imagem a partir de um Dockerfile
``` docker image build -t <image_name> ```
- Executar imagem criada:
``` docker container run <image_name> ```

### Gerenciar containers já criados
``` docker container ps ```
``` docker container ps -q (retorna apenas ID) ```
``` docker container start <container_name> ```
``` docker container stop <container_name> ```
``` docker container prune (remover todos os stopped) ```
``` docker container rm -f <id_do_container> (forca stop e remove) ```

### Modo Interativo
``` docker container run -it <container_name> ```
``` docker container start -ai <container_name> ```
``` docker container exec -t <container_name> ```


### Modo daemon
Modo daemon é uma opção utilizada para rodar containers em background. Para utilizar esse modo, apresente o parametro **-d** ao executar o comando. Exs:
``` docker container start <container_name> -d ```
``` docker container run <container_name> -d ```
``` docker-compose up -d ```


### Manipulação de containers em modo daemon
``` docker container ls ```
``` docker container ls -a ```
``` docker container inspect ```
``` docker container exec ```
``` docker container logs ```


### Comandos básicos no gerenciamento de imagens
``` docker image pull <tag> ```
Baixa a imagem solicitada, este comando pode ser executado implicitamente, quando o docker
precisa de uma imagem para outra operação e não consegue localiza-la no cache local.

``` docker image ls ```
Lista todas as imagens já baixadas, é possível ainda usar a sintaxe antiga: docker images

``` docker image rm <tag> ```
Remove uma imagem do cache local, é possível ainda usar a sintaxe antiga: docker rmi <tag>

``` docker image inspect <tag> ```
Extrai diversas informações utilizando um formato JSON da imagem indicada.

``` docker image tag <source> <tag> ```
Cria uma nova tag baseada em uma tag anterior ou hash.

``` docker image build -t <tag> ```
Permite a criação de uma nova imagem, como veremos melhor em build.

``` docker image push <tag> ```
Permite o envio de uma imagem ou tag local para um registry.


#### Roda container já criado integrando terminal
``` docker start -a -i <id_do_container> ```


#### Encaminhamento de porta
``` docker run -d -P <nome_da_imagem> ```
``` docker run -d -p 12345:80 <nome_da_imagem> ```
``` docker port <id_do_container> (Saber qual porta está encaminhando) ```

#### Dar nome ao container
``` docker run -d -P --name <nome_que_quiser> <nome_da_imagem> ```

#### Setar variável de ambiente
``` -e NOMEVAR="conteudo" ```

#### Uso de volumes externos (-v = volume) (docker host)
``` docker run -v "/var/www" <nome_da_imagem> ```
``` docker inspect <id_do_container> (inspeciona qual pasta o diretorio foi criado) ```
``` docker run -it -v "C:\Users\Alura\Desktop:/var/www" <nome_da_imagem> ```

#### Working directory pra iniciar comando em container
``` docker run -p 8080:3000 -v "C:\Users\Alura\Desktop\volume-exemplo:/var/www" -w "/var/www" node npm start (executa  comando "npm start" dentro de -w "/var/www") ```

#### Usar comandos dentro de comando docker
``` $(pwd) ```

# Create network
``` docker network create --driver bridge <name_network> ```
``` docker network ls ```
``` docker run -it --name <name_container> --network <name_network> <nome_da_imagem>```

# Docker Swarm
Orquestrar containers em cluster distintos

#### Dockerfile
##### Construção
> FROM
Especifica a imagem base a ser utilizada pela nova imagem.

> LABEL
Especifica vários metadados para a imagem como o mantenedor. A especificação do mantenedor era feita usando a instrução específica, MAINTAINER que foi substituída pelo LABEL.

> ENV
Especifica variáveis de ambiente a serem utilizadas durante o build.

> ARG
Define argumentos que poderão ser informados ao build através do parâmetro --build-arg.

> COPY
Copia arquivos e diretórios para dentro da imagem.

> ADD
Similar ao anterior, mas com suporte extendido a URLs. Somente deve ser usado nos casos que a instrução COPY não atenda.

> RUN
Executa ações/comandos durante o build dentro da imagem.

##### Execução
> EXPOSE
Informa ao Docker que a imagem expõe determinadas portas remapeadas no container. A exposição da porta não é obrigatória a partir do uso do recurso de redes internas do Docker. Recurso que veremos em Coordenando múltiplos containers. Porém a exposição não só ajuda a documentar como permite o mapeamento rápido através do parâmetro -P do docker container run.

> WORKDIR
Indica o diretório em que o processo principal será executado.

> ENTRYPOINT
Especifica o processo inicial do container.

> CMD
Indica parâmetros para o ENTRYPOINT.

> USER
Especifica qual o usuário que será usado para execução do processo no container (ENTRYPOINT e CMD) e instruções RUN durante o build.

> VOLUME
Instrui a execução do container a criar um volume para um diretório indicado e copia todo o conteúdo do diretório na imagem para o volume criado. Isto simplificará no futuro, processos de compartilhamento destes dados para backup por exemplo.

##### Exemplo
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
``` docker build -f nomearquivo.Dockerfile -t viniciosbarretos/nomeprojeto . ```

#### docker-compose
##### Comandos:
``` docker-compose up -d ```
Levanta todos os serviços em modo daemon

``` docker-compose ps ```
Similar ao docker container ps, mas se limitando aos serviços indicados no docker-compose.yml

``` docker-compose exec ```
Similar ao docker container exec, mas utilizando como referência o nome do serviço

``` docker-compose down ```
Para todos os serviços e remove os containers

``` docker-compose logs -f -t ```
Exibe a saída do log de serviços
- param -f: seguir log de saída
- param -t: exibir timestamps

``` docker-compose restart ```
Reinicia os serviços

``` docker exec -it <id_container> bash ```
Executar comando no modo interativo

##### Exemplo
arquivo docker-compose.yml (descreve como aplicação vai subir)
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
