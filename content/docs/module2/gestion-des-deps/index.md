---
title: "La gestion des dépendances"
weight: 40
---

# La gestion des dépendances

Dès qu'un programme devient un peu plus complexe, deux phénomènes interviennent
habituellement&nbsp;:

1. Le programme doit être décomposé en plusieurs modules
2. Certaines fonctionnalités du programmes, pouvant être accomplies par des
   programmes (ou des librairies) externes, doivent être détachées en des
   composantes ou des librairies distinctes, qu'il est possible de réutiliser
   (c'est un peu le [principe
   DRY](../module2/principes#dry-dont-repeat-yourself-ne-vous-répétez-pas-) que
   nous avons vu, appliqué dans le sens plus large d'un écosystème logiciel)

Dans le cas (2), on nomme parfois les librairies externes des "dépendances", ou des
"paquets" (packages en anglais, plus communément).

Dans les débuts de l'ingénierie logicielle, ces dépendances étaient gérées à la
main, en copiant et échangeant des fichiers.

Avec l'apparition de Linux au début des années 90, une innovation importante
allait vite apparaître : les gestionnaire de paquets. Des exemples fameux sont
[APT](https://fr.wikipedia.org/wiki/Advanced_Packaging_Tool) pour les
distributions Debian et Ubuntu de Linux, et
[RPM](https://fr.wikipedia.org/wiki/RPM_Package_Manager) pour d'autres
distributions comme Red Hat, Fedora, etc.

Ces paquets sont en général des programmes et des librairies qui sont
compatibles avec une version particulière de Linux et des sous-systèmes qui sont
présents dans une distribution particulière. Par exemple si on utilise Ubuntu
24.04, une version particulière d'une distribution particulière de Linux basée
sur Debian, le gestionnaire de paquets `apt` ne permettra que d'installer des
programmes compatibles avec Ubuntu 24.04. Un aspect important des systèmes de
gestion de ce genre est que les dépendances ne sont pas des objets atomiques et
isolées : en général une dépendance est liée à une autre, ce qui forme un graphe
de dépendances, qu'on dit transitives, ou récursives (si `X` dépend de `Y` qui
dépend de `Z`, alors `X` dépend de `Z`, de manière transitive). Donc non
seulement la compatibilité des versions est gérée au niveau du système global,
mais aussi entre les paquets (programmes ou librairies, entre eux). Si par
exemple l'installation de la version 7 de LibreOffice nécessite une version
particulière de Java, le gestionnaire de paquets gérera les dépendances
intelligemment, et automatiquement.

Par exemple on peut voir le graphe de dépendances du paquet `sl` sur Ubuntu,
en utilisant l'utilitaire `apt-rdepends` :

```shell
$ apt-rdepends sl
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
sl
  Depends: libc6 (>= 2.2.5)
  Depends: libncurses6 (>= 6)
  Depends: libtinfo6 (>= 6)
libc6
  Depends: libgcc-s1
libgcc-s1
  Depends: gcc-14-base (= 14.2.0-4ubuntu2~24.04)
  Depends: libc6 (>= 2.35)
gcc-14-base
libncurses6
  Depends: libc6 (>= 2.34)
  Depends: libtinfo6 (= 6.4+20240113-1ubuntu2)
libtinfo6
  Depends: libc6 (>= 2.34)
  ```

{{< image src="sl-deps.png" alt="" title="" loading="lazy" >}}

## La sécurité

Les gestionnaires de paquets jouent aujourd’hui un rôle central dans le
développement logiciel, mais ils introduisent également des enjeux importants de
sécurité. Lorsqu’un projet déclare une dépendance, il ne fait pas seulement
confiance à une bibliothèque précise, mais aussi à l’ensemble de ses dépendances
transitives, parfois très nombreuses, ainsi qu’aux personnes et aux
infrastructures qui les distribuent. Cette situation élargit considérablement la
surface d’attaque potentielle : un paquet compromis, un mainteneur malveillant,
une prise de contrôle d’un compte ou encore une simple faute de frappe dans le
nom d’une librairie (typosquatting) peuvent entraîner l’intégration de code
hostile dans un système sans que l’équipe de développement ne s’en aperçoive
immédiatement. Pour répondre à ces risques, les écosystèmes modernes proposent
divers mécanismes comme les audits automatiques de vulnérabilités connues, les
signatures cryptographiques, la vérification d’intégrité et la production
d’inventaires de composants (SBOM). Comprendre ces mécanismes fait désormais
partie des compétences essentielles du génie logiciel, car la gestion des
dépendances n’est plus seulement une question de commodité, mais aussi de
responsabilité opérationnelle et parfois légale. Les gestionnaires de
dépendances permettent

## Le versionnage sémantique (SemVer)

Au fil du temps et de l'évolution de la culture du développement logiciel, le
versionnage dit _sémantique_ (SemVer) s'est imposé en tant que convention pour
numéroter les versions d'un logiciel, que ce soit un programme ou une librairie.

{{< image src="semver.png" alt="" title="" loading="lazy" >}}

Bien que l'interprétation ne soit pas toujours identique, en général on
s'accorde pour dire que la composante "majeure" du numéro de version correspond
à un changement important, qui "brise" la compatibilité avec les versions
précédentes (s'il y en a). S'il s'agit d'une librairie logicielle par exemple,
la mise à jour demandera probablement un travail assez important, non-trivial.

Le deuxième nombre, la version mineure, indique l’ajout de nouvelles
fonctionnalités qui demeurent compatibles avec l’existant. Le logiciel
s’enrichit, mais sans obliger les utilisateurs à modifier leur code. Enfin, le
troisième nombre, appelé correctif ou patch, correspond à des réparations
internes : corrections de bugs, améliorations de performance ou failles de
sécurité qui ne modifient pas l’interface offerte aux autres programmes.

Grâce à cette structure simple, un numéro de version devient une véritable
promesse. Il permet aux équipes de décider automatiquement quelles mises à jour
peuvent être installées sans risque et lesquelles exigent une attention
particulière. Les gestionnaires de dépendances modernes s’appuient largement sur
cette logique pour éviter les surprises. Sans une convention partagée, une
nouvelle version pourrait tout aussi bien contenir une amélioration bénigne
qu’un changement radical rendant l’application inutilisable.

Adopter SemVer revient donc à instaurer une relation de confiance entre les
mainteneurs d’un logiciel et celles et ceux qui l’intègrent dans leurs propres
projets. Le numéro n’est plus simplement un identifiant : il devient une
information sur la stabilité, la continuité et l’effort requis pour évoluer avec
le produit.

{{< image src="versions.png" alt="" title="" loading="lazy" >}}

Il est à noter que bien que SemVer soit le schéma le plus répandu, il en existe
d'autres : Ubuntu par exemple utilise des numéros de version qui correspondent à
l'année et au mois de livraison d'une certaine version.

Il y a aussi une notion assez populaire, qui consiste à considérer qu'un
logiciel ou une librairie ne peut pas être considérée _stable_ tant qu'elle n'a
pas atteint au moins la version `1.0.0`. Certains projets restent d'ailleurs
indéfiniment dans un schéma de versions `0.x.y`, afin d'échapper à une certaine
pression sociale, et ainsi pouvoir conserver une certaine liberté créative et
d'action.

Il est à noter que pour les langages de programmation (qui sont à la fois des
programmes et des librairies), la question du numéro de version est
particulièrement cruciale et parfois même controversée. Un cas fameux est la
transition de la version 2 du langage Python, à la version 3, qui était prévue
devoir être relativement simple et rapide, mais qui a pris au moins 10 ans à
intervenir, étant donné toute la complexité technique et sociale que cette
transition a entraînée. En fait il est très probable que Python ne passe jamais
à la version 4, étant donné son historique et sa position centrale dans de
nombreux écosystèmes technologiques (web, IA, calcul scientifique, etc).

La notion de versionnage sémantique est tellement importante et omniprésente
qu'elle a de l'influence à l'extérieur de la sphère du développement logiciel.
On parlera ainsi de la Playstation 5, de la Mazda 3, etc. Même des choses qui
pourraient à priori ressembler à du logiciel, comme le [Web
2.0](https://fr.wikipedia.org/wiki/Web_2.0) et le
[Web3](https://fr.wikipedia.org/wiki/Web3), n'en sont pas vraiment, ils sont
plus des "évolutions culturelles et technologiques".

## Le gestionnaire `uv` pour Python

Pour explorer concrètement ces idées, nous allons utiliser le gestionnaire de
librairies `uv`, pour Python. `uv` est un outil très intéressant, car il est
apparu relativement tard dans l'histoire de Python (en 2024), à la suite d'une
longue lignée d'outils du même genre : `pipenv`, `poetry`, etc. Ces outils ne
doivent pas être confondus avec `pip`, qui est l'outil de base dans la
distribution Python, qui est moins puissant que les gestionnaires de
dépendances, dont il sera question ici. `uv` est apparu au bon moment, avec les
bonnes caractéristiques : extrêmement rapide (il est écrit lui-même dans le
langage Rust), avec une implémentation très complète et flexible des standards
de l'écosystème de packaging pour Python, qui ont pris un long moment avant de
maturer. Pendant de nombreuses années, la gestion du packaging en Python était
considéré un sujet pénible et beaucoup de controverse existait. L'apparition de
`uv` a introduit une certaine sérénité dans la culture de Python.

### La librairie `my-lib`

Nous allons tout d'abord créer, avec `uv`, une petite librairie simple, qui
n'offrira qu'une seule fonction&nbsp;:

```shell
$ uv init --lib my-lib
Initialized project `my-lib` at `.../uv-demo2/my-lib`
```

La structure initiale de notre nouveau projet de librairie devrait ressembler à
ceci :

```
my-lib/
├── pyproject.toml
├── src/
│   └── my_lib/
│       └── __init__.py
```

`uv` a créé la structure de notre librairie, mais il n'a évidemment pas fourni
le code qu'on veut y offrir, qu'il nous faut définir nous-même, dans le fichier
`src/my_lib/secret.py` (qui doit être créé) :

```python
def get_secret_number():
   return 42
```

Le projet ressemble donc maintenant à ceci :

```
my-lib/
├── pyproject.toml
├── src/
│   └── my_lib/
│       ├── __init__.py
│       └── secret.py      ← contient notre fonction get_secret_number
```

On peut tout d'abord tester notre librairie avec cette commande :

```shell
$ cd my-lib
$ uv run python -c 'from my_lib.secret import get_secret_number; print(get_secret_number())'
42
```

On doit ensuite produire un artefact de type `wheel`, qui va contenir la totalité
de notre librairie, en un seul fichier `.whl` (qu'il sera possible d'installer dans un
autre projet)&nbsp;:

```shell
$ cd my-lib
$ uv build --out-dir ../packages
Building source distribution (uv build backend)...
Building wheel from source distribution (uv build backend)...
Successfully built ../packages/my_lib-0.1.0.tar.gz
Successfully built ../packages/my_lib-0.1.0-py3-none-any.whl
```

Voici maintenant où nous en sommes&nbsp;:
```shell
$ tree
.
├── my-lib
│   ├── pyproject.toml
│   ├── README.md
│   ├── src
│   │   └── my_lib
│   │       ├── __init__.py
│   │       ├── __pycache__
│   │       │   ├── __init__.cpython-313.pyc
│   │       │   └── secret.cpython-313.pyc
│   │       ├── py.typed
│   │       └── secret.py
│   └── uv.lock
└── packages
    ├── my_lib-0.1.0-py3-none-any.whl
    └── my_lib-0.1.0.tar.gz

6 directories, 10 files
```

{{< image src="mylib.png" alt="" title="" loading="lazy" >}}

### L'application `my-app` (qui utilise la librairie `my-lib`)

Créons maintenant un nouveau projet avec `uv`, d'une application cette fois, que
nous appellerons `my-app` :

```shell
$ cd ..  # on doit être au même niveau que my-lib
$ uv init my-app
Initialized project `my-app` at `.../uv-demo2/my-app`
```

Étant donné que `my-app` devra utiliser le code de la librairie `my-lib`, nous
voudrons y ajouter une dépendance vers notre librairie `my-lib`. Mais tout
d'abord, étant donné que ceci est un exercice d'apprentissage, nous devons
modifier notre configuration quelque peu, en ajoutant le bloc `[tool.uv]` qui
contient deux lignes supplémentaires, à notre fichier `my-app/pyproject.toml` :

```toml {hl_lines="9-11"}
[project]
name = "my-app"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = []

[tool.uv]
no-index = true
find-links = ["../packages"]
```

Ces deux lignes sont très importantes dans notre contexte :

1. `no-index` empêche `uv` d'aller chercher `my-lib` sur registre de paquets public [PyPI](https://pypi.org), qui contient déjà apparemment une [librairie avec ce nom](https://pypi.org/project/my-lib/). Nous voulons que `uv` utilise notre version locale de `my-lib`, que nous avons sur notre propre disque
2. `find-links` indique le lieu où `uv` devra trouver les fichiers "wheels", qui sont des sortes de "zip" d'une librairie entière, optimisée pour un système particulier

Une fois cette configuration effectuée, on peut ajouter notre dépendance avec `uv add` :

```shell
$ cd my-app
$ uv add my-lib
Using CPython 3.13.5
Creating virtual environment at: .venv
Resolved 2 packages in 26ms
Prepared 1 package in 4ms
Installed 1 package in 3ms
 + my-lib==0.1.0
```

On peut constater l'état du projet de notre application avec cette commande :

```shell
$ uv tree
Resolved 2 packages in 3ms
my-app v0.1.0
└── my-lib v0.1.0
```

#### La notion de registre de paquets (PyPI)

Nous avons brièvement mentionné ci-haut la notion de _registre de paquets_.
Étant donné que nous voulions utiliser notre propre librairie `my-lib`, et non
celle qui se trouve en ligne, nous avons dû spécifier `no-index = true` dans la
configuration `pyproject.toml` de notre projet. Que se serait-il passé, sans
cette instruction particulière? `uv` est configuré, par défaut, pour fonctionner
avec le registre en ligne et officiel [PyPI](https://pypi.org), qui est un
endroit public où il est possible de télécharger et publier des paquets.
Explorons cette notion en créant un nouveau projet avec `uv` :

```shell
$ cd ..  # on ne doit pas être dans le répertoire d'un projet existant
$ uv init my-venv
Initialized project `my-venv` at `/Users/cjauvin/gh/inf1410-teluq/content/docs/module3/gestion-des-deps/uv-demo3/my-venv`
```

Ajoutons une dépendance vers la librairie [requests](https://pypi.org/project/requests/),
qui permet de faire des requêtes HTTP (web) en python :

```shell
$ cd my-venv
$ uv add requests
Using CPython 3.13.5
Creating virtual environment at: .venv
Resolved 6 packages in 183ms
Installed 5 packages in 4ms
 + certifi==2026.1.4
 + charset-normalizer==3.4.4
 + idna==3.11
 + requests==2.32.5
 + urllib3==2.6.3
```

On constate trois choses :

1. `uv` a téléchargé automatiquement la librairie `requests` de PyPI, sans que l'ait rien configuré
2. La librairie `requests` elle-même nécessite quelques dépendances additionnelles (des dépendances transitives donc) : `certifi`, `charset-normalizer`, etc.
3. Apparemment un "environnement virtuel" a été créé, dans le répertoire `.venv` de notre projet

Qu'est-ce qu'un environnement virtuel donc?

#### La notion d'environnement virtuel (virtual env, ou `venv`)

En fait une question reliée serait : qu'est-ce qu'un projet `uv`, au juste?
Quand j'ajoute une dépendance dans mon projet, quel est l'effet concret, et
comment puis-le constater? Voyons voir le contenu du répertoire du projet
dans lequel nous venons d'installer une dépendance :
:

```shell
$ cd my-venv
$ ls -la
total 24
drwxr-xr-x@ 8 cjauvin  staff  256 Feb 16 17:13 ./
drwxr-xr-x@ 5 cjauvin  staff  160 Feb 16 17:14 ../
-rw-r--r--@ 1 cjauvin  staff    5 Feb 12 14:55 .python-version
drwxr-xr-x@ 8 cjauvin  staff  256 Feb 12 14:58 .venv/
-rw-r--r--@ 1 cjauvin  staff  358 Feb 16 17:12 pyproject.toml
-rw-r--r--@ 1 cjauvin  staff    0 Feb 12 14:55 README.md
drwxr-xr-x@ 3 cjauvin  staff   96 Feb 12 14:55 src/
-rw-r--r--@ 1 cjauvin  staff  254 Feb 16 17:13 uv.lock
```

On constate un répertoire particulier, appelé `.venv` : il s'agit d'un
environnement virtuel (souvent appelé `venv`), un endroit particulier sur le
disque qui contient :

1. Une version particulière de l'interpréteur Python (par exemple la version
   3.13, dans cet exemple), ainsi qu'une série d'utilitaires reliés
2. Une série de paquets (packages) qui ont été installés dans le venv (dans le
   contexte de ce projet, un seul a été installé : `requests`)

Il est crucial de savoir et de comprendre que les paquets installés dans un venv
particulier fonctionnent exclusivement dans le contexte de l'interpréteur python
particulier, installé dans ce venv. On peut s'en convaincre avec cette commande :

```shell
$ ll .venv/lib/python3.13/site-packages/
total 24
-rw-r--r--@  1 cjauvin  staff    18B Feb 17 15:46 _virtualenv.pth
-rw-r--r--@  1 cjauvin  staff   4.2K Feb 17 15:46 _virtualenv.py
drwxr-xr-x@  7 cjauvin  staff   224B Feb 17 15:46 certifi/
drwxr-xr-x@  9 cjauvin  staff   288B Feb 17 15:46 certifi-2026.1.4.dist-info/
drwxr-xr-x@ 16 cjauvin  staff   512B Feb 17 15:46 charset_normalizer/
drwxr-xr-x@ 10 cjauvin  staff   320B Feb 17 15:46 charset_normalizer-3.4.4.dist-info/
drwxr-xr-x@ 11 cjauvin  staff   352B Feb 17 15:46 idna/
drwxr-xr-x@  8 cjauvin  staff   256B Feb 17 15:46 idna-3.11.dist-info/
drwxr-xr-x@ 20 cjauvin  staff   640B Feb 17 15:46 requests/
drwxr-xr-x@  9 cjauvin  staff   288B Feb 17 15:46 requests-2.32.5.dist-info/
drwxr-xr-x@ 18 cjauvin  staff   576B Feb 17 15:46 urllib3/
drwxr-xr-x@  8 cjauvin  staff   256B Feb 17 15:46 urllib3-2.6.3.dist-info/
```

Ceci permet un mécanisme d'isolation, qui permet d'éviter les conflits et les
problèmes de compatibilité entre les composantes. Par exemple, si un script de
traitement de données utilise la version 1 de `numpy`, on peut le rouler dans un
venv dont la version de `numpy` est gardée exclusivement à cette version. Si un
autre script utilise la version 2, il peut rouler dans son propre venv. Il est
en de même pour la version de l'interpréteur python utilisée, qui peut varier de
venv en venv, elle aussi.

{{< image src="two-venvs.png" alt="" title="" loading="lazy" >}}

> [!NOTE]
Chaque fois qu'une commande `uv` est exécutée, elle l'est dans le contexte d'un
venv particulier. Si le venv n'existe pas, il sera créé pour l'occasion.

#### La notion de reproductibilité (`uv.lock`)

Un autre fichier qu'il est intéressant de considérer dans notre projet est
`uv.lock`. Ce fichier (qu'on appelle parfois un "lockfile") contient la liste
des dépendances du projet, sous la forme de métadonnées précises et exactes,
permettant de reconstruire de manière parfaite et exhaustive le contenu d'un
venv particulier (c'est une manière de le copier donc, en le reproduisant avec
une recette). Chaque dépendance y est figée dans le temps, à l'aide
d'identifiants et de urls non-ambigus, qui permettent de faire en sorte de
réinstaller exactement le même environnement, dans un endroit différent. Quand
le projet vient d'être créé, `uv.lock` est vide. Mais dans notre cas, étant
donné que nous avons ajouté `requests` au projet, on peut constater, en
l'examinant, qu'il contient une série de références précises vers les artefacts
en ligne des packages correspondant :

```shell
$ cd my-venv
$ cat uv.lock
version = 1
revision = 3
requires-python = ">=3.13"

[[package]]
name = "certifi"
version = "2026.1.4"
source = { registry = "https://pypi.org/simple" }
sdist = { url = "https://files.pythonhosted.org/packages/e0/2d/a891ca51311197f6ad14a7ef42e2399f36cf2f9bd44752b3dc4eab60fdc5/certifi-2026.1.4.tar.gz", hash = "sha256:ac726dd470482006e014ad384921ed6438c457018f4b3d204aea4281258b2120", size = 154268, upload-time = "2026-01-04T02:42:41.825Z" }
wheels = [
    { url = "https://files.pythonhosted.org/packages/e6/ad/3cc14f097111b4de0040c83a525973216457bbeeb63739ef1ed275c1c021/certifi-2026.1.4-py3-none-any.whl", hash = "sha256:9943707519e4add1115f44c2bc244f782c0249876bf51b6599fee1ffbedd685c", size = 152900, upload-time = "2026-01-04T02:42:40.15Z" },
]
...
```

Cette notion de reproductibilité est extrêmement importante dans les
environnements de production, où certaines composantes logicielles sont parfois
installées à répétition, de manière scriptée et automatisée, dans des
environnements éphémères, comme des containers (docker ou autre), dans un
système Kubernetes, ou un environnement pour effectuer des tests.

#### Est-ce sécuritaire?

Qu'arriverait-il si, au lieu d'installer la librairie `requests`, nous tentions
d'installer, par mégarde, `request` (sans `s`)? Voyons voir :

```shell
$ uv add request
  × No solution found when resolving dependencies:
  ╰─▶ Because request was not found in the package registry and your project depends on request, we can
      conclude that your project's requirements are unsatisfiable.
  help: If you want to add the package regardless of the failed resolution, provide the `--frozen` flag
        to skip locking and syncing.
```

Dans ce cas, le comportement souhaité est le bon : PyPI ne reconnaît pas le nom
de cette librairie, et qui plus est, ce qui est le plus important : PyPI ne
permettrait pas à quelqu'un de publier une librairie dont le nom serait trop
proche d'une librairie très connue (`requests` est extrêmement populaire). Un
danger qui nous guette cependant avec les systèmes comme PyPI, est le
[typosquattage](https://fr.wikipedia.org/wiki/Typosquattage), le fait
d'exploiter une variante orthographique proche, pour publier du contenu
malicieux. Il est également possible, dans certains cas, que le compte PyPI
associé à une librairie soit piraté, et que son contenu soit remplacé par une
version malicieuse, qui contient un virus, ou encore un mécanisme pour voler des
données, par exemple.

---

Après ces détours, revenons maintenant à notre application `my-app` : tout comme
nous l'avons fait pour la librairie `my-lib`, c'est à nous qu'incombe la
responsabilité de fournir le code source pour l'application, qui sera très
simple. Dans le fichier `my-app/main.py` (qui a été créé automatiquement par
`uv` puisqu'il s'agissait d'une application), nous devons remplacer le contenu
par :

```python
from my_lib.secret import get_secret_number

def main():
    print(f"The secret number is: {get_secret_number()}")

if __name__ == "__main__":
    main()
```

On peut maintenant tester notre application :

```shell
$ cd my-app
$ uv run main.py
The secret number is: 42
```

{{< image src="mylib+myapp.png" alt="" title="" loading="lazy" >}}

Maintenant regardons attentivement notre fichier `my-app/pyproject.toml` :

```toml {hl_lines="7-9"}
[project]
name = "my-app"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "my-lib>=0.1.0",
]

[tool.uv]
no-index = true
find-links = ["../packages"]
```

On note que `my-app` définie sa dépendance à `my-lib` en utilisant une syntaxe
particulière : `my-lib>=0.1.0`, ce qui signifie évidemment : n'importe quelle
version de `my-lib` dont la version est plus grande ou égale à `0.1.0`. Ceci peut
être dangereux, car que se passerait-il dans le cas où une nouvelle version de `my-lib`
serait publiée, et qu'elle contiendra des changements qui feraient en sorte de modifier
le comportement de `my-app`? Soyons plus conservateur en fixant plus précisément notre
limite de version pour `my-lib`. Pour ce faire, on va utiliser de nouveau cette syntaxe
particulière :

```shell
$ uv add "my-lib>=0.1.0,<0.2.0"
Resolved 2 packages in 7ms
Audited 1 package in 0.28ms
```

> [!NOTE]
Avec `u`v, les contraintes de version servent à indiquer quelles versions d’un paquet peuvent être installées. Les opérateurs classiques `<`, `<=`, `>`, `>=` permettent de fixer des bornes (par exemple `>=1.2.0` signifie « version 1.2.0 ou supérieure »), tandis que `==` impose une version exacte. `uv` suit la spécification PEP 440 de l’écosystème Python : l’opérateur `~=` (compatible release) autorise les mises à jour compatibles selon la version indiquée (par exemple `~=1.4` accepte les versions `1.x` à partir de `1.4`, sans passer à `2.0`). Ces contraintes permettent de contrôler finement la stabilité d’un projet tout en autorisant, si souhaité, certaines mises à jour automatiques lors du verrouillage (uv lock).

> [!NOTE]
La commande `uv add "my-lib~=0.1.0"` serait ici équivalent à `uv add "my-lib>=0.1.0,<0.2.0"`.

On peut maintenant constater l'effet de cette précision dans notre même fichier
`my-app/pyproject.toml` :

```toml {hl_lines="7-9"}
[project]
name = "my-app"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "my-lib>=0.1.0,<0.2.0",
]

[tool.uv]
no-index = true
find-links = ["../packages"]
```

La version de `my-lib` pour `my-map` est "pinnée" (mot anglais) à `0.1.x`, ce
qui veut donc dire que seules les versions avec des "patchs" différents (pour la
résolution de bogues, en général) seront tolérées : par exemple `0.1.1`,
`0.1.2`, etc. La version `0.2.0`, si elle en venait à exister, serait
"interdite" par le gestionnaire `uv` (remarquez bien l'usage de `<0.2.0`, et
non `<=0.2.0`).

### La librairie `my-lib` évolue

Imaginons maintenant que la librairie `my-lib` évolue, et que la fonction
offerte change : par exemple, au lieu d'être 42, le nombre magique devient 99.
Modifions tout d'abord le code de `my-lib/src/secret.py` :

```python
def get_secret_number():
    return 99
```

Ensuite incrémentons la version de `my-lib` :

```shell
$ cd my-lib
$ uv version --bump minor  # 0.1.0 -> 0.2.0
Resolved 1 package in 16ms
      Built my-lib @ file:///Users/cjauvin/gh/inf1410-teluq/content/docs/module3/gestion-de
Prepared 1 package in 7ms
Uninstalled 1 package in 0.67ms
Installed 1 package in 1ms
 - my-lib==0.1.0 (from file:///Users/cjauvin/gh/inf1410-teluq/content/docs/module3/gestion-des-deps/uv-demo3/my-lib)
 + my-lib==0.2.0 (from file:///Users/cjauvin/gh/inf1410-teluq/content/docs/module3/gestion-des-deps/uv-demo3/my-lib)
my-lib 0.1.0 => 0.2.0
```

On peut constater dans notre fichier `my-lib/pyproject.toml` que le changement a
été fait :

```toml {hl_lines="3"}
[project]
name = "my-lib"
version = "0.2.0"
description = "Add your description here"
readme = "README.md"
authors = [
    { name = "Christian Jauvin", email = "cjauvin@gmail.com" }
]
requires-python = ">=3.13"
dependencies = []

[build-system]
requires = ["uv_build>=0.8.22,<0.9.0"]
build-backend = "uv_build"
```

On peut maintenant "builder" notre librairie afin d'en produire un artefact de type
"wheel" pour la nouvelle version `0.2.0`, comme nous avons fait pour la précédente
version `0.1.0` :

```shell
$ uv build --out-dir ../packages
Building source distribution (uv build backend)...
Building wheel from source distribution (uv build backend)...
Successfully built ../packages/my_lib-0.2.0.tar.gz
Successfully built ../packages/my_lib-0.2.0-py3-none-any.whl
```

On constate donc la présence du fichier `../packages/my_lib-0.2.0-py3-none-any.whl`, qui
contient la totalité de notre librairie, prête à être installée dans le contexte de
n'importe quel projet, par `uv`.

### L'application `my-app` devrait donc évoluer elle aussi !

Retournons maintenant à `my-app`. Si on tente de la mettre à jour :

```shell
$ uv sync --upgrade
Resolved 2 packages in 17ms
Audited 1 package in 0.28ms
```

{{< image src="myapp1.png" alt="" title="" loading="lazy" >}}

Rien ne se passe, car bien que `my-lib` version `0.2.0` soit disponible, la
configuration de `my-app` ne lui permet pas d'utiliser une version aussi élevée.
Pour changer cela, on peut utiliser la commande :

```shell
uv add "my-lib>=0.1.0,<=0.2.0"
Resolved 2 packages in 17ms
Audited 1 package in 0.23ms
```

qui devrait maintenant permettre de faire en sorte de laisser passer la version `0.2.0`
de `my-lib` :

```shell
$ uv sync --upgrade
Resolved 2 packages in 26ms
Prepared 1 package in 0.78ms
Uninstalled 1 package in 1ms
Installed 1 package in 1ms
 - my-lib==0.1.0
 + my-lib==0.2.0
```

On peut constater l'effet de cette mise à niveau de `my-lib`, sur notre
application elle-même :

```shell
$ uv run main.py
The secret number is: 99
```

Finalement, étant donné ce changement de comportement important, et pour éviter
de confondre les utilisateurs, on va vouloir mettre à jour la version de
`my-app` elle-même :

```shell
$ uv version --bump major
Resolved 2 packages in 7ms
Audited 1 package in 0.25ms
my-app 0.1.0 => 1.0.0
```

Dans ce cas étant donné qu'il s'agit d'une application, nous avons choisi de
changer la version majeure, qui passe donc à `1.0.0`.

{{< image src="myapp2.png" alt="" title="" loading="lazy" >}}

# Pas seulement pour Python !

Tout ce que nous avons dit et expliqué à propos de `uv` et python s'applique
également à d'autres langages et leur écosystème :

1. JavaScript avec Node et NPM
2. Rust et Cargo
3. Java avec Maven ou Gradle
4. Go avec go mod
5. .NET (C#, etc) et NuGet
6. Perl et CPAN
7. Ruby et Bundler

Pour votre culture personnelle, je vous suggère d'explorer une ou plusieurs de ces "paires",
et de tenter de répondre aux questions suivantes :

1. Est-ce qu'il y a une notion d'environnement virtuel?
2. Est-ce qu'il y a un dépôt centralisé de librairies et de packages (comme PyPI)?
3. Est-ce qu'il y a un mécanisme de lockfile (comme `uv.lock`)?
4. Est-ce que SemVer y est utilisé de la même manière?
5. Est-ce les contraintes au niveau des dépendances sont exprimées de la même
   manière (avec le même "mini-langage")?
6. Est-ce qu'on y retrouve la possibilité du même genre de vulnérabilité au
   niveau de la sécurité (typosquatting, etc)?