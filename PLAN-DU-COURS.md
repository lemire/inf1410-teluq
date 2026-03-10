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

## Introduction : la complexité comme problème central (complété)
- Le passage du programme au système
- Complexité essentielle vs accidentelle (Brooks, *No Silver Bullet*)
- La gestion de la complexité comme fil conducteur
- Les principes fameux comme heuristiques (introduction, avec renvoi vers
  la page de référence transversale `content/docs/principes.md`)
- Les principes sont développés en profondeur dans les sections où ils
  sont le plus pertinents (dans ce module et les suivants)

NOTE : revisiter les modules 1 et 2 pour y intégrer des références aux
principes pertinents (ex: DRY dans la gestion des dépendances, etc.)

## Architecture et modularité (complété)
- Introduction : pourquoi découper ?
- Parnas et l'information hiding (1972)
  - Exemple KWIC (Key Word In Context) reproduit en Python
  - Deux décompositions comparées : par flux vs par information hiding
  - Principe : Separation of Concerns
- Couplage et cohésion
  - Définitions avec exemples Python
  - Lien avec DRY : la duplication comme symptôme de mauvais découpage
- SOLID (principes de conception OO)
  - Focus sur S (Single Responsibility) et D (Dependency Inversion)
  - Les autres (O, L, I) traités plus brièvement
  - Perspective critique : lien avec YAGNI
- Composition over inheritance
  - Exemple Python : héritage vs composition
  - Lien avec le principe d'inversion des dépendances
- Les couches (layers)
  - Pattern présentation → logique → données
  - Exemple concret en Python
- Monolithe vs microservices
  - Le monolithe comme point de départ raisonnable
  - Quand et pourquoi découper
  - Loi de Conway (pont vers module 4)
- Patterns architecturaux
  - Client-server
  - MVC (Reenskaug, 1979) et frameworks web (Rails, Django, Flask, etc.)
  - Pipes and filters (philosophie Unix, lien avec KWIC)
  - Architecture événementielle (event-driven, Kafka, RabbitMQ)

## Les APIs (complété)

### L’API comme concept évolutif
- L’API au sens originel : l’interface d’une bibliothèque (fonctions qu’on appelle dans son propre code)
- Le virage réseau : l’API comme contrat entre systèmes distribués (introduction du réseau, latence, pannes, sérialisation)
- L’API comme produit : Stripe, Twilio, etc. Documentation, versioning, SLAs. L’économie de l’API.

### Les paradigmes d’APIs réseau
- RPC (Remote Procedure Call) : faire semblant que l’appel réseau est un appel de fonction
  - CORBA, XML-RPC, SOAP
  - Les fallacies of distributed computing (Peter Deutsch, 1994) : pourquoi l’illusion du RPC transparent est dangereuse
- REST (Roy Fielding, thèse de doctorat, 2000) : rupture conceptuelle, manipulation de ressources
  - HTTP comme protocole applicatif (verbes, codes de statut, headers)
  - Richardson Maturity Model (niveaux 0-3)
  - HATEOAS : la vision originale de Fielding vs la pratique (billet de 2008)
  - Exemple concret : API FastAPI avec curl
