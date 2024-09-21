
# Bootstrap configuration

```bash
sudo tee /etc/kubernetes/kubelet-config.yaml > /dev/null <<EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
staticPodPath: /etc/kubernetes/manifests
clusterDomain: cluster.local
clusterDNS:
  - 10.96.0.10
runtimeRequestTimeout: "15m"
cgroupDriver: systemd
failSwapOn: false
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/etc/kubernetes/pki/ca.crt"
authorization:
  mode: Webhook
readOnlyPort: 0
enforceNodeAllocatable: []
EOF
```

### kubelet.conf file

```bash
sudo tee /etc/kubernetes/kubelet.conf > /dev/null <<EOF
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://10.17.4.27:6443
  name: local
contexts:
- context:
    cluster: local
    user: kubelet
  name: local
current-context: local
users:
- name: kubelet
  user:
    client-certificate: /etc/kubernetes/pki/kubelet.crt
    client-key: /etc/kubernetes/pki/kubelet.key
EOF
```

### crio.conf file


```bash
sudo tee /etc/systemd/system/crio.service > /dev/null <<EOF
[Unit]
Description=CRI-O container runtime
After=network.target

[Service]
Type=notify
ExecStart=/opt/bin/crio/crio
Environment="PATH=/opt/bin/crio:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Restart=always
RestartSec=5
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
EOF
```


### kubelet.service file

```bash
sudo tee /etc/systemd/system/kubelet.service > /dev/null <<EOF

[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=https://kubernetes.io/docs/
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/opt/bin/kubelet \
    --kubeconfig=/etc/kubernetes/kubelet.conf \
    --config=/etc/kubernetes/kubelet-config.yaml \
    --container-runtime=remote \
    --container-runtime-endpoint=unix:///var/run/crio/crio.sock \
    --fail-swap-on=false \
    --cgroup-driver=systemd
Restart=always
StartLimitIntervalSec=0
RestartSec=10
CPUAccounting=true
MemoryAccounting=true
# Ensure kubelet has necessary permissions
ExecStartPre=/sbin/sysctl -w net.ipv4.ip_forward=1
ExecStartPre=/bin/mkdir -p /sys/fs/cgroup/systemd/system.slice/kubelet.service
# Commented out the problematic mount line to prevent errors
# ExecStartPre=/bin/mount --bind /sys/fs/cgroup/systemd/system.slice/kubelet.service /sys/fs/cgroup/systemd/system.slice/kubelet.service

[Install]
WantedBy=multi-user.target
EOF
```


```bash
sudo tee /etc/systemd/system/kubelet.service > /dev/null <<EOF

[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=https://kubernetes.io/docs/
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/opt/bin/kubelet \
    --kubeconfig=/etc/kubernetes/kubelet.conf \
    --config=/etc/kubernetes/kubelet-config.yaml \
    --container-runtime=remote \
    --container-runtime-endpoint=unix:///var/run/crio/crio.sock \
    --fail-swap-on=false \
    --cgroup-driver=systemd
Restart=always
StartLimitIntervalSec=0
RestartSec=10
CPUAccounting=true
MemoryAccounting=true
# Ensure kubelet has necessary permissions
ExecStartPre=/sbin/sysctl -w net.ipv4.ip_forward=1
ExecStartPre=/bin/mkdir -p /sys/fs/cgroup/systemd/system.slice/kubelet.service
ExecStartPre=/bin/mount --bind /sys/fs/cgroup/systemd/system.slice/kubelet.service /sys/fs/cgroup/systemd/system.slice/kubelet.service

[Install]
WantedBy=multi-user.target
EOF
```


```bash
sudo tee /etc/kubernetes/kubelet-config.yaml > /dev/null << 'EOF'
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  x509:
    clientCAFile: "/etc/kubernetes/pki/ca.crt"
authorization:
  mode: Webhook
serverTLSBootstrap: true
tlsCertFile: "/etc/kubernetes/pki/kubelet.crt"
tlsPrivateKeyFile: "/etc/kubernetes/pki/kubelet.key"
cgroupDriver: systemd
runtimeRequestTimeout: "15m"
containerRuntimeEndpoint: "unix:///var/run/crio/crio.sock"
EOF
```

# Recargar la configuración de systemd (solo una vez al inicio)
sudo systemctl daemon-reload

# Configuración y estado del servicio CRI-O
sudo systemctl enable crio
sudo systemctl start crio
sudo systemctl restart crio

# Configuración y estado del servicio kubelet
sudo systemctl enable kubelet
sudo systemctl start kubelet
sudo systemctl restart kubelet

# Verificar el estado del servicio kubelet después de todas las operaciones
sudo systemctl status kubelet
