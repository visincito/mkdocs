---
tags: 
  - k8s
---
# Errores al usar kubectl top y métricas en Kubernetes causados por certificados expirados

Uno de los problemas más confusos al trabajar con métricas en Kubernetes
aparece cuando comandos comunes como `kubectl top pod` dejan de
funcionar repentinamente. O cuando tenemos configurado HPA no recoge el valor de CPU y Memoria.
En este artículo explico un caso real usando
MicroK8s, cómo identificar el problema y cómo solucionarlo renovando
certificados expirados.

------------------------------------------------------------------------

## Síntomas

El primer indicio del problema aparece al ejecutar comandos relacionados
con métricas:

``` bash
kubectl top pod
```

Error:

``` text
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get pods.metrics.k8s.io)
```

También pueden aparecer errores como:

``` text
couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
```

Esto indica que el servidor de métricas (`metrics-server`) no está
respondiendo correctamente.

------------------------------------------------------------------------

## Diagnóstico

El siguiente paso es revisar los logs del metrics-server:

``` bash
kubectl logs deployment/metrics-server -n kube-system
```

En este caso, el error clave era:

``` text
Unable to authenticate the request err="[x509: certificate has expired or is not yet valid: current time 2026-03-02T12:58:28Z is after 2026-02-14T09:34:40Z]"
```

Este mensaje confirma claramente que un certificado ha expirado.

------------------------------------------------------------------------

## Verificación del estado de los certificados

Al tratarse de MicroK8s, podemos conectarnos al nodo control-plane y
ejecutar:

``` bash
sudo microk8s refresh-certs -c
```

Salida:

``` text
The CA certificate will expire in 3283 days.
The server certificate will expire in 364 days.
The front proxy client certificate will expire in -2 days.
```

Aquí vemos el problema:

``` text
The front proxy client certificate will expire in -2 days.
```

El certificado `front-proxy-client` está expirado.

------------------------------------------------------------------------

## Solución: Renovar el certificado expirado

Renovamos únicamente el certificado afectado, en este caso el front proxy client:

``` bash
sudo microk8s refresh-certs -e front-proxy-client.crt
```

------------------------------------------------------------------------

## Reiniciar el metrics-server

En MicroK8s, es recomendable reiniciar el addon para asegurar que
utiliza el nuevo certificado:

``` bash
sudo microk8s disable metrics-server
sudo microk8s enable metrics-server
```
## Notas

> :mega: Repetir la revisión y la renovación en todos los nodos del cluster.