- GraphQL (Facebook, 2015) : le client décide de la forme des données
  - Problème du over-fetching/under-fetching
  - Comparaison SQL vs GraphQL (niveaux différents de l'architecture)
  - Schéma introspectable comme alternative à HATEOAS pour la découvrabilité
  - Exemple concret : serveur Strawberry avec curl
- gRPC (Google, 2015) : retour au RPC avec les leçons apprises (Protobuf, HTTP/2, streaming)
- Webhooks : l’inversion du modèle requête-réponse (lien avec l’architecture événementielle)

### Design d’API et schémas
- Principes de design : nommage, versioning, gestion des erreurs, idempotence
- Schémas comme contrat : OpenAPI/Swagger (REST), fichiers .proto (gRPC), schéma introspectable (GraphQL)
- Lien avec DRY : le schéma comme source unique de vérité
- Lien avec la section sur les schémas (module 3, données)

## Les interfaces utilisateur (en cours)

### Perspective historique (complété)
- Les terminaux texte et les interfaces en ligne de commande (CLI)
  - Teletypes (TTY), terminaux vidéo (VT100), shells (Bourne Shell, Bash, Zsh, PowerShell)
  - Puissance de composition de la CLI, lien avec pipes and filters (Unix)
- Xerox PARC, Smalltalk et la naissance du GUI (années 70-80)
  - Alto (1973), métaphore du bureau, souris (Engelbart, 1968)
  - Smalltalk (Alan Kay), MVC (Reenskaug, 1979), renvoi vers la section architecture
  - Macintosh (1984), Windows (1985/1990)
- Le modèle événementiel : de la boucle d'événements aux callbacks
  - Inversion du contrôle, exemple comparé CLI vs Tkinter en Python
- Visual Basic (1991) : démocratisation du développement GUI par drag-and-drop (RAD)
- Les toolkits desktop : Tk, Win32/MFC, Cocoa, GTK, Java AWT/Swing, Qt
  - Tension multiplateforme : Java "write once, run anywhere" vs Qt code natif
- Flash/ActionScript (Macromedia, puis Adobe) : le web interactif avant HTML5, mort annoncée par Steve Jobs (2010)
- Web 2.0 (Tim O'Reilly, 2004) : AJAX, Google Maps, le web comme média de participation
- Le web comme plateforme UI : HTML/CSS/JS, avantage du déploiement, migration des applications desktop vers le web

### L'architecture des applications web (complété)
- Le modèle traditionnel : rendu côté serveur (SSR), pages complètes, formulaires (PHP, Django, Rails)
- AJAX : le navigateur devient un acteur, tension client/serveur
- Single Page Application (SPA) : principe, avantages, inconvénients, renvoi vers les frameworks JS
- Le retour du SSR et les approches hybrides : hydration, meta-frameworks (Next.js, Nuxt, SvelteKit)
- Static Site Generation (SSG) et Jamstack : Hugo, Jekyll, Gatsby, Astro, CDN

### Les frameworks JavaScript
- Le DOM (Document Object Model) : qu'est-ce que c'est, pourquoi c'est important
- La tension déclaratif vs impératif comme fil conducteur : HTML (déclaratif) → jQuery (impératif, manipulation directe du DOM) → React (retour au déclaratif avec JSX)
- jQuery (2006) : simplification du DOM, uniformisation des navigateurs
- Angular (2010/2016) : framework complet, two-way data binding, TypeScript
- React (2013, Facebook) : DOM virtuel, composants, flux de données unidirectionnel, JSX
- Vue.js (2014, Evan You) : approche progressive, réactivité, templates
- Svelte (2016, Rich Harris) : compilation, disparition du framework au runtime
- Concepts transversaux : composants, état (state management), réactivité, routing côté client

### Desktop, web et mobile : tensions et convergences
- Applications desktop natives : performance, accès système, distribution complexe
- Applications web : universalité, déploiement instantané, limitations (accès hardware)
- Applications mobiles natives (iOS/Swift, Android/Kotlin) : écosystèmes fermés, app stores
- Approches cross-platform : React Native, Flutter (Dart), .NET MAUI
- Progressive Web Apps (PWA) : service workers, installation, mode hors-ligne
- Responsive design et l'adaptation aux formats d'écran
- Electron et Tauri : le web comme runtime desktop
- WebAssembly (WASM) : exécuter du C++/Rust dans le navigateur, brouillage de la frontière web/natif
- Pattern BFF (Backend For Frontend, Sam Newman) : un backend adapté par type de client
- La tension fondamentale : write once run anywhere vs expérience native optimale

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
- The Twelve-Factor App (Adam Wiggins, Heroku, 2011) : méthodologie pour concevoir des applications cloud-native
  - Liens avec les modules précédents (dépendances, CI, architecture, APIs)
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
