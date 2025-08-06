
Habilitar o kubernetes no Docker Desktop
	- automaticamente cria um node
	- proximo passo é criar um pod com o container contendo o serviço

kubectl version

kubectl config view
kubectl config current-context

kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp

kubectl get nodes
kubectl get pods -o wide
kubectl get replicaset
kubectl get deployment 
kubectl get service

kubectl create deployment [deployment-name] --image=[dockeruser / dockerimg : imgversion]
	- cria um deployment / replicaset / pod
	
kubectl expose deployment [deployment-name] --type=LoadBalancer --port=8080
	- cria um service

### PODS

- Node
	- Pod
		- Container 1
		- Container 2
	- Pod 2 
		- Container 3
		- Container 4

kubectl explain pods

kubectl describe pod hello-kubernetes-6966cd586b-6dww6

### REPLICASETS - garante execucão de x pods

kubectl explain replicaset

kubectl get replicasets / rs
kubectl get pod -o wide
kubectl delete pod hello-kubernetes-6966cd586b-6dww6
- aqui o replicaset cria outro pod, apos o pod ser deletado

kubectl scale deployment hello-kubernetes --replicas=3
- escalando para fazer o minimo de pods em execução = 3

### DEPLOYMENT 

kubectl get rs -o wide
kubectl set image deployment hello-kubernetes hello-kubernetes=leandrocgsi/hello-kubernetes:erudio-0.0.2
	- *caso a imagem nao exista ou ocorra um erro* -> kubernetes confirma mesmo assim o deployment da imagem, mas internamente ocorre um erro, mostrando os rs, havera um pod com imagem invalida!
	- mudando a docker image do deployment
	- deployment cria outro replica set e abre 3 novos pods, depois desabilita o replicaset da versão antiga
		- deployment continuo


### SERVICE

kubectl get pods -o wide
	- diferentes ips nos pods, mas acessando o localhost8080, podemos acessar todas as portas
	- as cargas estao sendo balanceadas entre os pods pelo service
	- caso um pod seja deletado, outro é criado com um ip novo incrementado pelo service

pods -> unidades descartaveis (criados, deletados, mudados)
	- o objetivo do service é sempre manter uma interface disponivel mantendo os pods ativos

service foi criado no expose deployment
	- kubectl expose deployment [deployment-name] --type=LoadBalancer --port=8080

kubectl
	- ClusterIP
		- So pode ser acessado por dentro do cluster
		-  Não ha um IP externo
		-  Usado para serviços que são diretamentes usados de dentro do Cluster


### ARQUITETURA DO KUBERNETES

Cluster
 - Master Node(s)
	 - Gerencia o Cluster
 - Worker Node(s)
	 - Executa as nossas aplicações


Componenetes do Master Node
- etcd
	- Banco de Dados Distribuido
		- Todas as mudanças de config, deployments, operações de escalonamento são armazenadas aqui
		- Executando esses comandos estamos definindo o estado desejado ao Kubernetes
		- Estados desejados são armazenados aqui
- API Server
	- kube-apiserver
		- alterações são passadas primeiramente para o api-server e depois processada a partir
- Scheduler
	- kube-scheduler
		- responsavel por agendar os pods nos nodes
		- fatores
			- memoria disponivel
			- cpu disponivel
			- conflito de portas
			- etc
- Controller Manager
	- kube-controller-manager
		- gerencia a saude geral do cluster
		- assegura que o estado desejavel corresponda ao estado atual


Componentes do Worker Node
- PODS
	- Varios Pods executando containers
		- onde sua aplicação estará rodando
- Node Agent
	- kubelet
		- assegura e monitora o comportamento do worker node e comunica para o master node
- Componente de Rede
	- kube-proxy
		- ajuda expor os serviços em torno dos pods
- Container Runtime
	- CRI - Docker, rkt, Podman, etc
		- kubernetes possui compatibilidade com varias tecnologias runtime

Pontos Importantes
- Master node tipicamente so possui coisas relacionadas ao controlamento dos worker nodes
- Container deve ser compativel ao Open Container Interface (OCI) para rodar no Kubernetes 
- Caso o Master Node ou uma parte sua (api-server) cair, as aplicações irão continuar executando


### STATUS DOS COMPONENTES

Ferramentas de Monitoramento:
kubectl get componentstatuses / cs
- status dos componentes do master node
-  kubectl get --raw='/readyz?verbose'
	- pois get cs está deprecated
- kubectl get --raw='/livez?verbose'
- todas essas informações podem ser acessadas pelo browser (localhost:6443/readyz?verbose)

### KUBERNETES DASHBOARD

Links importantes:
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
https://github.com/kubernetes/dashboard

Passos para habilitar o dashboard:
1) kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
2) kubectl create serviceaccount admin-user -n kubernetes-dashboard
3) kubectl create clusterrolebinding admin-user-binding --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:admin-user
4) kubectl -n kubernetes-dashboard create token admin-user
5) kubectl proxy
6) Acessar -> http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

### Deletando os deployments, pods e services
kubectl delete deployment ...
kubectl delete service ...
kubectl delete pod ...
kubectl delete all -l app=hello-kubernetes


### Dando deploy nos Microserviços exchange-service e book-service
- alteramos o controller e o proxy para o feign fazer a comunicação entre correta entre os microserviços
- tiramos o naming-server, api-gateway e zipkin dos application.yml, docker-compose.yml e continuous-deployment.yml
  - pois o kubernetes ja faz o balanceamento de carga e descoberta de serviços
- adicionamos uma versao kube-v1 nas imagens dos microserviços
- dando deploy no kubernetes
  - kubectl create deployment exchange-service --image=ayrtonyoshii/exchange-service:kube-v1
  - kubectl set env deployment exchange-service SPRING_DATASOURCE_URL=jdbc:mysql://host.docker.internal:3306/exchange_service
    - datasource neste caso vai ser local, pois o mysql está rodando no docker desktop, SPRING_DATASOURCE_URL esta no docker-compose.yml
  - kubectl set env deployment exchange-service SPRING_DATASOURCE_USERNAME=root
  - kubectl set env deployment exchange-service SPRING_DATASOURCE_PASSWORD=root
  - kubectl expose deployment exchange-service --type=LoadBalancer --port=8000

### Mudando as configurações do deployment por meio do yml
-  kubectl get deployment exchange-service -o yaml >> deployment.yml
  - salva o deployment em um arquivo yml
  - podemos editar o arquivo e aplicar as mudanças
    -  kubectl diff -f deployment.yml
      - precisamos do diffutils instalado (chocolatey para instalar no WPS)
	-  kubectl apply -f deployment.yml

