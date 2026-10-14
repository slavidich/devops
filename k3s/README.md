# Install k3s

On the homelab host:

```bash
sudo apt update && sudo apt upgrade
curl -sfL https://get.k3s.io | sh -

mkdir -p $HOME/.kube
sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
sudo chown $(whoami):$(whoami) $HOME/.kube/config
sudo chmod 600 $HOME/.kube/config
echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc
```

Copy kubeconfig to the local machine:

```bash
scp sleber@homelab:~/.kube/config ~/.kube/k3s-config
export KUBECONFIG=~/.kube/k3s-config
kubectl get nodes
```
