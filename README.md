# 🛒 Projet Boutique — Architecture Microservices Spring Boot 4

> Mini-plateforme e-commerce construite avec une architecture microservices : deux services métier, un serveur Eureka, une API Gateway, et une application mobile Flutter/React Native. Orchestration complète via Docker Compose.

---

## 📁 Structure du dépôt

```
/projet-boutique/
├── produits-service/           # Microservice gestion des produits (port 8091)
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/boutique/produits/
│   │   │   │   ├── controller/
│   │   │   │   │   ├── ProduitController.java
│   │   │   │   │   └── CategorieController.java
│   │   │   │   ├── service/
│   │   │   │   │   ├── ProduitService.java
│   │   │   │   │   └── CategorieService.java
│   │   │   │   ├── repository/
│   │   │   │   │   ├── ProduitRepository.java
│   │   │   │   │   └── CategorieRepository.java
│   │   │   │   └── entity/
│   │   │   │       ├── Produit.java
│   │   │   │       └── Categorie.java
│   │   │   └── resources/
│   │   │       ├── application.properties
│   │   │       └── data.sql
│   │   └── test/
│   │       └── java/com/boutique/produits/
│   │           ├── service/ProduitServiceTest.java       # Tests unitaires Mockito
│   │           └── repository/ProduitRepositoryTest.java # Tests intégration @DataJpaTest
│   ├── Dockerfile
│   └── pom.xml
│
├── avis-service/               # Microservice gestion des avis (port 8092)
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/boutique/avis/
│   │   │   │   ├── controller/AvisController.java
│   │   │   │   ├── service/AvisService.java
│   │   │   │   ├── repository/AvisRepository.java
│   │   │   │   ├── entity/Avis.java
│   │   │   │   └── client/ProduitClient.java            # Feign client
│   │   │   └── resources/
│   │   │       └── application.properties
│   │   └── test/
│   ├── Dockerfile
│   └── pom.xml
│
├── eureka-server/              # Serveur de découverte (port 8761)
│   ├── src/main/java/com/boutique/eureka/
│   │   └── EurekaServerApplication.java
│   ├── src/main/resources/application.properties
│   ├── Dockerfile
│   └── pom.xml
│
├── api-gateway/                # API Gateway Spring Cloud (port 8090)
│   ├── src/main/java/com/boutique/gateway/
│   │   └── ApiGatewayApplication.java
│   ├── src/main/resources/application.yml
│   ├── Dockerfile
│   └── pom.xml
│
├── mobile-app/                 # Application mobile (Flutter ou React Native)
│   ├── lib/                    # Flutter
│   │   ├── main.dart
│   │   ├── screens/
│   │   │   ├── categories_screen.dart
│   │   │   ├── produits_screen.dart
│   │   │   └── avis_screen.dart
│   │   └── services/api_service.dart
│   └── pubspec.yaml
│
├── cypress/                    # Tests E2E (branche version2)
│   └── e2e/boutique.cy.js
│
├── docker-compose.yml          # Orchestration complète
└── README.md
```

---

## 🌿 Branches Git

| Branche | Contenu |
|---------|---------|
| `version1` | Parties 1 à 4 : microservices, Eureka, Gateway, Docker Compose |
| `version2` | Parties 5 à 6 : application mobile + tests unitaires, intégration et E2E |

---

## ⚙️ Configuration Java 25 — Résolution des erreurs Maven

> ⚠️ Java 25 est une version **preview** — des configurations spéciales sont nécessaires pour que Maven et Spring Boot l'acceptent.

### Erreurs courantes et solutions

#### ❌ `release version 25 not supported`
Le plugin `maven-compiler-plugin` est trop ancien. **Solution** : utiliser la version `3.13.0` avec `--enable-preview`.

#### ❌ `Unsupported class file major version 69`
Le plugin `spring-boot-maven-plugin` est trop ancien (version 3.2.x). Java 25 = class file version 69. **Solution** : monter Spring Boot à `3.4.5`.

