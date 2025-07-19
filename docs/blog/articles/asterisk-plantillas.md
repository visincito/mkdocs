---
draft: false
date: 2024-04-02T22:00:00
authors: [Visin]
categories:
  - VoIP
  - Asterisk
  - Blog
tags:
  - SIP
  - VoIP
---
# Plantillas en Asterisk

Una forma muy cómoda de configurar asterisk es usando las plantillas. Como en cualquier sistema, hay muchisimos valores de configuración comunes por esto el uso de plantillas puede ayudar mucho.

<!-- more -->

## Añadir parámetros adicionales a un contexto

Aunque esto no es una plantilla en si, se basa en el mismo principio de las plantillas. Podemos añadir a una configuración ya definida, parámetros adicionales.
Además, definir de nuevo otro parámetro con distinto valor sobre-escribe este valor. Esto también es muy util e interesante.

Por ejemplo, yo soy de los que le gusta disponer de los archivos originales en el directorio /etc/asterisk comentando todo lo que no voy a usar, y hacer un `include` de un nuevo archivo para mis configuraciones.

``` javascript title="Archivo pjsip.conf"
...

#include pjsip_personal.conf
```

Añadiendo la línea anterior al final del archivo `pjsip.conf` podemos crear un nuevo archivo para guardar nuestra configuración personal.

```javascript title="Archivo pjsip_personal.conf"
[global](+)
user_agent=Mi PBX

...
```

Añadiendo `(+)` a un contexto que ya exista definido, podemos añadir o cambiar alguna configuración. En el caso anterior, defino el `user_agent` que quiero que se muestre en los mensajes SIP

## Plantillas

Las plantillas se definen añadiendo `(!)` al nombre de la plantilla. Y para usarla se añade el nombre a lo que estemos configurando.
Un ejemplo es:

``` javascript title="Archivo pjsip_personal.conf"
...
[aor-personal](!)
type=aor
qualify_frequency=60


...

[1234](aor-personal)
contact=sip:1234@10.10.10.10:5060

```

El ejemplo anterior sería lo mismo que hacer:

``` javascript title="Archivo pjsip_personal.conf"

[1234]
type=aor
qualify_frequency=60
contact=sip:1234@10.10.10.10:5060

```

Si todas las extensiones que estés configurando en la definición de `aor` van a llevar el parámetro `qualify_frequency=60` usamos esa plantilla en todas las extensiones para no tener que repetir esos parámetros en todas las extensiones.
