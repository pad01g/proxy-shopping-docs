---
layout: page
title: Protocole
permalink: /fr/protocol/
lang: fr
ref: protocol
nav_order: 5
---

{% include langnav.html %}

La spécification complète est `docs/spec.md` dans proxy-shopping-go. Cette page en résume l'essentiel.

## Clés

Toutes les clés sont dérivées d'une seule phrase mnémonique (BIP39), une par usage (toutes en secp256k1).

| Usage | Chemin |
|---|---|
| identité (Nostr) | `m/44'/1237'/0'/0/0` (NIP-06) |
| libp2p | `m/7333'/0'/0'` |
| clé BTC de commande (utilisateur, acheteur) | `m/7333'/1'/{idx}'` (idx provient d'un hachage de l'identifiant de commande) |
| clé BTC de commande (séquestre) | `m/7333'/2'/{idx}`. La xpub est publique, ce qui permet aux autres de la dériver pendant que le séquestre est hors ligne |
| portefeuille BTC | `m/84'/1'/0'/0/0` |
| EVM | `m/44'/60'/0'/0/0` |

## Confiance

| kind | Signé par | Contenu |
|---|---|---|
| 30500 | coordinateur (*coordinator*) | délégation à un opérateur ; révoquée avec `revoked` |
| 30501 | opérateur (*operator*) | liste région × acheteur × séquestre, relais et points d'accès aux chaînes recommandés |
| 30502 / 30503 | acheteur (*shopper*) / séquestre (*escrow*) | profil (frais, régions de paiement en espèces, xpub, adresses de versement) |
| 10050 | tout le monde | les relais sur lesquels chacun reçoit ses messages |

La version dans la balise `v` détermine laquelle est la plus récente (normalement l'heure UNIX de la signature). Rien n'expire.
Les événements circulent à la fois sur les relais Nostr et sur le gossipsub de libp2p ; les nœuds Go retransmettent sur l'un ce qu'ils reçoivent sur l'autre.

## Messages

Les messages sont enveloppés avec NIP-59 (gift wrap → seal → contenu).
Le contenu est un événement **signé** (kind 5400), afin qu'un séquestre puisse le vérifier en tant que tiers lors d'un litige.

- Un message est envoyé à au moins k (2 par défaut) des relais de boîte de réception du destinataire.
- Le destinataire répond par un `ack` ; l'expéditeur renvoie jusqu'à le recevoir.
- Un message fait au plus 28000 octets. Les preuves volumineuses comme les captures d'écran ne sont référencées que par leur hachage ;
  en cas de litige, elles sont envoyées au séquestre en morceaux, sous forme de messages `attachment`.

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

T1 < T2 (hauteurs de bloc). Si l'utilisateur ne confirme jamais la réception et que T1 est passé, l'acheteur peut récupérer les fonds seul.
Si l'acheteur disparaît, l'utilisateur peut les reprendre seul après T2.
Le séquestre doit trancher un litige avant T1.

## USDC (Safe v1.4.1)

- Chaque commande reçoit un Safe avec trois propriétaires et un seuil de deux.
- Son adresse provient de CREATE2 ; elle peut donc être calculée à l'avance à partir de l'identifiant de commande.
- Les timelocks sont implémentés comme un module Safe (`PSEscrowModule`).
  - Après t1, l'acheteur peut récupérer les fonds avec `claimByShopper`.
  - Après t2, l'utilisateur peut les reprendre avec `refundToUser`.
- Les versements et les décisions sont des SafeTx signées en EIP-712. La répartition d'une décision utilise `MultiSendCallOnly`.

## Lier l'identité aux clés

Chaque demande de commande (`order.request`) contient une `key_proof` :
une signature de la clé de chaîne qui entre dans le multisig (la clé BTC de commande ou le compte EVM),
qui montre qu'elle appartient à la même personne que l'identité Nostr.
Sans elle, une autre identité pourrait copier les clés publiques de l'utilisateur dans sa propre demande et prétendre auprès du séquestre être l'utilisateur.

## Adresse de livraison

L'adresse est chiffrée avec une clé à usage unique K (XChaCha20-Poly1305).

- K est enveloppée avec NIP-44 pour l'acheteur et, séparément, pour le séquestre.
- La copie du séquestre ne fait pas partie de la demande : elle est remise à l'acheteur dans `order.escrow_key` (la demande n'en contient que le hachage).
  L'acheteur (ou l'utilisateur) la transmet au séquestre en cas de litige ; le séquestre ne peut donc pas lire l'adresse des commandes sans litige.

## Régions

Les codes sont comparés par préfixe : `JP` > `JP-13` (Tokyo) > `JP-13-13104` (Shinjuku).
Le niveau municipal utilise les codes des collectivités locales japonaises.