### Versions compatibles Java 25

| Composant | Version requise |
|-----------|----------------|
| Spring Boot Parent | **3.4.5** |
| Spring Cloud | **2024.0.1** |
| maven-compiler-plugin | **3.13.0** |
| spring-boot-maven-plugin | **3.4.5** |
| JDK Docker image | `eclipse-temurin:24-jdk-alpine` *(JDK 25 indisponible sur Docker Hub)* |

### Configuration `pom.xml` correcte pour Java 25

Dans **chaque** `pom.xml` des microservices, la section `<parent>` doit être :

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.5</version>
</parent>
```

La section `<properties>` :

```xml
<properties>
    <java.version>25</java.version>
    <spring-cloud.version>2024.0.1</spring-cloud.version>
</properties>
```

La section `<build>` complète :

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.13.0</version>
            <configuration>
                <release>25</release>
                <compilerArgs>
                    <arg>--enable-preview</arg>
                </compilerArgs>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>3.4.5</version>
            <configuration>
                <jvmArguments>--enable-preview</jvmArguments>
                <excludes>
                    <exclude>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Script PowerShell — Mise à jour automatique de tous les pom.xml

```powershell
$base = "C:\Users\cworo\IdeaProjects\projet-boutique"

Get-ChildItem -Path $base -Name "pom.xml" -Recurse | ForEach-Object {
    $full = Join-Path $base $_
    $content = Get-Content $full -Raw
    $content = $content -replace '<java\.version>\d+</java\.version>', '<java.version>25</java.version>'
    $content = $content -replace '<spring-cloud\.version>[^<]+</spring-cloud\.version>', '<spring-cloud.version>2024.0.1</spring-cloud.version>'
    Set-Content $full $content -Encoding UTF8
    Write-Host "✅ $_ mis à jour" -ForegroundColor Green
}
```

### Dockerfiles — utiliser JDK 24 (le plus récent sur Docker Hub)

```dockerfile
FROM eclipse-temurin:24-jdk-alpine AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN apk add --no-cache maven && mvn clean package -DskipTests

FROM eclipse-temurin:24-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8091
ENTRYPOINT ["java", "--enable-preview", "-jar", "app.jar"]
```

### Script PowerShell — Mise à jour automatique de tous les Dockerfiles

```powershell
$base = "C:\Users\cworo\IdeaProjects\projet-boutique"

Get-ChildItem -Path $base -Name "Dockerfile" -Recurse | ForEach-Object {
    $full = Join-Path $base $_
    $content = Get-Content $full -Raw
    $content = $content -replace 'eclipse-temurin:21-jdk-alpine', 'eclipse-temurin:24-jdk-alpine'
    $content = $content -replace 'eclipse-temurin:21-jre-alpine', 'eclipse-temurin:24-jre-alpine'
    $content = $content -replace 'ENTRYPOINT \["java", "-jar"', 'ENTRYPOINT ["java", "--enable-preview", "-jar"'
    Set-Content $full $content -Encoding UTF8
    Write-Host "✅ $_ corrigé" -ForegroundColor Green
}
```

### Ajouter le pom.xml parent (pour que IntelliJ voit tous les modules)

Créer `pom.xml` à la racine du projet :

```powershell
$content = '<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.boutique</groupId>
    <artifactId>projet-boutique</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    <name>projet-boutique</name>
    <modules>
        <module>eureka-server</module>
        <module>api-gateway</module>
        <module>produits-service</module>
        <module>avis-service</module>
    </modules>
</project>'

