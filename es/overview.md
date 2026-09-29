---
layout: page
title: Cómo funciona
permalink: /es/overview/
lang: es
ref: overview
nav_order: 2
---

{% include langnav.html %}

## Qué hace

Cuando la tienda en la que quieres comprar solo acepta efectivo o un medio de pago concreto, pagas con criptomonedas
(por ahora **BTC signet** y **USDC**) y un **comprador por encargo (proxy shopper)** compra el artículo por ti y te lo envía.

El dinero de cada pedido va a una **multifirma 2 de 3** (usuario (user), comprador por encargo y custodio (escrow)).

- Cuando llega el artículo, el usuario y el comprador firman para pagarle al comprador.
- Si hay una disputa, el custodio decide el reparto junto con uno de los otros dos.
- Aunque alguien deje de responder, los **bloqueos temporales (timelocks)** garantizan que el dinero acabe en manos de una de las partes.

## Arquitectura

```
Navegador (proxy-shopping-web)              Nodos Go (proxy-shopping-go)
  claves y firmas en el navegador       shopper / escrow / operator / relay
        │ WSS (solo salientes)                  │ WSS           │ libp2p
        ▼                                        ▼               ▼
   relays de Nostr (elegidos por los operators) ◀──▶  red de nodos Go
   = punto de entrada siempre activo + buzón    (gossipsub; atraviesa NAT con circuit relay v2 y DCUtR)
```

- **Los usuarios no ejecutan nada.** Basta con abrir la aplicación web pública para unirse.
  Las claves se crean en el navegador y nunca salen de él; toda autorización (firma) ocurre en el navegador.
- Los usuarios detrás de NAT también pueden conectarse, porque la conexión con un relay de Nostr es saliente.
- Los mensajes para quienes no están siempre en línea (usuarios, custodios, operadores (operators), coordinadores (coordinators))
  esperan, todavía cifrados, en relays de Nostr que hacen de buzón. Cada mensaje se envía a varios relays.
- Solo el comprador por encargo necesita estar siempre en línea. Ejecuta un nodo Go y una herramienta de automatización del navegador (shopper-bot) que opera las tiendas.

## Cómo fluye la confianza

```
coordinator (el usuario confía en su clave pública)
  └─ delegación: "este operator puede publicar listas"
       └─ lista: "en esta región, este shopper y este escrow son de confianza juntos"
```

- El usuario confía **solo en la clave pública del coordinador**. A partir de ahí la confianza se extiende a los operadores y a las combinaciones de comprador × custodio.
- Las listas tienen versiones y nunca caducan. Para quitar a alguien se publica una versión nueva.
- Un usuario "destituye" a un coordinador quitándolo de su configuración.

## Comisiones

| Para quién | ¿Se puede exigir? | Cómo |
|---|---|---|
| comprador por encargo | sí | incluida en el presupuesto |
| custodio (por adelantado) | sí | se paga directamente al custodio junto con el depósito. **Un custodio no tiene obligación de arbitrar pedidos que no pagaron la comisión por adelantado** |
| custodio (disputa) | sí | se descuenta del reparto del fallo |
| operador / coordinador | no | se acuerda fuera del protocolo, por ejemplo, tarifas por aparecer en la lista. Lo que vende una lista es "que te encuentren" |

La fianza del custodio y su confiscación no forman parte del protocolo.
Se tratan como condiciones que el operador y el custodio acuerdan entre ellos; se ofrece un contrato de referencia.

## Tiendas que solo aceptan efectivo

La ubicación de una tienda es un código de región (p. ej., `JP-13-13104` = Shinjuku, Tokio).
Un comprador por encargo declara las regiones a las que puede ir y pagar en efectivo, y solo acepta pedidos de tiendas de esas regiones.

## Riesgo de las tiendas

Antes de aceptar un pedido, el comprador puntúa la tienda:
si está en su lista de permitidas, si usa HTTPS y si su pago pasa por una pasarela de pago conocida.
Enviar los datos de una tarjeta a un sitio cualquiera es arriesgado, así que se rechazan los pedidos de tiendas con puntuación baja.
