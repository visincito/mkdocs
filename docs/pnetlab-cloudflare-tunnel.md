---
tags:
  - PNETLab
  - Cloudflare
  - Tunnel
  - Homelab
  - Networking
---
# 🌐 Accede a tu PNETLab desde cualquier lugar con Cloudflare Tunnel

¿Tienes PNETLab corriendo en tu homelab y quieres acceder desde fuera sin abrir puertos ni tener IP pública? **Cloudflare Tunnel** es la solución perfecta. En este artículo te explico paso a paso cómo configurarlo. 🚀

---

## 🤔 ¿Por qué Cloudflare Tunnel?

PNETLab es una plataforma increíble para emular redes, pero normalmente solo es accesible desde dentro de tu red local. Abrir puertos en el router no es seguro, y si tu operador usa **CGNAT**, ni siquiera es posible.

Cloudflare Tunnel crea un túnel **outbound-only** desde tu servidor PNETLab hasta la red perimetral de Cloudflare. Esto significa:

- 🔒 **Sin puertos abiertos** → más seguridad
- 🌍 **Acceso desde cualquier lugar**
- ✅ **SSL/TLS automático** con certificados de Cloudflare
- 🆓 **Gratuito** en el plan Free de Cloudflare

---

## 📋 Requisitos

Antes de empezar necesitas:

- ☁️ Una cuenta en [Cloudflare](https://dash.cloudflare.com/) (plan Free vale)
- 🌐 Un dominio gestionado por Cloudflare
- 🖥️ Tu servidor PNETLab instalado y funcionando (puerto `80` o `443` por defecto)
- 🐧 Acceso SSH a tu máquina PNETLab (Debian/Ubuntu)

---

## ⚙️ Instalación de cloudflared

Lo primero es instalar el cliente `cloudflared` en tu servidor PNETLab:

```bash
# Añadir el repositorio de Cloudflare
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloudflare-main.gpg

echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflared.list

# Instalar
sudo apt update && sudo apt install cloudflared
```

Verifica que se instaló correctamente:

```bash
cloudflared --version
```

---

## 🔐 Autenticación con Cloudflare

Ejecuta el siguiente comando para vincular tu servidor con tu cuenta de Cloudflare:

```bash
cloudflared tunnel login
```

Se abrirá un enlace en la terminal. Cópialo en tu navegador, inicia sesión en Cloudflare y selecciona el dominio que quieras usar. Esto generará un certificado en `~/.cloudflared/cert.pem`. ✅

---

## 🚇 Crear el túnel

Ahora creamos el túnel con un nombre descriptivo:

```bash
cloudflared tunnel create pnetlab
```

Este comando:
- 🆔 Asigna un **UUID único** al túnel
- 📄 Genera un archivo de credenciales `~/.cloudflared/<UUID>.json`
- 🌍 Crea un subdominio en `.cfargotunnel.com`

Puedes ver tus túneles activos con:

```bash
cloudflared tunnel list
```

---

## 📝 Configurar el túnel

Crea el archivo de configuración `~/.cloudflared/config.yml`:

```yaml
tunnel: <TU-UUID-DEL-TUNEL>
credentials-file: /root/.cloudflared/<TU-UUID>.json

ingress:
  - hostname: pnetlab.tudominio.com
    service: http://localhost:80
  - service: http_status:404
```

> ⚠️ Reemplaza `<TU-UUID-DEL-TUNEL>` por el UUID que te generó al crear el túnel.

El `ingress` define qué hacer con el tráfico que llega:
- Las peticiones a `pnetlab.tudominio.com` se redirigen al puerto `80` de tu PNETLab
- Cualquier otro hostname devuelve un `404`

---

## 🧭 Enrutar el DNS

Crea un registro CNAME en Cloudflare para enlazar tu subdominio con el túnel:

```bash
cloudflared tunnel route dns pnetlab pnetlab.tudominio.com
```

Esto crea automáticamente un registro `CNAME` apuntando a `<UUID>.cfargotunnel.com`. 🌐

---

## ▶️ Arrancar el túnel

Inicia el túnel para probar que funciona:

```bash
cloudflared tunnel run pnetlab
```

Si todo va bien, verás algo como:

```
Registered tunnel connection
```

Ahora abre en tu navegador `https://pnetlab.tudominio.com` y deberías ver tu PNETLab. 🎉

---

## 🔄 Instalar como servicio

Para que el túnel arranque automáticamente al iniciar el servidor, instálalo como servicio systemd:

```bash
sudo cloudflared --config /root/.cloudflared/config.yml service install
```

Verifica que está corriendo:

```bash
systemctl status cloudflared
```

Habilítalo para el arranque automático:

```bash
systemctl enable cloudflared
```

---

## 🧪 Solución de problemas

### ❌ Error 1033 (Argo Tunnel error)

El túnel no está corriendo o hay un problema de conectividad:

```bash
journalctl -u cloudflared -f
```

### ❌ Página en blanco al hacer login en PNETLab

Es un problema conocido con PNETLab y Cloudflare Tunnel 😅. La solución es modificar el archivo `app.js` de PNETLab para forzar el protocolo WebSocket:

```bash
# Busca y edita el archivo app.js
sudo sed -i 's/window.location.protocol === "https:" ? "wss:" : "ws:"/"wss:"/g' /var/www/html/app.js
```

Luego reinicia el servicio web de PNETLab.

### ❌ Error 522 (Connection timeout)

El túnel no llega al servicio. Revisa que PNETLab esté escuchando en `localhost:80`:

```bash
ss -tlnp | grep 80
```

---

## 🏁 Conclusión

Con Cloudflare Tunnel has conseguido:

- ✅ Acceder a tu PNETLab desde cualquier parte del mundo
- 🔒 Sin exponer ningún puerto en tu router
- 🆓 Totalmente gratuito
- 🛡️ Con la protección DDoS de Cloudflare incluida

Ahora ya puedes llevar tus laboratorios a todas partes sin complicaciones. ¡A laburar se ha dicho! 💪😄

---

*¿Te ha sido útil? Compártelo con tus colegas homelabbers o déjame un comentario abajo 👇*
