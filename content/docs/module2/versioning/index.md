---
title: "Le versioning avec git"
weight: 20
---

# Le versioning

## Quel est le problème qu'on cherche à résoudre?

Le logiciel (et le code source dont il est constitué) a une particularité
fondamentale : il change tout le temps. Même un programme simple est rapidement
modifié - pour corriger un bug, ajouter une fonctionnalité, améliorer la
performance ou simplement rendre le code plus lisible. Et très vite, une
question apparaît : comment savoir ce qui a changé, quand, et pourquoi ?

Sans système de versioning, le développement logiciel devient fragile. On copie
des fichiers, on renomme des dossiers (« version2 », « version_finale », «
version_finale_bis »), on échange du code par courriel ou sur un disque partagé.
Ces solutions fonctionnent… jusqu’au moment où elles ne fonctionnent plus : on
perd une modification importante, on ne sait plus quelle version est la bonne,
ou deux personnes écrasent mutuellement leur travail.

Le versioning n’est pas une simple sauvegarde. Sauvegarder, c’est garder une
photo du code à un instant donné. Versionner, c’est garder la mémoire de son
évolution. Un bon système de versioning permet de revenir en arrière, de
comprendre l’historique d’un projet, de travailler à plusieurs sans se gêner, et
d’expérimenter sans casser ce qui fonctionne.

À mesure que les projets grossissent et que le travail devient collectif, le
versioning devient indispensable. Il structure la façon dont on travaille, dont
on collabore, et même dont on réfléchit au code. C’est ce problème — gérer le
changement — qui a conduit à la création des systèmes de contrôle de versions,
et en particulier de Git, que nous allons étudier dans ce chapitre.

## Bref historique des systèmes de gestion de versions

### Première génération : local et fichier unique

**SCCS** (Source Code Control System, 1972) — Bell Labs
- Premier système de versioning, créé par Marc Rochkind
- Stockage par deltas (différences) pour économiser l'espace
- Un seul fichier à la fois, un seul utilisateur à la fois
- Verrouillage exclusif pour éviter les conflits

**RCS** (Revision Control System, 1982) — Walter Tichy, Purdue
- Amélioration de SCCS, plus rapide grâce aux deltas inversés (stocke la version courante en entier, reconstruit les anciennes)
- Toujours limité à des fichiers individuels
- Reste local à une machine

### Deuxième génération : centralisé et projets entiers

**CVS** (Concurrent Versions System, 1986–1990) — Dick Grune, puis Brian Berliner
- Construit au-dessus de RCS mais gère des arborescences de fichiers
- Premier système vraiment *concurrent* : plusieurs développeurs peuvent modifier simultanément
- Modèle client-serveur avec un dépôt central
- Faiblesses notoires : pas de commits atomiques, gestion pénible des renommages et des branches

**Perforce** (1995) — commercial
- Très performant sur de gros dépôts binaires
- Encore utilisé dans l'industrie du jeu vidéo et les grandes entreprises

**Subversion (SVN)** (2000) — CollabNet
- Conçu explicitement pour corriger les défauts de CVS
- Commits atomiques, versioning des répertoires, renommages suivis
- Toujours centralisé : le serveur est la source de vérité unique
- Numérotation linéaire des révisions (r1, r2, r3…)

### Troisième génération : distribué

**BitKeeper** (1998–2000) — Larry McVoy
- Premier système distribué utilisé à grande échelle (noyau Linux de 2002 à 2005)
- Licence propriétaire controversée, ce qui a mené à la création de Git

**GNU Arch / Bazaar / Darcs** (début 2000)
- Expérimentations diverses sur le versioning distribué
- Darcs introduit une approche par patches commutatifs (théorie des patches)

**Git** (2005) — Linus Torvalds
- Créé en quelques semaines après la rupture avec BitKeeper
- Objectifs : vitesse, intégrité des données, support massif des branches
- Caractéristiques clés :
 - Chaque clone est un dépôt complet avec tout l'historique
 - Stockage par snapshots (pas deltas) avec déduplication via SHA-1
 - Branches ultra-légères (simples pointeurs)
 - Travail hors ligne total
 - Modèle de staging (index) avant commit
- Devenu le standard de facto grâce à GitHub (2008)

**Mercurial** (2005) — Matt Mackall
- Lancé la même semaine que Git, mêmes motivations
- Interface plus simple, concepts similaires
- Utilisé par Facebook, Mozilla (historiquement)
- Moins dominant aujourd'hui, mais toujours actif

---

### Tableau récapitulatif

| Système | Année | Modèle | Granularité | Particularité |
|---------|-------|--------|-------------|---------------|
| SCCS | 1972 | Local | Fichier | Premier système, deltas |
| RCS | 1982 | Local | Fichier | Deltas inversés |
| CVS | 1990 | Centralisé | Projet | Accès concurrent |
| SVN | 2000 | Centralisé | Projet | Commits atomiques |
| Git | 2005 | Distribué | Projet | Snapshots, branches légères |
| Mercurial | 2005 | Distribué | Projet | Interface épurée |

L'évolution suit une logique claire : d'abord résoudre le problème du suivi pour
un fichier, puis pour un projet, puis permettre la collaboration, et enfin
décentraliser pour la résilience et la flexibilité.

## Un système de gestion des versions moderne et extrêmement populaire : git

### Fonction de hachage

Pour se construire un bon modèle mental du fonctionnement de git, il est essentiel de
bien comprendre tout d'abord la notion de **fonction de hachage**.

Une fonction de hachage est une fonction mathématique qui prend en entrée un
nombre, et qui retourne un autre nombre. Une fonction de hachage ne peut pas
être n'importe quelle fonction par contre (comme par exemple $f(x) = mx + b$),
elle doit avoir certaines caractéristiques :

* Déterminisme : Pour une même donnée en entrée, vous obtiendrez toujours exactement le même hash en sortie.

* Rapidité : Le calcul du hash doit être presque instantané.

* Effet d'avalanche : Si vous changez ne serait-ce qu'une virgule ou une
  majuscule dans le texte d'origine, le hash produit sera totalement différent.

* Irréversibilité (Hachage cryptographique) : Il est impossible de retrouver la
  donnée d'origine à partir du hash. C'est un sens unique.

* Résistance aux collisions : Il est extrêmement improbable que deux entrées
  différentes produisent le même hash.

