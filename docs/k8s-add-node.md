# ➕ Añadir un nuevo nodo a un cluster de Kubernetes

En este artículo vamos a añadir un nuevo nodo (worker) a un cluster de Kubernetes ya existente. Partimos de un cluster previamente inicializado en el nodo **control-plane**.

---

## 🧱 Preparación del nodo

Antes de unir el nuevo nodo al cluster, es necesario preparar el sistema exactamente igual que en la instalación inicial.

👉 Sigue todos los pasos descritos en la guía anterior hasta antes de ejecutar `kubeadm init`:

📄 [`Instalar k8s`](k8s-install.md)

Esto incluye:

- ⚙️ Configuración del sistema (swap, sysctl, módulos del kernel)
- 🔌 Instalación de CNI plugins
- 🐳 Instalación de CRI-O
- ☸️ Instalación de kubelet, kubeadm y kubectl

> ⚠️ **Importante:** No ejecutes `kubeadm init` en este nodo.

---

## 🔑 Generar comando de unión (join)

Desde el nodo **control-plane**, generamos un token de acceso para unir nuevos nodos al cluster:

```bash
kubeadm token create --print-join-command
```

Este comando devuelve una instrucción completa similar a:

```bash
kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> \
    --discovery-token-ca-cert-hash sha256:<HASH>
```

### 🧠 ¿Qué hace este comando?

- 🔐 **Token**: autentica el nodo contra el cluster
- 🧾 **CA cert hash**: valida la identidad del API Server
- 🌐 **Endpoint**: dirección del control-plane

---

## 🚀 Unir el nodo al cluster

Ahora, en el nodo que queremos añadir, simplemente ejecutamos el comando generado:

```bash
kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> \
    --discovery-token-ca-cert-hash sha256:<HASH>
```

Este proceso:

- 📡 Conecta el nodo con el API Server
- 📥 Descarga la configuración necesaria
- ⚙️ Registra el nodo en el cluster
- ▶️ Arranca los componentes necesarios (`kubelet`)

---

## 🔍 Verificación desde el control-plane

Volvemos al nodo principal y comprobamos que el nuevo nodo aparece correctamente:

```bash
kubectl get nodes
```

Deberíamos ver algo como:

```
NAME            STATUS   ROLES           AGE   VERSION
control-plane   Ready    control-plane   ...   ...
worker-node-1   Ready    <none>          ...   ...
```

> ⏳ Es posible que el nodo tarde unos segundos en aparecer como `Ready`.

---

## 🧪 Verificación de pods

También podemos comprobar que los pods del sistema se despliegan correctamente en el nuevo nodo:

```bash
kubectl get pods -A -o wide
```

Esto nos permitirá ver en qué nodo se está ejecutando cada pod.

