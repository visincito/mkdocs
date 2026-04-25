---
draft: false
date: 2026-04-25
authors: [Visin]
categories:
  - Kubernetes
  - Herramientas
tags:
  - k9s
  - kubernetes
  - cli
  - terminal
---

# 🐕️ Instalación y Configuración de K9s

K9s es un CLI visual para Kubernetes que te permitirá gestionar tus clusters de forma eficiente 🚀. En esta guía verás todos los métodos de instalación, configuración avanzada y comandos esenciales ⚡.

<!-- more -->
---

## 🎯 ¿Por qué usar K9s?

| Característica | 📝 Descripción |
|----------------|-----------------|
| **Interfaz visual** | Navega por tus recursos Kubernetes sin necesidad de recordar infinitos comandos `kubectl` |
| **Gestión de recursos** | Ver, editar, eliminar y crear recursos directamente desde la terminal |
| **Logs en tiempo real** | Follow de logs de múltiples pods simultáneamente |
| **Port forwarding** | Configura port-forwards de forma visual |
| **Plugin system** | Extiende funcionalidad con plugins personalizados |
| **Skin customizable** | themes yPersonaliza el aspecto visual |

> 📌 **Nota**: K9s requiere Kubernetes 1.28+ y conexión a un cluster configurado en `~/.kube/config`

---

## 📦 Métodos de Instalación

### 🐧 Linux

#### 🔧 Método 1: Binary directa (Recomendado 🔥)

```bash title="Instalar K9s"
# Descargar la última versión
K9S_VERSION=$(curl -sS https://api.github.com/repos/derailed/k9s/releases/latest | grep -oP '"tag_name":\s*"\K[^"]+' | sed 's/v//')
curl -sL "https://github.com/derailed/k9s/releases/download/v${K9S_VERSION}/k9s_linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/k9s /usr/local/bin/k9s
k9s version
```

#### 🔧 Método 2: APT/Dpkg (Ubuntu/Debian) 🐞

```bash title="Instalar vía .deb (Script directo)"
wget https://github.com/derailed/k9s/releases/latest/download/k9s_linux_amd64.deb && sudo apt install ./k9s_linux_amd64.deb && rm k9s_linux_amd64.deb
```

O versión manual:

```bash title="Instalar vía .deb (Manual)"
K9S_VERSION=$(curl -sS https://api.github.com/repos/derailed/k9s/releases/latest | grep -oP '"tag_name":\s*"\K[^"]+' | sed 's/v//')
wget -q "https://github.com/derailed/k9s/releases/download/v${K9S_VERSION}/k9s_linux_amd64.deb" -O k9s.deb
sudo apt install ./k9s.deb -y
rm k9s.deb
```

#### 🔧 Método 3: Pacman (Arch Linux) 🦌

```bash
sudo pacman -S k9s
```

#### 🔧 Método 4: Snap 🟦

```bash
sudo snap install k9s --devmode
```

#### 🔧 Método 5: Homebrew (Linux/macOS) 🍺

```bash
brew install derailed/k9s/k9s
```

---

### 🍎 macOS

#### 🍏 Método 1: Homebrew

```bash
brew install derailed/k9s/k9s
```

#### 🍏 Método 2: MacPorts

```bash
sudo port install k9s
```

#### 🍏 Método 3: Binary directa

```bash
K9S_VERSION=$(curl -sS https://api.github.com/repos/derailed/k9s/releases/latest | grep -oP '"tag_name":\s*"\K[^"]+' | sed 's/v//')
curl -sL "https://github.com/derailed/k9s/releases/download/v${K9S_VERSION}/k9s_darwin_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/k9s /usr/local/bin/k9s
```

> ⚠️ **macOS Silicon (Apple Silicon)**: Usa el binary `k9s_darwin_arm64.tar.gz`

---

### 🪟 Windows

#### 🖥️ Método 1: Winget

```powershell
winget install k9s
```

#### 🖥️ Método 2: Scoop

```powershell
scoop install k9s
```

#### 🖥️ Método 3: Chocolatey

```powershell
choco install k9s
```

#### 🖥️ Método 4: Powershell directa

```powershell
$version = (Invoke-RestMethod -Uri "https://api.github.com/repos/derailed/k9s/releases/latest" | Select-Object -ExpandProperty tag_name).TrimStart('v')
Invoke-WebRequest -Uri "https://github.com/derailed/k9s/releases/download/v$version/k9s_windows_amd64.tar.gz" -OutFile k9s.tar.gz
tar -xf k9s.tar.gz
Move-Item -Path k9s_windows_amd64\k9s.exe -Destination "$env:LOCALAPPDATA\Programs\k9s\k9s.exe"
```

---

### 🐳 Docker (Sin instalar 🐋)

```bash
# Opción 1: Dockerhub/Quay
docker run --rm -it -v $KUBECONFIG:/root/.kube/config quay.io/derailed/k9s

# Opción 2: Docker Desktop Extension
docker extension install spurin/k9s-dd-extension:latest
```

> 📌 **Nota**: Asegúrate de tener `KUBECONFIG` configurado correctamente o mounta tu `~/.kube/config`

