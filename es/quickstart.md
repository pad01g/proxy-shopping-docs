---
layout: page
title: Primeros pasos
permalink: /es/quickstart/
lang: es
ref: quickstart
nav_order: 4
---

{% include langnav.html %}

## 1. Ejecutar toda la red en local (laboratorio)

Un solo archivo de docker compose levanta una red cerrada. No se conecta a internet.

- dos relays de Nostr
- un relay de libp2p
- bitcoind en un signet propio y una API compatible con Esplora
- anvil (EVM) con Safe y los demás contratos
- cuatro tiendas ficticias y una pasarela de pago con tarjeta ficticia
- dos compradores por encargo (shoppers), tres custodios (escrows) (uno detrás de NAT), dos operadores (operators)
- la aplicación web pública y la aplicación de demostración

Requisitos: Docker (unos 8 GB de memoria).

Coloca proxy-shopping-go y proxy-shopping-web uno junto al otro en el mismo directorio padre y ejecuta:

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web
cd proxy-shopping-go
docker compose up -d --build
```

Para empezar de cero, ejecuta primero `docker compose down -v` para borrar también los datos guardados por los nodos y los relays
(las cadenas empiezan de cero en cada arranque, así que los datos de pedidos que queden no coincidirían con ellas).

### Ejecutar las pruebas e2e

```sh
docker compose run --rm runner         # all scenarios (about 10 minutes)
docker compose run --rm runner a e     # pick some
```

Los resultados van a `e2e/results/e2e-latest.md`. El runner trabaja desde dentro del NAT (la red `home`),
así que todos los escenarios suponen que el usuario está detrás de NAT.

| id | Escenario |
|---|---|
| a | Camino feliz: BTC (una tienda en JPY) y USDC (una tienda en USD). Se comprueba el tipo de cambio del presupuesto, el custodio recibe su comisión por adelantado y el comprador cobra. Un presupuesto con un tipo muy desviado recibe una advertencia seria |
| b | La entrega falla y el usuario abre una disputa. Un custodio honesto dictamina un reembolso y el usuario lo refrenda (BTC). El custodio puede descifrar la dirección y recibe la captura de pantalla como adjunto |
| c | Un custodio falla de forma deshonesta. El usuario lo denuncia; el operador lo quita de la lista y, según sus condiciones, le confisca la fianza como compensación (USDC) |
| d | Un coordinador (coordinator) revoca la delegación de un operador, y las combinaciones de esa lista desaparecen de las ofertas |
| e | Un navegador detrás de NAT hace un pedido a través de la aplicación web pública (los mensajes 1:1 pasan por relays de Nostr). Una consulta de estado llega al nodo del custodio detrás de NAT a través del circuit relay de libp2p |
| f | Un comprador rechaza una tienda que puntúa como arriesgada y una tienda de solo efectivo fuera de su región. Un comprador de esa región acepta el pedido en efectivo y lo entrega |
| g | Después de T1, el comprador puede quedarse con los fondos por su cuenta (BTC) |
| h | Si el comprador desaparece, el usuario recupera los fondos por su cuenta después de T2 (USDC) |
| i | Disputa en USDC con un custodio honesto. El reembolso se ejecuta aunque alguien envíe una pequeña cantidad al Safe antes del fallo |
| j | Un pedido que la tienda no pudo cumplir (agotado). El comprador ofrece un reembolso cooperativo; el usuario lo revisa y lo acepta |
| k | Si el comprador desaparece, el usuario recupera los fondos por su cuenta después de T2 (BTC) |

### Probar todo el flujo en la demo

**Pruébalo en tu navegador: <https://pad01g.github.io/proxy-shopping-web/> (todo simulado dentro de la página).**

Con el laboratorio en marcha, abre la demo en `http://localhost:8888/` (no hace falta archivo hosts ni certificados).
En una sola pantalla, el usuario, el custodio, el operador y el coordinador tienen cada uno su propia clave (en el almacenamiento local del navegador),
y el comprador por encargo es el propio nodo Go siempre en línea. Los caminos son reales: relays de Nostr, bitcoind, anvil, nodos Go.

- Elige un escenario arriba. La guía de la izquierda dice quién hace qué a continuación, y por qué.
- "Ir a este paso" cambia a la pestaña de ese rol y señala el botón que hay que pulsar. Los formularios vienen rellenados para el escenario, así que solo haces clic y confirmas.
- Escenarios: camino feliz (BTC / USDC), entrega fallida y reembolso, producto agotado y reembolso cooperativo, una tienda arriesgada rechazada, un custodio deshonesto denunciado y quitado de la lista, y un reembolso después de T2 cuando el comprador desaparece.
- También puedes abrir una ventana por rol, p. ej. `?role=user` y `?role=escrow,operator,coordinator` (las ventanas del mismo navegador comparten las claves y el progreso).
- En la realidad, cada rol está en otro lugar, en su propio navegador. La demo solo separa las claves.
- Solo en el laboratorio: cualquiera que pueda alcanzar el puerto 8888 puede usar el faucet, la minería, el salto en el tiempo y la API de administración del comprador (por eso está vinculado solo a 127.0.0.1).

También hay una e2e que maneja la demo siguiendo su guía: `docker compose run --rm runner demo`.

### Usar la aplicación web

Los nombres del laboratorio (`*.test`) solo se resuelven dentro de los contenedores.
Para usar la aplicación web desde tu navegador:

- Expón en local el puerto 443 del contenedor `edge` (p. ej., en `compose.override.yaml`: `services: {edge: {ports: ["127.0.0.1:443:443"]}}`).
  Esto también expone `faucet.test` y `evm.test`, exclusivos del laboratorio (cualquiera puede acuñar saldo y adelantar el tiempo), así que exponlo solo en localhost.
