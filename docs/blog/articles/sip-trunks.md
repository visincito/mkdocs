---
draft: false
date: 2024-04-02T21:00:00
authors: [Visin]
categories:
  - VoIP
  - Asterisk
  - Blog
tags:
  - SIP
  - VoIP
---
# PJSIP Trunk

Ejemplos de SIP Trunk con channel pjsip asterisk

<!-- more -->

## User Agent PJSIP

Bajo el contexto "[Global]" se puede definir el user_agent que mostrar en los paquetes  SIP.

O bien, podemos ponerlo en un archivo separado , llamado este con un `#include pjsip_personal.conf` y usar `[Global](+)` para que se añada al final de lo definido en Global.

``` javascript
...
[Global](+)
user_agent=SBC XXXX/v1.0
...
```

## OPTIONS PJSIP

PJSIP enviará el nombre del endpoint como contact para el OPTION
El que recibe ese OPTION tiene que tener en el contexto del SIP Trunk ese contact.
Por ejemplo:

``` javascript
OPTIONS sip:LABSIP@10.12.5.42:5060 SIP/2.0
Via: SIP/2.0/UDP 10.12.5.41:5060;rport;branch=z9hG4bKPjc7b3438d-9987-4d5f-8543-bb506667dd20
From: <sip:LABSIP@10.12.5.41>;tag=99614688-0a48-48df-9b3b-62172ac4da98
To: <sip:LABSIP@10.12.5.42>
Contact: <sip:LABSIP@10.12.5.41:5060>
Call-ID: 730b281f-04f1-4600-8e9e-77ef1910e0de
CSeq: 33191 OPTIONS
Max-Forwards: 70
User-Agent: SBC SCCOM/v1.0
Content-Length:  0
```

En este caso , usarmos el contexto "from-trunk" para este SIP Trunk, entonces definimos en nuestro "extensions":

``` javascript
[from-trunk]


exten => LABSIP,1,NoOp()
```

De esta forma ya responderemos un 200 OK

``` javascript
SIP/2.0 200 OK
Via: SIP/2.0/UDP 10.12.5.42:5060;rport=5060;received=10.12.5.42;branch=z9hG4bKPje9af206d-956f-4e6e-b2fd-548fa8080b16
Call-ID: d08befbb-0595-4506-a906-be08ec52c8c1
From: <sip:LABSIP@10.12.5.42>;tag=1fd91fc3-1c6d-481f-ba8a-fbe7b2f1526c
To: <sip:LABSIP@10.12.5.41>;tag=z9hG4bKPje9af206d-956f-4e6e-b2fd-548fa8080b16
CSeq: 61932 OPTIONS
Accept: application/sdp, application/simple-message-summary, application/xpidf+xml, application/cpim-pidf+xml, application/dialog-info+xml, application/pidf+xml, application/pidf+xml, application/dialog-info+xml, application/simple-message-summary, message/sipfrag;version=2.0
Allow: OPTIONS, REGISTER, SUBSCRIBE, NOTIFY, PUBLISH, INVITE, ACK, BYE, CANCEL, UPDATE, PRACK, MESSAGE, REFER
Supported: 100rel, timer, replaces, norefersub
Accept-Encoding: identity
Accept-Language: en
Server: SBC SCCOM/v1.0
Content-Length:  0
```

## Sin auth, solo por IP

``` javascript title="Archivo pjsip.conf"
[SBC1]
type=aor
qualify_frequency=30
contact=sip:10.9.94.251:5060

[fijos]
type=endpoint
transport=0.0.0.0-udp
context=from-trunk
allow=!all,g729
aors=SBC1

[SBC1]
type=identify
endpoint=fijos
match=10.9.94.251
```

## Con Auth, usuario y clave

