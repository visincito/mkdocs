# 🚀 Instalación de un nodo de Kubernetes paso a paso

En este artículo vamos a desplegar un nodo de Kubernetes desde cero utilizando herramientas estándar como `kubeadm`, `kubelet` y `kubectl`, junto con el runtime **CRI-O**. Todo el proceso está basado en una instalación manual, ideal para entender qué ocurre en cada capa del sistema.

---

## 🖥️ Preparación inicial del sistema

En mi caso he usado una máquina virtual con Debian 13 en proxmox, por lo que instalo el agente de qemu y habilito el ssh para conectarme remotamente:

```bash
apt install qemu-guest-agent
apt install openssh-server
apt install vim
apt update && apt upgrade
```

Configurar SSH es fundamental para administrar el nodo remotamente. Tras editar `/etc/ssh/sshd_config`, reiniciamos el servicio:

```bash
systemctl restart sshd
systemctl status sshd
```

También es recomendable establecer una contraseña segura:

```bash
passwd
```

Y comprobar la red:

```bash
ip a
```

---

## ⚙️ Configuración del sistema para Kubernetes

Kubernetes requiere ciertos ajustes a nivel de kernel y red.

### 🔴 Desactivar swap

Kubernetes no funciona correctamente con swap activo:

```bash
swapoff -a
cat /etc/fstab
```

---

### 🌐 Habilitar módulos del kernel

Necesitamos cargar módulos clave para networking:

```bash
modprobe br_netfilter
modprobe overlay
```

Verificamos:

```bash
lsmod | grep br_netfilter
lsmod | grep overlay
```

Los persistimos:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

---

### 🔧 Configurar parámetros de red

Configuramos sysctl para permitir el tráfico entre pods:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

Aplicamos los cambios:

```bash
sysctl --system
sysctl net.ipv4.ip_forward
```

---

## 🔌 Instalación de plugins CNI

Los plugins CNI permiten la comunicación entre pods.

```bash
apt install wget htop btop curl net-tools
wget https://github.com/containernetworking/plugins/releases/download/v1.8.0/cni-plugins-linux-amd64-v1.8.0.tgz
```

Verificamos la integridad:

```bash
echo "ab3bda535f9d90766cccc90d3dddb5482003dd744d7f22bcf98186bf8eea8be6 cni-plugins-linux-amd64-v1.8.0.tgz" | sha256sum --check
```

Instalamos:

```bash
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.8.0.tgz
```

---

## 🐳 Instalación de CRI-O y Kubernetes

Definimos versiones:

```bash
KUBERNETES_VERSION=v1.35
CRIO_VERSION=v1.35
```

Añadimos repositorios:

```bash
apt install gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/ /" | tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_VERSION/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_VERSION/deb/ /" | tee /etc/apt/sources.list.d/cri-o.list
```

Instalamos componentes:

```bash
apt update
apt-get install -y cri-o kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

---

## ▶️ Arranque de servicios

Activamos los servicios necesarios:

```bash
systemctl enable --now kubelet
systemctl start crio.service
systemctl enable crio.service
```

---

## 🧠 Inicialización del cluster

Creamos el cluster con `kubeadm`:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16
```

El parámetro `--pod-network-cidr` es importante para que el plugin de red (en este caso Flannel) funcione correctamente.

---

## 🔑 Configuración de kubectl

Para poder interactuar con el cluster:

```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

Verificamos:

```bash
kubectl get nodes
```

---

## 🌐 Despliegue de red (Flannel)

Instalamos Flannel como solución de red:

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

---

## 🔍 Despliegue de metrics-server
Instalamos metrics-server para poder monitorizar el cluster:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

En mi caso, quiero correr pods en el nodo control-plane, por lo que hago lo siguiente:

```bash
kubectl taint nodes k8s-cluster node-role.kubernetes.io/control-plane:NoSchedule-
```
Y le añado tambien un parámetro para que no se queje del certificado TLS:

```bash
kubectl edit deployment metrics-server -n kube-system
```
``` bash
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls     # <--- AQUI
```

---

## 🔍 Verificación del cluster

Comprobamos el estado de todos los recursos:

```bash
kubectl get all -A
kubectl get nodes
```
