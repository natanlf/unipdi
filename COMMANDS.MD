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

## Aula 8

### Usando Kubernates para orquestrar os containers

Kubernates é uma plataforma de orquestração de containers que automatiza a implantação, o dimensionamento e a gestão de aplicações em containers. Ele permite que você execute e gerencie seus containers em um cluster de máquinas, garantindo alta disponibilidade, escalabilidade e resiliência.

No docker desktop criarmos um cluester com 4 nós, e para acessar o cluster usamos o kubectl, que é a ferramenta de linha de comando do Kubernetes.
O visual studio code tem uma extensão do Kubernetes que facilita a visualização e o gerenciamento dos recursos do cluster.
Podemos visualizar os nós do cluster com o comando:

``kubectl get nodes``

Podemos ver os serviços rodando no cluster com o comando:

``kubectl get services``

``kubectl get all``

Vamos criar um novo pod com a aplicação do backend

``kubectl run backend-pod --image natanlfdeveloper/unipds pod /backend-pod created
pod/backend-pod created``

Para ver os pods rodando no cluster:

``kubectl get pods``

No Podman desktop, podemos ver os pods rodando no cluster, e também podemos acessar o terminal do pod para verificar os logs e o funcionamento da aplicação.

``kubectl logs backend-pod``

Conseguimos ver informações do pod, no exemplo ele foi para o worker3.

Vamos apagar o pod porque como não tem banco de dados rodando no cluster, a aplicação não vai funcionar, e para isso usamos o comando:

``kubectl delete pod backend-pod``

Vamos usar arquivos yaml para criar os recursos do Kubernetes, pois é uma forma mais organizada e fácil de gerenciar os recursos do cluster. O arquivo yaml é um arquivo de configuração que define os recursos do Kubernetes, como pods, serviços, deployments, etc.

Após criar o arquivo pod.yaml rodamos o seguinte comando para criar o pod:
``kubectl apply -f pod.yaml``

No menu pods/summary do podman desktop, podemos ver o pod criado, e também podemos acessar o terminal do pod para verificar os logs e o funcionamento da aplicação. 
Assim podemos ver como acessar o backend por exemplo, a porta disponível para isso. 

``kubectl logs backend-pod``

Sem o réplica set, o pod fica vulnerável a falhas, pois se o pod cair, a aplicação fica indisponível. Com o replica set, podemos garantir que sempre haverá uma cópia do pod rodando, garantindo alta disponibilidade e resiliência da aplicação.
Sem ele se o pod cair, a aplicação fica indisponível, e com ele, se o pod cair, o replica set cria um novo pod automaticamente para substituir o pod que caiu, garantindo que a aplicação continue disponível.

O arquivo **pod.yaml** foi apenas um exemplo de criação de podman sem réplica.

Vamos ter um pod com o database, outro backend e outro frontend, e para isso vamos criar um arquivo de deployment para cada um, e depois criar um serviço para cada um, para expor as portas e permitir a comunicação entre os pods.

Vamos ter o secret com usuário e senha do banco de dados, e o configmap para as variáveis de ambiente do backend, como a string de conexão com o banco de dados.

### Criando pod do secret do banco de dados

``kubectl apply -f db-secret.yaml``

Conseguimos ver no Podman desktop o secret criado, e também podemos acessar o terminal do secret para verificar as informações do secret.

``kubectl get secret db-secret -o yaml``

### Criando o configmap para o banco de dados

``kubectl apply -f db-configmap.yaml``

Precisamos disso porque não é apenas usuário e senha do secret, precisamos do host, nome do banco de daddos ..

### Subir o deployment do banco de dados

``kubectl apply -f db-deployment.yaml``

O deployment é responsável por criar e gerenciar os pods do banco de dados, garantindo que sempre haja uma cópia do pod rodando, e também permite o escalonamento da aplicação, ou seja, podemos aumentar ou diminuir o número de réplicas do pod para atender a demanda da aplicação.
Ele busca as informações do secret e do configmap para configurar o banco de dados, e também expõe a porta 27017 para permitir a comunicação com o backend.

### Para fazer o deploy na api temos o seguinte:  
``kubectl apply -f backend-deployment.yaml``

Nos deployments temos os services para manter a disponibilidade de ip e portas.
Até o momento temos dois pods de backend porque especificamos 2 réplicas e um do banco de dados.

### Vamos fazer deploy do frontend:

``kubectl apply -f frontend-deployment.yaml``

O NodePort  tem um range de portas entre 30000 e 32767, e é usado para expor o serviço para fora do cluster, ou seja, para permitir o acesso ao serviço a partir de fora do cluster. Ele atribui uma porta aleatória dentro desse range para o serviço, e essa porta pode ser usada para acessar o serviço a partir de fora do cluster.

