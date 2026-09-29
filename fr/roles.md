---
layout: page
title: Rôles
permalink: /fr/roles/
lang: fr
ref: roles
nav_order: 3
---

{% include langnav.html %}

| Rôle | Toujours en ligne | Utilise | Fait | S'il triche |
|---|---|---|---|---|
| **utilisateur** (*user*) | non | navigateur | commande, finance, confirme la réception, ouvre les litiges | ― |
| **acheteur mandataire** (*proxy shopper*) | **oui** | nœud Go + shopper-bot | établit les devis, achète pour le compte de l'utilisateur, signale l'expédition ; touche des frais | perd sa part dans la décision du séquestre ; l'opérateur le retire de la liste |
| **séquestre** (*escrow*) | non | navigateur ou nœud Go | tranche les litiges avant T1 | l'opérateur le retire de la liste ; selon leurs conditions, sa caution est confisquée et sert d'indemnisation |
| **opérateur** (*operator*) | non | navigateur, nœud Go ou `psctl` | signe, par région, la liste des combinaisons acheteur × séquestre dignes de confiance ; reçoit les signalements | le coordinateur révoque sa délégation |
| **coordinateur** (*coordinator*) | non | navigateur ou `psctl` | signe les délégations aux opérateurs | les utilisateurs le retirent de leurs paramètres (congé) |

## Utilisateur

1. Ouvrez l'application web, créez une phrase mnémonique (12 mots), notez-la et choisissez une phrase secrète qui chiffre la clé.
2. Saisissez l'URL de la boutique, la région de la boutique et les articles.
3. Choisissez l'une des combinaisons acheteur × séquestre proposées et passez commande.
4. Un devis arrive. Si son taux de change s'écarte de vos propres sources de plus de 3 %, vous recevez une mise en garde ; au-delà de 10 %, un avertissement fort.
5. Acceptez et financez. L'argent est réparti entre le multisig et les frais anticipés du séquestre.
6. Quand l'article arrive, appuyez sur « reçu » (après un écran indiquant le montant et le destinataire, votre signature partielle est envoyée à l'acheteur).
7. S'il n'arrive pas, ouvrez un litige. L'application rassemble les preuves pour vous.

## Acheteur mandataire

- Lancez le nœud Go avec `role: shopper` et connectez shopper-bot (l'outil d'automatisation du navigateur).
- Configurez les moyens de paiement, les devises, les régions où vous pouvez payer en espèces, vos frais et votre politique de risque des boutiques.
- Les données de carte restent uniquement dans shopper-bot ; elles ne sont jamais transmises au nœud ni au réseau.
- Si l'utilisateur ne confirme jamais la réception et que T1 est passé, vous pouvez récupérer les fonds seul.

## Séquestre

- Ne tranche que les litiges des commandes qui ont payé ses frais anticipés.
- Il ne peut normalement pas lire l'adresse de livraison. En cas de litige, l'acheteur (ou l'utilisateur) lui remet la clé de déchiffrement.
- Une décision parvient aux deux parties sous forme de transaction signée ; elle devient définitive dès que l'une d'elles la contresigne.

## Opérateur

- Signe ses régions (une ou plusieurs), la liste des combinaisons, ainsi que les relais et points d'accès aux chaînes qu'il recommande.
- Après un signalement, publie une nouvelle version de la liste sans le fautif. La caution suit les conditions convenues avec le séquestre.

## Coordinateur

- Émet des délégations aux opérateurs. Révoquer, c'est publier une nouvelle version avec `revoked`.

## Être inscrit (le registre)

Le registre de confiance du mainteneur est le dépôt GitHub
[pad01g/proxy-shopping-registry](https://github.com/pad01g/proxy-shopping-registry). **Une pull request fusionnée vaut
approbation :** après chaque fusion, la CI signe les nouvelles délégations et listes avec les clés du registre et les publie
(sur les relais Nostr publics et sur https://pad01g.github.io/proxy-shopping-registry/events.json).

| Vous voulez être | À ajouter dans une pull request | Effet de la fusion |
|---|---|---|
| acheteur | `shoppers/<name>.json` (pk, contact, description, régions de paiement en espèces, moyens de paiement, les séquestres avec qui vous travaillez) | l'opérateur du registre inscrit vos combinaisons acheteur × séquestre |
| séquestre | `escrows/<name>.json` (pk, contact, description, délai de SLA en jours) | les acheteurs peuvent vous désigner ; vos combinaisons apparaissent |
| opérateur | `operators/<name>.json` (pk, contact, description, régions) | le coordinateur du registre vous délègue ; vous signez ensuite vos propres listes |
| coordinateur | `coordinators/<name>.json` (pk, contact, description) | vous apparaissez dans l'annuaire des coordinateurs que les applications proposent aux utilisateurs (chaque utilisateur choisit toujours à qui se fier) |

`pk` est votre clé publique Nostr (64 caractères hexadécimaux) : l'application web l'affiche dans les Paramètres, et `psctl keys --mnemonic-file …`
l'affiche aussi. Retirer quelqu'un se fait par une pull request qui déplace son fichier dans `revoked/` avec un motif. Le README
du registre donne les formats de fichier exacts et les commandes.
