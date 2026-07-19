# steamtrack-status

Page passerelle de [steamtrack](https://github.com/cheapmanga/SteamTrack) :
l'adresse d'entree stable d'un service qui, lui, n'en a pas.

## Le probleme qu'elle resout

Le service tourne derriere un **quick tunnel Cloudflare**, qui tire une
adresse au hasard a chaque demarrage. Un lien partage meurt donc au premier
redemarrage de la VM. Plutot que d'acheter un domaine, la VM publie son
adresse courante dans `tunnel.json` du depot SteamTrack, et cette page l'y
lit, verifie que le service repond, puis redirige.

## Deploiement

Heberge sur **Cloudflare Pages**, branche a ce depot : **tout commit sur
`main` redeploie automatiquement**. Il n'y a rien a lancer a la main.

| Reglage Pages | Valeur |
|---|---|
| Production branch | `main` |
| Build command | *(vide)* |
| Build output directory | *(racine)* |

`_headers` interdit toute mise en cache : la page est un aiguillage, et
servir une adresse perimee est exactement le defaut qu'elle doit eviter.

## Deux pieges deja rencontres

- **`raw.githubusercontent` cache les fichiers 5 minutes**, cote serveur, et
  **rien ne le perce** : ni parametre d'horodatage, ni `Cache-Control:
  no-cache`. La page lisait donc une adresse perimee pendant cinq minutes
  apres chaque redemarrage, et annoncait le service hors ligne alors qu'il
  tournait. Elle passe desormais par l'**API GitHub contents**
  (`max-age=60`), avec repli sur raw si le quota anonyme -- 60 requetes par
  heure et par IP -- venait a s'epuiser.
- **Le favicon est inline en data URI**, pas en fichier. Cette page doit
  rester autonome : elle sert justement a diagnostiquer les moments ou tout
  le reste est injoignable.

## Source unique

Ce depot est **la seule copie** de la page. Une version anterieure a vecu
dans `gateway/` du depot SteamTrack ; ne pas la reintroduire, deux copies
divergeraient des la premiere correction.
