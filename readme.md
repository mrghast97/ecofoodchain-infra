**# EcoFoodChain Infrastructure (LocalStack EKS)**



Questo repository contiene l'infrastruttura necessaria per eseguire la piattaforma \*\*EcoFoodChain\*\* in locale utilizzando \*\*LocalStack\*\* per emulare AWS ed \*\*EKS (Elastic Kubernetes Service)\*\*.



**## Prerequisiti**



Assicurati di avere installato:

\- \*\*Docker \& Docker Compose\*\*

\- \*\*AWS CLI\*\* e wrapper \*\*`awslocal`\*\* (`pip install awscli-local`)

\- \*\*Kubectl\*\*

\- \*\*LocalStack\*\* in esecuzione.







**## Parte 1: Setup Infrastruttura Base (Rete e Cluster)**



Questi passaggi servono a creare la "scatola" (VPC e Cluster Kubernetes) dove gireranno i microservizi. Esegui questi comandi \*\*una volta sola\*\* all'inizio.



**### 1. Creazione Network (VPC e Subnet)**



**Crea la VPC:**

```bash

awslocal ec2 create-vpc --cidr-block 10.0.0.0/16



**Nota:** Copia il VpcId dall'output (es. vpc-xxxxxx). Sostituiscilo nei comandi successivi dove vedi vpc-xxxxxx.





**Crea due Subnet:**



**# Subnet 1**

awslocal ec2 create-subnet --vpc-id vpc-xxxxxx --cidr-block 10.0.1.0/24

\# Prendi nota del SubnetId (es. subnet-111111)



**# Subnet 2**

awslocal ec2 create-subnet --vpc-id vpc-xxxxxx --cidr-block 10.0.2.0/24

\# Prendi nota del SubnetId (es. subnet-222222)





**2. Creazione Security Group**

**Crea il Security Group per il cluster:**

awslocal ec2 create-security-group --group-name eks-sg --vpc-id vpc-xxxxxx --description "SG per EKS"



**Nota:** Copia il GroupId dall'output (es. sg-xxxxxx).





**3. Creazione Cluster EKS**

Avvia il cluster Kubernetes (sostituisci gli ID delle subnet e del security group che hai annotato):



Bash



awslocal eks create-cluster \\

&nbsp; --name mio-cluster \\

&nbsp; --role-arn arn:aws:iam::000000000000:role/eks-service-role \\

&nbsp; --resources-vpc-config subnetIds=subnet-111111,subnet-222222,securityGroupIds=sg-xxxxxx





**4. Configurazione Kubectl**

Collega il tuo client kubectl al cluster locale (necessario ogni volta che riavvii Docker/LocalStack):



Bash



awslocal eks update-kubeconfig --name mio-cluster





**Parte 2: Deployment Microservizi**



Per ogni microservizio, la procedura è:



* Creare il repository ECR.



* Buildare e pushare l'immagine Docker.



* Applicare i manifest Kubernetes.



* Esporre la porta (Port Forwarding).







**Sostituisci NOME\_SERVIZIO con il nome reale**



**# 1. ECR Setup**

awslocal ecr create-repository --repository-name NOME\_SERVIZIO



**# 2. Docker Build \& Push**

docker build -t NOME\_SERVIZIO:latest .

docker tag NOME\_SERVIZIO:latest 000000000000.dkr.ecr.eu-central-1.localhost.localstack.cloud:4566/NOME\_SERVIZIO:latest

docker push localhost.localstack.cloud:4566/NOME\_SERVIZIO:latest



**# 3. Kubernetes Apply**

kubectl apply -f NOME\_SERVIZIO-deployment.yaml

kubectl apply -f NOME\_SERVIZIO-service.yaml



**# 4. Port Forward (Scegli una porta locale libera, es. 8082)**

kubectl port-forward svc/NOME\_SERVIZIO 8082:80





**Troubleshooting Utile:**



**Verifica lo stato dei pod:**



Bash



kubectl get pods





**Controlla i servizi attivi:**



Bash



kubectl get svc





**Vedi i log di un pod specifico:**



Bash



kubectl logs <nome-pod>



**Se LocalStack non risponde:** Verifica che il container Docker sia attivo e che la porta 4566 sia esposta correttamente.



Bash



docker ps















