# Descomplicando o Kubernetes - Minikube

## Install Minikube:
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x ./minikube
sudo mv ./minikube /usr/local/bin/minikube
minikube version
```

## Selecionar qual hypervisor:
minikube config set driver <SEU_HYPERVISOR> 

Usando do docker:
```
minikube config set driver docker
```

comands:
```
minikube start
minikube stop
minijube delete 
minikube status
minikube dashboard
minikube ssh
minikube logs
```
Para criar um cluster com mais de um nó:
```
minikube start --nodes 2 -p multinode-cluster
```
Remover o cluster:
```
minikube delete
```
```
minikube delete --purge
```


