---
title: "Survol rapide de la programmation"
weight: 10
---

# Survol rapide de la programmation

Nous allons faire un survol rapide des idées les plus importantes et
fondamentales de la programmation logicielle. Pour aller droit au but, nous
allons utiliser le langage Python, qui est très facile à comprendre, même sans
aucune expérience explicite.

## Structures de données fondamentales

Au-delà des types de base, qui sont généralement des nombres de différents types
(entiers, réels, etc) et les chaines de caractères (strings), nous avons les
structures de données, qui sont des constructions logiques plus complexe
(non-atomique) qui impliquent, en général, les types de base, d'une manière ou
d'une autre.

### La liste et le `set`

La liste est une structure de données qui est une collection de valeurs :

{{< pyodide >}}
a = [10, 20, 30]
print(a)
{{< /pyodide >}}

Il est possible d'y ajouter, ou d'enlever des éléments :

{{< pyodide >}}
a.append(40)
print(a)
a.remove(10)
print(a)
print(len(a))
{{< /pyodide >}}

{{< image src="list.png" alt="" title="" loading="lazy" >}}

La liste a une taille et ses éléments ont un ordre. En contraste, le `set` a une taille, mais ses éléments
n'ont aucun ordre (il s'agit d'un "sac" de valeurs). Il est également possible de lui ajouter ou enlever
des éléments :

{{< pyodide >}}
s = {10, 20, 30, 40}
print(len(s))
s.add(50)
s.remove(20)
print(s)
{{< /pyodide >}}

Il est d'une importance **capitale** d'avoir un modèle mental **archi-clair** de
la différence entre ces deux structures de données à l'apparence si semblable,
pour la raison fondamentale suivante : **chercher un item dans une liste a un coût
proportionnel à la taille de la liste, tandis que chercher un item dans un `set`
a un coût fixe (et très faible)** :

{{< pyodide >}}
print(20 in a)  # coût proportionnel à la taille de `a`
print(20 in s)  # coût fixe, extrêmement faible !
{{< /pyodide >}}

{{< image src="set.png" alt="" title="" loading="lazy" >}}

Imaginez que vous cherchiez une aiguille dans une botte de foin :

{{< image src="haystack.png" alt="" title="" loading="lazy" >}}

Préférez-vous que votre botte de foin soit :

1. Une liste
2. Un `set`

### Le dictionnaire (table associative, etc)

La notion de "table associative" a plusieurs noms, selon les langages et les
cultures de programmation :

1. On parle d'un `dict` en Python (pour dictionnaire)
2. D'un `Object` en JavaScript (ou `{}`, à ne pas confondre avec les "objets" de la programmation orientée-objet, bien que ces concepts sont reliés)
3. Un `Hash` en Ruby
4. Un `array` (associatif) en PHP
5. Une `table` en Lua
6. Une `map` en Go
etc.

Il s'agit d'une autre famille de structures de données pour laquelle il est
crucial d'avoir un modèle mental clair et limpide. Une manière de se représenter
le fonctionnement d'une table associative est en tant qu'extension d'un `set` :
imaginons qu'à chaque élément (ou valeur) d'un `set`, nous attachons une valeur.

{{< image src="dict.png" alt="" title="" loading="lazy" >}}

On considère le `dict` comme une structure de données plus versatile et générale
que les autres parce qu'il est possible d'implémenter, par exemple, une liste
avec un `dict` :

{{< image src="dict-as-list.png" alt="" title="" loading="lazy" >}}

Il est aussi possible d'implémenter un `set` avec un `dict` :

{{< image src="dict-as-set.png" alt="" title="" loading="lazy" >}}

### Le dictionnaire en tant que `record`

Étant donné la structure du dictionnaire, il est souvent possible de l'utiliser
en tant
qu'[enregistrement](https://fr.wikipedia.org/wiki/Enregistrement_(structure_de_donn%C3%A9es))
(record en anglais, plus couramment), c'est-à-dire en tant que contenant pouvant
contenir des valeurs diverses pour un "object". Ce type d'usage est très courant dans
les langages de plus haut niveau. Par exemple en Python :

{{< pyodide >}}
# =========================
# 1) dict
# =========================

person_dict = {
    "name": "Alice",
    "age": 30
}

print("DICT:")
print(person_dict["name"])
print(person_dict["age"])


# =========================
# 2) class
# =========================

class PersonClass:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

person_class = PersonClass("Alice", 30)

print("\nCLASS:")
print(person_class.name)
print(person_class.age)


# =========================
# 3) dataclass
# =========================

from dataclasses import dataclass

@dataclass
class PersonDataClass:
    name: str
    age: int

person_dataclass = PersonDataClass("Alice", 30)