---

## 🛠️ Pre requisitos

### ✅ Verificar configuración de Kubernetes

```bash
# Verifica que kubectl funciona antes de usar k9s
kubectl cluster-info
kubectl get nodes
```

### 🖥️ Configurar variables de entorno

```bash
# Terminal con soporte 256 colores (OBLIGATORIO)
export TERM=xterm-256color

# Editor para editar recursos (opcional pero recomendado)
export KUBE_EDITOR=vim  # o nano, code, etc.
export EDITOR=vim

# Añadir a tu ~/.bashrc o ~/.zshrc
echo 'export TERM=xterm-256color' >> ~/.bashrc
echo 'export KUBE_EDITOR=vim' >> ~/.bashrc
```

---

## ⚙️ Configuración Avanzada

### 📁 Estructura de directorios

| 🖥️ Sistema Operativo | 📂 Ubicación por defecto |
|------------------------|---------------------------|
| Linux | `~/.config/k9s` |
| macOS | `~/Library/Application Support/k9s` |
| Windows | `%LOCALAPPDATA%\k9s` |

### 📝 Archivo de configuración principal

```yaml title="~/.config/k9s/config.yaml"
k9s:
  # Refresh automático de recursos (default: false)
  liveViewAutoRefresh: false
  
  # Intervalo de refresh en segundos (min: 2.0)
  refreshRate: 2
  
  # Timeout conexión al API server
  apiServerTimeout: 15s
  
  # Intentos de reconexión
  maxConnRetry: 5
  
  # Modo solo lectura (desactiva edición/eliminación)
  readOnly: false
  
  # Salir con Ctrl+C (default: false)
  noExitOnCtrlC: false
  
  # UI
  ui:
    # Soporte de mouse
    enableMouse: false
    
    # Ocultar header/logo
    headless: false
    logoless: false
    
    # Ocultar breadcrumbs
    crumbsless: false
    
    # Ocultar splash inicial
    splashless: false
    
    # Usar iconos
    noIcons: false
    
    # UI reactiva (watch cambios en disco)
    reactive: false
    
    # Skin por defecto
    skin: dracula
    
    # Vistas a pantalla completa por defecto
    defaultsToFullScreen: false
  
  # Configuración de logs
  logger:
    # Líneas a mostrar
    tail: 200
    
    # Buffer total
    buffer: 500
    
    # Segundos hacia atrás (-1 = tail)
    sinceSeconds: 300
    
    # Wrap de texto
    textWrap: false
    
    # Autoscroll
    disableAutoscroll: false
    
    # Timestamps
    showTime: false
```

### 🎨 Skins/Temas disponibles

```bash
# Listar skins disponibles
ls ~/.config/k9s/skins/  # o la ruta configurada
```

K9s incluye varias skins por defecto. Para usar una:

```yaml
k9s:
  ui:
    skin: nord  # Cambia a tu skin favorita
```

### 🔑 Aliases personalizados

```yaml title="~/.config/k9s/aliases.yaml"
aliases:
  # Formato: alias: gvr (Group/Version/Resource)
  pp: v1/pods
  svc: v1/services
  dp: apps/v1/deployments
  rs: apps/v1/replicasets
  sts: apps/v1/statefulsets
  ds: apps/v1/daemonsets
  cm: v1/configmaps
  sec: v1/secrets
  pvc: v1/persistentvolumeclaims
  ing: networking.k8s.io/v1/ingresses
  crb: rbac.authorization.k8s.io/v1/clusterrolebindings
  cr: rbac.authorization.k8s.io/v1/clusterroles
  rb: rbac.authorization.k8s.io/v1/rolebindings
  # Alias con filtros
  system: pod -n kube-system
  events: events --sort-by='.lastTimestamp'
```

### ⌨️ Hotkeys personalizados

```yaml title="~/.config/k9s/hotkeys.yaml"
hotKeys:
  # Prefix personalizado + número
  shift-0:
    shortCut: Shift-0
    description: Ver pods
    command: pods
  
  shift-1:
    shortCut: Shift-1
    description: Ver deployments
    command: dp
  
  shift-2:
    shortCut: Shift-2
    description: XRay Deployments
    command: xray deploy
  
  shift-s:
    shortCut: Shift-S
    override: true
    description: Ver recursos en namespace actual
    command: "$RESOURCE_NAME $NAMESPACE"
    keepHistory: true
```

### 🔌 Plugins personalizados

```yaml title="~/.config/k9s/plugins.yaml"
plugins:
  # Plugin básico con Ctrl+L para ver logs
  pod-logs:
    shortCut: Ctrl-L
    description: Ver logs del pod
    scopes:
      - pods
    confirm: false
    command: kubectl
    background: false
    args:
      - logs
      - -f
      - $NAME
      - -n
      - $NAMESPACE
  
  # Plugin para describir recurso
  describe-resource:
    shortCut: Ctrl-D
    description: Describir recurso
    scopes:
      - all
    command: kubectl
    args:
      - describe
      - $RESOURCE_NAME
      - -n
      - $NAMESPACE
  
  # Plugin para Port Forward
  port-forward:
    shortCut: Shift-F
    description: Port Forward
    scopes:
      - pods
      - svc
    command: kubectl
    background: true
    args:
      - port-forward
      - $NAME
      - $CONTAINER
```

