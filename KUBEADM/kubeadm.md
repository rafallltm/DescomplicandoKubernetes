
# Descomplicando o Kubernetes - kubeadm

Estas instruções são para o Kubernetes v1.3.

Neste guia, vamos utilizar o Multipass da Canonical para criar três instâncias com 2 CPUs e 2 GB de memória cada para montar o cluster Kubernetes.

## Criando as Instâncias

Execute os comandos abaixo para lançar as três instâncias:

```sh
multipass launch --cpus 2 --memory 2G --name k1 
multipass launch --cpus 2 --memory 2G --name k2 
multipass launch --cpus 2 --memory 2G --name k3
```

Verifique se as instâncias foram criadas corretamente:

```sh
multipass list
```

## Configurações nas Instâncias k1, k2 e k3

Abra o Terminator e divida a janela em três painéis. Conecte-se a cada uma das instâncias:

```sh
multipass shell k1
multipass shell k2
multipass shell k3
```

Ative o modo de transmissão para todos os terminais no Terminator para executar os comandos simultaneamente:

```sh
sudo apt update && sudo apt upgrade -y
```

## Desativando o Swap

```sh
sudo swapoff -a
```

## Carregando os Módulos do Kernel

```sh
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

## Configurando Parâmetros do Sistema

```sh
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

## Instalando Pacotes do Kubernetes

```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Nota: Em versões anteriores ao Debian 12 e Ubuntu 22.04, o diretório /etc/apt/keyrings não existe por padrão e deve ser criado antes do comando curl.

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Instalando o containerd

```sh
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update && sudo apt-get install -y containerd.io
```

## Configurando o containerd

```sh
sudo containerd config default | sudo tee /etc/containerd/config.toml

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl status containerd
```

## Habilitando o serviço kubelet

```sh
sudo systemctl enable --now kubelet
```

Liberar as portas:
TCP 6443, 10250-10255 e 6783

## Inicializando o Cluster no k1

Desative a transmissão para todos os terminais no Terminator. Verifique o endereço IP da interface de rede:

```sh
ip a
```

Copie o endereço IP da interface (exemplo: `10.145.242.77`). Inicialize o cluster:


```sh
sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=10.145.242.77
```
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.145.242.77:6443 --token uz5hre.fo7jzosl78mdycke \
	--discovery-token-ca-cert-hash sha256:cef31a0816282ba38ad749469e4a20687a3b6b84fa3dfd3d29d03c5d7d20f52a 
```


Anote a saída do token que será utilizado para adicionar os nós de trabalho k2 e k3. Configure o cluster para o usuário atual, copie e cole esse comando em seu terminal:

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Informações do arquivo:
```
cat $HOME/.kube/config
kubectl config view
```


## Adicionando os Nós de Trabalho (k2 e k3)

Nos nós k2 e k3, execute o comando `kubeadm join` que foi gerado na inicialização do cluster no k1. Por exemplo:

```sh
sudo kubeadm join 10.145.242.77:6443 --token uz5hre.fo7jzosl78mdycke \
	--discovery-token-ca-cert-hash sha256:cef31a0816282ba38ad749469e4a20687a3b6b84fa3dfd3d29d03c5d7d20f52a 
```

## Verificando os Nós

No nó k1, verifique o status dos nós:

```sh
kubectl get nodes
```


    NAME   STATUS     ROLES           AGE   VERSION
    k1     NotReady   control-plane   15m   v1.30.0
    k2     NotReady   <none>          38s   v1.30.0
    k3     NotReady   <none>          31s   v1.30.0


## Instalando o Weave Net

```sh
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

Verifique novamente o status dos nós:

```sh
kubectl get nodes
```


    NAME   STATUS   ROLES           AGE   VERSION
    k1     Ready    control-plane   41m   v1.30.0
    k2     Ready    <none>          26m   v1.30.0
    k3     Ready    <none>          26m   v1.30.0