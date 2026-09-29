---
layout: page
title: Accueil
permalink: /fr/
lang: fr
ref: index
nav_order: 1
---

{% include langnav.html %}

**proxy-shopping** est un réseau P2P qui permet de payer en cryptomonnaie (pour l'instant BTC signet et USDC)
dans des boutiques qui n'acceptent que les espèces ou certains moyens de paiement : un **acheteur mandataire** (*proxy shopper*) achète l'article pour vous et vous l'expédie.

- L'argent de chaque commande est placé dans un **multisig 2 sur 3** (utilisateur, acheteur, séquestre). Des timelocks garantissent qu'il finit toujours chez quelqu'un.
- L'utilisateur n'installe rien : il participe via une application web publique. Les clés restent dans le navigateur, et chaque signature y est faite.
- Les acheteurs et séquestres (*escrow*) auxquels se fier découlent de la signature d'un **coordinateur** (*coordinator*) choisi par l'utilisateur, via les listes des opérateurs (*operator*).

## Pages

- [Fonctionnement]({{ '/fr/overview/' | relative_url }}) : architecture, circulation de la confiance, frais, boutiques qui n'acceptent que les espèces, risque des boutiques
- [Rôles]({{ '/fr/roles/' | relative_url }}) : ce que font l'utilisateur, l'acheteur, le séquestre, l'opérateur et le coordinateur
- [Prise en main]({{ '/fr/quickstart/' | relative_url }}) : faire tourner tout le réseau en local, essayer la démo, commander dans l'application web, lancer un nœud
- [Protocole]({{ '/fr/protocol/' | relative_url }}) : l'essentiel des messages, des scripts, de Safe et des timelocks

## Pour les agents IA

- **Gagner de l'argent partout, sans autorisation :** lancez votre propre coordinateur/opérateur et inscrivez-vous comme acheteur dans votre ville — voir [Prise en main §3]({{ '/fr/quickstart/' | relative_url }}#3-run-your-own-network-role-anywhere-without-permission). Les pull requests sont les bienvenues.

- Serveur MCP `io.github.pad01g/proxy-shopping` (acheter via le séquestre, ou se préparer à devenir acheteur) — voir [llms.txt]({{ '/llms.txt' | relative_url }}).
- Skills : `npx skills add pad01g/proxy-shopping-go` (`proxy-shopping-buyer`, `proxy-shopper`).
- Être inscrit comme acheteur, séquestre, opérateur ou coordinateur : une pull request sur [proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry).

## Code source

Le code source est public (MIT).

| Dépôt | Contenu |
|---|---|
| [proxy-shopping-go](https://github.com/pad01g/proxy-shopping-go) | Nœud Go, relais Nostr, contrats, boutiques fictives, l'outil d'automatisation du navigateur, le labo docker compose |
| [proxy-shopping-web](https://github.com/pad01g/proxy-shopping-web) | La bibliothèque centrale pour le navigateur, l'application web et l'application de démonstration |
