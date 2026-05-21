# Lab 01 — IAM: Identity and Access Management

> **Série** : Le Café ☕ — AWS Hands-On Labs with LocalStack  
> **Niveau** : Débutant → Intermédiaire | **Durée** : ~75 min  
> **Prérequis** : Lab 00 complété, LocalStack running, AWS CLI configuré

---

## 🎯 Objectifs atteints

- [x] Expliquer la différence entre users, groups, roles et policies IAM
- [x] Créer une structure d'accès basée sur les rôles pour une organisation
- [x] Écrire une policy IAM custom en JSON
- [x] Attacher des policies aux groupes plutôt qu'aux users (bonne pratique)
- [x] Assumer un IAM role et comprendre les credentials temporaires
- [x] Reconnaître les erreurs IAM courantes qui mènent à des incidents de sécurité

---

## ⚙️ Environnement

- **OS** : Windows 11
- **Shell** : PowerShell (Administrateur)
- **Docker** : v28.5.1
- **LocalStack** : v3.0.0 (image Docker)
- **AWS CLI** : v2.34.51

---

## 🚀 Démarrage de LocalStack

```powershell
docker start localstack
docker ps --filter name=localstack
```
```
CONTAINER ID   IMAGE                         STATUS
6418e8988ca8   localstack/localstack:3.0.0   Up (healthy)   0.0.0.0:4566->4566/tcp
```

✅ LocalStack opérationnel sur `http://localhost:4566`

---

## 👤 Part 1 — Création des utilisateurs

### Step 2 — Créer les users humains

```powershell
aws --endpoint-url=http://localhost:4566 iam create-user --user-name alice
aws --endpoint-url=http://localhost:4566 iam create-user --user-name bob
aws --endpoint-url=http://localhost:4566 iam create-user --user-name charlie
```

```powershell
aws --endpoint-url=http://localhost:4566 iam list-users
```

✅ 3 utilisateurs créés : `alice` (dev), `bob` (dev), `charlie` (ops)

---

## 🗂️ Part 2 — Création des groupes

### Step 3 — Créer les groupes

```powershell
aws --endpoint-url=http://localhost:4566 iam create-group --group-name cafe-developers
aws --endpoint-url=http://localhost:4566 iam create-group --group-name cafe-operations
```

### Step 4 — Ajouter les users aux groupes

```powershell
aws --endpoint-url=http://localhost:4566 iam add-user-to-group --user-name alice --group-name cafe-developers
aws --endpoint-url=http://localhost:4566 iam add-user-to-group --user-name bob --group-name cafe-developers
aws --endpoint-url=http://localhost:4566 iam add-user-to-group --user-name charlie --group-name cafe-operations
```

```powershell
# Vérification
aws --endpoint-url=http://localhost:4566 iam get-group --group-name cafe-developers
aws --endpoint-url=http://localhost:4566 iam get-group --group-name cafe-operations
```

✅ `alice` et `bob` → `cafe-developers` | `charlie` → `cafe-operations`

---

## 📄 Part 3 — Policies custom