print("\nDATACLASS:")
print(person_dataclass.name)
print(person_dataclass.age)
{{< /pyodide >}}

Ou encore en JavaScript, avec l'[object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object), qui ressemble au `dict` Python :

```js
/*
Exemples simples de "record" en JavaScript :
- objet littéral
- class
- objet immuable (style record)
*/

////////////////////
// 1) Objet littéral
////////////////////

const personObject = {
  name: "Alice",
  age: 30
};

console.log("OBJECT:");
console.log(personObject.name);
console.log(personObject.age);


////////////////////
// 2) Class
////////////////////

class PersonClass {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

const personClass = new PersonClass("Alice", 30);

console.log("\nCLASS:");
console.log(personClass.name);
console.log(personClass.age);


////////////////////////////////////
// 3) Objet immuable (record-like)
////////////////////////////////////

const personRecord = Object.freeze({
  name: "Alice",
  age: 30
});

console.log("\nIMMUTABLE OBJECT:");
console.log(personRecord.name);
console.log(personRecord.age);
```

### L'implémentation du dictionnaire

Nous avons jusqu'ici parlé du dictionnaire du point de vue des langages de
programmation d'assez haut niveau, comme Python et JS. Ceux-ci permettent
d'utiliser un dictionnaire, mais en cache un aspect crucial : leur
implémentation ! Étant donné que Python, le langage lui-même, est écrit dans le
langage C, comment écrit-on un `dict` Python, en C? Il existe plusieurs manières
de le faire, mais l'une d'elles est [table de
hachage](https://fr.wikipedia.org/wiki/Table_de_hachage) (hash table en
anglais). En gros, l'idée est d'appliquer une [fonction de
hachage](docs/module2/versioning/#fonction-de-hachage) à un élément (la clé), ce
qui permet de déterminer l'index dans un tableau, où on pourra mettre la valeur
associée. Dans certaines implémentations, il est nécessaire de gérer les
collisions possibles : si deux clés mènent au même index par exemple, il sera
possible d'utiliser une liste, pour cette clé particulière.

{{< image src="hashtable.png" alt="" title="" loading="lazy" >}}

### La correspondance avec JSON

Une fois que vous avez des modèles mentaux clairs et efficaces par rapport aux
différences et similarités de ces trois structures de données dont nous avons
parlées dans cette section, il est utile de considérer leur correspondance avec
la notation JSON.

JSON (JavaScript Object Notation) est un format textuel léger destiné à
représenter des données structurées de façon simple et lisible, à partir de
quelques constructions de base : objets clé-valeur, tableaux ordonnés et valeurs
primitives. Conçu à l’origine pour les échanges de données dans les applications
Web, il correspond étroitement aux structures de données que l’on retrouve dans
la plupart des langages de programmation modernes, ce qui explique son adoption
massive comme format d’échange entre systèmes, en particulier dans les API et
les applications distribuées.

```json
{
  "nom": "Alice",
  "age": 30,
  "competences": ["Python", "JavaScript", "SQL"],
  "adresse": {
    "ville": "Montréal",
    "pays": "Canada"
  }
}
```

La totalité d'un document JSON est souvent un dictionnaire (que l'on reconnaît à
l'usage des `{}`), mais il existe toutefois d'autres formats (JSONL, NDJSON,
etc). Ce dictionnaire contient la plupart du temps des champs (clés) dont les
valeurs peuvent être, elles-mêmes, des dictionnaires, des listes ou des valeurs
scalaires (nombres, chaines de caractères ou valeurs booléennes).

## Complexité algorithmique et performance

Le sujet de la [complexité
algorithmique](https://fr.wikipedia.org/wiki/Th%C3%A9orie_de_la_complexit%C3%A9_(informatique_th%C3%A9orique))
est en général présenté et discuté de manière très mathématique (et même parfois
carrément
[philosophique](https://fr.wikipedia.org/wiki/Probl%C3%A8me_P_%E2%89%9F_NP)),
mais la plupart des programmeurs d'expérience en ont un modèle mental assez
simplifié.

La préoccupation fondamentale à la base de sujet est la performance d'un
algorithme, ou en termes encore plus simples le temps qu'il prend à exécuter.
Bien qu'en pratique les détails d'implémentation (comment un programme est
écrit) et les particularités matérielles de l'environnement où il est exécuté
(l'ordinateur particulier qui est utilisé) sont très importants, en général
quand on parle de complexité algorithmique on a en tête quelque chose de plus
abstrait.

Examinons certains exemples. Imaginons tout d'abord que nous avons une fonction
qui construit une liste de `N` nombres entiers aléatoires (entre 1 et `N`) :

{{< pyodide >}}
import random

def get_random_numbers(n):
    return [random.randint(1, n) for _ in range(n)]

print("Fonction 'get_random_numbers' définie.")
{{< /pyodide >}}

Imagions ensuite que nous aimerions savoir si une paire de nombres distincts,
dans cette liste, résulte en une somme donnée `K`.

{{< image src="two-sum.png" alt="" title="" loading="lazy" >}}

Une méthode naive pour résoudre ce problème consiste à considérer toutes les
paires :

{{< image src="two-sum-quadratic.png" alt="" title="" loading="lazy" >}}

{{< pyodide >}}
def two_sum_quadratic_version(a, k):
    for i, x in enumerate(a):
        for j, y in enumerate(a):
            if i == j: continue
            if x + y == k:
                return (x, y)

    return None

print("Fonction 'two_sum_quadratic_version' définie.")
{{< /pyodide >}}

Cette méthode est assez coûteuse parce que pour chaque item de la liste (qui
contient `N` items), il faut considérer une sous-liste (en fait la même liste,
implicitement), de `N` éléments. Il faut donc faire, au maximum, $N^2$ (`N` au
carré) vérifications. On parle donc d'un algorithme de complexité
[_quadratique_](https://fr.wikipedia.org/wiki/Fonction_quadratique). On utilise
parfois la notation appelée ["Grand
O"](https://fr.wikipedia.org/wiki/Comparaison_asymptotique#La_famille_de_notations_de_Landau_O,_o,_%CE%A9,_%CF%89,_%CE%98,_~)
pour décrire la complexité algorithmique d'une fonction, dans ce cas $O(N^2)$.

Si on y pense un peu, une méthode beaucoup plus efficace consisterait à utiliser
un `set` pour conserver les valeurs qu'on a déjà rencontrées, et à chaque
nouvelle valeur de la liste (à ne pas confondre avec le `set`!), tester si `K`
moins la nouvelle valeur se trouve dans le `set` :

{{< image src="two-sum-linear.png" alt="" title="" loading="lazy" >}}

{{< pyodide >}}
def two_sum_linear_version(a, k):
    seen = set()
    for x in a:
        target = k - x
        if target in seen:
            return (x, target)
        seen.add(x)

    return None

print("Fonction 'two_sum_linear_version' définie.")
{{< /pyodide >}}

La raison pour laquelle cet algorithme est beaucoup plus efficace est dû au fait
que tester l'inclusion d'un item dans un `set` (ce qui se fait, dans le code
ci-haut, à l'aide de l'opérateur `in`, dans la ligne `if target in seen`),
contrairement au fait de le faire dans une liste, est extrêmement efficace : si
on utilise la notation "grand O", on dit que chercher un item dans une liste est
$O(1)$, c'est-à-dire que cela se fait en temps constant. Mais étant donné que
nous devons tout de même passer à travers tous les items de la liste pour les
tester, la complexité finale de l'algorithme reste tout de même $O(N)$, ce qu'on
appelle une complexité linéaire.

Pour comparer de manière empirique la performance de ces deux algorithmes, nous
avons besoin de considérer plusieurs résultats (c-à-d plusieurs sommes
possibles), car un seul appel pourrait être "chanceux" (la première paire
considérée est tout de suite la bonne). On peut utiliser cette fonction, qui
prend en entrée une variable `fn`, qui va contenir la fonction qui sera exécutée
(`two_sum_linear_version` ou `two_sum_quadratic_version`) :

{{< pyodide >}}
def test_algo(fn, a):
    for k in range(len(a)):
        res = fn(a, k)
        if res:
            assert res[0] + res[1] == k

print("Fonction 'test_algo' définie.")
{{< /pyodide >}}

Quand la taille de la liste est raisonnable, la différence entre les deux
algorithmes n'est pas si notable (ici on utilise la console
[ipython](https://ipython.org/), qui possède des fonctionnalités pratiques pour
mesurer le temps d'exécution)&nbsp;

{{< pyodide >}}
import time

a = get_random_numbers(1000)

start = time.time()
test_algo(two_sum_quadratic_version, a)
print(f"Quadratique: {time.time() - start:.3f}s")

start = time.time()
test_algo(two_sum_linear_version, a)
print(f"Linéaire: {time.time() - start:.3f}s")
{{< /pyodide >}}

Mais si on augmente `N`, la différence devient rapidement assez dramatique :

{{< pyodide >}}
a = get_random_numbers(5000)

start = time.time()
test_algo(two_sum_quadratic_version, a)
print(f"Quadratique: {time.time() - start:.3f}s")

start = time.time()
test_algo(two_sum_linear_version, a)
print(f"Linéaire: {time.time() - start:.3f}s")
{{< /pyodide >}}

Il est très important de comprendre à quel point l'utilisation de la bonne
structure de données est cruciale, pour certains algorithmes. Si on remplace
le `set` de la méthode linéaire par une liste, elle redevient quadratique :

{{< pyodide >}}
def two_sum_falsely_linear_version(a, k):
    seen = []
    for x in a:
        target = k - x
        if target in seen:
            return (x, target)
        seen.append(x)

    return None

print("Fonction 'two_sum_falsely_linear_version' définie.")
{{< /pyodide >}}

{{< image src="two-sum-falsely-linear.png" alt="" title="" loading="lazy" >}}

{{< pyodide >}}
start = time.time()
test_algo(two_sum_falsely_linear_version, a)
print(f"Faussement linéaire: {time.time() - start:.3f}s")
{{< /pyodide >}}

Nous avons dit que cet algorithme est "faussement linéaire", et qu'il est en
fait quadratique, en raison de l'usage de la liste (plutôt qu'un `set`). Mais
pourquoi la performance semble être "entre les deux", dans ce cas particulier,
si on compare les temps d'exécution? La réponse est que l'opérateur `in` est
fortement optimisé en Python (qui est lui-même écrit dans le langage C), et
qu'il est beaucoup plus efficace que ce qu'on pourrait faire nous-même, avec
notre propre boucle `for`. On peut donc en dégager une idée générale, un modèle
mental simplifié avec lequel il est possible de faire beaucoup de chemin : la
complexité algorithmique se réduit souvent à la question du nombre de boucles
`for` imbriquées que notre code contient ! Chaque boucle imbriquée
supplémentaire correspond à un facteur par lequel on multiplie le temps
d'exécution de notre algorithme, en gros. On peut donc généraliser ce principe,
et comprendre qu'un algorithme qui contiendrait trois boucles `for` imbriquées,
par exemple, serait $O(N^3)$, ce qui serait évidemment encore plus coûteux.

### Au-delà et en-deça de la complexité polynomiale

Nous avons vu jusqu'ici des algorithmes et des structures de données qui ont
suggéré un ordre dans la complexité : $O(1)$ (temps constant), qui est le plus
rapide, à privilégier quand c'est possible évidemment, $O(N^2)$, le temps
quadratique, qui est beaucoup moins efficace. Et nous avons vu qu'il est
possible de généraliser au-delà : $O(N^3)$, temps cubique, etc. On appelle cette
classe de performance _polynomiale_.

Il existe cependant d'autres profils de performance dont il est utile de
connaître l'existence. Supposons qu'on ait une liste triée de nombres, et qu'on
cherche un nombre particulier dans cette liste :

{{< image src="sorted-list.png" alt="" title="" loading="lazy" >}}

Est-ce qu'il existe un algorithme plus performant que $O(N)$ pour ce problème?

{{< image src="binary-search.png" alt="" title="" loading="lazy" >}}

Cet algorithme très fameux s'appelle la "recherche binaire", et son temps
d'exécution est $O(log N)$, car il s'agit essentiellement d'une série de
"division par deux" (à chaque fois qu'on coupe la liste en deux), l'inverse donc
d'une multiplication. Dans le contexte de ce que nous avons vu jusqu'à
maintenant, il  s'agit donc d'une performance intermédiaire, entre $O(1)$ et
$O(N)$. Dans la pratique, si un tel algorithme est possible, c'est souvent
extrêmement avantageux, car la fonction $log(N)$ progresse beaucoup plus
lentement qu'une fonction linéaire ($O(N)$).

Dans le problème précédent, une difficulté était cachée : comment fait-on pour
trier une liste, et quel coût ça a? Il s'avère que la méthode la plus rapide
pour trier une liste est $O(N log N)$, ce qu'on appelle parfois une méthode
supra-linéaire, et qui se trouve entre $O(N)$ et $O(N^2)$. Il est
mathématiquement impossible de faire mieux qu'une performance supra-linéaire,
pour trier une liste de nombres ou d'objets quelconques.

Finalement, certains algorithmes ont des profils de performance encore plus
coûteux que la classe polynomiale : ils nécessitent un temps exponentiel, par
exemple $O(2^N)$. Voici certains exemples fameux de problèmes dont les algorithmes
pour produire une solution optimale sont exponentiels :

{{< image src="tsp.png" alt="" title="" loading="lazy" >}}

Le [problème du voyageur de
commerce](https://fr.wikipedia.org/wiki/Probl%C3%A8me_du_voyageur_de_commerce),
qui consister à déterminer le circuit le plus court passant par une série de
villes, dans un graphe.

Les échecs :

{{< image src="chess.png" alt="" title="" loading="lazy" >}}

Et finalement, le jeu de Sudoku :

{{< image src="sudoku.png" alt="" title="" loading="lazy" >}}
