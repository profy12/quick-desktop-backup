# Backup workstation → Backblaze B2

Sauvegarde chiffrée avec [restic](https://restic.net/) vers Backblaze B2 (API S3-compatible), orchestrée avec [Task](https://taskfile.dev/).

## Prérequis

```bash
# Ubuntu/Debian
sudo apt install restic
sudo snap install task --classic   # ou: go install github.com/go-task/task/v3/cmd/task@latest
```

## Démarrage rapide

### 1. Configurer les credentials

```bash
cp .env.example .env
```

Édite `.env` avec :
- **`AWS_ACCESS_KEY_ID`** et **`AWS_SECRET_ACCESS_KEY`** — clé applicative B2 créée sur [secure.backblaze.com/app_keys.htm](https://secure.backblaze.com/app_keys.htm)
- **`RESTIC_REPOSITORY`** — URL de ton bucket, ex. `s3:https://s3.us-west-004.backblazeb2.com/mon-bucket`
- **`RESTIC_PASSWORD`** — mot de passe de chiffrement restic (garde-le précieusement)

### 2. Initialiser le dépôt restic

```bash
task init
```

### 3. Tester un premier backup

```bash
task backup:run
```

### 4. Installer le timer systemd (backup automatique à 02h00)

```bash
task systemd:install
task systemd:enable
```

---

## Tâches disponibles

| Commande | Description |
|---|---|
| `task backup` | Backup complet + pruning |
| `task backup:run` | Backup sans pruning |
| `task prune` | Supprimer les anciens snapshots |
| `task check` | Vérifier l'intégrité du dépôt |
| `task check:full` | Vérification complète (lent) |
| `task snapshots` | Lister les snapshots |
| `task stats` | Statistiques du dépôt |
| `task restore` | Restaurer un snapshot |
| `task systemd:install` | Installer le service systemd |
| `task systemd:enable` | Activer le timer |
| `task systemd:disable` | Désactiver le timer |
| `task systemd:status` | État + logs du dernier backup |
| `task systemd:run-now` | Déclencher un backup via systemd |

## Restauration

```bash
# Restaurer le dernier snapshot dans ~/restore
task restore

# Restaurer un snapshot spécifique ailleurs
task restore SNAPSHOT_ID=abc123 TARGET=/tmp/restore
```

## Politique de rétention

| Période | Snapshots conservés |
|---|---|
| Quotidien | 7 jours |
| Hebdomadaire | 4 semaines |
| Mensuel | 6 mois |
| Annuel | 1 an |

## Sources sauvegardées

Définis la variable `BACKUP_SOURCES` dans ton fichier `.env` avec les chemins séparés par des espaces :

```bash
BACKUP_SOURCES=/home/aurelien/Documents /home/aurelien/dev /home/aurelien/.config
```

## Sécurité

- `.env` est ignoré par git — ne commite jamais tes credentials
- Les données sont chiffrées côté client par restic avant envoi
- Le mot de passe restic est la seule clé pour déchiffrer — fais-en une copie hors ligne
