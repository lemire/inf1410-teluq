---
title: "Historique"
slug: "historique"
weight: 20
---

# Historique du génie logiciel

L'histoire du génie logiciel n'est pas qu'une suite d'inventions techniques :
c'est aussi l'histoire de problèmes récurrents, de solutions qui créent de
nouveaux problèmes, et de débats qui traversent les décennies. Les tensions
entre complexité essentielle et accidentelle, entre rigueur formelle et
pragmatisme, y apparaissent dès les premières décennies, et n'ont jamais
vraiment disparu.

## Les années 40 à mi-60 : l'ère pré-GL

L'histoire du logiciel commence avec les premiers ordinateurs électroniques des
années 1940. Des machines comme l'ENIAC (1945) étaient programmées par câblage
physique : il fallait littéralement brancher et débrancher des connexions pour
changer le comportement de la machine. Il n'existait pas encore de notion de
« programme stocké en mémoire », et encore moins de génie logiciel. La
programmation, si on peut l'appeler ainsi, était une affaire de quelques
spécialistes qui travaillaient seuls, en contact direct avec le matériel.

{{< image src="eniac.png" alt="" title="" loading="lazy" >}}

Les choses commencent à changer avec l'apparition des premiers langages de
programmation dans les années 1950. FORTRAN (1957), conçu par John Backus chez
IBM, est généralement considéré comme le premier langage de haut niveau : il
permettait aux scientifiques d'écrire des formules mathématiques de façon
lisible, plutôt que de manipuler directement les instructions de la machine :

```fortran
      PROGRAM HELLO
      PRINT *, 'Hello, World!'
      STOP
      END
```

L'indentation à 6 espaces et les majuscules obligatoires sont typiques du
FORTRAN de l'époque, un langage conçu pour les cartes perforées où les colonnes
avaient un sens précis.

LISP (1958), créé par John McCarthy au MIT, introduisait des concepts
radicalement différents comme la récursivité et la manipulation de listes, qui
influenceront la programmation pendant des décennies :

```lisp
(defun factorielle (n)
  (if (<= n 1)
      1
      (* n (factorielle (- n 1)))))
```

La récursivité, les parenthèses omniprésentes et l'absence de distinction entre
code et données font de LISP un langage qui ne ressemble à rien d'autre.
Cinquante ans plus tard, ces mêmes idées se retrouveront dans des langages comme
Clojure et dans les fondements de l'intelligence artificielle.

COBOL (1959), développé
sous l'impulsion de Grace Hopper, ciblait le monde des affaires avec une syntaxe
qui se voulait proche de l'anglais :

```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. HELLO-WORLD.

PROCEDURE DIVISION.
    DISPLAY "Hello, World!".
    STOP RUN.
```

Même sans connaître COBOL, on peut voir l'ambition de rendre le code lisible par
des non-programmeurs, une préoccupation qui reviendra régulièrement dans
l'histoire du génie logiciel.

Cette période voit aussi naître les premiers systèmes d'exploitation, qui
commencent à gérer les ressources matérielles et à offrir une couche
d'abstraction entre le matériel et les programmes. Les projets logiciels restent
relativement modestes, et la programmation est encore vue comme un art individuel
plutôt que comme une discipline d'ingénierie. Mais les choses sont sur le point
de changer : à mesure que les ordinateurs deviennent plus puissants et plus
accessibles, les ambitions grandissent, et avec elles, les problèmes.

## Les années 60 et 70 : la crise logicielle et la naissance du GL

À mesure que les ordinateurs se répandent dans les grandes organisations, les
projets logiciels prennent de l'ampleur. Et c'est là que les ennuis commencent.
Les projets dépassent régulièrement leurs budgets et leurs échéanciers, les
logiciels livrés sont truffés de bogues, et certains projets échouent tout
simplement. On nommera ce phénomène la « crise du logiciel ». Le terme « génie
logiciel » (*software engineering*) est lui-même consacré lors de la conférence
de l'OTAN à Garmisch (Allemagne) en 1968. Le choix du mot *engineering* est
délibéré : il s'agit d'affirmer que le développement de logiciels doit devenir
une discipline d'ingénierie rigoureuse, avec des méthodes et des processus,
plutôt que de rester un artisanat improvisé. C'est le début d'une tension qui ne
se résoudra jamais complètement : combien de formalisme est « juste assez »?

