<img src="https://user-images.githubusercontent.com/49697760/111070205-4bc0c580-84af-11eb-9c12-bf3371b247cc.png" height="70" width="360">

Acompanhamento do treinamento de kubernetes no Alura.

O que é Kubernetes?
- Kubernetes é um sistema de orquestração de contêineres open-source que automatiza a implantação, o dimensionamento e a gestão de aplicações em contêineres. Ele foi originalmente projetado pelo Google e agora é mantido pela Cloud Native Computing Foundation. O
- Kubernetes trabalham com pods que são capsulas que podem conter um ou mais containeres. Ao ser criado, ele ganha um endereço IP. Podemos ter mais de um container dentro deste pod com portas diferentes. Não pode existir mais de um container com a mesma porta dentro do mesmo pod. Caso no pod haja apenas um container e ele deixe de funcionar o pod tmbém deixa de funcionar e o Kubernetes pode criar um novo pod com um novo numero IP. Se toda vez que um po d é criado ele assume um numero IP diferente, como manter uma operação com vários pods se comunicando entre si funcionando? Veja abaixo o item Service.
 
## Instalação kubernetes no Linux:

Os comandos necessários para a instalação e inicialização do cluster no Linux podem ser obtidos abaixo:

Primeiro para o kubectl:

    sudo apt-get install curl -y
    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl

Agora para o minikube:

    curl -Lo minikube https://storage.googleapis.com/minikube/releases/v1.12.1/minikube-linux-amd64 \ && chmod +x minikube
    sudo install minikube /usr/local/bin/
    
Instalação de um drive virtual: no Linux é necessário fazer a instalação de um drive virtual(Virtual Box, por exemplo) - será utilizado apenas como driver de virtualização.

    sudo dpkg –i <nome do arquivo>.deb

Iniciando o minikube

    minikube start --vm-driver=virtualbox
    
[Página do minikube](kubernetes.io/docs/tutorials/hello-minikube)

* ATENÇÃO: utilizando o Linux este comando deve ser executado sempre antes de utilizar o Kubernetes

## Comandos 

- Vendo os pods já criados: <code>kubectl get pods</code>
      - <code>kubectl get pods -o wide</code> (com mais detalhes)
- Criando um pod: <code>kubectl run nginx-pod --image=nginx:latest</code>
- Visualizando a descrição de um pod: <code>kubectl describe pod nginx-pod</code>
- Editando um pod: <code>kubectl edit pod nginx-pod</code>
- Deletando um pod: <code>kubectl delete pod nginx-pod</code> (caso pod não declarativo)
                    <code>kubectl delete -f .\primeiro-pod.yaml</code> (caso pod declarativo) 
- Enviando requisição de um pod para outro: 
     - <code>kubectl exec -it pod-1 -- bash</code>
     - <code>curl "ip do serviço:porta"</code>
- Vendo nodes criados: <code>kubectl get nodes -o wide</code>  
- Acessando o banco de dados:
   - <code>kubectl exec -it db-noticias -- bash</code>
   - <code>mysql -u root -p</code>
- Criando um pod, svc, etc... via arquivo yaml: <code>kubectl apply -f ".\nome do arquivo.yaml"</code> 
- Vendo o histórico de versões dos Pods: <code>kubectl roolout history deployment "nome-do-deployment"</code>
- Sobreescrevendo arquivo de deployment com alterações: <code>kubectl apply -f .\"nome do arquivo" --record</code>
- Melhorando o histórico de versões descrevendo a ação feita: <code>kubectl annotate deployment "nome-deployment" kubernetes.io/change-cause="Descrição que queira colocar"</code>
- Voltando versão do histórico: <code>kubectl undo deployment "nome-do-deployment" --to-revision-"número-da-revisão"</code>
- Voltando um Deployment para uma revisão específica: <code>kubectl rollout undo deployment <nome do deployment> --to-revision=<versão a ser retornada></code>
- Acessando o volume nginx-container: <code>kubectl exec -it pod-volume --container nginx-container</code>
- Acessando a pasta que foi criada: <code>cd volume-dentro-do-container</code>
- Criando o arquivo "arquivo.txt": <code>touch arquivo.txt</code>
- Visualizando o IP local <code>kubectl get nodes -o wide</code>

## Criando pods de maneira declarativa

- No editor de texto entre com os comandos abaixo:

      apiVersion: v1
      kind: Pod
      metadata:
        name: primeiro-pod-declarativo
        labels:
          app: primeiro-pod
      spec:
        containers:   
          - name: nginx-container
            image: nginx:latest
            ports: 
              - containerPort: 80            
            
- Salve o arquivo com nome primeiro-pod.yaml
- No powershell execute o comando(dentro da pasta onde está o arquivo yaml):  kubectl apply -f .\primeiro-pod.yaml          
      