[System.IO.File]::WriteAllText("C:\chemin\vers\projet-boutique\pom.xml", $content, [System.Text.Encoding]::UTF8)
```

Puis dans IntelliJ : **File → Open** → sélectionner ce `pom.xml` → **Open as Project**.

---

## 🛠️ Prérequis

Avant de lancer le projet, assurez-vous d'avoir installé :

| Outil | Version minimale | Vérification |
|-------|-----------------|--------------|
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.20+ | `docker compose version` |
| Java / JDK | 25 | `java --version` |
| Maven | 3.9+ | `mvn --version` |
| Flutter *(mobile)* | 3.x | `flutter --version` |
| Node.js *(tests E2E)* | 18+ | `node --version` |
| Cypress *(tests E2E)* | 13+ | `npx cypress --version` |

---

## 🚀 Lancement du projet

### Option 1 — Démarrage complet via Docker Compose *(recommandé)*

```bash
# 1. Cloner le dépôt
git clone https://github.com/<votre-username>/projet-boutique.git
cd projet-boutique

# 2. Basculer sur la branche souhaitée
git checkout version1   # ou version2

# 3. Construire et démarrer tous les services
docker compose up --build -d

# 4. Vérifier que tous les conteneurs sont actifs
docker compose ps
```

> ⏳ **Patience** : le démarrage complet peut prendre 2–3 minutes, le temps qu'Eureka soit prêt et que les microservices s'y enregistrent.

**Vérification rapide :**

```bash
# Eureka Dashboard
curl http://localhost:8761

# API Gateway — liste des produits
curl http://localhost:8090/api/produits

# API Gateway — liste des catégories
curl http://localhost:8090/api/categories

# API Gateway — avis du produit 1
curl http://localhost:8090/api/avis/1
```

### Option 2 — Démarrage en mode développement (sans Docker)

> Requiert PostgreSQL et Redis en local ou via Docker séparément.

```bash
# Démarrer les dépendances uniquement
docker compose up -d postgres redis

# eureka-server
cd eureka-server
mvn spring-boot:run

# produits-service (dans un nouveau terminal)
cd produits-service
mvn spring-boot:run

# avis-service (dans un nouveau terminal)
cd avis-service
mvn spring-boot:run

# api-gateway (dans un nouveau terminal)
cd api-gateway
mvn spring-boot:run
```

---

## 🌐 URLs et points d'accès

| Service | URL | Description |
|---------|-----|-------------|
| API Gateway | `http://localhost:8090` | Point d'entrée unique |
| produits-service | `http://localhost:8091` | Accès direct (dev) |
| avis-service | `http://localhost:8092` | Accès direct (dev) |
| Eureka Dashboard | `http://localhost:8761` | Tableau de bord de découverte |
| Swagger produits | `http://localhost:8091/swagger-ui.html` | Documentation API produits |
| Swagger avis | `http://localhost:8092/swagger-ui.html` | Documentation API avis |

---

## 📡 Référence des API

### produits-service (via Gateway : `http://localhost:8090`)

| Méthode | Endpoint | Description | Cache Redis |
|---------|----------|-------------|-------------|
| `GET` | `/api/produits` | Liste tous les produits | ✅ `@Cacheable` |
| `GET` | `/api/produits?categorieId={id}` | Produits d'une catégorie | — |
| `GET` | `/api/produits/{id}` | Détail d'un produit | — |
| `POST` | `/api/produits` | Créer un produit | ✅ `@CacheEvict` |
| `GET` | `/api/categories` | Liste toutes les catégories | — |
| `GET` | `/api/categories/{id}` | Détail d'une catégorie | — |

**Exemple — Créer un produit :**
```bash
curl -X POST http://localhost:8090/api/produits \
  -H "Content-Type: application/json" \
  -d '{
    "nom": "Laptop Pro",
    "prix": 1299.99,
    "stock": 10,
    "categorie": { "id": 1 }
  }'
```

### avis-service (via Gateway : `http://localhost:8090`)

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| `GET` | `/api/avis/{produitId}` | Liste les avis d'un produit |
| `POST` | `/api/avis` | Soumettre un avis (note 1–5) |