En réponse à la crise, Edsger Dijkstra publie en 1968 sa célèbre lettre « Go To
Statement Considered Harmful ». Son
[argument](https://en.wikipedia.org/wiki/Considered_harmful) : l'instruction
`goto`, qui permet de sauter arbitrairement d'un endroit à un autre dans le
code, rend les programmes impossibles à comprendre et à maintenir. Dijkstra
prône la **programmation structurée**, fondée sur des structures de contrôle
claires (boucles, conditions, sous-routines). L'idée semble évidente
aujourd'hui, mais à l'époque elle est controversée : certains programmeurs y
voient une contrainte inutile imposée par des théoriciens. Ce débat préfigure un
schéma qui se répétera souvent : une bonne pratique, née de l'observation de
problèmes réels, est d'abord résistée par ceux qui la perçoivent comme un dogme
académique.

{{< image src="dijkstra.jpg" alt="" title="" loading="lazy" >}}

La première tentative de formaliser le processus de développement vient en 1970,
quand Winston Royce décrit ce qui deviendra le **modèle en cascade**
(*waterfall*) : le développement progresse de façon linéaire à travers des
phases séquentielles (analyse des besoins, conception, implémentation, tests,
déploiement, maintenance). Chaque phase doit être complétée avant de passer à la
suivante. L'ironie, c'est que Royce présentait ce modèle pour en montrer les
limites. Mais c'est justement cette version rigide qui sera massivement adoptée
par l'industrie et les gouvernements, séduits par sa clarté apparente et sa
compatibilité avec les structures bureaucratiques. Il faudra attendre trente ans
et le Manifeste Agile pour qu'un modèle alternatif prenne véritablement le
dessus.

{{< image src="waterfall.webp" alt="" title="" loading="lazy" >}}

Les années 70 voient aussi la naissance de **Unix** (1969-1971) chez Bell Labs,
créé par Ken Thompson et Dennis Ritchie. Unix introduit une philosophie de
conception qui deviendra extrêmement influente : des programmes simples, qui
font une seule chose bien, et qui peuvent être combinés entre eux. Le langage
**C** (1972), créé par Ritchie pour réécrire Unix, incarne cette même
philosophie de concision pragmatique :

```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

Comparé à COBOL, le contraste est frappant. Là où COBOL visait la lisibilité
pour les gestionnaires, C vise l'efficacité pour les programmeurs. C'est un
autre niveau d'abstraction par rapport à l'assembleur, mais qui reste proche de
la machine. Cette proximité, combinée à une portabilité inédite, en fera un
outil révolutionnaire.

{{< image src="unix.webp" alt="" title="" loading="lazy" >}}

{{< image src="c-book.png" alt="" title="" loading="lazy" >}}

En 1975, **Fred Brooks** publie *The Mythical Man-Month*, un livre fondé sur son
expérience comme chef de projet du système OS/360 chez IBM, l'un des plus grands
projets logiciels de l'époque. Son observation centrale est devenue la **loi de
Brooks** : « Ajouter des personnes à un projet en retard le retarde davantage. »
La raison est mathématique : la communication entre $n$ personnes croît en
$n(n-1)/2$. Doubler la taille d'une équipe ne double pas sa productivité, loin
de là. Le titre même du livre dénonce un mythe tenace : un projet de 12
« mois-hommes » ne peut pas être réalisé par 12 personnes en un mois, car les
tâches ne sont pas toutes parallélisables. Ces observations, formulées il y a
cinquante ans, restent parmi les vérités les plus fondamentales du génie
logiciel. Brooks reviendra sur le sujet en 1986 avec l'essai « No Silver
Bullet », dans lequel il argue qu'aucune technologie ou méthode ne peut à elle
seule produire une amélioration d'un ordre de grandeur en productivité
logicielle. La complexité fondamentale du logiciel, dit-il, n'est pas dans les
outils, mais dans le problème lui-même.

{{< image src="mythical-man-month.jpg" alt="" title="" loading="lazy" >}}

En 1970, **Edgar F. Codd** publie son article fondateur sur le **modèle
relationnel**, qui propose de structurer les données en tables liées par des
relations mathématiques. L'idée est typique de cette époque : appliquer la
rigueur formelle pour résoudre un problème pratique. Le modèle de Codd donnera
naissance à SQL et aux systèmes de gestion de bases de données relationnelles
qui dominent encore aujourd'hui. On y reviendra en détail dans un module
ultérieur.

En 1986, Barry Boehm propose le **modèle en spirale**, une alternative au modèle
en cascade qui intègre explicitement la gestion des risques. Plutôt que de
progresser linéairement, le développement passe par des cycles répétés de
planification, analyse de risques, développement et évaluation. L'idée d'itérer
plutôt que de tout planifier d'avance est encore minoritaire à l'époque, mais
elle préfigure directement les approches agiles qui émergeront une quinzaine
d'années plus tard.

{{< image src="spiral.png" alt="" title="" loading="lazy" >}}

## Les années 80 : l'ère des abstractions, du PC et du GUI

Les années 80 sont marquées par deux révolutions parallèles : l'essor de la
**programmation orientée objet** (POO) comme paradigme dominant, et la
démocratisation de l'informatique avec le **PC** et les **interfaces
graphiques**. Bien que les concepts de la POO remontent à Simula (1967) et
Smalltalk (1972), c'est dans les années 80 qu'ils se généralisent, portés
notamment par **C++** (créé par Bjarne Stroustrup, première version publique en
1985).

{{< image src="stroustrup.jpg" alt="" title="" loading="lazy" >}}

C++ ajoute les classes, l'héritage et le polymorphisme au langage C, permettant
de modéliser les problèmes en termes d'objets qui encapsulent données et
comportements :

```cpp
class Forme {
public:
    virtual double aire() const = 0;
    virtual ~Forme() {}
};

class Cercle : public Forme {
    double rayon;
public:
    Cercle(double r) : rayon(r) {}
    double aire() const override {
        return 3.14159 * rayon * rayon;
    }
};
```

L'idée de gérer la complexité par l'abstraction et l'héritage deviendra la façon
dominante de penser le logiciel pendant les deux décennies suivantes, pour le
meilleur et pour le pire : on découvrira avec le temps que les hiérarchies
d'héritage profondes créent autant de problèmes qu'elles en résolvent.

L'apparition du Macintosh (1984) et plus tard de Windows popularise les
interfaces graphiques. Ce changement a un impact profond sur la programmation :
on passe d'un modèle séquentiel (le programme contrôle le flux d'exécution) à un
modèle **événementiel** (le programme réagit aux actions de l'utilisateur :
clics, mouvements de souris, frappes clavier). En parallèle, la standardisation
progressive des protocoles réseau, avec le modèle OSI et la suite TCP/IP, pose
les bases de ce qui deviendra Internet. Le logiciel ne vit plus seulement sur
une machine isolée : il commence à communiquer.

{{< image src="pc.jpg" alt="" title="" loading="lazy" >}}

{{< image src="mac.jpg" alt="" title="" loading="lazy" >}}

## Les années 90 : la naissance du web

Quand Tim Berners-Lee crée le World Wide Web au CERN en 1991, peu de gens
imaginent l'impact que cela aura. En quelques années, le web passe d'un outil de
partage de documents scientifiques à une plateforme universelle. Les développeurs
doivent soudainement penser en termes de client-serveur, de protocoles HTTP et
de HTML.

{{< image src="timbl.jpg" alt="" title="" loading="lazy" >}}

Puis JavaScript arrive en 1995, créé en dix jours par Brendan Eich chez
Netscape. Ce langage, conçu dans l'urgence pour ajouter de l'interactivité aux
pages web, deviendra contre toute attente l'un des langages les plus utilisés au
monde. Le niveau d'abstraction monte encore d'un cran : on ne programme plus
pour une machine, mais pour un navigateur qui tourne sur n'importe quelle
machine :

```javascript
document.getElementById("btn").addEventListener("click", function() {
    alert("Hello, World!");
});
```

On voit ici le modèle événementiel en action : plutôt que d'exécuter du code de
façon séquentielle, on attache un comportement à un événement (le clic sur un
bouton).

{{< image src="brendaneich.jpg" alt="" title="" loading="lazy" >}}

**Java** (1995), créé par James Gosling chez Sun Microsystems, arrive avec la
promesse « *Write Once, Run Anywhere* » : grâce à sa machine virtuelle (JVM), un
programme Java peut s'exécuter sur n'importe quelle plateforme :

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

La verbosité de Java saute aux yeux : pour afficher une ligne de texte, il faut
une classe, une méthode `main` avec sa signature complète, et
`System.out.println`. C'est le prix de l'architecture « tout est objet ».

Java popularise
aussi la gestion automatique de la mémoire et un écosystème d'outils de
développement. Un an plus tôt, en 1994, le livre *Design Patterns* du « Gang of
Four » avait catalogué 23 patrons de conception récurrents (Singleton, Observer,
Factory, Strategy...), offrant un vocabulaire commun pour discuter
d'architecture. Java et les design patterns forment ensemble l'orthodoxie des
années 90 : tout est objet, tout est patron. Avec le recul, on réalisera que
certains de ces patrons sont moins des solutions universelles que des
contournements de limitations spécifiques à certains langages.

{{< image src="james-gosling.jpg" alt="" title="" loading="lazy" >}}

En 1991, **Guido van Rossum** publie la première version de **Python**, un
langage qui prend le contre-pied de la verbosité de Java. La philosophie de
Python, résumée dans le « Zen of Python », tient en quelques principes : la
lisibilité compte, il devrait y avoir une façon évidente de faire les choses, et
le simple vaut mieux que le complexe :

```python
def factorielle(n):
    if n <= 1:
        return 1
    return n * factorielle(n - 1)
```

Comparé à la même fonction en Java, qui exigerait une classe, une méthode
statique et des déclarations de types, la concision de Python est frappante. Ce
choix de design n'est pas qu'esthétique : en réduisant le « bruit syntaxique »,
Python permet au programmeur de se concentrer sur le problème plutôt que sur le
langage. D'abord populaire comme outil de script et d'enseignement, Python
deviendra au fil des décennies l'un des langages les plus utilisés au monde,
porté notamment par l'essor de la science des données et de l'apprentissage
automatique.

Le **Unified Modeling Language** (UML), standardisé en 1997, représente l'apogée
de l'approche « concevoir d'abord, coder ensuite ». UML propose une notation
graphique élaborée pour modéliser les systèmes logiciels : diagrammes de
classes, de séquence, de cas d'utilisation, d'activité, et bien d'autres. Dans
les universités et les grandes organisations, UML devient presque une fin en
soi : des équipes passent des mois à produire des diagrammes détaillés avant
d'écrire une seule ligne de code. Le problème, c'est que ces modèles
vieillissent mal. Dès que le code évolue, les diagrammes deviennent obsolètes,
et personne ne les met à jour. UML reste un outil de communication utile quand
il est utilisé avec parcimonie, mais son usage systématique est un exemple
classique de formalisme qui s'éloigne de la réalité du terrain.

En 1999, **Kent Beck** publie *Extreme Programming Explained*, qui prend le
contre-pied radical de l'approche UML/cascade. L'Extreme Programming (XP) prône
des cycles de développement très courts, la programmation en binôme (*pair
programming*), le développement piloté par les tests (*test-driven development*)
et le refactoring constant. Plutôt que de tout planifier d'avance, on embrasse
le changement. Ces idées, jugées « extrêmes » à l'époque, deviendront la norme
deux décennies plus tard.

En parallèle, le mouvement du **logiciel libre**, initié par Richard Stallman
dans les années 80, prend une dimension nouvelle avec l'essor de l'**open
source**.

{{< image src="rms.jpg" alt="" title="" loading="lazy" >}}

**Linux** (1991), créé par Linus Torvalds, démontre qu'une communauté
mondiale de bénévoles peut créer et maintenir un système d'exploitation complet.
C'est une révolution dans la dimension individu/équipe : on passe de l'équipe de
projet traditionnelle à une collaboration distribuée, asynchrone, entre des
milliers de contributeurs qui ne se sont jamais rencontrés. Ce modèle influencera
profondément les pratiques du génie logiciel et préfigure la façon dont la
majorité du logiciel est développé aujourd'hui.

{{< image src="linus.webp" alt="" title="" loading="lazy" >}}

La même année, **Andrew Hunt** et **David Thomas** publient *The Pragmatic
Programmer*, un ouvrage qui offre des conseils pratiques et terre-à-terre pour le
métier de développeur. Le livre popularise des principes comme **DRY** (*Don't
Repeat Yourself*) et insiste sur l'importance d'un apprentissage continu. Son
titre même est un manifeste : face au formalisme ambiant des années 90, il
revendique le pragmatisme comme vertu cardinale.

{{< image src="pragprog.jpg" alt="" title="" loading="lazy" >}}

## Les années 00 : l'ère de l'agilité

En février 2001, dix-sept développeurs se réunissent dans un chalet de ski à
Snowbird, Utah, et rédigent le **Manifeste Agile**. Ce court document affirme
quatre valeurs : les individus et les interactions plutôt que les processus et
les outils; un logiciel fonctionnel plutôt qu'une documentation exhaustive; la
collaboration avec le client plutôt que la négociation de contrats; la réponse au
changement plutôt que le suivi d'un plan. Le Manifeste ne prescrit pas de
méthode spécifique : il pose un ensemble de valeurs. Plusieurs cadres
méthodologiques s'en réclament, notamment **Scrum** (sprints, mêlées
quotidiennes, rétrospectives) et **Kanban** (visualisation du flux de travail,
limitation du travail en cours). L'adoption de l'agilité transforme la relation
entre développeurs et clients, mais elle génère aussi ses propres dérives :
certaines organisations adoptent les rituels agiles sans en intégrer les valeurs,
ce que certains appellent ironiquement l'« agile d'entreprise ».

La décennie est aussi celle du **Web 2.0** : le web cesse d'être un média de
consultation passive et devient une plateforme participative. Des applications
comme Wikipedia, Facebook, YouTube et Gmail montrent que le navigateur peut
rivaliser avec les logiciels de bureau. Ce changement a des conséquences directes
sur le génie logiciel : les applications web doivent gérer des millions
d'utilisateurs simultanés, évoluer rapidement et être disponibles en permanence.
Les cycles de livraison de 18 mois deviennent intenables. C'est dans ce contexte
que le cloud et les outils de collaboration modernes prennent tout leur sens.

{{< image src="faang.webp" alt="" title="" loading="lazy" >}}

En 2006, **Amazon Web Services** (AWS) lance ses premiers services de cloud
computing, permettant à quiconque de louer de la puissance de calcul et du
stockage à la demande. Le déploiement et la mise à l'échelle deviennent des
opérations logicielles plutôt que matérielles : un changement fondamental. Deux
ans plus tard, **GitHub** (2008) transforme la collaboration entre développeurs.
En combinant l'hébergement de dépôts Git avec des fonctionnalités sociales
(fork, pull requests, issues), GitHub démocratise la contribution open source et
établit de nouvelles normes pour le travail en équipe. Le *pull request*, en
particulier, devient un mécanisme central de revue de code dans l'industrie.

## Les années 10 : l'ère du cloud et du DevOps

Les années 2010 sont marquées par l'accélération des cycles de développement et
la fusion progressive entre développement et opérations. Le mouvement **DevOps**
n'est pas qu'un ensemble d'outils : c'est un changement culturel qui affirme que
ceux qui écrivent le code doivent aussi être responsables de son fonctionnement
en production. L'**intégration continue** (*Continuous Integration*, CI) et le
**déploiement continu** (*Continuous Deployment*, CD) deviennent des pratiques
standard. Des outils comme Jenkins, Travis CI puis GitHub Actions automatisent la
compilation, les tests et le déploiement à chaque modification du code. L'idée
est de détecter les problèmes le plus tôt possible et de pouvoir livrer des
nouvelles versions à tout moment, parfois plusieurs fois par jour.

**Docker** (2013) popularise la **conteneurisation** : la possibilité de
packager une application avec toutes ses dépendances dans un conteneur isolé et
portable. Un `Dockerfile` décrit de façon déclarative comment construire
l'environnement d'exécution :

```dockerfile
FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

Docker résout le fameux problème « ça marche sur ma machine » en garantissant que
l'environnement de développement est identique à l'environnement de production.
Combiné avec **Kubernetes** (2014, créé par Google), qui orchestre le déploiement
et la mise à l'échelle automatique de conteneurs, Docker transforme la façon dont
les applications sont opérées. En parallèle, l'architecture en **microservices**
émerge comme alternative aux applications monolithiques : plutôt qu'un seul
programme qui fait tout, on découpe l'application en services indépendants qui
communiquent par API. Chaque service peut être développé, déployé et mis à
l'échelle indépendamment. Le niveau d'abstraction continue sa montée : on ne
gère plus des serveurs, mais des conteneurs orchestrés automatiquement.

## Les années 20 : l'ère de l'IA

Dans les années 2020, l'**open source** n'est plus un mouvement alternatif :
c'est le mode de développement dominant. La quasi-totalité des outils, frameworks
et bibliothèques utilisés quotidiennement par les développeurs sont open source.
Des projets comme Linux, Python, React ou Kubernetes sont maintenus par des
communautés mondiales, souvent soutenues financièrement par les grandes
entreprises technologiques. Le modèle open source a aussi transformé la façon
dont les développeurs apprennent : le code source des meilleurs logiciels du
monde est accessible à tous, et contribuer à un projet open source est devenu un
rite de passage pour beaucoup de développeurs.

Mais le changement le plus spectaculaire de cette décennie est l'irruption des
**grands modèles de langage** (LLM) dans la pratique du développement. Avec
l'arrivée d'outils comme GitHub Copilot (2021) puis ChatGPT (2022), les
développeurs disposent pour la première fois d'assistants capables de générer du
code, d'expliquer des programmes existants, de rédiger des tests et de déboguer.
Des outils comme Claude Code vont encore plus loin, en offrant un agent capable
de naviguer dans un projet entier, de comprendre son architecture et de proposer
des modifications cohérentes. On commence à parler de *vibe coding*, une approche
où le développeur décrit ce qu'il veut en langage naturel et laisse l'IA écrire
le code. Cette évolution soulève des questions fondamentales que ce cours abordera
dans un module ultérieur : quel est le rôle du développeur quand l'IA peut écrire
du code? Comment valider du code généré par une machine? Les principes du génie
logiciel (tests, revue de code, architecture) deviennent-ils plus ou moins
importants dans ce contexte?

{{< image src="ai.webp" alt="" title="" loading="lazy" >}}

---

Cette traversée de huit décennies révèle des tensions qui n'ont jamais été
résolues, seulement déplacées. La quête de formalisme (modèle en cascade, UML,
processus lourds) a systématiquement été rattrapée par le pragmatisme des
praticiens (programmation structurée, open source, agilité). Le niveau
d'abstraction n'a cessé de monter, du code machine aux conteneurs orchestrés par
des fichiers de configuration, et maintenant au langage naturel lui-même. Et le
développement logiciel, activité solitaire à ses débuts, est devenu un travail
d'équipe à l'échelle mondiale, avec ses propres défis de coordination et de
communication. Ce sont ces tensions, et les réponses que le génie logiciel a
tenté d'y apporter, que nous explorerons dans la suite de ce cours.
