---
title: "Les tests"
slug: "tests"
weight: 20
---

# Les tests

## Quel est le problème qu'on cherche à résoudre?

Dans la section précédente, nous avons vu que la programmation repose sur des
structures de données, des algorithmes et des systèmes de types. Mais écrire du
code qui *semble* correct ne suffit pas : il faut pouvoir *vérifier* qu'il se
comporte comme on le pense. C'est exactement ce que Peter Naur décrivait dans
*Programming as Theory Building* : le programmeur construit un modèle mental de
ce que le programme est censé faire. Les tests sont la manière la plus directe
de confronter ce modèle mental à la réalité.

Un compilateur, dans un langage comme C ou Java, détecte automatiquement toute
une classe d'erreurs avant même que le programme ne s'exécute : une variable mal
nommée, un type incompatible, une fonction appelée avec le mauvais nombre
d'arguments. En Python, rien de tout cela n'est vérifié à l'avance. La raison
est simple : Python est un langage interprété, qui exécute le code ligne par
ligne, au moment de l'exécution. Il n'y a pas d'étape de compilation qui
analyserait l'ensemble du programme avant de le lancer. Un programme Python peut
donc contenir une faute de frappe dans un nom de variable et ne planter que
lorsque cette ligne précise est atteinte à l'exécution. Les tests jouent donc un
double rôle en Python : ils vérifient la logique du programme, mais ils servent
aussi de filet de sécurité de base, un peu comme le ferait un compilateur, en
s'assurant que le code s'exécute sans erreur dans les cas prévus. Dans un
contexte où une grande quantité de code Python est développée et maintenue par
une grosse équipe, cette absence de vérification statique rend les tests
carrément essentiels : sans eux, il devient pratiquement impossible de s'assurer
que les modifications d'une personne ne cassent pas le travail des autres.

## Le mécanisme de base : `assert`

L'idée fondamentale d'un test est de comparer le comportement observé d'un
programme avec le comportement attendu. En Python, le mécanisme le plus simple
pour exprimer cela est l'instruction `assert` : elle prend une condition, et si
cette condition est fausse, le programme plante immédiatement avec une erreur.

{{< pyodide >}}
def addition(a, b):
    return a + b

assert addition(2, 3) == 5
assert addition(-1, 1) == 0
assert addition(0, 0) == 0
print("Tous les tests passent!")
{{< /pyodide >}}

On peut essayer de modifier la fonction `addition` pour qu'elle retourne une
valeur incorrecte, et observer ce qui se passe lorsqu'un `assert` échoue :

{{< pyodide >}}
def addition(a, b):
    return a * b  # bug volontaire!

assert addition(2, 3) == 5
{{< /pyodide >}}

## La pyramide des tests

L'instruction `assert` suffit pour vérifier qu'une petite fonction se comporte
correctement, mais un logiciel réel est composé de nombreuses parties qui
interagissent entre elles. La question devient alors : à quel niveau faut-il
tester? Une fonction individuelle? L'interaction entre deux modules? Le système
au complet, du point de vue de l'utilisateur? La *pyramide des tests*,
popularisée par Mike Cohn dans *Succeeding with Agile* (2009), propose un modèle
pour y réfléchir. Elle distingue trois niveaux, du plus fin au plus large :

- **Tests unitaires** (la base, la plus large) : ils testent une fonction ou une
  petite unité de code en isolation. Ils sont rapides, nombreux, et faciles à
  écrire.
- **Tests d'intégration** (le milieu) : ils vérifient que plusieurs composants
  fonctionnent correctement ensemble, par exemple qu'une fonction qui appelle une
  base de données obtient les bons résultats.
- **Tests end-to-end** (le sommet, la plus petite) : ils simulent le parcours
  complet d'un utilisateur à travers le système. Ils sont lents, fragiles, et
  coûteux à maintenir, mais ils vérifient que tout fonctionne de bout en bout.

{{< image src="pyramid.png" alt="" title="" loading="lazy" >}}

La forme de pyramide reflète une règle pratique : on devrait avoir *beaucoup* de
tests unitaires, *quelques* tests d'intégration, et *peu* de tests end-to-end.
La raison est économique : plus on monte dans la pyramide, plus les tests sont
lents à exécuter, difficiles à écrire, et fragiles face aux changements. Un test
unitaire qui vérifie une fonction de calcul prend une fraction de seconde; un
test end-to-end qui simule un utilisateur navigant dans une application web peut
prendre plusieurs secondes, et casser dès qu'un bouton change de place.

Dans le cadre de ce cours, nous allons surtout nous concentrer sur les tests
unitaires, qui sont à la fois les plus fondamentaux et les plus accessibles.

## pytest

