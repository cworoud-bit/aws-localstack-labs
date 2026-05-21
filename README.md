# aws-localstack-labs
# Lab 00 — Discover AWS Locally with LocalStack

> **Série** : Le Café ☕ — AWS Hands-On Labs with LocalStack  
> **Niveau** : Débutant | **Durée** : ~60 min | **Prérequis** : Docker, Python 3.8+, AWS CLI

---

## 🎯 Objectifs atteints

- [x] Comprendre ce qu'est LocalStack et pourquoi il existe
- [x] Installer et démarrer LocalStack sur ma machine
- [x] Configurer l'AWS CLI pour pointer vers LocalStack
- [x] Interagir avec S3, IAM et SQS entièrement en local
- [x] Comprendre comment LocalStack s'intègre dans un workflow DevOps

---

## ⚙️ Environnement

- **OS** : Windows 11
- **Shell** : PowerShell (Administrateur)
- **Docker** : v28.5.1
- **LocalStack** : v3.0.0 (image Docker)
- **LocalStack CLI** : v2026.3.0
- **AWS CLI** : v2.34.51
- **Python** : 3.13

---

## 📦 Part 1 — Installation

### Docker
```powershell
docker --version
# Docker version 28.5.1, build e180ab8

docker ps
# Plusieurs conteneurs en cours d'exécution confirmés
```

### LocalStack CLI
```powershell
pip install localstack
localstack --version
# LocalStack CLI 2026.3.0
```

### awscli-local
```powershell
pip install awscli-local
# Installation réussie (awscli-local 0.22.2)
```

> ⚠️ **Problème rencontré** : `awslocal --version` échouait avec l'erreur :
> `RuntimeError: Could not determine home directory`  
> **Cause** : Conflit entre AWS CLI v2 (binaire PyInstaller) et le contexte Administrateur PowerShell.  
> **Solution** : Utilisation de `aws --endpoint-url=http://localhost:4566` directement à la place de `awslocal`, ce qui est fonctionnellement identique.

---

## 🚀 Part 2 — Démarrage de LocalStack

### Lancement via Docker
```powershell
docker run --rm -d -p 4566:4566 --name localstack localstack/localstack:3.0.0
```

> ⚠️ **Note** : La nouvelle LocalStack CLI (2026.x) requiert un compte payant (`LOCALSTACK_AUTH_TOKEN`).  
> L'image Docker `localstack/localstack:3.0.0` reste gratuite et suffisante pour ce lab.

### Vérification du conteneur
```powershell
docker ps --filter name=localstack
```
```
CONTAINER ID   IMAGE                         STATUS
6418e8988ca8   localstack/localstack:3.0.0   Up (healthy)   0.0.0.0:4566->4566/tcp
```

### Health Check
```powershell
curl http://localhost:4566/_localstack/health
```
```json
{
  "services": {
    "s3": "running",
    "iam": "running",
    "sqs": "running",
    "dynamodb": "available",
    ...
  }
}
```
✅ LocalStack opérationnel sur `http://localhost:4566`

---

## 🔑 Part 3 — Configuration AWS CLI

### Création du profil localstack
```powershell
aws configure --profile localstack
```
```
AWS Access Key ID:     test
AWS Secret Access Key: test
Default region name:   us-east-1
Default output format: json
```

### Définir le profil par défaut
```powershell
# Sur PowerShell (Windows) — pas "export" comme sur Linux/Mac
$env:AWS_PROFILE = "localstack"
echo $env:AWS_PROFILE
# localstack
```

> ⚠️ **Différence Windows/Linux** : Sur PowerShell on utilise `$env:VAR = "valeur"` au lieu de `export VAR=valeur`.

---

## 🪣 Part 4 — Ressources AWS en local

### Step 8 — Bucket S3 (Menus de Le Café)

```powershell
# Créer le bucket
aws --endpoint-url=http://localhost:4566 s3 mb s3://lecafe-menus

# Vérifier
aws --endpoint-url=http://localhost:4566 s3 ls

# Uploader un fichier menu
echo "Espresso: 2.50 | Latte: 3.50 | Croissant: 2.00" > menu.txt
aws --endpoint-url=http://localhost:4566 s3 cp menu.txt s3://lecafe-menus/menu-paris.txt

# Confirmer l'upload
aws --endpoint-url=http://localhost:4566 s3 ls s3://lecafe-menus/

# Télécharger pour vérifier
aws --endpoint-url=http://localhost:4566 s3 cp s3://lecafe-menus/menu-paris.txt menu-downloaded.txt
```

✅ Bucket `lecafe-menus` créé et fichier uploadé/téléchargé avec succès.

---

### Step 9 — Utilisateur IAM (lecafe-app)

```powershell
# Créer l'utilisateur
aws --endpoint-url=http://localhost:4566 iam create-user --user-name lecafe-app

# Lister les utilisateurs
aws --endpoint-url=http://localhost:4566 iam list-users

# Attacher une policy S3 ReadOnly
aws --endpoint-url=http://localhost:4566 iam attach-user-policy `
  --user-name lecafe-app `
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Vérifier les policies attachées
aws --endpoint-url=http://localhost:4566 iam list-attached-user-policies --user-name lecafe-app
```

✅ Utilisateur `lecafe-app` créé avec la politique `AmazonS3ReadOnlyAccess`.

---

### Step 10 — Queue SQS (Traitement des commandes)

```powershell
# Créer la queue
aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name lecafe-orders
```
```json
{
    "QueueUrl": "http://sqs.us-east-1.localhost.localstack.cloud:4566/000000000000/lecafe-orders"
}
```

