---
layout: page
title: Roles
permalink: /es/roles/
lang: es
ref: roles
nav_order: 3
---

{% include langnav.html %}

| Rol | Siempre en línea | Usa | Hace | Si hace trampa |
|---|---|---|---|---|
| **usuario (user)** | no | navegador | hace pedidos, deposita, confirma la recepción, abre disputas | ― |
| **comprador por encargo (proxy shopper)** | **sí** | nodo Go + shopper-bot | presupuesta, compra en nombre del usuario, informa del envío; cobra una comisión | pierde su parte en el fallo del custodio; el operador lo quita de la lista |
| **custodio (escrow)** | no | navegador o nodo Go | resuelve las disputas antes de T1 | el operador lo quita de la lista; según sus condiciones, se le confisca la fianza y se usa como compensación |
| **operador (operator)** | no | navegador, nodo Go o `psctl` | firma, por región, la lista de combinaciones fiables de comprador × custodio; recibe denuncias | el coordinador revoca su delegación |
| **coordinador (coordinator)** | no | navegador o `psctl` | firma delegaciones a operadores | los usuarios lo quitan de su configuración (destitución) |

## Usuario

1. Abre la aplicación web, crea una frase semilla (12 palabras), anótala y elige una frase de contraseña que cifre la clave.
2. Introduce la URL de la tienda, la región de la tienda y los artículos.
3. Elige una de las combinaciones de comprador × custodio que se ofrecen y haz el pedido.
4. Llega un presupuesto. Si su tipo de cambio difiere de tus propias fuentes en más de un 3 %, verás un aviso; por encima del 10 %, una advertencia seria.
5. Acepta y deposita. El dinero se reparte entre la multifirma y la comisión por adelantado del custodio.
6. Cuando llegue el artículo, pulsa "recibido" (tras una pantalla que muestra el importe y el destinatario, tu firma parcial se envía al comprador).
7. Si no llega, abre una disputa. La aplicación reúne las pruebas por ti.

## Comprador por encargo

- Ejecuta el nodo Go con `role: shopper` y conecta shopper-bot (la herramienta de automatización del navegador).
- Configura los medios de pago, las monedas, las regiones donde puedes pagar en efectivo, tu comisión y tu política de riesgo de tiendas.
- Los datos de las tarjetas viven solo dentro de shopper-bot; nunca se entregan al nodo ni a la red.
- Si el usuario nunca confirma la recepción y pasa T1, puedes quedarte con los fondos por tu cuenta.

## Custodio

- Solo resuelve disputas de pedidos que pagaron su comisión por adelantado.
- Normalmente no puede leer la dirección de entrega. En una disputa, el comprador (o el usuario) le entrega la clave de descifrado.
- Un fallo llega a ambas partes como una transacción firmada; se vuelve definitivo cuando una de ellas lo refrenda.

## Operador

- Firma sus regiones (una o varias), la lista de combinaciones y los relays y endpoints de cadena que recomienda.
- Ante una denuncia, publica una nueva versión de la lista sin el infractor. La fianza se rige por las condiciones que acordó con el custodio.

## Coordinador

- Emite delegaciones a operadores. Revocar significa publicar una nueva versión con `revoked`.

## Aparecer en las listas (el registro)

El registro de confianza del mantenedor es el repositorio de GitHub
[pad01g/proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry). **Una pull request fusionada es la
aprobación:** tras cada fusión, la CI firma las nuevas delegaciones y listas con las claves del registro y las publica
(en los relays públicos de Nostr y en https://pad01g.github.io/proxy-shopping-registry/events.json).

| Quieres ser | Añade en una pull request | Qué hace la fusión |
|---|---|---|
| comprador por encargo | `shoppers/<name>.json` (pk, contacto, descripción, regiones de efectivo, medios de pago, los custodios con los que trabajas) | el operador del registro incluye tus combinaciones de comprador × custodio en su lista |
| custodio | `escrows/<name>.json` (pk, contacto, descripción, días de SLA) | los compradores pueden nombrarte; aparecen tus combinaciones |
| operador | `operators/<name>.json` (pk, contacto, descripción, regiones) | el coordinador del registro te delega; a partir de ahí firmas tus propias listas |
| coordinador | `coordinators/<name>.json` (pk, contacto, descripción) | apareces en el directorio de coordinadores que las aplicaciones ofrecen a los usuarios (cada usuario sigue eligiendo en quién confiar) |

`pk` es tu clave pública de Nostr (64 caracteres hexadecimales): la aplicación web la muestra en Configuración, y `psctl keys --mnemonic-file …`
la imprime. Quitar a alguien es una pull request que mueve su archivo a `revoked/` con un motivo. El README del
registro tiene los formatos exactos de los archivos y los comandos.
