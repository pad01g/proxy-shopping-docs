---
layout: page
title: Fonctionnement
permalink: /fr/overview/
lang: fr
ref: overview
nav_order: 2
---

{% include langnav.html %}

## Ce que ça fait

Quand la boutique où vous voulez acheter n'accepte que les espèces ou un moyen de paiement particulier, vous payez en cryptomonnaie
(pour l'instant **BTC signet** et **USDC**) et un **acheteur mandataire** (*proxy shopper*) achète l'article pour vous et vous l'expédie.

L'argent va dans un **multisig 2 sur 3** propre à chaque commande (utilisateur, acheteur, séquestre — *escrow*).

- Quand l'article arrive, l'utilisateur et l'acheteur signent pour payer l'acheteur.
- En cas de litige, le séquestre décide de la répartition avec l'une des deux autres parties.
- Même si quelqu'un cesse de répondre, des **timelocks** garantissent que l'argent finit chez l'une des parties.

## Architecture

```
Browser (proxy-shopping-web)                Go nodes (proxy-shopping-go)
  keys and signing in the browser       shopper / escrow / operator / relay
        │ WSS (outbound only)                   │ WSS           │ libp2p
        ▼                                        ▼               ▼
   Nostr relays (named by the operators) ◀──▶  network of Go nodes
   = always-online entry point + mailbox        (gossipsub; NAT traversal via circuit relay v2 and DCUtR)
```

- **L'utilisateur ne fait rien tourner.** Ouvrir l'application web publique suffit pour participer.
  Les clés sont créées dans le navigateur et n'en sortent jamais ; chaque autorisation (signature) se fait dans le navigateur.
- Les utilisateurs derrière un NAT peuvent aussi se connecter, car la connexion à un relais Nostr est sortante.
- Les messages destinés à ceux qui ne sont pas toujours en ligne (utilisateurs, séquestres, opérateurs, coordinateurs) attendent,
  toujours chiffrés, sur des relais Nostr qui servent de boîtes aux lettres. Chaque message est envoyé à plusieurs relais.
- Seul l'acheteur doit être toujours en ligne. Il fait tourner un nœud Go et un outil d'automatisation du navigateur (shopper-bot) qui opère les boutiques.

## Circulation de la confiance

```
coordinator (the user trusts its public key)
  └─ delegation: "this operator may publish lists"
       └─ list: "in this region, this shopper and this escrow can be trusted together"
```

- L'utilisateur ne fait confiance **qu'à la clé publique du coordinateur** (*coordinator*). La confiance s'étend de là aux opérateurs (*operator*) et aux combinaisons acheteur × séquestre.
- Les listes sont versionnées et n'expirent jamais. Retirer quelqu'un, c'est publier une nouvelle version.
- Un utilisateur « congédie » un coordinateur en le supprimant de ses paramètres.

## Frais

| Pour qui | Garanti ? | Comment |
|---|---|---|
| acheteur | oui | inclus dans le devis |
| séquestre (à l'avance) | oui | payé directement au séquestre en même temps que le financement. **Un séquestre n'est pas tenu d'arbitrer les commandes sans ces frais payés à l'avance** |
| séquestre (litige) | oui | prélevé sur la répartition de la décision |
| opérateur / coordinateur | non | convenu hors protocole, par exemple des frais d'inscription. Ce qu'une liste vend, c'est « être trouvé » |

La caution du séquestre et sa confiscation ne font pas partie du protocole.
Ce sont des conditions que l'opérateur et le séquestre conviennent entre eux ; un contrat de référence est fourni.

## Boutiques qui n'acceptent que les espèces

L'emplacement d'une boutique est un code de région (par ex. `JP-13-13104` = Shinjuku, Tokyo).
Un acheteur déclare les régions où il peut se rendre et payer en espèces, et n'accepte que les commandes pour les boutiques de ces régions.

## Risque des boutiques

Avant d'accepter une commande, l'acheteur attribue une note à la boutique :
figure-t-elle sur sa liste d'autorisation (*allowlist*), utilise-t-elle HTTPS, et son paiement passe-t-il par une passerelle connue ?
Envoyer ses données de carte à un site quelconque est risqué, donc les commandes pour les boutiques mal notées sont refusées.
