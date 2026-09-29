---
layout: page
title: Prise en main
permalink: /fr/quickstart/
lang: fr
ref: quickstart
nav_order: 4
---

{% include langnav.html %}

## 1. Faire tourner tout le réseau en local (labo)

Un seul fichier docker compose démarre un réseau fermé. Il ne se connecte pas à Internet.

- deux relais Nostr
- un relais libp2p
- bitcoind sur un signet dédié, et une API compatible Esplora
- anvil (EVM) avec Safe et les autres contrats
- quatre boutiques fictives et une passerelle fictive de paiement par carte
- deux acheteurs (*shopper*), trois séquestres (*escrow*, dont un derrière un NAT), deux opérateurs (*operator*)
- l'application web publique et l'application de démonstration

Prérequis : Docker (environ 8 Go de mémoire).

Placez proxy-shopping-go et proxy-shopping-web côte à côte dans le même répertoire parent et lancez :

```sh
git clone https://github.com/pad01g/proxy-shopping-go
git clone https://github.com/pad01g/proxy-shopping-web
cd proxy-shopping-go
docker compose up -d --build
```

Pour repartir de zéro, lancez d'abord `docker compose down -v` afin de supprimer aussi les données stockées par les nœuds et les relais
(les chaînes repartent de zéro à chaque démarrage, donc des données de commande résiduelles seraient en désaccord avec elles).

### Lancer les tests e2e

```sh
docker compose run --rm runner         # all scenarios (about 10 minutes)
docker compose run --rm runner a e     # pick some
```

Les résultats sont écrits dans `e2e/results/e2e-latest.md`. Le runner fonctionne depuis l'intérieur du NAT (le réseau `home`),
donc chaque scénario suppose que l'utilisateur est derrière un NAT.

| id | Scénario |
|---|---|
| a | Cas nominal : BTC (une boutique en JPY) et USDC (une boutique en USD). Le taux de change du devis est vérifié, le séquestre touche ses frais anticipés, l'acheteur est payé. Un devis au taux très éloigné déclenche un avertissement fort |
| b | La livraison échoue et l'utilisateur ouvre un litige. Un séquestre honnête décide un remboursement et l'utilisateur contresigne (BTC). Le séquestre peut déchiffrer l'adresse et reçoit la capture d'écran en pièce jointe |
| c | Un séquestre tranche de manière malhonnête. L'utilisateur le signale ; l'opérateur le retire de la liste et, selon leurs conditions, confisque sa caution à titre d'indemnisation (USDC) |
| d | Un coordinateur (*coordinator*) révoque la délégation d'un opérateur, et les combinaisons de cette liste disparaissent des offres |
| e | Un navigateur derrière un NAT commande via l'application web publique (les messages 1:1 passent par les relais Nostr). Une requête d'état atteint le nœud du séquestre derrière un NAT via le circuit relay de libp2p |
| f | Un acheteur refuse une boutique qu'il juge risquée, ainsi qu'une boutique n'acceptant que les espèces hors de sa région. Un acheteur de la région accepte la commande en espèces et livre |
| g | Après T1, l'acheteur peut récupérer les fonds seul (BTC) |
| h | Si l'acheteur disparaît, l'utilisateur reprend les fonds seul après T2 (USDC) |
| i | Litige USDC avec un séquestre honnête. Le remboursement s'exécute même si quelqu'un envoie un petit montant au Safe avant la décision |
| j | Une commande que la boutique n'a pas pu honorer (rupture de stock). L'acheteur propose un remboursement coopératif ; l'utilisateur le vérifie et l'accepte |
| k | Si l'acheteur disparaît, l'utilisateur reprend les fonds seul après T2 (BTC) |

### Essayer tout le parcours dans la démo

Avec le labo en marche, ouvrez la démo à `http://localhost:8888/` (ni fichier hosts ni certificats nécessaires).
Sur un seul écran, l'utilisateur, le séquestre, l'opérateur et le coordinateur ont chacun leur propre clé (dans le stockage local du navigateur),
et l'acheteur est le nœud Go toujours en ligne lui-même. Les chemins sont réels : relais Nostr, bitcoind, anvil, nœuds Go.

