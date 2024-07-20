# Descomplicando o Kubernetes - Kubectl


###  Instalando e customizando o Kubectl no GNU/Linux:

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

```
chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version --client
```

Verifique qual shell:
```
echo $SHELL
```

No Bash:
```
source <(kubectl completion bash)
```

```
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

No ZSH:
```
source <(kubectl completion zsh)
```
```
echo "[[ $commands[kubectl] ]] && source <(kubectl completion zsh)"
```


### Crie o alias k para kubectl:

No bash:
```
echo "alias k=kubectl" >> ~/.bashrc
source ~/.bashrc
```

No Zsh:
```
echo "alias k=kubectl" >> ~/.zshrc
source ~/.zshrc
```

### Uninstall:
```
sudo rm /usr/local/bin/kubectl
rm -rf ~/.kube
```


