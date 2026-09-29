---
layout: page
title: Inicio
permalink: /es/
lang: es
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping** es una red P2P que te permite pagar con criptomonedas (por ahora BTC signet y USDC)
en tiendas que solo aceptan efectivo o ciertos medios de pago: un **comprador por encargo (proxy shopper)** compra el artículo por ti y te lo envía.

- El dinero de cada pedido queda en una **multifirma 2 de 3** (usuario, comprador por encargo y custodio (escrow)). Los bloqueos temporales (timelocks) garantizan que siempre acabe en manos de alguien.
- Los usuarios no instalan nada: entran a través de una aplicación web pública. Las claves se quedan en el navegador y todas las firmas se hacen allí.
- En qué compradores y custodios confiar se deriva de la firma de un **coordinador (coordinator)** que elige el usuario, a través de las listas de los operadores (operators).

## Páginas

- [Cómo funciona]({{ '/es/overview/' | relative_url }}): arquitectura, cómo fluye la confianza, comisiones, tiendas que solo aceptan efectivo, riesgo de las tiendas
- [Roles]({{ '/es/roles/' | relative_url }}): qué hace cada uno: usuario, comprador por encargo, custodio, operador y coordinador
- [Primeros pasos]({{ '/es/quickstart/' | relative_url }}): ejecutar toda la red en local, probar la demo, hacer un pedido en la aplicación web, operar un nodo
- [Protocolo]({{ '/es/protocol/' | relative_url }}): lo esencial de los mensajes, los scripts, Safe y los bloqueos temporales

## Para agentes de IA

- **Gana dinero en cualquier lugar, sin pedir permiso:** opera tu propio coordinador u operador y inscríbete como comprador por encargo en tu ciudad; consulta [Primeros pasos §3]({{ '/es/quickstart/' | relative_url }}#3-run-your-own-network-role-anywhere-without-permission). Las pull requests son bienvenidas.

- Servidor MCP `io.github.pad01g/proxy-shopping` (comprar a través del custodio o prepararse para ser comprador por encargo): consulta [llms.txt]({{ '/llms.txt' | relative_url }}).
- Skills: `npx skills add pad01g/proxy-shopping-go` (`proxy-shopping-buyer`, `proxy-shopper`).
- Para aparecer en las listas como comprador por encargo, custodio, operador o coordinador: una pull request a [proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry).

## Código fuente

El código fuente es público (MIT).

| Repositorio | Contenido |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | Nodo en Go, relay de Nostr, contratos, tiendas ficticias, la herramienta de automatización del navegador, el laboratorio con docker compose |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | La biblioteca central para el navegador, la aplicación web y la aplicación de demostración |
