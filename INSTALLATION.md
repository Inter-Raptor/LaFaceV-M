# La Face V&M

Site de collection de Vivien et Myriam, prévu pour GitHub Pages.

## État de cette première version

L'interface est prête. La collection démarre vide : aucun disque d'exemple n'est présenté comme un disque réellement possédé. La consultation fonctionne dès l'activation de Pages. Les boutons d'ajout, modification et suppression ne sont accessibles qu'après connexion du service sécurisé et authentification. Aucune sauvegarde locale n'est présentée comme une synchronisation.

## 1. Publier le site

Dans le dépôt : Settings → Pages → Build and deployment → Source : Deploy from a branch → branche main → dossier /docs → Save.

Adresse attendue après publication : https://inter-raptor.github.io/LaFaceV-M/

## 2. Activer l'enregistrement et le PIN

Un compte Cloudflare est nécessaire pour héberger le petit service du dossier `worker`. Il conserve seulement les compteurs de tentatives et un cache temporaire ; les vinyles et pochettes restent dans GitHub.

Sur un ordinateur avec Node.js et le dépôt téléchargé, ouvrir un terminal dans `worker` :

```
npx wrangler login
npx wrangler deploy
npx wrangler secret put GITHUB_TOKEN
npx wrangler secret put ADMIN_PIN
npx wrangler secret put SESSION_SECRET
```

Chaque commande `secret put` demande la valeur dans le terminal. Ne jamais écrire ces valeurs dans un fichier du dépôt ou les envoyer dans une conversation.

- `GITHUB_TOKEN` : jeton GitHub à permissions fines, limité à **Inter-Raptor/LaFaceV-M**, avec **Contents: Read and write**. Choisir une date d'expiration ; le renouveler dans Cloudflare lorsqu'il expire.
- `ADMIN_PIN` : code aléatoire de 6 à 12 chiffres partagé entre Vivien et Myriam.
- `SESSION_SECRET` : secret aléatoire d'au moins 32 caractères. Le changer invalide toutes les sessions.

Renseigner ensuite l'adresse HTTPS du Worker dans `docs/config.js`, par exemple `https://lafacevm-api.VOTRE-SOUS-DOMAINE.workers.dev` (remplacer par l'adresse réellement fournie). Aucun secret dans ce fichier.

## 3. Fonctionnement

- Lecture publique ; édition protégée côté serveur.
- 5 PIN incorrects entraînent un blocage global des nouvelles connexions de 15 minutes, conservé même si le service redémarre. Ce choix protège un PIN court, mais un tiers peut volontairement provoquer ce blocage. Les sessions déjà ouvertes continuent de fonctionner.
- Session de modification de 30 minutes, conservée uniquement en mémoire de la page. « Verrouiller » efface cette session du navigateur ; le jeton émis expire côté serveur au bout des 30 minutes.
- Chaque ajout génère `docs/data/vinyles/IDENTIFIANT.json` et une pochette carrée JPEG dans `docs/images/`. `docs/data/index.json` rassemble les fiches pour la consultation.
- Un seul commit regroupe la fiche, la pochette et l'index. Une modification concurrente est refusée plutôt que d'écraser la version récente.
- La consultation connectée lit les données courantes depuis le service (cache de 10 secondes). Les pochettes peuvent demander un court délai de disponibilité sur GitHub.
- La suppression retire les fichiers courants. L'historique Git conserve les versions précédentes, y compris les images : ne pas y stocker de documents privés.
- Les textes, images et fiches sont publics dans ce dépôt. Le PIN protège la modification, pas la consultation.

## Vérifications avant utilisation réelle

Tester après configuration : connexion PIN, ajout avec photo et deux disques, consultation sur un second appareil, modification, suppression et refus d'une requête sans session. La version initiale a été vérifiée localement ; la connexion réelle Cloudflare/GitHub reste à vérifier après installation.
