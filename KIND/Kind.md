# Descomplicando o Kubernetes - kind

###  Para instalar o Docker:
```
curl -fsSL https://get.docker.com | bash

sudo usermod -aG docker "$USER"

newgrp docker
```

###  Instalando o kind no GNU/Linux:
```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.14.0/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind 
```


###  Criando um cluster com o Kind:
```
kind create cluster --name giropops

kind get clusters

kubectl get nodes
```


### Criando um arquivo.yaml de configuração com 1 control-plane e 2 worker:
```
cat << EOF > $HOME/kind-3nodes.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF
```

###  Criar um cluster chamado kind-multinodes utilizando o arquivo kind-3nodes.yaml:
```
kind create cluster --name kind-multinodes --config $HOME/kind-3nodes.yaml
```

###  Para informações do cluster:
```
kubectl cluster-info
kubectl get nodes
```

###  Para deletar: 
```
kind delete cluster --name kind-multinodes
```








