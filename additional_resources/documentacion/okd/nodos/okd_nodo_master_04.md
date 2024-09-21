

:one: Verificar el estado de los servicios

```bash
sudo systemctl status crio
sudo systemctl status etcd
sudo systemctl status kube-apiserver
sudo systemctl status kube-controller-manager
sudo systemctl status kube-scheduler
sudo systemctl status kubelet
```

```bash
sudo systemctl restart crio
sudo systemctl restart etcd
sudo systemctl restart kube-apiserver
sudo systemctl restart kube-controller-manager
sudo systemctl restart kube-scheduler
sudo systemctl restart kubelet
```



1. Set Up the KUBECONFIG Environment Variable
To use the oc command without specifying the kubeconfig file each time, set the KUBECONFIG environment variable to point to your kubeconfig file.

bash
Copiar código
export KUBECONFIG=/path/to/your/kubeconfig
Since your kubeconfig file is located at auth/kubeconfig, use the following command:

bash
Copiar código
export KUBECONFIG=/home/core/okd-install/auth/kubeconfig
You can add this command to your ~/.bashrc file to make it persistent:

bash
Copiar código
echo 'export KUBECONFIG=/home/core/okd-install/auth/kubeconfig' >> ~/.bashrc
source ~/.bashrc