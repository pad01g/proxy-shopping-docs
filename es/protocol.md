---
layout: page
title: Protocolo
permalink: /es/protocol/
lang: es
ref: protocol
nav_order: 5
---

{% include langnav.html %}

La especificación completa es `docs/spec.md` en proxy-shopping-go. Esta página resume lo esencial.

## Claves

Todas las claves se derivan de una única frase semilla (BIP39), una por propósito (todas secp256k1).

| Propósito | Ruta |
|---|---|
| identidad (Nostr) | `m/44'/1237'/0'/0/0` (NIP-06) |
| libp2p | `m/7333'/0'/0'` |
| clave BTC del pedido (usuario, comprador por encargo) | `m/7333'/1'/{idx}'` (idx sale de un hash del ID del pedido) |
| clave BTC del pedido (custodio) | `m/7333'/2'/{idx}`. La xpub es pública, así que otros pueden derivarla mientras el custodio está desconectado |
| cartera BTC | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## Confianza

| kind | Firmado por | Contenido |
|---|---|---|
| 30500 | coordinador (coordinator) | delegación a un operador; se revoca con `revoked` |
| 30501 | operador (operator) | lista de región × comprador × custodio, relays y endpoints de cadena recomendados |
| 30502 / 30503 | comprador por encargo (shopper) / custodio (escrow) | perfil (comisiones, regiones de efectivo, xpub, direcciones de cobro) |
| 10050 | todos | los relays donde uno recibe mensajes |

La versión de la etiqueta `v` decide cuál es más reciente (normalmente la hora UNIX de la firma). Nada caduca.
Los eventos viajan tanto por relays de Nostr como por libp2p gossipsub; los nodos Go reenvían a uno lo que reciben del otro.

## Mensajes

Los mensajes se envuelven con NIP-59 (gift wrap → seal → contenido).
El contenido es un evento **firmado** (kind 5400), para que un custodio pueda verificarlo como tercero en una disputa.

- Un mensaje se envía a al menos k (por defecto 2) de los relays de entrada del destinatario.
- El destinatario responde con un `ack`; el remitente reenvía hasta recibirlo.
- Un mensaje ocupa como máximo 28000 bytes. Las pruebas grandes, como capturas de pantalla, se referencian solo por su hash;
  en una disputa se envían al custodio por partes como mensajes `attachment`.

```
order.request + order.escrow_key → order.quote → order.accept → (funding) → order.funded + escrow.notice
→ order.purchased → order.shipping → order.release → order.completed
cancel and refund: order.cancel (before funding), order.refund (cooperative refund from the shopper)
other: ack (receipt), chat
dispute: dispute.open → dispute.evidence_request → dispute.evidence (+ attachment) → dispute.ruling → dispute.countersigned
report: report (to the operator)
```

## BTC (P2WSH)

```
OP_IF
  OP_2 <user> <shopper> <escrow> OP_3 OP_CHECKMULTISIG
OP_ELSE
  OP_IF   <T1> OP_CHECKLOCKTIMEVERIFY OP_DROP <shopper> OP_CHECKSIG
  OP_ELSE <T2> OP_CHECKLOCKTIMEVERIFY OP_DROP <user> OP_CHECKSIG
  OP_ENDIF
OP_ENDIF
```

T1 < T2 (alturas de bloque). Si el usuario nunca confirma la recepción y pasa T1, el comprador puede quedarse con los fondos por su cuenta.
Si el comprador desaparece, el usuario puede recuperarlos por su cuenta después de T2.
El custodio debe resolver una disputa antes de T1.

## USDC (Safe v1.4.1)

- Cada pedido tiene un Safe con tres propietarios y un umbral de dos.
- Su dirección sale de CREATE2, así que se puede calcular de antemano a partir del ID del pedido.
- Los bloqueos temporales se implementan como un módulo de Safe (`PSEscrowModule`).
  - Después de t1, el comprador puede quedarse con los fondos con `claimByShopper`.
  - Después de t2, el usuario puede recuperarlos con `refundToUser`.
- Los pagos y los fallos son SafeTx firmadas con EIP-712. El reparto de un fallo usa `MultiSendCallOnly`.

## Vincular la identidad con las claves

Cada solicitud de pedido (`order.request`) lleva un `key_proof`:
una firma de la clave de cadena que entra en la multifirma (la clave BTC del pedido o la cuenta EVM),
que demuestra que pertenece a la misma persona que la identidad de Nostr.
Sin ella, otra identidad podría copiar las claves públicas del usuario en su propia solicitud y afirmar ante el custodio que es el usuario.

## Dirección de entrega

La dirección se cifra con una clave de un solo uso K (XChaCha20-Poly1305).

- K se envuelve con NIP-44 para el comprador y, por separado, para el custodio.
- La copia del custodio no forma parte de la solicitud: se entrega al comprador en `order.escrow_key` (la solicitud solo lleva su hash).
  El comprador (o el usuario) se la pasa al custodio en una disputa, así que el custodio no puede leer la dirección de los pedidos sin disputa.

## Regiones

Los códigos se comparan por prefijo: `JP` > `JP-13` (Tokio) > `JP-13-13104` (Shinjuku).
El nivel municipal usa los códigos de gobiernos locales de Japón.
