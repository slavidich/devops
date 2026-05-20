# установка 
понятное дело обновляем все  
```bash
sudo apt update && sudo apt upgrade
```
импортируем ключи ssh 
затем k3s ставится просто 
```bash
curl -sfL https://get.k3s.io | sh -
```
helm обязательно нужен 

# команды 
`sudo k3s kubectl get nodes`

Чтобы каждый раз перед kubectl не писать k3s 
```bash
mkdir -p $HOME/.kube
sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
sudo chown $(whoami):$(whoami) $HOME/.kube/config
sudo chmod 600 $HOME/.kube/config
# и 
export KUBECONFIG=$HOME/.kube/config
```