### Step 6 — Policy Developer S3 (`developer-s3-policy.json`)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3BucketListing",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::lecafe-assets"
    },
    {
      "Sid": "AllowS3ObjectOperations",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::lecafe-assets/*"
    }
  ]
}
```

```powershell
aws --endpoint-url=http://localhost:4566 iam create-policy `
  --policy-name LeCafe-Developer-S3 `
  --policy-document file://developer-s3-policy.json
```

✅ Policy `LeCafe-Developer-S3` créée — ARN : `arn:aws:iam::000000000000:policy/LeCafe-Developer-S3`

---

### Step 7 — Policy Operations EC2 Read-Only (`operations-policy.json`)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2ReadOnly",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus",
        "ec2:DescribeRegions",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeVpcs"
      ],
      "Resource": "*"
    },
    {
      "Sid": "AllowCloudWatchReadOnly",
      "Effect": "Allow",
      "Action": [
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams",
        "logs:GetLogEvents",
        "logs:FilterLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

```powershell
aws --endpoint-url=http://localhost:4566 iam create-policy `
  --policy-name LeCafe-Operations-ReadOnly `
  --policy-document file://operations-policy.json
```

✅ Policy `LeCafe-Operations-ReadOnly` créée

---

### Step 8 — Attacher les policies aux groupes

```powershell
aws --endpoint-url=http://localhost:4566 iam attach-group-policy `
  --group-name cafe-developers `
  --policy-arn arn:aws:iam::000000000000:policy/LeCafe-Developer-S3

aws --endpoint-url=http://localhost:4566 iam attach-group-policy `
  --group-name cafe-operations `
  --policy-arn arn:aws:iam::000000000000:policy/LeCafe-Operations-ReadOnly
```

```powershell
# Vérification
aws --endpoint-url=http://localhost:4566 iam list-attached-group-policies --group-name cafe-developers
aws --endpoint-url=http://localhost:4566 iam list-attached-group-policies --group-name cafe-operations
```

✅ Policies attachées aux groupes — alice et bob héritent automatiquement des permissions S3

---

## 🎭 Part 4 — Service Role

### Step 10 — Trust Policy (`trust-policy.json`)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2ToAssumeRole",
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### Step 11 — Créer le role

```powershell
aws --endpoint-url=http://localhost:4566 iam create-role `
  --role-name lecafe-app-role `
  --assume-role-policy-document file://trust-policy.json `
  --description "Role assumed by the Le Cafe ordering application"
```

✅ Role `lecafe-app-role` créé — ARN : `arn:aws:iam::000000000000:role/lecafe-app-role`

---

### Step 12 — Policy permissions application (`app-role-policy.json`)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3AssetRead",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::lecafe-assets",
        "arn:aws:s3:::lecafe-assets/*"
      ]
    },
    {
      "Sid": "AllowSQSOrderWrite",
      "Effect": "Allow",
      "Action": [
        "sqs:SendMessage",
        "sqs:GetQueueUrl",
        "sqs:GetQueueAttributes"
      ],
      "Resource": "arn:aws:sqs:us-east-1:000000000000:lecafe-orders"
    }
  ]
}
```

```powershell
aws --endpoint-url=http://localhost:4566 iam put-role-policy `
  --role-name lecafe-app-role `
  --policy-name LeCafe-App-Permissions `
  --policy-document file://app-role-policy.json
```

✅ Inline policy `LeCafe-App-Permissions` attachée au role

---

### Step 13 — Simuler l'assumption du role (STS)

```powershell
aws --endpoint-url=http://localhost:4566 sts assume-role `
  --role-arn arn:aws:iam::000000000000:role/lecafe-app-role `
  --role-session-name ordering-app-session
```

**Résultat obtenu :**
```json
{
  "Credentials": {
    "AccessKeyId": "LSIAQAAAAAAAEHTHX7VF",
    "SecretAccessKey": "Zopy3fdDaquUkreYj9ALIzl5Tpl85cA8EvGX92jK",
    "SessionToken": "FQoGZXIvYXdzEBYaD43CMHostu...",
    "Expiration": "2026-05-21T09:16:39.714000+00:00"
  },
  "AssumedRoleUser": {
    "AssumedRoleId": "AROAQAAAAAAACO3L7TMYC:ordering-app-session",
    "Arn": "arn:aws:sts::000000000000:assumed-role/lecafe-app-role/ordering-app-session"
  }
}
```

✅ Credentials temporaires générés — expirent automatiquement après 1h

---

## 🔍 Part 5 — Inspection de la configuration IAM

### Step 14 — Créer les ressources S3 et SQS

```powershell
aws --endpoint-url=http://localhost:4566 s3 mb s3://lecafe-assets
aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name lecafe-orders
```

### Step 15 — Inspecter la configuration complète

```powershell
aws --endpoint-url=http://localhost:4566 iam get-role --role-name lecafe-app-role
aws --endpoint-url=http://localhost:4566 iam get-role-policy --role-name lecafe-app-role --policy-name LeCafe-App-Permissions
aws --endpoint-url=http://localhost:4566 iam list-groups-for-user --user-name alice
aws --endpoint-url=http://localhost:4566 iam list-attached-group-policies --group-name cafe-developers
aws --endpoint-url=http://localhost:4566 iam get-policy --policy-arn arn:aws:iam::000000000000:policy/LeCafe-Developer-S3
aws --endpoint-url=http://localhost:4566 iam get-policy-version --policy-arn arn:aws:iam::000000000000:policy/LeCafe-Developer-S3 --version-id v1
```

✅ Configuration IAM complète vérifiée

---

## 🧹 Cleanup

```powershell
# Détacher les policies
aws --endpoint-url=http://localhost:4566 iam detach-group-policy --group-name cafe-developers --policy-arn arn:aws:iam::000000000000:policy/LeCafe-Developer-S3
aws --endpoint-url=http://localhost:4566 iam detach-group-policy --group-name cafe-operations --policy-arn arn:aws:iam::000000000000:policy/LeCafe-Operations-ReadOnly

# Retirer les users des groupes
aws --endpoint-url=http://localhost:4566 iam remove-user-from-group --user-name alice --group-name cafe-developers
aws --endpoint-url=http://localhost:4566 iam remove-user-from-group --user-name bob --group-name cafe-developers
aws --endpoint-url=http://localhost:4566 iam remove-user-from-group --user-name charlie --group-name cafe-operations

# Supprimer groupes et users
aws --endpoint-url=http://localhost:4566 iam delete-group --group-name cafe-developers
aws --endpoint-url=http://localhost:4566 iam delete-group --group-name cafe-operations
aws --endpoint-url=http://localhost:4566 iam delete-user --user-name alice
aws --endpoint-url=http://localhost:4566 iam delete-user --user-name bob
aws --endpoint-url=http://localhost:4566 iam delete-user --user-name charlie

# Supprimer le role
aws --endpoint-url=http://localhost:4566 iam delete-role-policy --role-name lecafe-app-role --policy-name LeCafe-App-Permissions
aws --endpoint-url=http://localhost:4566 iam delete-role --role-name lecafe-app-role

# Supprimer les policies
aws --endpoint-url=http://localhost:4566 iam delete-policy --policy-arn arn:aws:iam::000000000000:policy/LeCafe-Developer-S3
aws --endpoint-url=http://localhost:4566 iam delete-policy --policy-arn arn:aws:iam::000000000000:policy/LeCafe-Operations-ReadOnly

# Arrêter LocalStack
docker stop localstack
```

✅ Toutes les ressources supprimées, LocalStack arrêté

---

## ⚠️ Problèmes rencontrés et solutions

| Problème | Cause | Solution |
|----------|-------|----------|
| `cat > /tmp/...` ne fonctionne pas | Syntaxe Linux/Mac | Utiliser `notepad fichier.json` sur Windows |
| `export AWS_PROFILE` ne fonctionne pas | Commande Linux | Utiliser `$env:AWS_PROFILE = "localstack"` |

---

## 🤔 Réponses aux questions de réflexion

**1. Attacher une policy directement à un user ?**  
Cela semble tentant pour un cas "exceptionnel" (ex: donner un accès temporaire à un consultant). Mais ça crée un problème de maintenance : ces permissions directes sont invisibles quand on audit les groupes. Avec le temps, les users accumulent des permissions oubliées qui violent le principe du moindre privilège. La bonne approche est de créer un groupe temporaire même pour un seul user.

**2. Changer le Principal de `ec2.amazonaws.com` à `lambda.amazonaws.com` ?**  
Le role ne pourrait plus être assumé par EC2 — seulement par Lambda. Cela montre que le trust policy est un contrôle d'accès à part entière : un role EC2 ne peut pas être utilisé par Lambda et vice versa. Chaque service AWS a son propre identifiant de service principal, ce qui permet de scoper précisément qui peut assumer un role.

**3. Valeur des credentials temporaires ?**  
Si des credentials temporaires sont volés, ils expirent automatiquement (1h par défaut) sans action humaine requise. Pour l'application, cela signifie qu'elle doit appeler `AssumeRole` régulièrement avant expiration — les AWS SDKs gèrent ce renouvellement automatiquement via le mécanisme de credential provider chain, ce qui simplifie le code applicatif.

---

## 📋 Référence rapide

| Concept | Commande |
|---------|----------|
| Créer user | `aws --endpoint-url=http://localhost:4566 iam create-user --user-name NAME` |
| Créer groupe | `aws --endpoint-url=http://localhost:4566 iam create-group --group-name NAME` |
| Ajouter user au groupe | `aws --endpoint-url=http://localhost:4566 iam add-user-to-group --user-name U --group-name G` |
| Créer policy | `aws --endpoint-url=http://localhost:4566 iam create-policy --policy-name N --policy-document file://PATH` |
| Attacher policy au groupe | `aws --endpoint-url=http://localhost:4566 iam attach-group-policy --group-name G --policy-arn ARN` |
| Créer role | `aws --endpoint-url=http://localhost:4566 iam create-role --role-name N --assume-role-policy-document file://PATH` |
| Attacher inline policy au role | `aws --endpoint-url=http://localhost:4566 iam put-role-policy --role-name N --policy-name N --policy-document file://PATH` |
| Assumer un role | `aws --endpoint-url=http://localhost:4566 sts assume-role --role-arn ARN --role-session-name NAME` |
| Account ID LocalStack | `000000000000` (toujours) |
