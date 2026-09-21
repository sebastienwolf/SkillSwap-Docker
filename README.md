# SkillSwap — Docker

Lance l'application SkillSwap complète (API Laravel + frontend Vue) en une seule commande, avec la communication entre les deux services déjà configurée. Aucune installation préalable requise sur la machine hôte (ni PHP, ni Node, ni base de données) : seul Docker est nécessaire.

Ce dépôt ne contient pas le code applicatif : il orchestre les deux dépôts existants, inclus en [git submodules](https://git-scm.com/book/fr/v2/Utilitaires-Git-Sous-modules) :

- [`skillswap-api`](https://github.com/sebastienwolf/skillswap-api) — API Laravel
- [`SkillswapFrontend`](https://github.com/sebastienwolf/skillswap-vue.js) — frontend Vue 3

## Démarrage

```bash
git clone --recurse-submodules git@github.com:sebastienwolf/skillswap-docker.git
cd skillswap-docker
docker compose up --build
```

(Sur un dépôt déjà cloné sans `--recurse-submodules`, initialiser les submodules avec `git submodule update --init --recursive`.)

Une fois les conteneurs démarrés (le premier lancement inclut la génération de la clé d'application Laravel, les migrations et un jeu de données de démonstration) :

- Frontend : http://localhost:5173
- API : http://localhost:8000/api

## Comment ça communique

- Le frontend est buildé en assets statiques et servi par un Nginx dans son propre conteneur (voir `SkillswapFrontend/Dockerfile` et `SkillswapFrontend/docker/nginx.conf`).
- Ce Nginx relaie tous les appels `/api/*` du navigateur vers le conteneur `api` via le réseau Docker interne créé par `docker-compose.yml`. Le navigateur ne voit donc qu'une seule origine (celle du frontend) : pas de configuration CORS à gérer, et ça fonctionne quel que soit l'hôte utilisé pour accéder au conteneur (`localhost`, IP LAN, nom de domaine...).
- Le conteneur `api` n'est prêt (`front` ne démarre qu'après) que lorsque son port répond (`healthcheck` dans `docker-compose.yml`).

## Personnaliser les ports

Par défaut : frontend sur `5173`, API sur `8000`. Pour changer :

```bash
cp .env.example .env
# éditer .env (API_PORT, FRONT_PORT)
docker compose up --build
```

## Mettre à jour le code applicatif

Les submodules pointent vers un commit précis de chaque dépôt, pas vers leur branche `main` en continu. Pour récupérer les derniers changements :

```bash
git submodule update --remote --merge
git add skillswap-api SkillswapFrontend
git commit -m "chore: update submodules"
docker compose up --build
```

## Arrêter / nettoyer

```bash
docker compose down          # arrête les conteneurs, garde les données (base SQLite)
docker compose down -v       # arrête et supprime aussi les données
```
