# RAG help desk — Orchestration / Infrastructure

## Présentation du projet

RAG Incident Desk est une plateforme d’assistance documentaire basée sur l’IA.

Le système permet à des utilisateurs de poser des questions techniques à partir d’une documentation interne.

Le moteur RAG :

1. recherche les passages les plus pertinents dans les documents,
2. enrichit le prompt utilisateur,
3. génère une réponse contextualisée grâce à un LLM.

Le projet répond au besoin suivant :

> Réduire le temps de recherche d’informations techniques dans une documentation interne volumineuse.

---

## Utilisateurs cibles

### Support technique

Recherche rapide de procédures.

### Equipes IT

Consultation documentaire contextualisée.

### Administrateurs

Gestion de la base documentaire.

---

## Architecture globale

```text
Angular Frontend
       ↓
Spring Boot Backend
       ↓
PostgreSQL + pgvector
       ↓
Ollama LLM
```

---

## Stack technique globale

| Domaine               | Technologie           |
| --------------------- | --------------------- |
| Frontend              | Angular               |
| Backend               | Spring Boot           |
| Sécurité              | Spring Security + JWT |
| IA                    | Ollama                |
| Embeddings            | nomic-embed-text      |
| Chat Model            | llama3.2:1b           |
| Base de données       | PostgreSQL            |
| Recherche vectorielle | pgvector              |
| Conteneurisation      | Docker                |
| Orchestration         | Docker Compose        |
| CI/CD                 | GitHub Actions        |

---

## Lancement du projet

### Structure attendue

```text
rag-system/
├── rag-backend/
├── rag-frontend/
└── rag-orchestration/
```

---

## Configuration

Créer un fichier :

```text
.env
```

à partir de :

```text
.env.example
```

---

## Démarrage

Depuis le dossier orchestration :

```bash
docker compose up --build
```

Le système démarre automatiquement :

- PostgreSQL,
- pgvector,
- Ollama,
- backend Spring Boot,
- frontend Angular.

---

## APIs documentées

### Authentification

| Méthode | Endpoint           |
| ------- | ------------------ |
| POST    | /api/auth/register |
| POST    | /api/auth/login    |

### RAG

| Méthode | Endpoint     |
| ------- | ------------ |
| POST    | /api/rag/ask |

### Ingestion

| Méthode | Endpoint                    |
| ------- | --------------------------- |
| POST    | /api/admin/documents/upload |

---

# 4. ADR — Choix techniques

# ADR 001 — PostgreSQL + pgvector vs NoSQL vectoriel

## Contexte

Le projet nécessite :

- stockage documentaire,
- persistance des utilisateurs,
- gestion RBAC,
- recherche vectorielle.

---

## Décision

Le projet utilise PostgreSQL avec l’extension pgvector.

---

## Raisons

### Avantages

- intégration Spring Data JPA,
- stockage vectoriel natif,
- simplification de l’architecture,
- réduction du nombre de technologies.

### Trade-offs

- couplage relationnel/vectoriel.

---

## Alternatives considérées

### MongoDB Atlas Vector Search

Avantages :

- NoSQL flexible,
- vector search intégré.

Inconvénients :

- architecture plus complexe,
- moins adapté aux besoins relationnels du projet.

### Pinecone / Weaviate

Avantages :

- spécialisés IA/vector search.

Inconvénients :

- dépendance SaaS,
- coût potentiel,
- complexité supplémentaire.

---

# ADR 002 — Ollama vs OpenAI

## Décision

Le projet utilise Ollama localement.

---

## Raisons

### Avantages

- exécution locale,
- absence de coût API,
- confidentialité des données,
- fonctionnement offline,
- reproductibilité Docker.

### Trade-offs

- modèles plus petits,
- performances parfois inférieures,
- consommation mémoire locale.
- l'image docker reste lourde et pendant le premier lancement le pull peut durer longtemps

---

## Alternative : OpenAI

### Avantages

- meilleures performances,
- modèles plus puissants,
- simplicité d’intégration.

### Inconvénients

- coût d’utilisation,
- dépendance externe

---

# 5. Règles métier

## BR-001 — Seuls les administrateurs peuvent ingérer des documents

L’upload documentaire est réservé aux utilisateurs possédant le rôle ADMIN.

---

## BR-002 — Une question ne peut être posée que par un utilisateur authentifié

Le moteur RAG nécessite un JWT valide.

---

## BR-003 — Les réponses doivent être générées uniquement à partir de documents indexés

Le système ne doit pas répondre sans récupération préalable de chunks pertinents.

---

# 6. RBAC — Contrôle d’accès

## Objectif

Limiter les fonctionnalités selon le rôle utilisateur.

---

## Rôles

### USER

Peut :

- se connecter,
- poser des questions,
- consulter les réponses.

---

### ADMIN

Peut :

- ingérer des documents,
- administrer la base documentaire,
- accéder au dashboard d’administration.

---

## Implémentation backend

Spring Security :

```java
.requestMatchers("/api/admin/documents/**").hasRole("ADMIN")
.requestMatchers("/api/rag/**").hasAnyRole("USER", "ADMIN")
```

---

## Implémentation frontend

Angular Guards :

- redirection selon le rôle,
- protection des routes
