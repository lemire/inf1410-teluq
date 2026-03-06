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
