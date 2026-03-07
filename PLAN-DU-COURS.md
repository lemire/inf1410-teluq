# Module 1 — Le génie logiciel

## Introduction au génie logiciel (complété)
- Le problème fondamental : pourquoi le logiciel est difficile (rapport CHAOS, SAAQclic)
- Définition du GL : programmer vs faire du génie logiciel
- Complexité essentielle vs accidentelle (Fred Brooks, *No Silver Bullet*)
- Présentation de la structure du cours (les six modules)

## Histoire du GL (complété)

### Quelques livres et articles fameux référencés
- *The Mythical Man-Month* (Fred Brooks)
- *The Pragmatic Programmer* (Hunt, Thomas)
- *Clean Code* (Robert Martin)
- *Design Patterns* (Gang of Four)
- *No Silver Bullet* (Fred Brooks)
- *Programming as Theory Building* (Peter Naur)

### Les années 40 à mi-60 : l'ère pré-GL
- ENIAC et les premiers ordinateurs
- Premiers assembleurs et compilateurs
- FORTRAN, LISP, ALGOL et COBOL
- Premiers OS

### Les années 60 et 70 : la crise logicielle et la naissance du GL
- Conférence NATO (crise du logiciel)
- Programmation structurée (Dijkstra)
- Modèle en cascade (waterfall)
- Modèle relationnel (Edgar Codd)
- Modèle en spirale (Barry Boehm)
- *The Mythical Man-Month* et la loi de Brooks
- *No Silver Bullet*
- Unix et C

### Les années 80 : l'ère des abstractions, du PC et du GUI
- OOP et C++
- GUI et programmation événementielle (Macintosh, Windows)
- Standardisation de la stack de networking (TCP/IP)

### Les années 90 : la naissance du web
- Web (Tim Berners-Lee)
- JavaScript
- Java
- Python (Guido van Rossum)
- Open source, Linux (Linus Torvalds)
- Design patterns (Gang of Four)
- UML
- Extreme Programming (Kent Beck)
- *The Pragmatic Programmer* et le principe DRY

### Les années 00 : l'ère de l'agilité
- Manifeste Agile (Snowbird 2001)
- Scrum, Kanban
- Web 2.0
- AWS (2006)
- GitHub (2008)

### Les années 10 : l'ère du cloud et du DevOps
- CI/CD
- Microservices
- Docker, Kubernetes
- Serverless

### Les années 20 : l'ère de l'IA
- LLM, GitHub Copilot, ChatGPT, Claude Code

# Module 2 - Concevoir un logiciel correct

## Introduction (complété)
- *Programming as Theory Building* (Peter Naur) : programmer comme construction d'une théorie / d'un modèle mental
- Fil conducteur du module : les outils pour formaliser, vérifier et préserver cette théorie

## Survol rapide de la programmation (complété)
- Structures de données (listes, ensembles, dictionnaires)
- Complexité algorithmique (Big O, Two Sum, recherche binaire)
- Types : compilé vs interprété, statique vs dynamique, fort vs faible,
  type hints Python/mypy, lien avec les schémas JSON et SQL

## Les tests (complété)
- Pourquoi tester : confronter le modèle mental (Naur) à la réalité
- Tests comme substitut partiel au compilateur en Python
- Le mécanisme de base : `assert`
- Pyramide des tests : unitaires, d'intégration, end-to-end
- pytest comme framework de test
- TDD (Test-Driven Development) : cycle Red-Green-Refactor (Kent Beck)
- Coverage (coverage.py, pytest-cov) et ses limites
- Fixtures : `@pytest.fixture`, scopes, yield/teardown, `conftest.py`
- Mocking : `monkeypatch` (setattr, setenv/delenv), mention de `unittest.mock`
- Property-based testing (Hypothesis) et lien avec le fuzzing

## Le versioning avec git (complété)
- Problème fondamental du changement
- Histoire du versioning (SCCS → CVS → SVN → Git)
- Objets git (blobs, trees, commits), branches, merges, DAG

## La gestion des dépendances (complété)
- Problème de la réutilisation et de la décomposition
- Sécurité de la chaîne d'approvisionnement
- Versionnement sémantique (SemVer)
- uv comme outil concret (bibliothèques, applications, venv, lock files)

## L'intégration continue - CI (complété)
- Le problème : vérifications manuelles, "ça marche sur ma machine"
- Historique : Extreme Programming (Kent Beck), article de Martin Fowler (2006)
- Aparté sur YAML : syntaxe de base, pièges (indentation, types implicites)
- GitHub Actions : concepts (workflow, event, job, step, action, runner)
- Exemple concret : projet Python avec pytest, workflow CI, démo sur GitHub
- Au-delà des tests : linting (ruff), formatage, type checking (mypy), sécurité des dépendances
- Lien CI → CD (déploiement continu), renvoi au module 5

# Module 3 - Du programme au système

## Introduction : la complexité comme problème central (à développer)
- Le passage du programme au système
- Complexité essentielle vs accidentelle (Brooks, *No Silver Bullet*)
- La gestion de la complexité comme fil conducteur
- Les principes fameux comme heuristiques (introduction, avec renvoi vers
  la page de référence transversale `content/docs/principes.md`)
- Les principes sont développés en profondeur dans les sections où ils
  sont le plus pertinents (dans ce module et les suivants)

NOTE : revisiter les modules 1 et 2 pour y intégrer des références aux
principes pertinents (ex: DRY dans la gestion des dépendances, etc.)

## Architecture et modularité (à développer)
- Parnas et le papier fondateur sur la modularité (1972)
- Couplage et cohésion
- Couches (layers) et séparation des responsabilités
- Monolithe vs microservices
- Patterns architecturaux (event-driven, pipes and filters, etc.)

## Les APIs (à développer)
- L’API comme interface entre composants
- REST, GraphQL, gRPC
- Design d’API
- Lien avec les schémas (JSON Schema, OpenAPI, Protobuf)

## Les données (en cours)

### La représentation des données (complété)
- Données vs information vs connaissances (pyramide DIKW)
- Encodage et sérialisation (ASCII, UTF-8, CSV, XML, JSON, Protobuf)
- La notion de schéma (SQL DDL, type hints, OpenAPI, JSON Schema)
- Évolution des schémas (compatibilité avant/arrière)

### Le stockage des données (en cours)
- Modèle hiérarchique (IBM, années 60)
- Modèle en réseau (CODASYL)
- Modèle relationnel et SQL
- Transactions (à développer)
- ORMs (à développer)
- Révolution NoSQL (à développer)
- Bases de données de séries temporelles (à développer)
- Bases de données vectorielles (à développer)

### Les autres paradigmes de données (complété)
- Logging, Messaging, Streaming, Caching
- Recherche et indexation, Object storage, Analytics/OLAP
- Séries temporelles, Configuration/coordination
- Données collaboratives/répliquées (CRDTs), Ledgers/blockchain

# Module 4 - Construire en équipe

- Git distribué (github)
- Agile
- Kanban
- Scrum
- Communication (importance d’outils comme Slack, etc).
- Estimation
- Coordination
- Dette technique (dimension sociale et organisationnelle)

# Module 5 - Faire vivre le logiciel

- Cloud
- DevOps
- Observabilité
- Monitoring
- Logging
- Sécurité
- Scalabilité

# Module 6 - Au-delà du logiciel

- Open source
- AI-assisted dev
- Économie du logiciel
- Plateformes (iOS, Android, etc)