**Exemple — Soumettre un avis :**
```bash
curl -X POST http://localhost:8090/api/avis \
  -H "Content-Type: application/json" \
  -d '{
    "produitId": 1,
    "auteur": "Alice",
    "commentaire": "Excellent produit !",
    "note": 5
  }'
```

> ⚠️ Si `produitId` n'existe pas dans `produits-service`, `avis-service` retourne une erreur `404 Not Found` via le client Feign.

---

## 🏗️ Architecture technique

```
                        ┌─────────────────┐
                        │   API Gateway   │  :8090
                        │ Spring Cloud GW │
                        └────────┬────────┘
                                 │ Route selon le path
              ┌──────────────────┼──────────────────┐
              │                  │                  │
    /api/produits/**    /api/categories/**    /api/avis/**
              │                  │                  │
    ┌─────────▼──────────────────▼┐    ┌────────────▼─────────┐
    │      produits-service        │    │      avis-service     │
    │          :8091               │    │         :8092         │
    │  Controller / Service /      │    │  Controller / Service │
    │  Repository / Entity         │◄───┤  Feign Client         │
    │  Cache Redis (@Cacheable)    │    │  Repository / Entity  │
    └──────────────┬───────────────┘    └──────────┬────────────┘
                   │                               │
         ┌─────────▼─────┐                ┌────────▼──────┐
         │  PostgreSQL DB │                │ PostgreSQL DB │
         │  (produits)    │                │   (avis)      │
         └───────────────┘                └───────────────┘
                   │                               │
         ┌─────────▼─────┐
         │  Redis Cache   │
         └───────────────┘

                    Tous enregistrés dans :
                   ┌─────────────────┐
                   │  Eureka Server  │  :8761
                   │  @EnableEureka  │
                   └─────────────────┘
```

### Flux Feign (avis-service → produits-service)

Avant d'enregistrer un avis, `avis-service` vérifie l'existence du produit via un `@FeignClient` :

```
POST /api/avis
    └─► AvisService.creerAvis()
            └─► ProduitClient.getProduit(produitId)   ← Feign → produits-service
                    ├─ 200 OK  → avis enregistré ✅
                    └─ 404     → exception levée → 404 retourné au client ❌
```

---

## 🗄️ Données initiales (`data.sql`)

Au démarrage de `produits-service`, les données suivantes sont insérées automatiquement :

**3 Catégories :**

| ID | Nom |
|----|-----|
| 1 | Informatique |
| 2 | Électroménager |
| 3 | Mode |

**5 Produits :**

| ID | Nom | Prix | Stock | Catégorie |
|----|-----|------|-------|-----------|
| 1 | Laptop Pro | 1299.99 | 15 | Informatique |
| 2 | Souris Sans Fil | 29.99 | 100 | Informatique |
| 3 | Réfrigérateur XL | 799.00 | 8 | Électroménager |
| 4 | Lave-linge 8kg | 499.00 | 12 | Électroménager |
| 5 | Veste en Cuir | 189.90 | 25 | Mode |

---

## 🐳 Docker Compose — Détail des services

```yaml
# Extrait illustratif du docker-compose.yml
services:
  postgres:
    image: postgres:16
    ports: ["5432:5432"]

  redis:
    image: redis:7
    ports: ["6379:6379"]

  eureka-server:
    build: ./eureka-server
    ports: ["8761:8761"]

  produits-service:
    build: ./produits-service
    ports: ["8091:8091"]
    depends_on: [postgres, redis, eureka-server]

  avis-service:
    build: ./avis-service
    ports: ["8092:8092"]
    depends_on: [postgres, eureka-server]

  api-gateway:
    build: ./api-gateway
    ports: ["8090:8090"]
    depends_on: [eureka-server]
```

**Arrêter et nettoyer :**
```bash
# Arrêter tous les services
docker compose down

# Arrêter ET supprimer les volumes (reset BDD)
docker compose down -v

# Voir les logs d'un service
docker compose logs -f produits-service
```

