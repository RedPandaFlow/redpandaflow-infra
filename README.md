# RedPandaFlow Infra

Stack Docker Compose de RedPandaFlow, une application de kanban collaboratif.

## Présentation

Ce dépôt orchestre tout le projet avec un unique fichier docker-compose : l'API
backend, le frontend React, une base de données PostgreSQL et pgAdmin.
L'architecture complète (services, réseaux, communication) est documentée dans
le [dépôt documentation](https://github.com/RedPandaFlow/documentation/blob/main/architecture.md),
et les choix de conteneurisation dans
[conteneurisation.md](https://github.com/RedPandaFlow/documentation/blob/main/conteneurisation.md).

## Équipe

Travail collaboratif sur l'ensemble du projet (backend, frontend, infra,
CI/CD, documentation) :

- Nathan FERRE
- Ylan Dessenne

## Services

- **db** — PostgreSQL 16 (alpine)
- **pgadmin** — pgAdmin 4, interface web pour la base de données
- **backend** — API ASP.NET Core construite depuis [redpandaflow-backend](https://github.com/RedPandaFlow/redpandaflow-backend)
- **frontend** — SPA React construite depuis [redpandaflow-frontend](https://github.com/RedPandaFlow/redpandaflow-frontend), servie par nginx

## Lancement en local

Le fichier compose construit les images backend et frontend depuis les dépôts
voisins : il faut donc cloner les trois côte à côte :

```bash
RedPandaFlow/
├── redpandaflow-backend/
├── redpandaflow-frontend/
└── redpandaflow-infra/
```

Copier `.env.example` vers `.env`, renseigner les valeurs (identifiants de la
base, secret JWT, ports), puis :

```bash
docker compose up -d --build
```

## Variables d'environnement

| Variable          | Rôle                                                       |
| ----------------- | ---------------------------------------------------------- |
| `DB_USER`         | Utilisateur PostgreSQL                                     |
| `DB_PASSWORD`     | Mot de passe PostgreSQL                                    |
| `DB_REDPANDAFLOW` | Nom de la base PostgreSQL                                  |
| `PGADMIN_EMAIL`   | Email de connexion pgAdmin                                 |
| `PGADMIN_PASSWORD`| Mot de passe de connexion pgAdmin                          |
| `PGADMIN_PORT`    | Port hôte mappé vers pgAdmin                               |
| `BACKEND_PORT`    | Port hôte mappé vers l'API                                 |
| `FRONTEND_PORT`   | Port hôte mappé vers la SPA                                |
| `JWT_SECRET_KEY`  | Clé de signature des JWT (`openssl rand -base64 48`)       |
| `VITE_API_URL`    | URL de l'API intégrée au frontend au moment du build       |

## Ports par défaut

- Frontend → `http://localhost:5000`
- Backend → `http://localhost:5090`
- pgAdmin → `http://localhost:80`

PostgreSQL n'est **pas** publié sur l'hôte : il n'est joignable que sur le
réseau interne `backend-network` (par l'API et pgAdmin).

## Volumes persistants

- `postgres_data` — fichiers de la base de données
- `pgadmin_data` — configuration de pgAdmin

Le contenu de la base survit à `docker compose down`. Utiliser
`docker compose down -v` pour tout effacer.

## Sauvegarde et restauration

Deux scripts dans `scripts/` sauvegardent et restaurent la base PostgreSQL. Ils
lisent les identifiants et le nom de la base directement dans le conteneur `db`
en cours d'exécution : aucune configuration supplémentaire n'est nécessaire, il
suffit que la stack soit démarrée (`docker compose up -d`).

Créer une sauvegarde (dump au format custom écrit dans `scripts/../backups/`) :

```bash
./scripts/backup.sh
```

Restaurer une sauvegarde (supprime et recrée les objets existants) :

```bash
./scripts/restore.sh backups/redpandaflow_db-20260605-110131.dump
```

Le dossier `backups/` est ignoré par git. Les valeurs par défaut peuvent être
surchargées avec les variables d'environnement `DB_CONTAINER` et `BACKUP_DIR`.

## Dépôts liés

- [redpandaflow-backend](https://github.com/RedPandaFlow/redpandaflow-backend) — API ASP.NET Core
- [redpandaflow-frontend](https://github.com/RedPandaFlow/redpandaflow-frontend) — React
- [documentation](https://github.com/RedPandaFlow/documentation) — documentation du projet