```powershell
# Récupérer l'URL
aws --endpoint-url=http://localhost:4566 sqs get-queue-url --queue-name lecafe-orders

# Envoyer un message (commande café)
aws --endpoint-url=http://localhost:4566 sqs send-message `
  --queue-url http://sqs.us-east-1.localhost.localstack.cloud:4566/000000000000/lecafe-orders `
  --message-body '{"item": "Latte", "size": "large", "table": 7}'
```
```json
{
    "MD5OfMessageBody": "4b52e0224c62fc65e9a5cea9703b3746",
    "MessageId": "827427c1-e45e-43f7-8cce-93c3c5da6400"
}
```

```powershell
# Lire le message (simulation cuisine)
aws --endpoint-url=http://localhost:4566 sqs receive-message `
  --queue-url http://sqs.us-east-1.localhost.localstack.cloud:4566/000000000000/lecafe-orders
```
```json
{
    "Messages": [{
        "MessageId": "827427c1-e45e-43f7-8cce-93c3c5da6400",
        "ReceiptHandle": "YmMzZmRkZTIt...",
        "Body": "{item: Latte, size: large, table: 7}"
    }]
}
```

✅ Queue créée, message envoyé et reçu avec `ReceiptHandle`.

> ⚠️ **Différence PowerShell** : Les guillemets `"..."` causent des erreurs d'échappement.  
> **Solution** : Utiliser des guillemets simples `'...'` pour le `--message-body` sur PowerShell.

---

## 🔍 Part 5 — Inspection de LocalStack

### Step 11 — Swagger UI
Ouvert dans le navigateur : `http://localhost:4566/_localstack/swagger`  
✅ Documentation complète de tous les endpoints LocalStack visible.

### Step 12 — Logs
```powershell
localstack logs
```
✅ Tous les appels API (HTTP method, path, status code) visibles dans les logs.

---

## 🧹 Cleanup

```powershell
docker stop localstack

docker ps --filter name=localstack
# Liste vide — conteneur arrêté
```

✅ LocalStack arrêté, toutes les ressources supprimées (stateless par défaut).

---

## ⚠️ Problèmes rencontrés et solutions

| Problème | Cause | Solution |
|----------|-------|----------|
| `awslocal` non reconnu | Pas encore installé | `pip install awscli-local` |
| `RuntimeError: Could not determine home directory` | AWS CLI v2 + contexte Admin PowerShell | Utiliser `aws --endpoint-url=http://localhost:4566` |
| `export` non reconnu | Commande Linux/Mac | Utiliser `$env:AWS_PROFILE = "localstack"` sur PowerShell |
| `python3` non reconnu | Windows utilise `python` | Utiliser `python -m json.tool` |
| `awslocal` cherche mauvais chemin `nektos.act` | Mauvaise installation AWS CLI | Réinstaller AWS CLI v2 via MSI officiel |
| `localstack start -d` demande auth token | LocalStack CLI 2026.x payant | Lancer via `docker run localstack/localstack:3.0.0` |
| Guillemets `\"` dans PowerShell | Échappement différent | Utiliser guillemets simples `'...'` |
| `localstack stop` : container `localstack-main` introuvable | Nom du conteneur différent | Utiliser `docker stop localstack` |

---

## 🤔 Réponses aux questions de réflexion

**1. Ephémère vs persistant (volume-mounted) ?**  
Le stockage éphémère (par défaut) est idéal pour les tests isolés — chaque session repart de zéro, sans pollution entre développeurs. Le stockage persistant (volume Docker) est utile quand l'équipe veut partager un état commun ou éviter de recréer des ressources à chaque démarrage. En CI/CD, l'éphémère est préférable pour garantir la reproductibilité.

**2. Risque d'exécuter `aws` contre le vrai AWS ?**  
Un développeur pourrait accidentellement créer/supprimer des ressources réelles et générer des coûts ou des incidents. Solution : utiliser des profils AWS distincts (`--profile localstack` vs `--profile prod`), définir `AWS_PROFILE=localstack` dans le `.env` du projet, et configurer des alertes de budget sur le compte AWS réel.

**3. Pourquoi `ReceiptHandle` plutôt que `MessageId` pour supprimer ?**  
Le `ReceiptHandle` est unique par tentative de réception, pas par message. Dans un système distribué, plusieurs consommateurs peuvent recevoir le même message simultanément (before timeout). Le `ReceiptHandle` garantit que seul le consommateur qui a effectivement traité le message peut le supprimer, évitant les suppressions prématurées par d'autres instances.

---

## 📋 Référence rapide (Windows PowerShell)

| Tâche | Commande |
|-------|----------|
| Démarrer LocalStack | `docker run --rm -d -p 4566:4566 --name localstack localstack/localstack:3.0.0` |
| Arrêter LocalStack | `docker stop localstack` |
| Health check | `curl http://localhost:4566/_localstack/health` |
| Créer bucket S3 | `aws --endpoint-url=http://localhost:4566 s3 mb s3://nom-bucket` |
| Uploader vers S3 | `aws --endpoint-url=http://localhost:4566 s3 cp fichier.txt s3://bucket/cle` |
| Créer utilisateur IAM | `aws --endpoint-url=http://localhost:4566 iam create-user --user-name nom` |
| Créer queue SQS | `aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name nom` |
| Envoyer message SQS | `aws --endpoint-url=http://localhost:4566 sqs send-message --queue-url URL --message-body '...'` |
| Lire message SQS | `aws --endpoint-url=http://localhost:4566 sqs receive-message --queue-url URL` |