``` javascript title="Archivo pjsip.conf"
;=====================Visin=====
; Configuracion para casa Lalin
; AOR
[eXXXXXXXXXX]
type=aor
qualify_frequency=120
contact=sip:eXXXXXXXXXX@ims.operador.com:5060
outbound_proxy=sip:evimsemad.operador.com\;lr

; AUTH
[eXXXXXXXXXX]
type=auth
auth_type=userpass
password=******enc******
username=eXXXXXXXXXX

; ENDPOINT
[eXXXXXXXXXX]
type=endpoint
transport=0.0.0.0-udp
context=from-trunk
disallow=all
allow = g729,alaw,gsm
aors=eXXXXXXXXXX
send_connected_line=false
language=es
outbound_proxy=sip:evimsemad.operador.com\;lr
outbound_auth=eXXXXXXXXXX
from_domain=ims.operador.com
from_user=+34988222222
contact_user=+34988222222
user_eq_phone=yes
t38_udptl=no
t38_udptl_ec=none
fax_detect=no
trust_id_inbound=no
t38_udptl_nat=no
identify_by=auth_username
direct_media=no
rewrite_contact=yes
rtp_symmetric=yes
dtmf_mode=auto

; IDENTIFY
[eXXXXXXXXXX]
type=identify
endpoint=eXXXXXXXXXX
match=evimsemad.operador.com

; REGISTRATION
[eXXXXXXXXXX]
type=registration
transport=0.0.0.0-udp
outbound_auth=eXXXXXXXXXX
retry_interval=60
fatal_retry_interval=30
forbidden_retry_interval=30
max_retries=10000
expiration=3600
auth_rejection_permanent=no
line=no
contact_user=+34988222222
server_uri=sip:ims.operador.com
client_uri=sip:+34988222222@ims.operador.com
outbound_proxy=sip:evimsemad.operador.com\;lr
```

``` javascript
;========= OPENSIPS =====
; Configuracion para OPENSIPS
; AOR
[visincito]
type=aor
qualify_frequency=120
contact=sip:visincito@opensips.org
outbound_proxy=sip:opensips.org\;lr

; AUTH
[visincito]
type=auth
auth_type=userpass
password=******enc******
username=visincito

; ENDPOINT
[visincito]
type=endpoint
transport=0.0.0.0-udp
context=from-trunk
disallow=all
allow = g729,alaw,gsm
aors=visincito
send_connected_line=false
language=es
;outbound_proxy=sip:evimsemad.operador.com\;lr
;outbound_auth=eXXXXXXXXXX
;from_domain=ims.operador.com
;from_user=+34988222222
;contact_user=+34988222222
user_eq_phone=yes
t38_udptl=no
t38_udptl_ec=none
fax_detect=no
trust_id_inbound=no
t38_udptl_nat=no
identify_by=auth_username
direct_media=no
rewrite_contact=yes
rtp_symmetric=yes
dtmf_mode=auto

; IDENTIFY
[visincito]
type=identify
endpoint=visincito
match=opensips.org

; REGISTRATION
[visincito]
type=registration
transport=0.0.0.0-udp
;outbound_auth=eXXXXXXXXXX
retry_interval=60
fatal_retry_interval=30
forbidden_retry_interval=30
max_retries=10000
expiration=3600
auth_rejection_permanent=no
line=no
;contact_user=+34988222222
;server_uri=sip:ims.operador.com
;client_uri=sip:+34988222222@ims.operador.com
;outbound_proxy=sip:evimsemad.operador.com\;lr
```

``` javascript
;========= sip2sip.info =====
; Configuracion para OPENSIPS
; AOR
[visincito]
type=aor
qualify_frequency=120
contact=sip:visincito@sip2sip.info
outbound_proxy=proxy.sipthor.net\;lr

; AUTH
[visincito]
type=auth
auth_type=userpass
password=******enc******
username=visincito

; ENDPOINT
[visincito]
type=endpoint
transport=0.0.0.0-udp
context=from-trunk
disallow=all
allow = g729,alaw,gsm
aors=visincito
send_connected_line=false
language=es
identify_by=auth_username
direct_media=yes
rewrite_contact=yes
rtp_symmetric=yes
dtmf_mode=auto

; IDENTIFY
[visincito]
type=identify
endpoint=visincito
match=sip2sip.info
match=proxy.sipthor.net

; REGISTRATION
[visincito]
type=registration
transport=0.0.0.0-udp
retry_interval=60
fatal_retry_interval=30
forbidden_retry_interval=30
max_retries=10000
expiration=3600
auth_rejection_permanent=no
line=no
```