L'instruction `assert` est un bon point de départ, mais elle a ses limites. Si
on a des dizaines de tests répartis dans plusieurs fichiers, comment les
découvrir et les exécuter automatiquement? Comment obtenir un rapport clair de ce
qui passe et de ce qui échoue? Comment organiser les tests de manière lisible?

C'est le rôle d'un *framework de test*. En Python, le plus populaire est
**pytest**. Son principe est simple : on écrit des fonctions dont le nom commence
par `test_`, on y met des `assert`, et pytest se charge du reste. Voici un
exemple. Supposons qu'on ait un fichier `calcul.py` :

```python
# calcul.py
def addition(a, b):
    return a + b

def factorielle(n):
    if n <= 1:
        return 1
    return n * factorielle(n - 1)
```

On crée un fichier `test_calcul.py` à côté :

```python
# test_calcul.py
from calcul import addition, factorielle

def test_addition_simple():
    assert addition(2, 3) == 5

def test_addition_negatifs():
    assert addition(-1, 1) == 0

def test_factorielle_base():
    assert factorielle(0) == 1
    assert factorielle(1) == 1

def test_factorielle_recursive():
    assert factorielle(5) == 120
```

On lance ensuite `pytest` dans le terminal, et il découvre et exécute
automatiquement tous les fichiers `test_*.py` :

```shell
$ pytest
=================== test session starts ====================
collected 4 items

test_calcul.py ....                                  [100%]

==================== 4 passed in 0.01s =====================
```

Si on introduit un bug dans la fonction `factorielle` :

```python
def factorielle(n):
    if n <= 1:
        return 0  # bug!
    return n * factorielle(n - 1)
```

pytest produit un rapport détaillé qui montre exactement quelle assertion a
échoué, avec les valeurs comparées :

```shell
$ pytest
=================== test session starts ====================
collected 4 items

test_calcul.py ..F.                                  [100%]

======================== FAILURES ==========================
_______________ test_factorielle_base ______________________

    def test_factorielle_base():
>       assert factorielle(0) == 1
E       assert 0 == 1
E        +  where 0 = factorielle(0)

test_calcul.py:10: AssertionError
================ 1 failed, 3 passed in 0.02s ==============
```

C'est un avantage important par rapport à un `assert` brut, qui se contente de
planter avec un message `AssertionError` sans aucun détail. pytest décompose
l'expression et montre les valeurs intermédiaires, ce qui facilite
considérablement le diagnostic.

## TDD : Test-Driven Development

Jusqu'ici, nous avons écrit le code d'abord, puis les tests ensuite. Le
*Test-Driven Development* (TDD), popularisé par Kent Beck au début des années
2000, propose d'inverser l'ordre : on écrit le test *avant* le code.

Le cycle TDD se déroule en trois étapes, souvent appelées "Red-Green-Refactor" :

1. **Red** : écrire un test qui échoue (parce que la fonctionnalité n'existe pas
   encore)
2. **Green** : écrire le code *minimal* pour faire passer le test
3. **Refactor** : améliorer le code tout en s'assurant que le test passe toujours

L'idée peut sembler contre-intuitive, mais elle a un avantage profond : elle
force le programmeur à clarifier ce qu'il attend du code *avant* de l'écrire. En
termes de Naur, le test devient une manière d'expliciter sa théorie du programme
avant de la construire.

Prenons un exemple concret : on veut écrire une fonction `est_palindrome` qui
vérifie si une chaine de caractères se lit de la même manière dans les deux sens.

**Étape 1 (Red)** : on écrit les tests d'abord, sans avoir écrit la fonction :

```python
# test_palindrome.py
from palindrome import est_palindrome

def test_palindrome_simple():
    assert est_palindrome("kayak") == True

def test_non_palindrome():
    assert est_palindrome("bonjour") == False

def test_palindrome_vide():
    assert est_palindrome("") == True
```

```shell
$ pytest test_palindrome.py
E   ModuleNotFoundError: No module named 'palindrome'
```

Le test échoue : c'est normal, le module n'existe même pas encore.

**Étape 2 (Green)** : on écrit le code minimal pour faire passer les tests :

```python
# palindrome.py
def est_palindrome(s):
    return s == s[::-1]
```

```shell
$ pytest test_palindrome.py
=================== test session starts ====================
collected 3 items

test_palindrome.py ...                               [100%]

==================== 3 passed in 0.01s =====================
```

Les tests passent.

**Étape 3 (Refactor)** : on peut maintenant enrichir les tests pour couvrir des
cas plus subtils, par exemple ignorer les majuscules et les espaces :

```python
def test_palindrome_majuscules():
    assert est_palindrome("Kayak") == True

def test_palindrome_avec_espaces():
    assert est_palindrome("esope reste ici et se repose") == True
```

Ces nouveaux tests échouent, ce qui nous pousse à améliorer la fonction :

