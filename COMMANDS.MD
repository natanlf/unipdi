## Aula 3

### Construindo imagem docker da api

``docker build -t unipdi-backend .``

### Para ver se criou a imagem

``docker images``

Também é possível ver a imagem criada pelo Docker Desktop
Dessa forma a imagem não está otimizada.

### Rodando a imagem

``docker run -p 8080:8080 unipdi-backend``

Ao rodar a api vamos ter um erro de conexão com o banco de dados, pois a aplicação está tentando se conectar ao MongoDB.
Precisamos informar o endereço do MongoDB para a aplicação, para isso podemos usar a variável de ambiente ``SPRING_DATA_MONGODB_URI``

``docker run -p 8080:8080 -e SPRING_DATA_MONGODB_URI=mongodb://host.docker.internal:27017/unipdi unipdi-backend``

### Fazendo push da imagem para o Docker Hub

``docker tag unipdi-backend <username>/unipds:v1.0.0``

``docker push <username>/unipds:v1.0.0``

O usuário é possível ver ao fazer login do Docker Hub, e o nome da imagem é o nome do repositório criado no Docker Hub.

### Detalhes para fazer o push
O Docker Hub precisa saber exatamente em qual conta e repositório colocar a sua imagem. Para isso, você deve criar uma tag seguindo estritamente a estrutura: seu-usuario/nome-do-repositorio:tag.

Use o comando docker tag passando o nome atual (ou o ID) e o novo nome formatado:

Explicação do comando
### Sintaxe: docker tag [imagem-atual] [seu-usuario-dockerhub]/[nome-do-repositorio]:[versao]
``docker tag minha-api-java username/openfinance-api:v1.0.0``

## Aula 4
### Subindo a aplicação com Docker Compose

``docker-compose up``

Ao ajustar o docker-compose.yml e executar o comando acima, deu o seguinte erro:

``ERROR: Network "unipdi_default" needs to be recreated - option "com.docker.network.enable_ipv4" has changed``

Liste as redes do Docker:

``docker network ls``

Remova a rede problemática:

``docker network rm unipdi_default``

Se tiver algum container atrapalhando, remova os containers:

``docker ps -a``

``docker rm <container_id>``

Caso tenha algum problema no docker compose e precise derrubar os containers e remover os containers, redes e volumes criados, use o comando:

``docker compose down -v``

### Multi stage build
Podemos no docker file usar o multi stage build para otimizar a imagem, ou seja, podemos usar uma imagem para compilar a aplicação e outra imagem para rodar a aplicação, assim a imagem final fica menor e mais leve.

Abaixo criamos um argumento para pegar o nome do jar gerado no target

``ARG JAR_FILE=target/*.jar``

Copia da imagem de build para a imagem final, usando o argumento criado acima para pegar o nome do jar gerado no target e transferir para a imagem final, renomeando para app.jar

```COPY --from=build /app/${JAR_FILE} app.jar```

Ao criar uma nova imagem usando o multi stage build, a imagem final fica menor e mais leve, pois ela só contém o necessário para rodar a aplicação, sem as dependências de build. Faz muita diferença no tamanho da imagem gerado ao comparar com a anterior. A anterior deu 607MB e a com multi stage build apenas 341MB.