No foward port do frontend, podemos acessar a aplicação pelo navegador, usando o endereço http://localhost:30000, e assim conseguimos acessar a aplicação do frontend, que está se comunicando com o backend, que por sua vez está se comunicando com o banco de dados.

Só liberamos a porta 5173 no backend, então temos problemas de cors.

### Vamos mudar o frontend para rodar na porta 5173

``kubectl port-forward service/svc-frontend 5173:80``

###  Cenários de mudanças na aplicação

Se precisar fazer alguma mudança na aplicação, como por exemplo, mudar a porta do backend, ou mudar a string de conexão com o banco de dados, ou até mesmo mudar o código da aplicação, precisamos fazer o build da nova imagem, subir a nova imagem para o Docker Hub, e depois atualizar o deployment do backend para usar a nova imagem.
O réplica set vai garantir que a nova imagem seja usada em todos os pods do backend, e assim a aplicação vai continuar disponível durante a atualização, sem downtime.

Tem como ver o histórico de atualizações de deployment com o comando:

``kubectl rollout history deployment/backend-deployment``

Criei a seguinte imagem após atualizar a api:
``docker build -t unipdi-backend-local .``

Para atualizar o deployment do backend para usar a nova imagem, usamos o comando:

``kubectl apply -f backend-deployment.yaml``

Agora conseguimos rotular a mudança feita, e assim conseguimos ver o histórico de atualizações do deployment, e também podemos fazer rollback para uma versão anterior caso necessário.

``kubectl rollout history deployment/backend-deployment``

Para anotar:
``kubectl annotate deployment backend-deployment kubernates.io/change-cause="usando imagem local"``

Podemos fazer rollback para a versão anterior caso necessário, usando o comando:

``kubectl rollout undo deployment/backend-deployment``

## Aula 9

Vamos usar a livenessProd que faz uma requisição para a porta de onde a aplicação está rodando para saber se realmente está funcionando.
No arquivo de frontend-deployment.yaml
No livenessProd do frontend a porta usada é a mesma do container que é a 80, o path vamos usar o raiz porque ele é o suficiente para fazermos 
um get com retorno 200 é suficiente para saber que a aplicação está funcionando.

periodSeconds = é a quantidade de segundos que espera para confirmar se aplicação está pronta. Ou seja, a cada 10 segundos 
verifica se a aplicação está ok.

failureThreshold: 3 = testa 3 vezes para ter certeza que está funcionando.

initialDelaySeconds: 20 = espera 20 segundos a aplicação iniciar, para assim poder fazer os testes.

Liveness = aplicação rodando.

O código abaixo reinicia o container se falhar 3 vezes.

          livenessProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 10
            failureThreshold: 3
            initialDelaySeconds: 20


Readness = aplicação pronta para receber requisição.

O código abaixo vai tirar o pod do tráfego e não existe um reinicio do pod.
Se a checagem falhar 3 vezes vai para NOT READY.

          readinessProbe: 
            httpGet:
              path: /
              port: 80
            periodSeconds: 10  
            failureThreshold: 3    
            initialDelaySeconds: 20  

Precismos também definir o limite de recursos devem podem ser consumidos.

### Escala

[HPA = Horizontal Pod Autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/)

Para escalar temos o arquivo frontend-hpa onde temos o número mínimo de réplicas e o máximo.

Ao aplicar:

``kubectl apply -f frontend-hpa.yaml``

Assim conseguimos criar um servidor de métricas [Repositório Servidor de Métricas](https://github.com/kubernetes-sigs/metrics-server)

No Docker conseguimos ver a versão do Kubernates que nesse caso é a **v1.31.1** que é compatível com a versão **0.8.x** do servidor de métricas.
No repositório do servidor de métricas tem a tabela de compatibilidade.
Em metric releases vamos fazer download da versão **0.8.x**

Colamos o arquivo do download para a pasta com os outro arquivos **yaml**

Adicionar em args essa linha no arquivo **components.yaml**, arquivo do download:
``- --kubelet-insecure-tls``

Executar: ``kubectl apply -f components.yaml``

Em algum momento depois de aplicar o yaml vai atualizar com as métricas e podemos ver com o seguinte comando:
``kubectl get hpa``

Nesse caso está usando 50% de cpu sendo que não queremos que ultrapasse 1%.

Isso está acontecendo porque no arquivo **frontend-hpa** averageUtilization coloquei como 1, estava 50 antes.

O frontend-deployment.yaml coloquei 2m

Por isso estoura o limite e cria novas réplicas. Por exemplo **frontend-deployment.yaml** ficou com 10 réplicas de frontend

Não esquecer de aplicar os yamls que atualizar.