- Apunta `app.test` y los demás nombres a 127.0.0.1 en tu archivo hosts.

Los certificados son autofirmados.

1. Abre `https://app.test/` y elige "crear nueva" o "restaurar desde palabras".
   Elige una frase de contraseña (8 caracteres o más) que cifre la clave (también puedes elegir explícitamente no cifrarla, o usar una extensión NIP-07).
   La próxima vez, desbloquea la clave con la frase de contraseña.
2. En "pedido", introduce la URL de la tienda (p. ej., `https://safe-shop.test/`), la región de la tienda (p. ej., `JP-13-13104`) y el artículo (p. ej., `A-100`), y busca ofertas.
3. Elige una combinación de comprador × custodio, introduce la dirección de entrega y haz el pedido.
4. Cuando llegue el presupuesto, revisa la diferencia de tipo de cambio y el resultado de la comprobación de la dirección multifirma, y acéptalo.
5. En el laboratorio, llena tu cartera con "obtener del faucet" y luego pulsa "depositar en la multifirma".
   Toda acción que mueve dinero (depositar, pagar, refrendar) pasa por una pantalla que muestra el importe y el destinatario.
6. Cuando llegue el artículo, pulsa "recibido" para pagar al comprador. "Completado" aparece solo después de que el pago se confirma en la cadena.

## 2. Ejecutar en una red pública

La arquitectura es la misma que en el laboratorio. Las diferencias:

- usa certificados ACME;
- no uses las claves de `lab/keys`;
- usa un signet real y una cadena EVM real.

### Comprador por encargo

Siempre en línea; ejecuta el nodo Go y shopper-bot.

```yaml
role: shopper
name: my-shopper
network: ps-main
mnemonic_file: /keys/shopper.mnemonic
nostr: {relays: ["wss://relay.example"], k: 2}
trust: {coordinators: ["<coordinator public key>"]}
chain:
  btc: {network: signet, esplora: "https://mempool.space/signet/api"}
shopper:
  bot_url: "http://shopper-bot:7000"
  cash_regions: [JP-13]
  risk: {allowlist: [shop.example], known_gateways: [pay.example], threshold: 70}
```

- Los datos de las tarjetas van solo en el archivo de configuración de shopper-bot (`BOT_CARDS_FILE`); el nodo nunca los recibe.
- Los pasos de cada tienda se escriben como un driver de shopper-bot. Un driver que usa IA para operar tiendas tiene la misma entrada y salida (`PurchaseRequest` / `PurchaseResult`).

### Custodio / operador / coordinador

No necesitan estar siempre en línea.

- Pueden usar las pantallas "Custodio", "Operador" y "Coordinador" de la aplicación web.
- Para funcionar de forma continua, ejecuta el nodo Go con `role: escrow` / `role: operator`.
- Para solo firmar, también sirve `psctl`.

La versión (`v`) es por defecto la hora UNIX, así que normalmente no hace falta pasarla.

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator public key> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```

<a id="3-run-your-own-network-role-anywhere-without-permission"></a>

## 3. Opera tu propio rol en la red, en cualquier lugar, sin pedir permiso

proxy-shopping no tiene operador central ni nada en lo que registrarse. La cadena de confianza son solo claves y eventos de Nostr firmados,
así que tú (o un agente de IA que trabaje para ti) puedes abrir un mercado local en tu propia ciudad:

1. **Sé coordinador:** crea una clave (`psctl keys --mnemonic-file coordinator.mnemonic`). Eso es todo lo que es un coordinador.
2. **Delega en un operador** (una segunda clave tuya, o alguien de confianza):
   `psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator pk> --publish wss://relay.damus.io,wss://nos.lol,wss://relay.primal.net`
3. **Incluye en la lista compradores y custodios para tu región**: tú como comprador por encargo, un amigo como custodio:
   `psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish …`
4. **Pide a la gente que confíe en tu clave de coordinador** (se añade en la Configuración de la aplicación web, o con `PS_COORDINATORS` para el servidor MCP).
   Para que todo el mundo te encuentre, abre una pull request al [registro](https://github.com/pad01g/proxy-shopping-registry)
   añadiendo `coordinators/<name>.json`; para gestionar aprobaciones de la misma manera en tu propia comunidad, haz un fork del registro.

**Dónde ejecutarlo:** basta con una máquina en casa. El nodo solo hace conexiones salientes (relays de Nostr y, si está detrás de NAT, un
circuit relay de libp2p), así que no tienes que abrir puertos. Usa [Tailscale](https://tailscale.com/) o cualquier VPN para acceder a la
API de administración de tu nodo y a la aplicación web desde el móvil cuando estés fuera; un VPS pequeño también sirve.

**Qué puede hacer un agente de IA:** un agente puede llevar la rutina del operador y del comprador: vigilar los pedidos a través de la API de
administración del nodo o del servidor MCP, mantener las listas al día, manejar shopper-bot para las tiendas con tarjeta, informar de problemas. Y tu propio
conocimiento local (qué tiendas, qué regiones, qué tiendas de solo efectivo tienes a un paseo) es la parte que nadie más puede ofrecer.
La skill `proxy-shopper` (`npx skills add pad01g/proxy-shopping-go`) guía a un agente paso a paso.

Sé honesto con tus usuarios: la red pública es nueva y funciona con BTC signet (monedas de prueba), así que las ganancias llegarán cuando la gente la use.

## Contribuir {#contributing}

**Las pull requests son bienvenidas**, en todos los repositorios: drivers de tiendas para shopper-bot, nuevos medios de pago y cadenas,
traducciones de esta documentación, revisiones del protocolo, correcciones de errores e inscripciones en el
[registro](https://github.com/pad01g/proxy-shopping-registry). Abre un issue o una pull request en GitHub.
