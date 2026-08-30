# install 

```bash
sudo apt update && sudo apt upgrade
# install k3s 
curl -sfL https://get.k3s.io | sh -
# config 
mkdir -p $HOME/.kube
sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
sudo chown $(whoami):$(whoami) $HOME/.kube/config
sudo chmod 600 $HOME/.kube/config
echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc
```

next copy config file to local machine 
```bash
# on local machine 
scp sleber@homelab:~/.kube/config ~/.kube/k3s-config
export KUBECONFIG=~/.kube/k3s-config
kubectl get nodes
```