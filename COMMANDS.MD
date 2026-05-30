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