---
tags:
  - Gmail
  - Cloudflare
  - Email
  - SMTP
  - 2FA
  - Google
---

# 📧 Enviar correo desde Gmail con tu dominio Cloudflare

Guía para configurar Gmail y poder enviar correos usando una dirección de tu propio dominio gestionado con Cloudflare.

---

## 🔑 Paso 1: Activar verificación en dos pasos (2FA)

Para que este método funcione, necesitas tener la [verificación en dos pasos](https://www.google.com/landing/2step/) habilitada en tu cuenta de Google. Si aún no la tienes:

→ [Activar 2FA en tu cuenta de Google](https://myaccount.google.com/signinoptions/two-step-verification)

---

## 🔐 Paso 2: Crear una contraseña de aplicación para Mail

Con el 2FA activado, genera una contraseña de aplicación específica para Mail:

→ [Crear contraseña de aplicación](https://security.google.com/settings/security/apppasswords)

La usarás más adelante junto con tu dirección de Gmail en la configuración SMTP.

---

## 📥 Paso 3: Añadir tu dirección Cloudflare en Gmail

En Gmail ve a **Configuración → Cuentas → Enviar correo como**. Haz clic en **Añadir otra dirección de correo** y rellena el formulario:

- **Nombre**: el que quieras que aparezca como remitente
- **Dirección de correo**: tu dirección con dominio Cloudflare
- ❌ **Desmarca** "Tratar como un alias"
- Haz clic en **Siguiente paso >>**

---

## ⚙️ Paso 4: Rellenar el formulario SMTP

| Campo | Valor |
|-------|-------|
| **Servidor SMTP** | `smtp.gmail.com` |
| **Puerto** | `587` |
| **Usuario** | Tu Gmail completo (`usuario@gmail.com`) |
| **Contraseña** | La contraseña de aplicación del Paso 2 |
| **Cifrado (TLS)** | Activado ✅ |

Haz clic en **Añadir cuenta >>**

---

## ✅ Paso 5: Verificar la propiedad

Recibirás un email de Gmail en tu dirección Cloudflare con un código de confirmación. Introduce el código en el diálogo (o haz clic en el enlace del correo) y ¡listo! 🎉

Ahora puedes seleccionar tu nueva dirección en el campo **De:** al redactar un mensaje. Cuando respondas a un correo recibido en esa dirección, se seleccionará automáticamente.

---

## 📝 Resumen

| Item | Detalle |
|------|---------|
| 🔑 **2FA** | Obligatorio en la cuenta de Google |
| 🔐 **App Password** | Se genera en [security.google.com](https://security.google.com/settings/security/apppasswords) |
| 📡 **SMTP Server** | `smtp.gmail.com:587` con TLS |
| 🚫 **Alias** | No tratar como alias |

---

[:octicons-arrow-left-24: Volver a inicio](../index.md)
