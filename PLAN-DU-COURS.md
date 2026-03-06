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

## Introduction (à développer)
- *Programming as Theory Building* (Peter Naur) : programmer comme construction d'une théorie / d'un modèle mental
- Fil conducteur du module : les outils pour formaliser, vérifier et préserver cette théorie

## Survol rapide de la programmation (complété)
- Structures de données (listes, ensembles, dictionnaires)
- Complexité algorithmique (Big O, Two Sum, recherche binaire)
- Types : compilé vs interprété, statique vs dynamique, fort vs faible,
  type hints Python/mypy, lien avec les schémas JSON et SQL

## Les tests (à développer)
- Pourquoi tester (coût des bugs, confiance dans le code)
- Pyramide des tests : unitaires, d'intégration, end-to-end
- pytest comme outil concret
- TDD (Test-Driven Development)
- Coverage et ses limites
- Property-based testing (hypothesis)

## Le versioning avec git (complété)
- Problème fondamental du changement
- Histoire du versioning (SCCS → CVS → SVN → Git)
- Objets git (blobs, trees, commits), branches, merges, DAG

## La gestion des dépendances (complété)
- Problème de la réutilisation et de la décomposition
- Sécurité de la chaîne d'approvisionnement
- Versionnement sémantique (SemVer)
- uv comme outil concret (bibliothèques, applications, venv, lock files)

## L'intégration continue - CI (à développer)
- Concept de CI : intégrer souvent, automatiser la vérification
- GitHub Actions comme outil concret
- Exemple de workflow (tests automatiques sur push)
- Lien tests + CI comme filet de sécurité

# Module 3 - Évolution du logiciel

## Principes importants et fameux
- Principe DRY
- KISS
- YAGNI
- Principes SOLID
- Separation of Concerns
- Law of Demeter
- Composition over inheritance
- Boy scout rule
  > “Always leave the campground cleaner than you found it.”
- Principle of Least Astonishment (POLA)
  - Un composant doit se comporter comme on s’y attend
  - Une fonction `calculateTotal()` ne devrait pas aussi envoyer un courriel

- Architecture
- Modularité
- API
- Données
- Couplage
- Patterns
- Dette technique

# Module 4 - Construire en équipe

- Git distribué (github)
- Agile
- Kanban
- Scrum
- Communication (importance d'outils comme Slack, etc).
- Estimation
- Coordination

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