Para saber se um yaml é valido, utilize: [validador](https://kubeyaml.com/)

## Service

- Abstrações para expor aplicaçõesexecutando em um ou mais pods;
- Proveem IPs fixos para comunicação;
- Proveem um DNS para um ou mais pods;
- São capazes de fazer balanceamento de carga.

![services kubernetes](https://user-images.githubusercontent.com/49697760/113857395-db892500-9778-11eb-95db-5175c7339d31.jpg)

### Criando services:

      apiVersion: v1
      kind: Service
      metadata:
        name: svc-pod-2
      spec:
        type: ClusterIP
        selector:
          app: segundo-pod
        ports:
          - port: 9000  
            targetPort: 80

## Node port

- Permitem a comunicação com o mundo externo;
- Também pode funcionar como ClusterIP

## Criando nodePod de maneira declarativa

    apiVersion: v1
    kind: Service
    metadata:
      name: svc-pod-1
    spec:
      type: NodePort
      ports:
        - port: 80  
          #targetPort: 80
          nodePort: 30000
      selector:
        app: primeiro-pod 
        
## Loadbalancer

- Utilizam automaticamente os balanceadores de carga de cloud providers;
- Por ser um Load Balancer, também são um NodePort e ClusterIP ao mesmo tempo.

## Criando loadBalancer de maneira declarativa

    apiVersion: v1
    kind: Service
    metadata:
      name: svc-pod-1-loadbalancer
    spec:
      type: loadBalancer
      ports:
        - port: 80  
          #targetPort: 80
          nodePort: 30000
      selector:
        app: primeiro-pod

## Definindo variáveis de ambiente

Este é um exemplo da criação de variáveis de ambiente no serviço. 
Elas foram criadas em "env" para acesso ao banco de dados:

    apiVersion: v1
    kind: Pod
    metadata:
      name: db-noticias
      labels:
        app: db-noticias
    spec:
      containers:
        - name: db-noticias-container
          image: aluracursos/mysql-db:1
          ports:
            - containerPort: 3306
          env:
            - name: "MYSQL_ROOT_PASSWORD"  
              value: "q1w2e3r4"
            - name: "MYSQL_DATABASE"  
              value: "empresa"
            - name: "MYSQL_PASSWORD"  
              value: "q1w2e3r4" 

## Criando um Config Map

Um config map é uma arquivo com as configurações que são necessárias dentro de determinados pods permitindo a reutilização e desacoplamento.

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: db-configmap
    data:
      MYSQL_ROOT_PASSWORD: q1w2e3r4
      MYSQL_DATABASE: empresa
      MYSQL_PASSWORD: q1w2e3r4  
        
## Importando apenas uma linha do configmap:

    apiVersion: v1
    kind: Pod
    metadata:
      name: db-noticias
      labels:
        app: db-noticias
    spec:
      containers:
        - name: db-noticias-container
          image: aluracursos/mysql-db:1
          ports:
            - containerPort: 3306
          # veja daqui para baixo
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: db-configmap
                  key: MYSQL_ROOT_PASSWORD 
                  
## Importando todas as informações do configmap:

    apiVersion: v1
    kind: Pod
    metadata:
      name: db-noticias
      labels:
        app: db-noticias
    spec:
      containers:
        - name: db-noticias-container
          image: aluracursos/mysql-db:1
          ports:
            - containerPort: 3306
              #Nome do arquivo configmap
          envFrom:
            - configMapRef:
                name: db-configmap
              
## ReplicaSets              
              
- ReplicaSet nada mais é que uma estrutura que encapsula, pode encapsular, na verdade, um ou mais Pods. No caso de um dos Pods falhar, e ele nunca mais vai voltar, é o ReplicaSet quem vai conseguir criar um novo para nós. E isso também vai funcionar para diversos Pods gerenciados por um mesmo ReplicaSet.              

Exemplo:

    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: portal-noticias-replicaset
    spec:
      template:
        metadata:
          name: portal-noticias
          labels:
            app: portal-noticias
        spec:
          containers:
            - name: portal-noticias-container
              image: aluracursos/portal-noticias:1
              ports:
                - containerPort: 80
              envFrom:
                - configMapRef:  
                    name: portal-configmap
      replicas: 3
      selector:
        matchLabels:
          app: portal-noticias      

## Deployments:

- Está uma camada acima do repliSet. Quando um deployment é definido, automaticamente definindo um replicaSet.

Exemplo:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec: 
      replicas: 3
      template:
        metadata:
          name: nginx-pod
          labels: 
            app: nginx-pod
        spec:
          containers:
            - name: nginx-container
              image: nginx:stable
              ports:
                - containerPort: 80
      selector:
        matchLabels:
          app: nginx-pod


## Volumes

Recurso do kubernetes para armazenar arquivos e possuem ciclo de vida independente dos containeres, mas dependente dos Pods.

Exemplo:

    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-volume
    spec:
      containeres: 
        - name: nginx-container
          image: nginx:latest
          volumeMounts:
            - mountPath: /volume-dentro-do-container
              name: primeiro-volume
        - name: jenkins-container
          image: jenkins:alpine
          volumeMounts:
            - mountPath: /volume-dentro-do-container
              name: primeiro-volume      
      volumes:
        name: primeiro-volume    
        hostPath:
          path: /C/Users/Daniel/Desktop/primeiro-volume
          type: Directory
          
- O exemplo acima funciona quando o diretório já foi criado. Se não foi, em type, utilize DirectoryOrCreate

Exmplo Linux:

    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-volume
    spec:
      containers: 
        - name: nginx-container
          image: nginx:latest
          volumeMounts:
            - mountPath: /volume-dentro-do-container
              name: segundo-volume
        - name: jenkins
          image: jenkins/jenkins:alpine 
          volumeMounts:
            - mountPath: /volume-dentro-do-container
              name: segundo-volume      
      volumes:
        - name: segundo-volume    
          hostPath:
            path: /home/segundo-volume
            type: DirectoryOrCreate
            
## Persistent Volumes

[Documentação oficial](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

## Storage Classes

- Com ele nós vamos conseguir criar Volumes, Persistent Volumes, no caso, e discos dinamicamente.

Exemplo:

    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
        name: slow
    provisioner: kubernetes.io/gce-pd
    parameters:
        type: pd-standard
        fstype: ext4
        replication-type: none

## StatefulSets

Quando um pod é criado dentro de um StatefulSet, também são criados o Persistent Volue e Persistent Volume Claim e, quando um pod falhar, um novo pod é criado assumindo a mesma identidade do pod que falhou podendo acessar as mesmas informações que já vinham sendo acessadas.

Exemplo: 

    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: sistema-noticias-statefulset
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: sistema-noticias
          name: sistema-noticias
        spec:
          containers:
            - name: sistema-noticias-container
              image: aluracursos/sistema-noticias:1
              ports:
                - containerPort: 80
              envFrom:
                - configMapRef:
                    name: sistema-configmap
              volumeMounts:
                - name: imagens
                  mountPath: /var/www/html/uploads
                - name: sessao
                  mountPath: /tmp
          volumes:
            - name: imagens
              persistentVolumeClaim:
                claimName: imagens-pvc
            - name: sessao
              persistentVolumeClaim:
                claimName: sessao-pvc
      selector:
        matchLabels:
          app: sistema-noticias
      serviceName: svc-sistema-noticias

- No exemplo acima o StatefulSet vai recriar os pods de sessão e de imagens. Desta forma, para as pessoas que estão acessando a aplicação, caso o pod venha a dar problemas, vão continuar com sua sessão e as imagens inseridas vão continuar a disposição.

## Probes

Tem como função tornar visível ao Kubernetes que uma aplicação não está se comportando da maneira esperada. Assim o Kubernetes pode tormar uma atitude como eliminar o pod e criar automaticamente outro.

## Liveness Probes
 
 O Liveness Probe nada mais do que uma prova de vida que a aplicação dentro de um container de um Pod está funcionando. 
 
 Exemplo:
 
           livenessProbe:
            httpGet:
              path: /
              port: 80     
            periodSeconds: 10
            failureThreshold: 3
            initialDelaySeconds: 20  
            
- O exemplo acima foi extraido do porta-noticias-deployment.yaml abaixo da definição de envFrom.            

## Readiness Probes

O Readiness Probe informa quando um Pod está pronto para receber requisições. Sua definição é semelhante a do Liveness Probe e deve ser colocado no código abaixo deste: 

           readinessProbe:
            httpGet:
              path: /
              port: 80     
            periodSeconds: 10
            failureThreshold: 5
            initialDelaySeconds: 3  
            
## Startup Probes

Este proble não foi visto no curso, mas tem sua definição no link: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-startup-probes


## Horizontal Pod Autoescaler

- É um recurso que visa manter a utilização de memória e/ou cpu dentro de padrões pré estabelecidos criando novos pods e, quando o consumo cair, deletando os mesmos.

No arquivo de deployment que será monitorado, inclua o código abaixo em containeres:

          resources:
            requests:
              cpu: 10m    
Exemplo:

    apiVersion: autoscaling/v2beta2
    kind: HorizontalPodAutoscaler
    metadata:
      name: portal-noticias-hpa
    spec:
      scaleTargetRef:
        apiVersion: apps?v1
        kind: Deployment
        name: portal-noticias-deployment
      minReplicas: 1
      maxReplicas: 10
      metrics:
        - type: Resource
          resource:
            name: cpu
            target:
              type: Utilization
              averageUtilization: 50
          

## Servidor de métricas

[Site do kubernetes no git sobre servidor de métricas](https://github.com/kubernetes-sigs/metrics-server)

### No Linux: 

Para verificar as extensões permitidas: minikube addons list
Para habilitar alguma extensão: minikube addons enable metrics-server

- Arquivo de stress: 

      #!/bin/bash
      for i in {1..10000}; do
        curl localhost:30000
        sleep $1
      done
      
- Executando arquivo de stress: bash stress.sh 0.001 > out.txt

