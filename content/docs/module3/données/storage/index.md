---
title: "Les bases de données"
weight: 20
---

# L'évolution des bases de données

Nous allons faire un survol de l'évolution des bases de données d'une manière un
peu particulière : au lieu de donner leurs caractéristiques de manière abstraite
et théorique, nous allons étudier un modèle simplifié sous la forme d'un
programme python simple.

## Le modèle hiérarchique (IBM, années 69)

Le premier modèle que nous allons examiner est le modèle hiérarchique. Il est
fondamentalement basé sur la notion d'un arbre. Pour implémenter un modèle
simplifié en python, c'est relativement facile.

{{< pyodide >}}

"""
Modèle hiérarchique — illustration en Python
==============================================

Le modèle hiérarchique (IMS d'IBM, 1966) organise les données
en arbre : chaque enregistrement a exactement un parent,
et on navigue de haut en bas dans l'arbre.

Ce programme illustre les idées centrales :
  1. La structure est un ARBRE (pas un graphe quelconque)
  2. Chaque nœud a UN SEUL parent (sauf la racine)
  3. L'accès aux données est NAVIGATIONNEL (on descend dans l'arbre)
  4. La duplication est inévitable pour les relations M:N
"""

class Noeud:
    """Un enregistrement dans la base hiérarchique."""

    def __init__(self, type_segment: str, donnees: dict):
        self.type_segment = type_segment  # le « type de segment » (≈ type d'entité)
        self.donnees = donnees
        self.enfants: list["Noeud"] = []

    def ajouter_enfant(self, type_segment: str, donnees: dict):
        enfant = Noeud(type_segment, donnees)
        self.enfants.append(enfant)
        return enfant                     # pratique pour le chaînage

    def __repr__(self):
        return f"{self.type_segment}({self.donnees})"

{{< /pyodide >}}

L'implémentation d'un arbre est fondamentalement très simple, comme on le constate.
Nous allons maintenant créer une petite base de données dont le schéma sera :

```
Université
  └── Département
        ├── Professeur
        └── Cours
              └── Étudiant
```

{{< pyodide >}}

# Racine
uqam = Noeud("Université", {"nom": "UQAM"})

# Départements
info = uqam.ajouter_enfant("Département", {"nom": "Informatique", "code": "INFO"})
math = uqam.ajouter_enfant("Département", {"nom": "Mathématiques", "code": "MATH"})

# Professeurs
info.ajouter_enfant("Professeur", {"nom": "Tremblay", "bureau": "PK-4150"})
info.ajouter_enfant("Professeur", {"nom": "Gagnon",   "bureau": "PK-4920"})
math.ajouter_enfant("Professeur", {"nom": "Lavoie",   "bureau": "PK-5230"})

# Cours
bd   = info.ajouter_enfant("Cours", {"sigle": "INF3080", "titre": "Bases de données"})
algo = info.ajouter_enfant("Cours", {"sigle": "INF3105", "titre": "Structures de données"})
stat = math.ajouter_enfant("Cours", {"sigle": "MAT2080", "titre": "Statistiques"})

# Étudiants inscrits aux cours
bd.ajouter_enfant("Étudiant", {"matricule": "TRAA01", "nom": "Alice"})
bd.ajouter_enfant("Étudiant", {"matricule": "MORB02", "nom": "Bob"})
algo.ajouter_enfant("Étudiant", {"matricule": "TRAA01", "nom": "Alice"})  # ⚠ DUPLICATION
stat.ajouter_enfant("Étudiant", {"matricule": "MORB02", "nom": "Bob"})    # ⚠ DUPLICATION

{{< /pyodide >}}

Nous avons besoin de quelques mécanismes supplémentaires pour pouvoir faire des requêtes dans
notre base de données :

{{< pyodide >}}

def afficher_arbre(noeud: Noeud, niveau: int = 0):
    """Parcours en profondeur — la façon naturelle de lire une base hiérarchique."""
    indent = "    " * niveau
    print(f"{indent}📂 {noeud.type_segment} : {noeud.donnees}")
    for enfant in noeud.enfants:
        afficher_arbre(enfant, niveau + 1)

afficher_arbre(uqam)
{{< /pyodide >}}