- Choisissez un scénario en haut. Le guide à gauche indique qui fait quoi ensuite, et pourquoi.
- « Aller à cette étape » bascule sur l'onglet du rôle concerné et désigne le bouton à presser. Les formulaires sont pré-remplis pour le scénario : il suffit de cliquer et de confirmer.
- Scénarios : cas nominal (BTC / USDC), livraison échouée et remboursement, rupture de stock et remboursement coopératif, boutique risquée refusée, séquestre malhonnête signalé et retiré de la liste, et remboursement après T2 quand l'acheteur disparaît.
- Vous pouvez aussi ouvrir une fenêtre par rôle, par ex. `?role=user` et `?role=escrow,operator,coordinator` (les fenêtres d'un même navigateur partagent les clés et la progression).
- En réalité, chaque rôle est ailleurs, dans son propre navigateur. La démo ne sépare que les clés.
- Labo uniquement : toute personne pouvant atteindre le port 8888 peut utiliser le faucet, le minage, le saut dans le temps et l'API d'administration de l'acheteur (il n'écoute que sur 127.0.0.1).

Il existe aussi un e2e qui pilote la démo en suivant son guide : `docker compose run --rm runner demo`.

### Utiliser l'application web

Les noms du labo (`*.test`) ne se résolvent qu'à l'intérieur des conteneurs.
Pour utiliser l'application web depuis votre navigateur :