Le premier critère implique qu'en dépit des apparences, un hash n'est **pas** un
nombre aléatoire, il est important de le réaliser. Le dernier critère implique
que l'_image_ de la fonction (l'ensemble de ses valeurs possibles) doit être
extrêmement grand (car s'il ne l'était pas, les collisions seraient fréquentes).
La fonction de hachage [SHA-1](https://fr.wikipedia.org/wiki/SHA-1), utilisée
couramment par git, produit par exemple des valeurs de 160 bits, ce qui
correspond à $2^{160}$ valeurs possibles, ce qui est un nombre vraiment très
grand, mais tout de même plus petit que ceux produits par la fonction
[SHA-256](https://fr.wikipedia.org/wiki/SHA-2), aussi utilisée par les versions
plus modernes de git, et dont la taille de l'image se rapproche du nombre estimé
d'atomes dans l'univers (environ $2^{266}$, ou $10^{80}$).

Il est important de comprendre que le but d'une fonction de hachage, dans le
contexte de git, est de calculer un nombre unique, à partir d'un "objet". Cet
"objet" (au sens digital) peut être de plusieurs natures :

1. Une chaîne de caractères
2. Un fichier (ou document)
3. Un groupe de plusieurs fichiers
4. Une arborescence (récursive) de fichiers
5. Etc.

Chacun de ces objets peut être considéré comme étant un nombre, avec les
conversions nécessaires. C'est ici qu'entrent en jeu les qualités particulières
que nous avons énumérées ci-haut&nbsp;: pour un objet donné (un fichier par
exemple), la valeur de hash doit être unique (résistance aux collisions), et
toujours la même, sans exception (déterminisme). Si on ne change ne serait-ce
qu'un bit d'un fichier, la valeur _doit_ changer (effet d'avalanche).
L'implémentation de cette fonction (dans un langage comme Python, avec la
librairie [hashlib](https://docs.python.org/3/library/hashlib.html) par exemple
doit être suffisamment rapide, parce que git l'utilisera extrêmement souvent,
étant donné qu'il s'agit d'une de ses opérations fondamentales.

{{% details "Implémentation de SHA-256 en Python pur (sans la libraire `hashlib`), pour les curieux" %}}

```python
# Implémentation pédagogique de SHA-256 en Python pur (sans hashlib)

def rotation_droite(valeur, decalage):
    """
    Rotation circulaire à droite sur 32 bits
    """
    return ((valeur >> decalage) | (valeur << (32 - decalage))) & 0xFFFFFFFF


def sha256(message: bytes) -> str:
    """
    Calcule le hash SHA-256 d'un message (bytes)
    et retourne une chaîne hexadécimale
    """

    # Valeurs initiales du hash
    # (32 premiers bits des parties fractionnaires
    # des racines carrées des 8 premiers nombres premiers)
    H = [
        0x6a09e667,
        0xbb67ae85,
        0x3c6ef372,
        0xa54ff53a,
        0x510e527f,
        0x9b05688c,
        0x1f83d9ab,
        0x5be0cd19,
    ]

    # Constantes de ronde
    # (32 premiers bits des parties fractionnaires
    # des racines cubiques des 64 premiers nombres premiers)
    K = [
        0x428a2f98, 0x71374491, 0xb5c0fbcf, 0xe9b5dba5,
        0x3956c25b, 0x59f111f1, 0x923f82a4, 0xab1c5ed5,
        0xd807aa98, 0x12835b01, 0x243185be, 0x550c7dc3,
        0x72be5d74, 0x80deb1fe, 0x9bdc06a7, 0xc19bf174,
        0xe49b69c1, 0xefbe4786, 0x0fc19dc6, 0x240ca1cc,
        0x2de92c6f, 0x4a7484aa, 0x5cb0a9dc, 0x76f988da,
        0x983e5152, 0xa831c66d, 0xb00327c8, 0xbf597fc7,
        0xc6e00bf3, 0xd5a79147, 0x06ca6351, 0x14292967,
        0x27b70a85, 0x2e1b2138, 0x4d2c6dfc, 0x53380d13,
        0x650a7354, 0x766a0abb, 0x81c2c92e, 0x92722c85,
        0xa2bfe8a1, 0xa81a664b, 0xc24b8b70, 0xc76c51a3,
        0xd192e819, 0xd6990624, 0xf40e3585, 0x106aa070,
        0x19a4c116, 0x1e376c08, 0x2748774c, 0x34b0bcb5,
        0x391c0cb3, 0x4ed8aa4a, 0x5b9cca4f, 0x682e6ff3,
        0x748f82ee, 0x78a5636f, 0x84c87814, 0x8cc70208,
        0x90befffa, 0xa4506ceb, 0xbef9a3f7, 0xc67178f2,
    ]

    # --- Prétraitement (padding) ---

    longueur_originale = len(message) * 8  # en bits
    message += b'\x80'  # ajout du bit 1

    # Ajout de zéros jusqu'à atteindre 448 bits modulo 512
    while (len(message) * 8) % 512 != 448:
        message += b'\x00'

    # Ajout de la longueur originale sur 64 bits
    message += longueur_originale.to_bytes(8, 'big')

    # --- Traitement par blocs de 512 bits ---

    for debut_bloc in range(0, len(message), 64):
        bloc = message[debut_bloc:debut_bloc + 64]
        W = [0] * 64  # planification du message

        # Les 16 premiers mots proviennent directement du bloc
        for i in range(16):
            W[i] = int.from_bytes(bloc[i*4:(i+1)*4], 'big')

        # Extension des 16 mots en 64
        for i in range(16, 64):
            s0 = (
                rotation_droite(W[i-15], 7)
                ^ rotation_droite(W[i-15], 18)
                ^ (W[i-15] >> 3)
            )
            s1 = (
                rotation_droite(W[i-2], 17)
                ^ rotation_droite(W[i-2], 19)
                ^ (W[i-2] >> 10)
            )
            W[i] = (W[i-16] + s0 + W[i-7] + s1) & 0xFFFFFFFF

        # Initialisation des registres de travail
        a, b, c, d, e, f, g, h = H

        # --- Boucle principale de compression ---
        for i in range(64):
            S1 = (
                rotation_droite(e, 6)
                ^ rotation_droite(e, 11)
                ^ rotation_droite(e, 25)
            )
            choix = (e & f) ^ (~e & g)
            temp1 = (h + S1 + choix + K[i] + W[i]) & 0xFFFFFFFF

            S0 = (
                rotation_droite(a, 2)
                ^ rotation_droite(a, 13)
                ^ rotation_droite(a, 22)
            )
            majorite = (a & b) ^ (a & c) ^ (b & c)
            temp2 = (S0 + majorite) & 0xFFFFFFFF

            h = g
            g = f
            f = e
            e = (d + temp1) & 0xFFFFFFFF
            d = c
            c = b
            b = a
            a = (temp1 + temp2) & 0xFFFFFFFF

        # Ajout du résultat du bloc au hash courant
        H = [
            (x + y) & 0xFFFFFFFF
            for x, y in zip(H, [a, b, c, d, e, f, g, h])
        ]

    # Conversion finale en hexadécimal
    return ''.join(f'{valeur:08x}' for valeur in H)


# Exemple d'utilisation
if __name__ == "__main__":
    print(sha256(b"hello world"))
```

{{% /details %}}

Une valeur de hash particulière pourrait être par exemple&nbsp;:

`2123f2435e1dbe255a323c90e97e38f759fe8946`

ce qui correspond à un nombre hexadécimal (en base 16 donc, qui s'exprime avec
les dix chiffres 0 à 10, ainsi que les 6 premières lettres de l'alphabet, de `a`
à `f`) de 40 "chiffres". Chaque caractère hexadécimal de cette chaîne étant
équivalent à 4 bits, nous avons donc un nombre de $40 \times 4 = 160$ bits, qui
permet d'exprimer donc $2^{160}$ valeurs possibles.

Faites-vous une idée concrète du fonctionnement d'une fonction de hachage à
l'aide de cette applet interactive :

{{< applet src="/html/applets/hashing.html" width="100%" scale="1.0" >}}

Finalement, une manière de comprendre le rôle que peut jouer une fonction de
hachage est en tant qu'une sorte de "signature" : une signature _représente_ une
personne ou un document de manière unique, sans aucune ambiguïté possible. Il
est également possible de se représenter un hash en tant que _pointeur_ vers
quelque chose, ce qui est la métaphore utilisée par git, comme nous le verrons
concrètement.

> [!NOTE]
La notion de hachage est utilisée de manière centrale par git, mais il s'agit d'une
notion fondamentale en informatique. Elle est particulièrement importante dans le monde
de la cryptographie, qui est la science de l'échange et de la manipulation de "secrets".
Elle joue également un rôle fondateur dans des domaines plus récents et émergents,
comme les cryptomonnaies (Bitcoin est l'exemple le plus fameux) et de manière plus
générale, la notion de chaîne de blocs (blockchain en anglais).

### Les fondements de git

Git est un outil pour la ligne de commande qui utilise un modèle de données
particulier pour représenter et manipuler le contenu et l'évolution historique
de l'ensemble de fichiers d'un projet de développement logiciel. Nous allons
l'étudier tout en l'utilisant concrètement. Je vous recommande d'entrer les
commandes qui vont suivre dans votre propre ligne de commande au moment de la
lecture, afin de rendre plus concrète les notions que nous verrons.

La première notion importante de git est celle de _dépôt_ (repository en anglais), qui
correspond à un répertoire particulier, et qui constitue un projet au sens de git. Ce répertoire
peut déjà exister (et contenir des fichiers) ou être créé pour l'occasion et donc être vide.
Nous allons donc tout d'abord créer un nouveau répertoire :

```shell
$ mkdir mon_premier_depot
$ cd mon_premier_depot
$ ls -al
total 0
drwxr-xr-x@  2 cjauvin  staff    64 Jan 13 16:54 ./
drwxr-x---+ 75 cjauvin  staff  2400 Jan 13 16:54 ../
```

Notre répertoire n'est pas encore un dépôt git, mais avec cette commande il le devient :

```shell
$ git init
Initialized empty Git repository in /Users/cjauvin/mon_premier_depot/.git/
```

On peut constater que git a ajouté un sous-répertoire spécial dans notre répertoire :

```shell
$ ls -al
total 0
drwxr-xr-x@  3 cjauvin  staff    96 Jan 13 16:58 ./
drwxr-x---+ 75 cjauvin  staff  2400 Jan 13 16:54 ../
drwxr-xr-x@  9 cjauvin  staff   288 Jan 13 17:10 .git/
```

Ce répertoire contient toutes les données dont git aura besoin pour gérer notre
dépôt. On peut le voir comme une sorte de base de données aussi, qui est pour le
moment vide :

```shell
$ ls -al .git
total 24
drwxr-xr-x@  9 cjauvin  staff  288 Jan 13 17:10 ./
drwxr-xr-x@  3 cjauvin  staff   96 Jan 13 16:58 ../
-rw-r--r--@  1 cjauvin  staff  137 Jan 13 16:58 config
-rw-r--r--@  1 cjauvin  staff   73 Jan 13 16:58 description
-rw-r--r--@  1 cjauvin  staff   21 Jan 13 16:58 HEAD
drwxr-xr-x@ 16 cjauvin  staff  512 Jan 13 16:58 hooks/
drwxr-xr-x@  3 cjauvin  staff   96 Jan 13 16:58 info/
drwxr-xr-x@  4 cjauvin  staff  128 Jan 13 16:58 objects/
drwxr-xr-x@  4 cjauvin  staff  128 Jan 13 16:58 refs/
```

Commençons notre exploration en créant tout d'abord un premier fichier :

```shell
$ echo "allo!" >> toto.txt
```

À ce stade, l'état du fichier est "untracked", c'est-à-dire qu'il ne fait pas partie
du dépôt (même si git a tout de même remarqué sa présence) :

```shell
$ git status
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        toto.txt

nothing added to commit but untracked files present (use "git add" to track)
```

On peut ajouter le fichier dans le "staging area" de git, qui constitue une sorte d'espace de travail
temporaire, qui sert à accumuler les changement que nous voudrons "committer" plus tard :

```shell
$ git add toto.txt
```

Une fois le fichier ajouté, la même commande permet de constater le changement d'état. Pour
le moment, le nouveau fichier est dans le staging area, prêt à être committé :

```shell
$ git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   toto.txt
```

On est maintenant prêt à faire notre premier commit, qui est simplement l'ajout
officiel de ce fichier dans la base de données git :

```shell
$ git commit -m "Premier commit"
[main (root-commit) 1359fb1] Premier commit
 1 file changed, 1 insertion(+)
 create mode 100644 toto.txt
```

Une fois ce premier commit effectué, notre dépôt git est démarré en bonne et due
forme, et on peut commencer à l'inspecter en profondeur pour en comprendre le
fonctionnement et le modèle de données. Il est important de savoir que les
commandes qui vont suivre sont très rarement utilisées dans l'usage quotidien de
git (pratiquement jamais en fait). On s'en sert ici simplement en tant qu'outils
pédagogiques.

#### Premier type d'objet fondamental git : le blob

Le modèle de données de Git est constitué de quatre types fondamentaux d'objets.
Le premier type est le _blob_, qui correspond aux données données binaires
compressées d'un fichier donné (ici `toto.txt`) :

```shell
$ git ls-tree -r HEAD
100644 blob 4c7d057645ac149446d1289aaa6f9fd74e91ce13    toto.txt
```

Notre premier blob a donc comme identifiant le hash
`4c7d057645ac149446d1289aaa6f9fd74e91ce13`. Voici comment git calcule le hash du
blob, à partir du fichier original qu'on a ajouté :

```shell
$ git hash-object toto.txt
4c7d057645ac149446d1289aaa6f9fd74e91ce13
```

Cette valeur particulière devrait être la même pour vous, dans votre
environnement, car elle n'est pas une valeur aléatoire, elle est entièrement
déterministe. La seule chose qui pourrait changer la donne serait que votre git
soit configuré pour utiliser une autre fonction de hachage que `SHA1`, ce qui
est possible.

Si on entre la valeur `allo!` dans l'applet interactif ci-haut, on n'obtient pas
cette valeur particulière de hash pourtant (essayez-le, avec la fonction
`SHA1`).. pourquoi donc? Parce que git hache en fait un peu plus que simplement
le contenu du fichier : il hache la valeur `blob <taille>\0<contenu>`, où
`<contenu>` correspond dans notre cas à `allo!\n` (le `\n` est un caractère
spécial pour le retour de chariat, newline en anglais) et la `<taille>`, en
nombre de caractères, est donc 6. Donc si on entre la valeur `blob 6\0allo!\n`
dans l'applet, le même hash que celui calculé par git devrait apparaître.

Ce blob est en fait un fichier, qui est sauvegardé dans la base de données
spéciale de git (dans le répertoire caché `.git`). Le chemin vers ce fichier (de
blob) est créé à partir du hash, qui est séparé en deux parties (la première
correspondant à son suffixe de deux lettres) :

```shell
$ ll .git/objects/
total 0
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 12:16 4c/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 12:16 9e/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 12:16 cf/
drwxr-xr-x@ 2 cjauvin  staff    64B Jan 15 12:16 info/
drwxr-xr-x@ 2 cjauvin  staff    64B Jan 15 12:16 pack/
$
$ ll .git/objects/4c
total 8
-r--r--r--@ 1 cjauvin  staff    21B Jan 15 12:16 7d057645ac149446d1289aaa6f9fd74e91ce13
```

Contrairement à `toto.txt`, qui contient du texte, le blob n'est pas lisible,
car il est constitué de données binaires compressées :

```shell
cat .git/objects/4c/7d057645ac149446d1289aaa6f9fd74e91ce13
xK��OR0cH���W�+�⏎
```

{{< image src="git-blob.png" alt="Un blob git" title="" loading="lazy" >}}

Essayons maintenant de modifier notre fichier texte en y ajoutant une ligne :

```shell
$ echo "bonjour" >> toto.txt
$ cat toto.txt
allo!
bonjour
```

Et vérifions l'effet sur git avec la commande `git status` :

```shell
$ git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   toto.txt
no changes added to commit (use "git add" and/or "git commit -a")
```

Pour voir le changement précis que git détecte on peut utiliser `git diff` :

```shell
$ git diff
diff --git a/toto.txt b/toto.txt
index 4c7d057..48bed0c 100644
--- a/toto.txt
+++ b/toto.txt
@@ -1 +1,2 @@
 allo!
+bonjour
```

Que se passerait-il si on tentait de committer notre changement à ce point?

```shell
$ git commit -m "Dexuième commit"
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   toto.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

Il n'y a rien (encore) à committer, car le changement n'a pas été ajouté
dans le staging area avec la commande `git add`, dont on peut tout de suite
voir l'effet avec `git status` :

```shell
$ git add toto.txt
$
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   toto.txt
```

Voici le modèle mental qu'il faut préférablement avoir des trois "endroits"
importants qui sont impliqués dans le pipeline de création d'un commit :

{{< image src="git-3-places.png" alt="Un blob git" title="" loading="lazy" >}}

Nous sommes maintenant prêt pour le deuxième commit :

```shell
$ git commit -m "Deuxième commit"
[main 34c85ab] Deuxième commit
 1 file changed, 1 insertion(+)
$
$ git status
On branch main
nothing to commit, working tree clean
```

Examinons ce qui s'est passé au niveau des objets git :

```shell
git ls-tree -r HEAD
100644 blob 48bed0c35d92c60539832929142859e3f5ae6eda    toto.txt
```

On remarque que `toto.txt` est maintenant associé à ce qui semble être un blob
différent, car le hash est différent (`48bed0c35d92c60539832929142859e3f5ae6eda`
alors qu'on avait `4c7d057645ac149446d1289aaa6f9fd74e91ce13` avant). Que
s'est-il passé? Un nouveau blob a été créé, correspondant à la nouvelle version
du fichier. Si on regarde la liste des objets, on peut voir notre nouveau blob
et son contenu :

```shell
$ ll .git/objects/
total 0
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 14:57 34/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 14:53 38/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 14:39 48/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 12:16 4c/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 14:53 88/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 12:16 9e/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 12:16 cf/
drwxr-xr-x@ 2 cjauvin  staff    64B Jan 15 12:16 info/
drwxr-xr-x@ 2 cjauvin  staff    64B Jan 15 12:16 pack/
$
$ cat .git/objects/48/bed0c35d92c60539832929142859e3f5ae6eda
xK��OR04aH���W�J����/-�O⏎
```

{{< image src="git-2-blobs.png" alt="Un blob git" title="" loading="lazy" >}}

Mais ce qu'il est fondamental de remarquer et de comprendre, c'est que notre
blob précédent (`4c7d057645ac149446d1289aaa6f9fd74e91ce13`) est toujours là, il
n'a pas disparu ! Ceci veut donc dire que tous les stages de transformation d'un
fichier sont sauvegardés **en tant que fichiers complets et indépendants**. Git
ne gère donc pas les "différences" entre les fichiers, mais conserve plutôt
l'état complet de chaque version (en anglais on parle souvent de "snapshot",
soit l'état précis d'un fichier à un certain point dans une chaîne de
changements), de manière à pouvoir "calculer" la différence, au moment où c'est
nécessaire. Est-ce que ceci n’entraîne pas un coût supplémentaire par contre, en
terme d'espace disque? Si on a un fichier de 100 lignes et qu'on fait 10 commits
qui ajoutent chacun 1 ligne, nous aurons au final 10 blobs de plus de 100 lignes
(un de 100, un de 101, etc). Il serait plus compact de ne sauvegarder que les
différences successives. Pourtant, git choisit de fonctionner avec des
snapshots, parce que ça lui permet d'opérer de manière plus rapide et efficace,
il s'agit d'un compromis. Les comparaisons et la navigation entre les versions
d'un fichier sont donc, avec git, particulièrement efficaces, ce qui a
certainement joué un rôle dans son adoption généralisée spectaculaire. Il faut
mentionner également qu'il y a un niveau plus "bas" que les blobs, les
_packfiles_, où le contenu des blobs est compressé efficacement (et donc la
redondance éliminée), mais comme ce niveau est plus complexe et moins facile
d'accès, il ne fera donc pas partie de notre analyse.

Le fait d'avoir un mécanisme comme la fonction de hachage permet à git de
rapidement répondre à la question : est-ce qu'un fichier a changé ou non? Si le
hash d'un fichier est identique aujourd'hui à ce qu'il était hier, nous avons la
certitude, par définition, que le fichier n'a pas changé. Car le moindre
changement (ne serait-ce que l'ajout dune virgule) produit une valeur de hash
complètement différente. Par contre, cette différence, au niveau des valeurs du
hash, ne disent rien au sujet de ce qui a changé ! Elle dit SEULEMENT que
quelque chose a changé. Si git veut savoir exactement ce qui a changé (pour par
exemple le fournir à l'utilisateur de la commande `git diff`), il doit le
calculer en temps réel, au moment du besoin. On pourrait penser que ceci est
coûteux et inefficaces, mais dans les faits ça ne l'est pas, car un algorithme
pour déterminer la différence entre deux fichiers textes peut être exécuté très
rapidement. Le modèle de donnés de git est entièrement conçu de manière à
optimiser les cas s'usage les plus courants.

Poursuivons notre analyse du modèle de données de git en nous demandant ensuite :
qu'est-ce qu'un commit, au juste? Jusqu'à maintenant, nous avons créé deux commits,
en séquence :

```shell
$ git log
commit 34c85abbbbee1c21c34da2b5ed126cb888fcb498 (HEAD -> main)
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Thu Jan 15 14:57:15 2026 -0500

    Deuxième commit

commit 9e6072c8df904f021474a6173cc1b17b2908c0f1
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Thu Jan 15 12:16:21 2026 -0500

    Premier commit
```

Le premier commit est associé au hash `9e6072c8df904f021474a6173cc1b17b2908c0f1`.
On peut utiliser `git cat-file` pour décrire le contenu de ce commit :

```shell
$ git cat-file -p 9e6072c8df904f021474a6173cc1b17b2908c0f1
tree cff192ac57ced2fea4977bcf9dcc6c3590af9cbd
author Christian Jauvin <cjauvin@gmail.com> 1768497381 -0500
committer Christian Jauvin <cjauvin@gmail.com> 1768497381 -0500

Premier commit
```

#### Deuxième type d'objet git fondamental : le tree

On voit que le commit contient un "tree" (un arbre), qui est le deuxième type
fondamental d'objet dans le modèle de données de git que nous voyons, après le
blob. Si un blob est un fichier, alors le tree est un répertoire (une
arborescence récursive de fichiers). Si on inspecte notre tree avec `git cat-file`,
on constate qu'il ne contient qu'un seul blob, celui qui correspond à notre fichier `toto.txt` :

```shell
$ git cat-file -p cff192ac57ced2fea4977bcf9dcc6c3590af9cbd
100644 blob 4c7d057645ac149446d1289aaa6f9fd74e91ce13    toto.txt
```

Pour se faire une meilleure idée du fonctionnement des trees, complexifions un peu
la structure de notre projet en ajoutant des fichiers et des répertoires :

```shell
$ mkdir src
$ touch src/app.js
$ mkdir src/components
$ touch src/components/menu.js
```

La commande `git add` effectuée sur un répertoire ajoute récursivement au
staging area tous les fichiers se trouvant à n'importe quel niveau sous le
répertoire, d'un coup :

```shell
$ git add src
$
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   src/app.js
        new file:   src/components/menu.js
```

On peut maintenant faire notre troisième commit :

```shell
$ git commit -m "Troisième commit"
[main 2e6fdee] Troisième commit
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 src/app.js
 create mode 100644 src/components/menu.js
```

On peut récupérer le hash de notre troisième et dernier commit en faisant :

```shell
$ git log -1
commit 2e6fdee379fbe0d258b2d5f04e7cdd0b4d5cbccb (HEAD -> main)
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Thu Jan 15 19:05:29 2026 -0500

    Troisième commit
```

Inspectons tout d'abord ce commit :

```shell
$ git cat-file -p 2e6fdee379fbe0d258b2d5f04e7cdd0b4d5cbccb
tree fb1722eff4b73668ddec99b3452d692b1e4aa9ba
parent 34c85abbbbee1c21c34da2b5ed126cb888fcb498
author Christian Jauvin <cjauvin@gmail.com> 1768521929 -0500
committer Christian Jauvin <cjauvin@gmail.com> 1768521929 -0500

Troisième commit
```

Inspectons ensuite son tree (dont le hash est spécifié à la première ligne) :

```shell
$ git ls-tree -r -t fb1722eff4b73668ddec99b3452d692b1e4aa9ba
040000 tree 6edfc5b49bc2a7de3402cd734abfee5c4a0c42bd    src
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    src/app.js
040000 tree e0408d67c1864db2c5bfae47d89933d6c714b810    src/components
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    src/components/menu.js
100644 blob 48bed0c35d92c60539832929142859e3f5ae6eda    toto.txt
```

On constate la nature récursive du tree, qui est lui-même composé de blobs et de
sous-trees.

Il faut comprendre que les trees sont des "objets" git au même sens que les blobs
(seulement d'un type différent), et qu'ils sont stockés sous la forme de fichiers
binaires, au même endroit que les blobs :

```shell
$ ll .git/objects/
total 0
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 19:05 2e/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 14:57 34/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 14:53 38/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 16 15:40 3d/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 14:39 48/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 12:16 4c/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 16 15:40 4d/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 16 15:40 69/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 19:05 6e/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 16 15:37 76/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 16 15:37 82/
drwxr-xr-x@ 4 cjauvin  staff   128B Jan 16 15:37 88/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 12:16 9e/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 16 15:37 c7/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 12:16 cf/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 19:05 e0/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 19:02 e6/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 16 15:37 e7/
drwxr-xr-x@ 3 cjauvin  staff    96B Jan 15 19:05 fb/
drwxr-xr-x@ 2 cjauvin  staff    64B Jan 15 12:16 info/
drwxr-xr-x@ 2 cjauvin  staff    64B Jan 15 12:16 pack/
```

Les objets git sont organisés sur le disque en fonction du préfixe de deux
chiffres de leur hash.

> [!NOTE]
Le diagramme qui suit introduit une idée importante à comprendre avec git : si
cela n'introduit pas d’ambiguïté, on peut référer à un hash (qui est une valeur
très longue et difficile à lire ou manipuler) avec seulement son préfixe, soit
une valeur plus courte. On peut ainsi faire référence à
`fb1722eff4b73668ddec99b3452d692b1e4aa9ba` avec son préfixe plus court des 6
premiers chiffres, soit `fb1722`, car il n'y a aucun autre objet, dans notre
dépôt git, qui a ce même préfixe (si c'était le cas, git produirait un message
d'erreur pour s'en plaindre). Ce préfixe peut être d'une longueur autre que 6,
mais on préfère en général une longueur qui maximise la lisibilité, selon le
contexte. Il est à noter que ce système de préfixe simplifié fonctionne
également avec tous les outils git de la ligne de commande.

{{< image src="git-tree.png" alt="Une arborescence git" title="" loading="lazy" >}}

La notion d'arbre git permet d'introduire une notion importante, fondamentale à
git : [l'arbre de Merkle](https://fr.wikipedia.org/wiki/Arbre_de_Merkle). Un
arbre de Merkle est un type d'arbre particulier où chaque noeud a un hash qui
est _fonction de_ (qui inclut, ou prend en considération, donc) tout ce qui est
"dessous". Si on change un élément de l'arbre (un blob par exemple) alors
l'effet est que les hash de tous les éléments "au-dessus" (les ancêtres) vont
devoir changer aussi (car leur valeur dépend de tout ce qu'ils ont sous eux).
Examinons un exemple concret pour s'en convaincre, modifions tout d'abord un
fichier au milieu de notre arborescence. Profitons-en pour apprendre que l'ajout
de `-a` à la commande `git commit` (en plus de `-m` qu'on avait déjà, pour faire
donc `-am`) permet de sauter l'étape d'ajouter le changement au staging area
(`git add`) quand on veut aller un peu plus rapidement :

```shell
$ echo "alert('allo!')" >> src/app.js
$
$ git commit -am "Quatrième commit"
[main 3dd71fb] Quatrième commit
 1 file changed, 1 insertion(+)
```

> [!NOTE]
Il est intéressant de noter en passant que l'arbre de Merkle est la
structure de données qui est au coeur de la cryptomonnaie (comme Bitcoin) et de
la chaîne de blocs (blockchain). Git et les cryptomonnaies constituent donc deux
exemples fameux et extrêmement influents de cette technologie particulière.

Notons tout d'abord le hash du commit qu'on vient de produire :

```shell
$ git log -1
commit 3dd71fb4950c1be35ec335cd0ac40f3b5807303b (HEAD -> main)
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Fri Jan 16 15:40:53 2026 -0500

    Quatrième commit
```

Examinons ensuite le commit et son tree, et notons qu'on peut utiliser son préfixe court de 4 chiffres (`3dd7`) car il n'est pas ambigu :

```shell
$ git cat-file -p 3dd7
tree 6954fc566d4abe842409c9265ca5bb94044af1c4
parent 2e6fdee379fbe0d258b2d5f04e7cdd0b4d5cbccb
author Christian Jauvin <cjauvin@gmail.com> 1768596053 -0500
committer Christian Jauvin <cjauvin@gmail.com> 1768596053 -0500

Quatrième commit
$
$ git ls-tree -r -t 6954
040000 tree 4d74086f34491859490bd4a3695876e23798b8f0    src
100644 blob 76aa73d7349a507efaf3206cd21428d4452eae9e    src/app.js
040000 tree e0408d67c1864db2c5bfae47d89933d6c714b810    src/components
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    src/components/menu.js
100644 blob 48bed0c35d92c60539832929142859e3f5ae6eda    toto.txt
```

On peut constater le fait intéressant que seuls les arbres `6954fc` et `4d74086` ont changés,
parce qu'ils sont les ancêtres du blob `76aa73`, que notre commit a changé. On constate
donc clairement qu'un arbre git est un arbre de Merkle au sens classique.

{{< image src="git-tree2.png" alt="Une arborescence git" title="" loading="lazy" >}}

#### Troisième type d'objet git fondamental : le commit

Tournons maintenant notre attention vers les commits, qui sont le troisième type
d'objet git fondamental.

Chaque fois qu'un commit est créé (avec la commande `git commit`), un nouvel
objet (de type commit) est ajouté à la base de données de git.

Examinons encore une fois notre commit le plus récent :

```shell
$ git cat-file -p 3dd7
tree 6954fc566d4abe842409c9265ca5bb94044af1c4
parent 2e6fdee379fbe0d258b2d5f04e7cdd0b4d5cbccb
author Christian Jauvin <cjauvin@gmail.com> 1768596053 -0500
committer Christian Jauvin <cjauvin@gmail.com> 1768596053 -0500

Quatrième commit
```

Un commit a tout d'abord un hash (comme tous les objets git) : `3dd71fb4950c1be35ec335cd0ac40f3b5807303b`,
ou encore `3dd7`, son préfixe équivalent, plus facile à manipuler. On constate ensuite que
le commit contient trois choses :

1. Un pointeur (identifiant, sous la forme d'un hash), `6954`, vers un arbre qui
   représente (de manière récursive) les fichiers du dépôt dans un état
   particulier, un moment dans le temps.

2. Un pointeur (c-à-d un hash) vers le commit _parent_ (le commit qui a précédé
   ce commit particulier, soit `2e6f`). Nous verrons qu'il est possible pour un commit
   d'avoir plusieurs parents, dans certaines circonstances.

3. Des métadonnées diverses, comme la date et l'heure de création du commit,
   l'adresse de courriel du committeur, etc.

La présence du pointeur vers un commit parent suggère un concept fondamental
avec git : les commits forment une chaîne :

{{< image src="git-chain.png" alt="" title="" loading="lazy" >}}

Chaque commit a donc un hash qui "pointe" vers son commit prédécesseur (nous
verrons plus loin qu'il est possible pour un commit d'avoir plus d'un parents,
et en quoi c'est utile), pour former une chaîne.

Jusqu'ici, git ne nous permet que d'évoluer de manière linéaire, étant donné que
n'avons qu'une chaîne de commits. Pourtant, le développement logiciel, surtout
s'il est effectué par une équipe, est tout sauf linéaire. Des embranchements
peuvent être nécessaires dans le processus d'évolution du code source. Pour
illustrer cela, imaginons que les 4 commits que nous avons jusqu'à maintenant
correspondent à la version 1 (v1) d'une application qu'on a mise en ligne, et
qui est utilisée par le grand public. Mais étant donné qu'on perd rarement
rarement du temps en développement logiciel, le travail a déjà commencé pour
concevoir la version 2 du même logiciel (qu'on prévoit mettre en ligne plus
tard). Deux commits ont donc été ajoutés à notre chaîne, qui constituent le
début du travail sur la version 2. Notez qu'à partir d'ici, nous allons utiliser
des noms de commit plus simples pour les diagrammes, donc le commit `C1` va
correspondre `9e60`, `C2` à `34c8`, et ainsi de suite. Nos deux nouveaux commits
pour la version 2 de l'application seraient donc `C5` et `C6`

{{< image src="git-linear-versions.png" alt="" title="" loading="lazy" >}}

Maintenant imaginons ce qui se passerait si un bogue était découvert dans la
version 1, présentement en ligne (et qui correspond au commit `C4` (`3dd7`) dans
notre exemple) :

{{< image src="git-linear-versions-bug.png" alt="" title="" loading="lazy" >}}

Le problème est donc que la chaîne de commits est déjà rendue plus loin, au
deuxième commit de la version 2, soit le commit `C6` ! Il faudrait produire un
commit `C7`, qui serait une correction de `C4`, mais qui n’inclurait pas les
changements introduits par la version 2 (`C5` et `C6`). Comment serait-il
possible de résoudre un tel problème. Arrêtez-vous un instant pour y réfléchir
si vous le désirez, avant de prendre connaissance de la réponse, car il s'agit
d'un problème fondamental et intéressant !

#### La branche

La façon dont git permet de résoudre ce problème est à l'aide d'une _branche_ :

{{< image src="git-with-branch-bugfix.png" alt="" title="" loading="lazy" >}}

Donc au moment où le travail sur la version 1 est complété (au commit `C4`
donc), on doit créer une nouvelle branche (nommée `v2`), pour le travail qui
doit être effectué sur la nouvelle version 2. De cette manière, quand un
problème survient avec la version 1, il sera toujours possible de le résoudre en
ajoutant simplement un commit (`C7`) à la branche `main`, correspondant à la
version 1.

Il est souvent dit que créer une branche est sans contredit une des opérations
les plus simples et les moins coûteuses de git :

```shell
$ git branch v2
```

En comparaison, les systèmes de versioning plus anciens que git faisaient en
sorte que cette opération était beaucoup plus coûteuse et complexe. La raison
qui fait en sorte que la branche est si simple, avec git, est le fait qu'elle
est techniquement extrêmement simple : ce n'est même pas un "objet fondamental"
au sens où le blob, le tree et le commit le sont.. il s'agit plutôt simplement
d'un fichier qui "pointe" vers le hash d'un commit particulier :

```shell
$ cat .git/refs/heads/v2
3dd71fb4950c1be35ec335cd0ac40f3b5807303b
```

On voit ici que le fichier `.git/refs/heads/v2`, dont le nom correspond à notre
nouvelle branche `v2`, contient simplement le commit `3dd7` (ou encore, `C4`,
dans nos diagrammes, pour simplifier), qui est l'endroit de départ (ou
l'embranchement) pour notre nouvelle branche.

Mais si une branche est si simple, qu'elle n'est qu'une étiquette qui pointe
vers un commit, comment peut-elle fonctionner? Nous sommes présentement dans
l'état suivant :

{{< image src="git-just-2-branches.png" alt="" title="" loading="lazy" >}}

Si nous ajoutons un nouveau commit dans cet état, comment git fera pour savoir dans
quelle branche l'ajouter? Nous devons donc introduire un nouveau concept : `HEAD`, qui
est un fichier qui "pointe" vers la branche active, celle dont nous voulous nous servir :

```shell
$ cat .git/HEAD
ref: refs/heads/main
```

Voici donc la représentation réelle de notre état actuel :

{{< image src="git-just-2-branches-with-head.png" alt="" title="" loading="lazy" >}}

Git manipule donc deux types de pointeur :

1. Un pointeur vers un commit (soit une branche)
2. Un pointeur vers une branche (soit `HEAD`)

Si on utilise cette commande :

```shell
$ git branch
* main
  v2
```

git nous dit que la branche courante est toujours `main` (qui est la branche par
défaut, créée au moment de la création du dépôt), avec la petite étoile (`*`).
Si nous ajoutions un commit à ce moment-ci, il serait donc dans la branche
`main`. Mais comme nous voulons maintenant travailler dans la branche `v2`, nous
devons utiliser cette nouvelle commande :

```shell
$ git switch v2
Switched to branch 'v2'
$
$ git branch
  main
* v2
```

pour nous retrouver maintenant dans cet état (avec lequel le pointeur `HEAD` pointe simplement
vers la nouvelle branche `v2`) :

{{< image src="git-just-2-branches-with-head-to-v2.png" alt="" title="" loading="lazy" >}}

Continuons donc notre expérience de pensée, et imaginons que nous sommes en train de travailler
dans le contexte de la version 2, et que nous ajoutons deux nouveaux commits (`C5` et `C6` dans notre
notation simplifiée pour les diagrammes) :

```shell
$ echo "aa" >> toto.txt
$ git commit -am "Cinquième commit (premier de v2)"
[v2 b1d097c] Cinquième commit (premier de v2)
 1 file changed, 1 insertion(+)
$
$ echo "bb" >> toto.txt
$
$ git commit -am "Sixième commit (deuxième de v2)"
[v2 daa06a6] Sixième commit (deuxième de v2)
 1 file changed, 1 insertion(+)
 ```

Notre dépôt est maintenant comme ceci :

{{< image src="git-two-new-commits.png" alt="" title="" loading="lazy" >}}

Notez que la structure du diagramme pourrait aisément suggérer que nous sommes
toujours dans une structure linéaire (une chaîne), et on pourrait se demander :
en quoi est-ce différent du modèle précédent, à une seule branche? La clé est
d'avoir le bon modèle mental : une branche n'est qu'un pointeur ! Et notre
diagramme montre que nous avons deux pointeurs, donc deux branches : `main` et
`v2`. Les commits `C1` à `C4` sont communs aux deux branches, et les commits
`C5` et `C6` appartiennent seulement à la branche `v2`. Et comme `HEAD` montre
(pointe) que nous nous trouvons sur la branche `v2`, tout nouveau commit s'y
retrouverait également.

Si on reprend le cours de notre expérience de pensée, on arrive au moment où un
bogue serait découvert avec la version 1 ! Que peut-on faire, avec la solution? Si
on ne fait qu'ajouter un commit (pour la solution), elle se retrouvera à suivre `C6` dans la
branche `v2`, ce qui n'est pas ce que l'on veut (car la version 1 n'inclut pas encore les nouveautés
de la version 2). La solution est donc de changer tout d'abord de branche :

```shell
$ git switch main
Switched to branch 'main'
$
$ git branch
* main
  v2
```

ce qui ne fait que déplacer le pointeur `HEAD` vers la branche `main`, là où se trouve notre
fameux bogue :

{{< image src="git-move-head-back-to-main.png" alt="" title="" loading="lazy" >}}

Nous sommes donc maintenant en position de fournir la solution à notre bogue, et
de faire en sorte qu'elle se retrouve à l'endroit logique là où elle devrait
être :

```shell
$ echo "solution" >> toto.txt
$
$ git commit -am "Septième commit (bugfix v1)"
[main f229f82] Septième commit (bugfix v1)
 1 file changed, 1 insertion(+)
```

Et avec ceci, nous nous retrouvons donc dans la situation suivante :

{{< image src="git-c7-bugfix.png" alt="" title="" loading="lazy" >}}

qui correspond exactement à la configuration souhaitée que nous avons décrite
plus haut (prenez le temps de vous en convaincre, car c'est important).

#### Le merge (de deux branches)

Poursuivons notre scénario imaginé : une fois le bogue dans la version 1
corrigé, il se trouve que la version 2 est maintenant prête, et nous aimerions
la mettre en ligne, pour que les utilisateurs puissent l'utiliser. Ceci veut
donc dire que nous aimerions que le travail effectué dans les commits `C5` et
`C6`, de la branche `v2`, soit intégré à la version 1, dont l'état le plus
récent se trouve dans notre commit actuel, `C7`. La commande pour faire cela
est `git merge` :

```shell
$ git merge v2
Auto-merging toto.txt
CONFLICT (content): Merge conflict in toto.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Que s'est-il passé? Il semble y avoir un problème ! Le merge a causé un
"conflit", une incohérence entre la version du fichier `toto.txt` de la branche
`v2`, et celle de la branche `main`. Git nous informe qu'il n'a pas pu résoudre
lui-même ce conflit (très souvent il est en mesure de le faire, mais pas cette
fois) et que nous devons donc intervenir manuellement, avant de pouvoir
poursuivre. La commande `git diff` permet de mieux comprendre le problème :

```$ git diff
diff --cc toto.txt
index 650d061,d521aa0..0000000
--- a/toto.txt
+++ b/toto.txt
@@@ -1,3 -1,4 +1,8 @@@
  allo!
  bonjour
++<<<<<<< HEAD
 +solution
++=======
+ aa
+ bb
++>>>>>>> v2
```

La syntaxe de cette sortie n'est pas si évidente, mais en gros, ce qui se trouve
entre `<<<<<<< HEAD` et `=======` correspond à une version (de la branche
`main`), et ce qui se trouve entre `=======` et `>>>>>>>` à l'autre version (de
la branche `v2`). Ces deux parties sont en contradiction, dans le sens qu'elles
occupent le même endroit dans le fichier, et git ne sait pas comment choisir. Il
faut donc éditer le fichier nous-même, et décider ce qu'on veut garder.

{{< image src="git-conflict.png" alt="" title="" loading="lazy" >}}

Dans ce cas particulier, éditez le fichier (vous pouvez par exemple utiliser les
éditeurs `vim` ou `nano`, sur la ligne de commande) pour qu'il ait l'air de ceci
:

```shell
$ cat toto.txt
allo!
bonjour
solution
aa
bb
```

Une fois le fichier modifié, git nous informe très clairement de la situation :

```shell
$ git status
On branch main
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   toto.txt
```

Il faut donc ajouter le fichier dont on a résolu les conflits, et ensuite on peut
faire notre commit, ce qui complétera le merge :

```shell
$ git add toto.txt
$
$ git commit -m "Huitième commit (merge)"
[main 55f1243] Huitième commit (merge)
```

{{< image src="git-merge.png" alt="" title="" loading="lazy" >}}

Si on utilise maintenant la commande `git log`, la totalité de l'évolution de la
branche `main` (incluant son passage temporaire dans la branche `v2`) devient
claire :

```shell
$ git log
commit 55f1243bb2bd319d2bfefc8e9040dba986456285 (HEAD -> main)
Merge: f229f82 daa06a6
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Wed Jan 21 09:40:53 2026 -0500

    Huitième commit (merge)

commit f229f82b53ca118d69daa1cccbbe7a296da87758
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Tue Jan 20 11:15:54 2026 -0500

    Septième commit (bugfix v1)

commit daa06a6ceb25c2d008b491b1faba20ce4f5b16d5 (v2)
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Tue Jan 20 10:45:38 2026 -0500

    Sixième commit (deuxième de v2)

commit b1d097c40b6b69dc342cdac2bb09c11bbe853fc0
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Tue Jan 20 10:45:19 2026 -0500

    Cinquième commit (premier de v2)

commit 3dd71fb4950c1be35ec335cd0ac40f3b5807303b
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Fri Jan 16 15:40:53 2026 -0500

    Quatrième commit

commit 2e6fdee379fbe0d258b2d5f04e7cdd0b4d5cbccb
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Thu Jan 15 19:05:29 2026 -0500

    Troisième commit

commit 34c85abbbbee1c21c34da2b5ed126cb888fcb498
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Thu Jan 15 14:57:15 2026 -0500

    Deuxième commit

commit 9e6072c8df904f021474a6173cc1b17b2908c0f1
Author: Christian Jauvin <cjauvin@gmail.com>
Date:   Thu Jan 15 12:16:21 2026 -0500

    Premier commit
```

Si on examine plus attentivement notre dernier commit (`C8` dans le diagramme), on
constate un détail intéressant :

```shell
$ git cat-file -p 55f1
tree d9b5afa4d6f37608b115b489fbca9ac5eaa4b67b
parent f229f82b53ca118d69daa1cccbbe7a296da87758
parent daa06a6ceb25c2d008b491b1faba20ce4f5b16d5
author Christian Jauvin <cjauvin@gmail.com> 1769006453 -0500
committer Christian Jauvin <cjauvin@gmail.com> 1769006453 -0500

Huitième commit (merge)
```

Ce commit a donc deux pointeurs parents (`C6` et `C7`) ! Ceci est dû au fait
qu'il est un commit de merge. Sur le diagramme, ceci correspond au fait que deux
flèches partent de `C8`, au lieu d'une seule, comme les autres commits.

> [!NOTE]
Il est utile de savoir que cet aspect structural de l'évolution des commits d'un
dépôt git forme un [graphe orienté
acyclique](https://fr.wikipedia.org/wiki/Graphe_orient%C3%A9_acyclique) (ou plus
communément un DAG, en anglais). Ceci est une structure intermédiaire entre un
arbre (dont les noeuds ne peuvent avoir qu'un seul parent) et une graphe
général, où les noeuds peuvent avoir plusieurs parents et plusieurs enfants). Un
DAG ne permet donc pas les "circuits".

<!--
{{< applet src="/html/applets/git.html" width="140%" scale="1.0" >}}
-->