---

## 📱 Application Mobile

L'application mobile se connecte **exclusivement** à l'API Gateway (`http://<IP_MACHINE>:8090`).

### Parcours utilisateur

```
Écran 1 — Catégories
  GET /api/categories
  └─► Sélection d'une catégorie
        │
        ▼
Écran 2 — Produits de la catégorie
  GET /api/produits?categorieId={id}
  └─► Clic sur un produit
        │
        ▼
Écran 3 — Avis du produit
  GET /api/avis/{produitId}
```

### Lancer l'application Flutter

```bash
cd mobile-app

# Installer les dépendances
flutter pub get

# Configurer l'IP de l'API Gateway
# Dans lib/services/api_service.dart, modifier :
# const String baseUrl = 'http://<VOTRE_IP>:8090';

# Lancer sur émulateur ou appareil physique
flutter run

# Build APK release
flutter build apk --release
```

> 💡 Sur Android Emulator, utilisez `http://10.0.2.2:8090` à la place de `localhost`.

### Lancer l'application React Native *(alternative)*

```bash
cd mobile-app

# Installer les dépendances
npm install

# iOS
npx react-native run-ios

# Android
npx react-native run-android
```

---

## 🧪 Exécuter les tests (branche `version2`)

```bash
git checkout version2
```

### 1. Tests unitaires — Mockito (`ProduitServiceTest`)

Testent la logique métier du service en isolation (sans Spring context, sans BDD).

```bash
cd produits-service
mvn test -Dtest=ProduitServiceTest
```

**Ce qui est testé :**
- `getAllProduits()` → retourne la liste mockée
- `getProduitById(id)` → retourne le bon produit / lève une exception si introuvable
- `createProduit(dto)` → appelle bien `repository.save()`
- Vérification des interactions Mockito (`verify`, `times`)

### 2. Tests d'intégration — `@DataJpaTest` (`ProduitRepositoryTest`)

Testent le repository avec une base H2 en mémoire (ou Testcontainers PostgreSQL).

```bash
cd produits-service
mvn test -Dtest=ProduitRepositoryTest
```

**Ce qui est testé :**
- `findAll()` → retourne les bons enregistrements
- `findByCategorieId(id)` → filtre correctement par catégorie
- `save()` / `findById()` → persistance et récupération
- Contraintes de validation (stock négatif, prix nul, etc.)

### 3. Lancer tous les tests Spring en une commande

```bash
cd produits-service
mvn verify
```

### 4. Tests E2E — Cypress

Simulent le parcours complet depuis la liste des produits jusqu'aux avis, **via l'API Gateway** avec `cy.request()`.

```bash
# Prérequis : tous les services Docker doivent être en cours d'exécution
docker compose up -d

# Installer Cypress
cd cypress
npm install

# Lancer en mode headless (CI)
npx cypress run

# Lancer avec l'interface graphique
npx cypress open
```

**Scénarios couverts (`cypress/e2e/boutique.cy.js`) :**

```
✅ GET /api/produits        → status 200, tableau non vide
✅ GET /api/produits/{id}   → status 200, champs attendus présents
✅ GET /api/avis/{produitId}→ status 200, tableau (peut être vide)
✅ POST /api/avis            → status 201, avis créé avec bonne note
✅ POST /api/avis (produit inexistant) → status 404
```

---

## 🔍 Vérification du cache Redis

```bash
# Ouvrir un shell Redis dans le conteneur
docker exec -it redis redis-cli

# Lister les clés de cache
KEYS *

# Inspecter le contenu d'une clé
GET produits::SimpleKey []

# Vider le cache manuellement
FLUSHALL
```

**Comportement attendu :**
- Après `GET /api/produits` : une clé apparaît dans Redis
- Après `POST /api/produits` : la clé est supprimée (`@CacheEvict`)
- Second `GET /api/produits` : recréation de la clé