---

## ⌨️ Atajos de Teclado Esenciales

### 🎯 Navegación General

| 🏷️ Atajo | 📝 Acción |
|----------|------------|
| `?` | Mostrar ayuda |
| `ctrl-a` | Mostrar todos los aliases |
| `:q`, `:quit`, `ctrl-c` | Salir de k9s |
| `esc` | Volver atrás / cancelar |
| `[` | Historial anterior |
| `]` | Historial siguiente |
| `/` | Filtrar recursos |
| `-` | Volver al recurso anterior |

### 🔍 Búsqueda de Recursos

| 🏷️ Atajo | 📝 Acción |
|----------|------------|
| `:pod` | Ver pods (acepta singular/plural/alias) |
| `:pod ns-namespace` | Ver pods en namespace |
| `:pod /texto` | Filtrar pods por texto |
| `:pod app=label,env=dev` | Filtrar por labels |
| `:pod @contexto` | Ver pods en otro contexto |
| `/:!filter` | Filtrar inverso (excluir) |
| `/:-l label-selector` | Filtrar por label |
| `/:-f filtro` | Fuzzy find |

### 📊 Gestión de Recursos

| 🏷️ Atajo | 📝 Acción |
|----------|------------|
| `d` | Describe recurso |
| `v` | Ver recurso (YAML/JSON) |
| `e` | Editar recurso |
| `l` | Ver logs |
| `s` | Shell en contenedor |
| `ctrl-d` | Eliminar recurso (con confirmación) |
| `ctrl-k` | Kill recurso (sin confirmación) |
| `ctrl-f` | Port forward |
| `tab` | Seleccionar en tablas |

### 🔄 Vistas Especiales

| 🏷️ Atajo | 📝 Acción |
|----------|------------|
| `:ctx` | Ver/Switch de contexto |
| `:ns` | Ver/Switch de namespace |
| `:pulses` o `:pu` | Ver métricas en tiempo real |
| `:xray [recurso]` | XRay del recurso |
| `:popeye` o `:pop` | Sanitizer de cluster |
| `:screendump` o `:sd` | Guardar screenshot |

---

## 🔧 Comandos CLI

```bash
# Version
k9s version

# Info (directorios, configuración)
k9s info

# Help
k9s help

# En namespace específico
k9s -n mi-namespace

# En contexto específico
k9s --context mi-contexto

# Modo solo lectura
k9s --readonly

# Logs
k9s info logs
k9s -l debug  # Modo debug
```

---

## 🐛 Solución de Problemas

### ❌ Error: "Unable to locate Kubeconfig"

```bash
# Verificar que existe el archivo
ls -la ~/.kube/config

# Definir manualmente
export KUBECONFIG=/ruta/custom/config
```

### ❌ Error: "Invalid configuration"

```bash
# Validar formato de kubeconfig
kubectl config view
kubectl cluster-info
```

### ❌ Error: "K9s no muestra nada"

```bash
# Verificar versión de kubeconfig
kubectl version --client

# Actualizar k9s
k9s version
```

### ❌ Error: "Colores raros/Pantalla corrupta"

```bash
# Verificar variable TERM
echo $TERM  # Debe ser xterm-256color

# Forzar en .bashrc/.zshrc
export TERM=xterm-256color
```

### 🐞 Debug mode

```bash
# Arrancar en modo debug
k9s -l debug

# Ver logs
k9s info  # Muestra ubicación de logs
tail -f ~/.local/state/k9s/k9s.log
```

---

## 📊 Compatibilidad K8s

| 📦 Versión K9s | 🎯 Versión K8s mínima |
|----------------|----------------------|
| >= v0.27.0 | 1.26.1 |
| v0.26.7 - v0.26.6 | 1.25.3 |
| v0.26.5 - v0.26.4 | 1.25.1 |
| v0.26.3 - v0.26.1 | 1.24.3 |
| v0.26.0 - v0.25.19 | 1.24.2 |
| v0.25.18 - v0.25.3 | 1.22.3 |
| <= v0.24 | 1.21.3 |

---

## 🧹 Desinstalación

```bash
# Linux
sudo rm /usr/local/bin/k9s
rm -rf ~/.config/k9s

# macOS (Homebrew)
brew uninstall k9s

# macOS (Ports)
sudo port uninstall k9s

# Snap
sudo snap remove k9s

# Windows (Scoop)
scoop uninstall k9s

# Windows (Chocolatey)
choco uninstall k9s

# Windows (Winget)
winget uninstall k9s
```

---

## 📚 Recursos

- 📖 [Documentación oficial](https://k9scli.io)
- 🐙 [GitHub](https://github.com/derailed/k9s)
- 💬 [Slack](https://k9sers.slack.com/)

---

> 🔔 **Tip**: Usa `:q` o `:quit` para salir de k9s de forma limpia.Ctrl+C pregunta si quieres salir depending de la configuración

¡Happy K8s-ing! 🎉