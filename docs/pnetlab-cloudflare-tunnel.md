---
tags:
  - PNETLab
  - Cloudflare
  - Tunnel
  - Fix
  - Troubleshooting
---
# 🛠️ PNETLab + Cloudflare Tunnel: Solucionar pantalla blanca al hacer login

Si estás exponiendo tu PNETLab con Cloudflare Tunnel, igual te ha pasado esto: la página de login carga perfecta, pero al introducir credenciales... **pantalla en blanco** 😱. El dashboard nunca llega a cargarse.

En este tip te cuento cómo arreglarlo tocando **una sola línea** en el servidor. 🔧

---

## 🤔 El problema

PNETLab (basado en UNETLAB) tiene un check en JavaScript que espera que la respuesta HTTP contenga el texto `"OK"` (`statusText == 'OK'`). Cuando el tráfico pasa por Cloudflare (sobre todo vía HTTP/2), ese `statusText` puede venir **vacío** o con otro valor, y la validación falla en silencio, dejándote con la pantalla en blanco.

---

## ✅ La solución

Eliminar la comprobación del `statusText` y validar solo el código de estado HTTP (`200`).

### 1️⃣ Accede por SSH a tu PNETLab

```bash
ssh usuario@<ip-de-tu-pnetlab>
```

### 2️⃣ Edita el archivo `app.js`

La ruta exacta es:

```bash
nano /opt/unetlab/html/store/public/main/js/angularjs/app.js
```

### 3️⃣ Localiza y cambia la línea problemática

Está aproximadamente en la **línea 93**. El código original es así:

```javascript
if (response.status == '200' && response.statusText == 'OK')
```

Cámbialo por:

```javascript
if (response.status == '200')
```

Guarda (`Ctrl+O`, `Enter`) y sal (`Ctrl+X`). ✅

### 4️⃣ Limpia cachés y reinicia

Para que los cambios surtan efecto:

- 🗑️ **Purga la caché de Cloudflare** → Dashboard > Caching > Configuration > **Purge Everything**
- 🧹 **Limpia la caché del navegador** (el `.js` viejo puede estar cacheado)
- 🔄 (Opcional) **Reinicia el servidor**:

```bash
reboot
---

## 🛡️ Seguridad extra con Cloudflare Zero Trust

⚠️ **Importante:** PNETLab no está diseñado para exponerse directamente a Internet. Tiene vulnerabilidades conocidas y corre con privilegios altos.

Te recomiendo encarecidamente añadir una capa de seguridad con **Cloudflare Zero Trust (Access)**:

1. Ve a **Zero Trust > Access > Applications**
2. Crea una **Self-hosted Application**
3. Configura un **Identity Provider** (Google, GitHub, One-time PIN, etc.)
4. Así los usuarios autentican **antes** de llegar siquiera al login de PNETLab

De esta forma solo usuarios autorizados pueden acceder, aunque PNETLab tenga vulnerabilidades sin parchear. 🔒

---

## 📝 Resumen técnico

| Item | Detalle |
|------|---------|
| 📂 **Archivo** | `/opt/unetlab/html/store/public/main/js/angularjs/app.js` |
| 🐛 **Causa** | `statusText == 'OK'` falla en conexiones vía Cloudflare Tunnel |
| 🩹 **Fix** | Eliminar la validación de texto, dejar solo `response.status == '200'` |