Ensuite on peut naviguer dans l'arborescence des données :

{{< pyodide >}}

def naviguer(noeud: Noeud, *chemin: str) -> list[Noeud]:
    """
    Navigation descendante par types de segments.

    Exemple : naviguer(uqam, "Département", "Cours")
    → retourne tous les Cours de tous les Départements.

    C'est l'équivalent simplifié de la commande GN (Get Next)
    d'IMS : on spécifie le chemin dans l'arbre.
    """
    if not chemin:
        return [noeud]

    type_cherche, *reste = chemin
    resultats = []
    for enfant in noeud.enfants:
        if enfant.type_segment == type_cherche:
            resultats.extend(naviguer(enfant, *reste))
    return resultats

for cours in naviguer(uqam, "Département", "Cours"):
    print(f"  → {cours.donnees}")
{{< /pyodide >}}

On peut également y chercher quelque chose :

{{< pyodide >}}
def chercher(noeud: Noeud, type_segment: str, cle: str, valeur) -> list[Noeud]:
    """
    Recherche par type + valeur — toujours en descendant dans l'arbre.
    """
    resultats = []
    if noeud.type_segment == type_segment and noeud.donnees.get(cle) == valeur:
        resultats.append(noeud)
    for enfant in noeud.enfants:
        resultats.extend(chercher(enfant, type_segment, cle, valeur))
    return resultats

alices = chercher(uqam, "Étudiant", "nom", "Alice")
print(f"  Trouvé {len(alices)} occurrence(s) :")
for a in alices:
    print(f"    → {a.donnees}")
{{< /pyodide >}}

Alice est inscrite à INF3080 et INF3105. Dans le modèle hiérarchique, chaque
nœud a UN SEUL parent, donc Alice doit apparaître DEUX FOIS dans l'arbre :

```
    Cours INF3080
    └── Étudiant Alice  ← copie 1
    Cours INF3105
    └── Étudiant Alice  ← copie 2
```