---

## ⚙️ Variables d'environnement

Chaque service peut être configuré via variables d'environnement (ou `application.properties`) :

### produits-service

| Variable | Valeur par défaut | Description |
|----------|-------------------|-------------|
| `SPRING_DATASOURCE_URL` | `jdbc:postgresql://postgres:5432/produits_db` | URL PostgreSQL |
| `SPRING_DATASOURCE_USERNAME` | `postgres` | Utilisateur BDD |
| `SPRING_DATASOURCE_PASSWORD` | `postgres` | Mot de passe BDD |
| `SPRING_REDIS_HOST` | `redis` | Hôte Redis |
| `SPRING_REDIS_PORT` | `6379` | Port Redis |
| `EUREKA_CLIENT_SERVICEURL_DEFAULTZONE` | `http://eureka-server:8761/eureka` | URL Eureka |

### avis-service

| Variable | Valeur par défaut | Description |
|----------|-------------------|-------------|
| `SPRING_DATASOURCE_URL` | `jdbc:postgresql://postgres:5432/avis_db` | URL PostgreSQL |
| `SPRING_DATASOURCE_USERNAME` | `postgres` | Utilisateur BDD |
| `SPRING_DATASOURCE_PASSWORD` | `postgres` | Mot de passe BDD |
| `EUREKA_CLIENT_SERVICEURL_DEFAULTZONE` | `http://eureka-server:8761/eureka` | URL Eureka |

---

## ❗ Dépannage

### Les microservices ne s'enregistrent pas dans Eureka

```bash
# Vérifier que eureka-server est bien démarré
docker compose logs eureka-server

# Attendre 30–60 secondes après le démarrage d'Eureka
# puis vérifier : http://localhost:8761
```

### Erreur de connexion à la base de données

```bash
# Vérifier que PostgreSQL est prêt
docker compose logs postgres

# Tester la connexion manuellement
docker exec -it postgres psql -U postgres -c "\l"
```

### Le cache Redis ne fonctionne pas

```bash
# Vérifier que Redis est accessible
docker compose logs redis
docker exec -it redis redis-cli ping   # doit répondre PONG
```

### L'API Gateway retourne 503

```bash
# Vérifier que les services sont bien enregistrés dans Eureka
curl http://localhost:8761/eureka/apps

# Vérifier les routes configurées
docker compose logs api-gateway
```

### Problème de CORS sur l'application mobile

Si l'application mobile rencontre des erreurs CORS, assurez-vous que l'API Gateway autorise les origines cross-domain en ajoutant dans `application.yml` :

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins: "*"
            allowedMethods: "*"
            allowedHeaders: "*"
```

---

## 📦 Stack technique

| Couche | Technologie | Version |
|--------|-------------|---------|
| Backend | Spring Boot | 4.x |
| ORM | Spring Data JPA + Hibernate | — |
| BDD | PostgreSQL | 16 |
| Cache | Redis + Spring Cache | 7 |
| Discovery | Netflix Eureka | Spring Cloud |
| Communication | OpenFeign | Spring Cloud |
| Gateway | Spring Cloud Gateway | — |
| Documentation | springdoc-openapi (Swagger UI) | — |
| Conteneurisation | Docker + Docker Compose | 24+ |
| Mobile | Flutter (Dart) ou React Native | 3.x / 0.73+ |
| Tests unitaires | JUnit 5 + Mockito | — |
| Tests intégration | @DataJpaTest + H2 / Testcontainers | — |
| Tests E2E | Cypress | 13+ |
| Build | Maven | 3.9+ |
| JDK | Java | 25 |

---

## 👤 Auteur

Projet réalisé dans le cadre du test pratique **Architecture Microservices Spring Boot 4**.

**Formateur :** Wahid Hamdi

---

*Pour toute question, ouvrez une issue sur le dépôt GitHub.*