- Exposez localement le port 443 du conteneur `edge` (par ex. dans `compose.override.yaml` : `services: {edge: {ports: ["127.0.0.1:443:443"]}}`).
  Cela expose aussi `faucet.test` et `evm.test`, réservés au labo (n'importe qui peut créer des soldes et avancer le temps), donc n'exposez que sur localhost.
- Faites pointer `app.test` et les autres noms vers 127.0.0.1 dans votre fichier hosts.

Les certificats sont auto-signés.

1. Ouvrez `https://app.test/` et choisissez « créer » ou « restaurer à partir des mots ».
   Choisissez une phrase secrète (8 caractères ou plus) qui chiffre la clé (vous pouvez aussi choisir explicitement de ne pas la chiffrer, ou utiliser une extension NIP-07).
   La fois suivante, déverrouillez la clé avec la phrase secrète.
2. Sous « commande », saisissez l'URL de la boutique (par ex. `https://safe-shop.test/`), la région de la boutique (par ex. `JP-13-13104`) et l'article (par ex. `A-100`), puis recherchez des offres.
3. Choisissez une combinaison acheteur × séquestre, saisissez l'adresse de livraison et passez commande.
4. Quand le devis arrive, vérifiez l'écart de taux et le résultat de la vérification de l'adresse du multisig, puis acceptez.
5. Dans le labo, alimentez votre portefeuille avec « obtenir du faucet », puis appuyez sur « financer le multisig ».
   Chaque action qui déplace de l'argent (financer, payer, contresigner) passe par un écran indiquant le montant et le destinataire.
6. Quand l'article arrive, appuyez sur « reçu » pour payer l'acheteur. « Terminé » n'apparaît qu'une fois le paiement confirmé sur la chaîne.

## 2. Faire tourner sur un réseau public

L'architecture est la même que dans le labo. Les différences :

- utilisez des certificats ACME ;
- n'utilisez pas les clés de `lab/keys` ;
- utilisez un vrai signet et une vraie chaîne EVM.

### Acheteur

Toujours en ligne ; fait tourner le nœud Go et shopper-bot.

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

- Les données de carte ne figurent que dans le fichier de configuration de shopper-bot (`BOT_CARDS_FILE`) ; le nœud ne les reçoit jamais.
- Les étapes propres à chaque boutique sont écrites sous forme de driver shopper-bot. Un driver qui utilise l'IA pour opérer les boutiques a les mêmes entrées et sorties (`PurchaseRequest` / `PurchaseResult`).

### Séquestre / opérateur / coordinateur

Ceux-ci n'ont pas besoin d'être toujours en ligne.

- Ils peuvent utiliser les écrans « Escrow », « Operator » et « Coordinator » de l'application web.
- Pour tourner en continu, lancez le nœud Go avec `role: escrow` / `role: operator`.
- Pour signer uniquement, `psctl` convient aussi.

La version (`v`) vaut par défaut l'heure UNIX, vous n'avez donc normalement pas à la passer.

```sh
psctl keys --mnemonic-file coordinator.mnemonic
psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator public key> --publish wss://relay.example
psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish wss://relay.example
```

<a id="3-run-your-own-network-role-anywhere-without-permission"></a>

## 3. Tenir votre propre rôle dans le réseau, partout, sans autorisation

proxy-shopping n'a ni opérateur central ni inscription. La chaîne de confiance n'est faite que de clés et d'événements Nostr signés :
vous (ou un agent IA travaillant pour vous) pouvez donc lancer une place de marché locale dans votre propre ville :

1. **Devenez coordinateur :** créez une clé (`psctl keys --mnemonic-file coordinator.mnemonic`). Un coordinateur, ce n'est rien de plus.
2. **Déléguez à un opérateur** (votre propre seconde clé, ou quelqu'un de confiance) :
   `psctl delegate --network ps-main --mnemonic-file coordinator.mnemonic --operator <operator pk> --publish wss://relay.damus.io,wss://nos.lol,wss://relay.primal.net`
3. **Inscrivez des acheteurs et des séquestres pour votre région** — vous-même comme acheteur, un ami comme séquestre :
   `psctl list --network ps-main --mnemonic-file operator.mnemonic --file list.json --publish …`
4. **Demandez aux gens de faire confiance à votre clé de coordinateur** (en l'ajoutant dans les Paramètres de l'application web, ou via `PS_COORDINATORS` pour le serveur MCP).
   Pour être trouvé par tout le monde, ouvrez une pull request sur le [registre](https://github.com/pad01g/proxy-shopping-registry)
   ajoutant `coordinators/<name>.json` ; pour gérer les approbations de la même façon pour votre propre communauté, forkez le registre.

**Où le faire tourner :** une machine chez vous suffit. Le nœud n'établit que des connexions sortantes (relais Nostr et, derrière un NAT,
un circuit relay libp2p) : vous n'ouvrez aucun port. Utilisez [Tailscale](https://tailscale.com/) ou n'importe quel VPN pour accéder
à l'API d'administration de votre nœud et à l'application web depuis votre téléphone quand vous êtes dehors ; un petit VPS convient aussi.

**Ce qu'un agent IA peut faire :** un agent peut assurer la routine de l'opérateur et de l'acheteur — suivre les commandes via l'API d'administration
du nœud ou le serveur MCP, tenir les listes à jour, piloter shopper-bot pour les boutiques acceptant la carte, signaler les problèmes — et votre
connaissance locale (quelles boutiques, quelles régions, quels magasins n'acceptant que les espèces sont à deux pas de chez vous) est ce que personne d'autre ne peut offrir.
La skill `proxy-shopper` (`npx skills add pad01g/proxy-shopping-go`) guide un agent pas à pas.

Soyez honnête avec vos utilisateurs : le réseau public est nouveau et fonctionne sur BTC signet (des pièces de test), les gains viendront donc quand les gens l'utiliseront.

## Contribuer {#contributing}

**Les pull requests sont les bienvenues** — dans tous les dépôts : drivers de boutiques pour shopper-bot, nouveaux moyens de paiement et nouvelles chaînes,
traductions de cette documentation, relectures du protocole, corrections de bugs et inscriptions dans le
[registre](https://github.com/pad01g/proxy-shopping-registry). Ouvrez une issue ou une pull request sur GitHub.