Conséquences :
* Gaspillage d'espace
* Risque d'incohérence si on met à jour une copie mais pas l'autre
* Pas de moyen simple de poser la question :
    « À quels cours Alice est-elle inscrite ? »
    (il faut parcourir TOUT l'arbre)

## Le modèle en réseau (CODASYL, fin des années 60)

Le deuxième modèle important de base de données qui est apparu dans l'histoire
est le modèle en réseau. Si le modèle hiérarchique était basé sur un arbre, le
modèle en réseau est basé sur un graphe :

{{< pyodide >}}

"""
Modèle réseau (CODASYL) — illustration en Python
==================================================

Le modèle réseau (CODASYL/DBTG, 1969) est une évolution du modèle
hiérarchique. L'idée clé : un enregistrement peut avoir PLUSIEURS
parents, grâce à des ensembles nommés (SETs) qui relient un type
« owner » à un type « member ».

Ce programme illustre les idées centrales :
  1. Les enregistrements (RECORDS) existent indépendamment
  2. Les liens sont des ENSEMBLES NOMMÉS (SETs) de type owner → member
  3. Un enregistrement peut être member de PLUSIEURS sets
     → résout le problème de duplication du modèle hiérarchique
  4. L'accès reste NAVIGATIONNEL (FIND, GET, FIND NEXT WITHIN SET…)
"""

# ============================================================
# 1. Définitions : Record, Set, et la base réseau
# ============================================================

class Record:
    """Un enregistrement dans la base réseau."""

    def __init__(self, type_record: str, donnees: dict):
        self.type_record = type_record
        self.donnees = donnees

    def __repr__(self):
        return f"{self.type_record}({self.donnees})"


class Set:
    """
    Un SET CODASYL : un lien nommé entre un owner et ses members.

    Contrainte fondamentale : chaque member ne peut appartenir
    qu'à UNE SEULE instance d'un set donné (un seul owner par set).
    Mais un record peut être member de PLUSIEURS sets différents.
    """

    def __init__(self, nom: str, owner: Record):
        self.nom = nom
        self.owner = owner
        self.members: list[Record] = []

    def inserer(self, member: Record):
        self.members.append(member)

    def __repr__(self):
        return f"Set '{self.nom}' : {self.owner} → {self.members}"


class BaseReseau:
    """
    Une base de données réseau simplifiée.

    Stocke tous les records et tous les sets,
    et fournit des opérations de navigation.
    """

    def __init__(self):
        self.records: list[Record] = []
        self.sets: list[Set] = []

    def creer_record(self, type_record: str, donnees: dict) -> Record:
        """STORE record — crée un enregistrement dans la base."""
        rec = Record(type_record, donnees)
        self.records.append(rec)
        return rec

    def creer_set(self, nom: str, owner: Record) -> Set:
        """Déclare une instance de set avec son owner."""
        s = Set(nom, owner)
        self.sets.append(s)
        return s

    # --- Navigation dans les sets ---

    def find_members(self, nom_set: str, owner: Record) -> list[Record]:
        """
        FIND NEXT WITHIN SET — retourne tous les members
        d'un owner dans un set nommé.
        """
        for s in self.sets:
            if s.nom == nom_set and s.owner is owner:
                return list(s.members)
        return []

    def find_owner(self, nom_set: str, member: Record) -> Record | None:
        """
        FIND OWNER WITHIN SET — navigation INVERSE.
        Étant donné un member, retrouve son owner dans un set nommé.
        """
        for s in self.sets:
            if s.nom == nom_set and member in s.members:
                return s.owner
        return None

    def find_all_owners(self, nom_set: str, member: Record) -> list[Record]:
        """
        Retrouve TOUS les owners d'un member dans un type de set.
        (Utile quand un record est member de plusieurs instances du même set.)
        """
        return [s.owner for s in self.sets
                if s.nom == nom_set and member in s.members]

    def find_records(self, type_record: str, cle: str, valeur) -> list[Record]:
        """FIND record by type + valeur."""
        return [r for r in self.records
                if r.type_record == type_record and r.donnees.get(cle) == valeur]

{{< /pyodide >}}

À partir de cette mécanique, nous pouvons donc maintenant créer notre base de données
en réseau :

{{< pyodide >}}

# ============================================================
# 2. Construction de la base — même scénario universitaire
# ============================================================
#
# Schéma réseau :
#
#   Département ──[DEPT_PROF]──→ Professeur
#   Département ──[DEPT_COURS]──→ Cours
#   Cours ────────[INSCRIPTION]──→ Étudiant   ← un étudiant peut
#   (plusieurs cours)                            être member de
#                                                PLUSIEURS sets
#                                                INSCRIPTION

db = BaseReseau()

# --- Records (existent indépendamment, pas dans un arbre) ---
info = db.creer_record("Département", {"nom": "Informatique", "code": "INFO"})
math = db.creer_record("Département", {"nom": "Mathématiques", "code": "MATH"})

tremblay = db.creer_record("Professeur", {"nom": "Tremblay", "bureau": "PK-4150"})
gagnon   = db.creer_record("Professeur", {"nom": "Gagnon",   "bureau": "PK-4920"})
lavoie   = db.creer_record("Professeur", {"nom": "Lavoie",   "bureau": "PK-5230"})

bd   = db.creer_record("Cours", {"sigle": "INF3080", "titre": "Bases de données"})
algo = db.creer_record("Cours", {"sigle": "INF3105", "titre": "Structures de données"})
stat = db.creer_record("Cours", {"sigle": "MAT2080", "titre": "Statistiques"})

alice = db.creer_record("Étudiant", {"matricule": "TRAA01", "nom": "Alice"})  # ← UNE SEULE fois !
bob   = db.creer_record("Étudiant", {"matricule": "MORB02", "nom": "Bob"})    # ← UNE SEULE fois !

# --- Sets : les liens nommés entre records ---

# Département → Professeurs
set_info_prof = db.creer_set("DEPT_PROF", info)
set_info_prof.inserer(tremblay)
set_info_prof.inserer(gagnon)

set_math_prof = db.creer_set("DEPT_PROF", math)
set_math_prof.inserer(lavoie)

# Département → Cours
set_info_cours = db.creer_set("DEPT_COURS", info)
set_info_cours.inserer(bd)
set_info_cours.inserer(algo)

set_math_cours = db.creer_set("DEPT_COURS", math)
set_math_cours.inserer(stat)

# Cours → Étudiants (INSCRIPTION)
# Alice est inscrite à BD et ALGO — même objet, deux sets différents
set_inscr_bd = db.creer_set("INSCRIPTION", bd)
set_inscr_bd.inserer(alice)
set_inscr_bd.inserer(bob)

set_inscr_algo = db.creer_set("INSCRIPTION", algo)
set_inscr_algo.inserer(alice)       # ✅ PAS de duplication !

set_inscr_stat = db.creer_set("INSCRIPTION", stat)
set_inscr_stat.inserer(bob)         # ✅ PAS de duplication !

{{< /pyodide >}}

On peut afficher la structure de notre base de données :

{{< pyodide >}}

print("=" * 60)
print("STRUCTURE DE LA BASE RÉSEAU")
print("=" * 60)
for dept in db.find_records("Département", "nom", "Informatique") + \
            db.find_records("Département", "nom", "Mathématiques"):
    print(f"\n📂 {dept}")
    profs = db.find_members("DEPT_PROF", dept)
    if profs:
        print("    Professeurs :")
        for p in profs:
            print(f"        → {p.donnees}")
    cours = db.find_members("DEPT_COURS", dept)
    if cours:
        print("    Cours :")
        for c in cours:
            etudiants = db.find_members("INSCRIPTION", c)
            noms = [e.donnees["nom"] for e in etudiants]
            print(f"        → {c.donnees}  [{', '.join(noms)}]")

{{< /pyodide >}}

On peut faire une requête pour tous les étudiants inscrits à un cours particulier :

{{< pyodide >}}

print("\n" + "=" * 60)
print("NAVIGATION AVANT : étudiants inscrits à INF3080")
print("=" * 60)
for e in db.find_members("INSCRIPTION", bd):
    print(f"  → {e.donnees}")

{{< /pyodide >}}

Ou encore, une requête pour tous les cours suivis par une étudiante particulière :

{{< pyodide >}}

print("\n" + "=" * 60)
print("NAVIGATION INVERSE : cours d'Alice")
print("=" * 60)
cours_alice = db.find_all_owners("INSCRIPTION", alice)
for c in cours_alice:
    print(f"  → {c.donnees}")

{{< /pyodide >}}

Alice n'existe qu'UNE SEULE FOIS dans la base de données. Elle est membre de
deux sets INSCRIPTION (un par cours).

* Plus de duplication !
* La question « quels cours suit Alice ? » se résout
par navigation inverse (FIND OWNER WITHIN SET).

Mais il reste des problèmes :
* L'accès est toujours NAVIGATIONNEL : le programmeur doit connaître le schéma
    des sets et écrire des boucles pour traverser les liens.
* Ajouter un nouveau type de lien exige de modifier le schéma et le code de
    navigation.
* Pas de langage déclaratif : on dit COMMENT chercher, pas CE QU'ON cherche.

## Le modèle relationnel et SQL

On en arrive finalement au modèle relationnel.

{{< sql >}}

-- ============================================================
-- Modele relationnel -- illustration en SQL
-- ============================================================
--
-- Le modele relationnel (Codd, 1970) remplace les arbres
-- et les liens navigationnels par des TABLES et des REQUETES
-- DECLARATIVES : on decrit CE QU'ON CHERCHE, pas COMMENT
-- le trouver.
--
-- Meme scenario universitaire que les programmes Python
-- pour le modele hierarchique et le modele reseau.
-- ============================================================


-- ============================================================
-- 1. Definition du schema : des TABLES plates
-- ============================================================
-- Plus d'arbre, plus de sets/owners/members.
-- Chaque entite a sa propre table, et les relations
-- sont exprimees par des CLES ETRANGERES.

CREATE TABLE departement (
    code  VARCHAR(10) PRIMARY KEY,
    nom   VARCHAR(50) NOT NULL
);

CREATE TABLE professeur (
    id     INTEGER     PRIMARY KEY,
    nom    VARCHAR(50) NOT NULL,
    bureau VARCHAR(20),
    dept   VARCHAR(10) REFERENCES departement(code)
);

CREATE TABLE cours (
    sigle  VARCHAR(10) PRIMARY KEY,
    titre  VARCHAR(80) NOT NULL,
    dept   VARCHAR(10) REFERENCES departement(code)
);

CREATE TABLE etudiant (
    matricule VARCHAR(10) PRIMARY KEY,    -- UNE SEULE FOIS
    nom       VARCHAR(50) NOT NULL
);

-- La relation M:N (cours <-> etudiants) est une TABLE DE JOINTURE.
-- Ni duplication (modele hierarchique),
-- ni navigation par sets (modele reseau) :
-- juste une table avec deux cles etrangeres.

CREATE TABLE inscription (
    matricule VARCHAR(10) REFERENCES etudiant(matricule),
    sigle     VARCHAR(10) REFERENCES cours(sigle),
    PRIMARY KEY (matricule, sigle)
);

{{< /sql >}}

Une fois notre schéma en place, on peut ajouter des données.

{{< sql >}}

-- ============================================================
-- 2. Insertion des donnees
-- ============================================================

INSERT INTO departement VALUES ('INFO', 'Informatique');
INSERT INTO departement VALUES ('MATH', 'Mathematiques');

INSERT INTO professeur VALUES (1, 'Tremblay', 'PK-4150', 'INFO');
INSERT INTO professeur VALUES (2, 'Gagnon',   'PK-4920', 'INFO');
INSERT INTO professeur VALUES (3, 'Lavoie',   'PK-5230', 'MATH');

INSERT INTO cours VALUES ('INF3080', 'Bases de donnees',        'INFO');
INSERT INTO cours VALUES ('INF3105', 'Structures de donnees',   'INFO');
INSERT INTO cours VALUES ('MAT2080', 'Statistiques',            'MATH');

INSERT INTO etudiant VALUES ('TRAA01', 'Alice');
INSERT INTO etudiant VALUES ('MORB02', 'Bob');

INSERT INTO inscription VALUES ('TRAA01', 'INF3080');   -- Alice -> BD
INSERT INTO inscription VALUES ('TRAA01', 'INF3105');   -- Alice -> Algo
INSERT INTO inscription VALUES ('MORB02', 'INF3080');   -- Bob   -> BD
INSERT INTO inscription VALUES ('MORB02', 'MAT2080');   -- Bob   -> Stat

{{< /sql >}}

Et finalement on peut faire des requêtes :

{{< sql >}}

-- ============================================================
-- 3. Requetes declaratives -- CE QU'ON CHERCHE, pas COMMENT
-- ============================================================

-- Equivalent de naviguer(uqam, "Departement", "Cours")
-- du modele hierarchique, mais sans connaitre aucun chemin :

SELECT d.nom AS departement, c.sigle, c.titre
  FROM departement d
  JOIN cours c ON c.dept = d.code;

-- Equivalent de find_members("INSCRIPTION", bd)
-- du modele reseau -- etudiants inscrits a INF3080 :

SELECT e.matricule, e.nom
  FROM etudiant e
  JOIN inscription i ON i.matricule = e.matricule
 WHERE i.sigle = 'INF3080';


-- La question qui etait penible dans le modele hierarchique
-- et qui necessitait une navigation inverse dans le modele reseau
-- "A quels cours Alice est-elle inscrite ?"
-- s'ecrit naturellement :

SELECT c.sigle, c.titre
  FROM cours c
  JOIN inscription i ON i.sigle = c.sigle
 WHERE i.matricule = 'TRAA01';


-- Une question impossible a poser simplement dans les
-- modeles precedents -- "Quels etudiants partagent
-- au moins un cours avec Alice ?"

SELECT DISTINCT e.nom
  FROM etudiant e
  JOIN inscription i1 ON i1.matricule = e.matricule
  JOIN inscription i2 ON i2.sigle = i1.sigle
 WHERE i2.matricule = 'TRAA01'
   AND e.matricule != 'TRAA01';

{{< /sql >}}

### Les transactions

### Les ORMs (object relational mappers)

## La révolution NoSQL

### Key-Value Stores

### Document Stores

### Columnar Stores

### Graph Databases

## Les bases de données de séries temporelles

## Les bases de données vectorielles