```python
def est_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]
```

Et le cycle recommence.

## La couverture de code

Une question naturelle se pose : comment savoir si nos tests sont suffisants? La
*couverture de code* (code coverage) est une métrique qui mesure quelle
proportion du code source est effectivement exécutée par les tests. L'outil de
référence en Python est **coverage.py**, créé et maintenu par Ned Batchelder
depuis 2004. L'extension **pytest-cov** est simplement un pont qui permet
d'utiliser coverage.py directement depuis pytest.

Reprenons notre fichier `calcul.py`, en y ajoutant une fonction `division` :

```python
# calcul.py
def addition(a, b):
    return a + b

def factorielle(n):
    if n <= 1:
        return 1
    return n * factorielle(n - 1)

def division(a, b):
    if b == 0:
        raise ValueError("Division par zéro")
    return a / b
```

Si nos tests ne couvrent que `addition` et `factorielle`, la couverture sera
incomplète :

```shell
$ pytest --cov=calcul
=================== test session starts ====================
collected 4 items

test_calcul.py ....                                  [100%]

---------- coverage: 80% ----------
Name        Stmts   Miss  Cover
-------------------------------
calcul.py       9      2    78%
-------------------------------
```

Les deux lignes manquantes correspondent à la fonction `division`, que nos tests
ne touchent pas du tout.

Il est tentant de viser une couverture de 100%, mais c'est un objectif trompeur.
Une couverture élevée garantit que le code a été *exécuté*, pas qu'il est
*correct*. On pourrait exécuter chaque ligne sans jamais vérifier que les
résultats sont bons. La couverture est un indicateur utile pour repérer du code
non testé, mais elle ne remplace pas la réflexion sur la qualité des tests
eux-mêmes.

## Le property-based testing

Dans tous les exemples vus jusqu'ici, nous avons écrit des tests avec des valeurs
spécifiques choisies à la main : `addition(2, 3)`, `factorielle(5)`,
`est_palindrome("kayak")`. Cette approche fonctionne bien, mais elle dépend
entièrement de notre capacité à imaginer les bons cas de test. Or, les bugs se
cachent souvent dans des cas auxquels on n'a pas pensé.

Le *property-based testing* propose une approche différente : au lieu de tester
des cas précis, on décrit une *propriété* que la fonction devrait toujours
respecter, et on laisse l'ordinateur générer automatiquement des centaines de cas
aléatoires pour essayer de la violer. En Python, la bibliothèque de référence
pour cela est **Hypothesis**.

Prenons un exemple. Une propriété fondamentale de notre fonction `addition` est
qu'elle devrait être commutative : `addition(a, b)` devrait toujours être égal à
`addition(b, a)`, peu importe les valeurs de `a` et `b`.

```python
from hypothesis import given
from hypothesis.strategies import integers
from calcul import addition

@given(a=integers(), b=integers())
def test_addition_commutative(a, b):
    assert addition(a, b) == addition(b, a)
```

Quand on lance ce test, Hypothesis génère automatiquement des centaines de paires
d'entiers (positifs, négatifs, très grands, zéro) et vérifie la propriété pour
chacune. Si elle trouve un contre-exemple, elle le simplifie automatiquement pour
donner le cas le plus petit qui fait échouer le test.

On peut appliquer la même idée à notre fonction `est_palindrome`. Une propriété
intéressante : pour n'importe quelle chaine de caractères `s`, la concaténation
de `s` avec son inverse (`s + s[::-1]`) devrait *toujours* être un palindrome.

```python
from hypothesis import given
from hypothesis.strategies import text
from palindrome import est_palindrome

@given(s=text())
def test_concatenation_inverse_est_palindrome(s):
    assert est_palindrome(s + s[::-1]) == True
```

Ce test va générer des centaines de chaines aléatoires, y compris des chaines
vides, des chaines avec des accents, des caractères spéciaux, des emojis, etc.
Si notre fonction a un bug subtil qui ne se manifeste qu'avec certains
caractères, Hypothesis a de bonnes chances de le trouver.

L'intérêt est puissant : on n'a plus besoin de deviner les bons cas de test, on
décrit ce qui *devrait être vrai*, et la machine se charge de chercher ce qui ne
l'est pas.

Le property-based testing est un cousin du *fuzzing*, une technique plus ancienne
issue du domaine de la sécurité informatique. Le fuzzing consiste à bombarder un
programme avec des entrées aléatoires ou semi-aléatoires pour provoquer des
crashes et révéler des vulnérabilités, sans formuler de propriété explicite. Il
est surtout utilisé dans les langages bas niveau comme C et C++, où les erreurs
mémoire sont courantes. Hypothesis peut être vu comme une version plus structurée
et plus intelligente du fuzzing, adaptée au monde du développement applicatif.