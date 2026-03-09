---
title: "Module 3 - Du programme au système"
weight: 300
bookCollapseSection: true
---

# Du programme au système

Dans le module précédent, on s'est concentré sur les outils du programmeur
individuel : les types, les tests, git, les dépendances, l'intégration continue.
Ces outils permettent de construire et de vérifier si un programme est correct
ou non. Mais un logiciel réel n'est presque jamais un programme simple. C'est un
ensemble de composants qui interagissent : une base de données, une API, un
système de cache, une file de messages, des services qui communiquent entre eux.
Le passage du programme au système est l'un des sauts conceptuels les plus
importants en génie logiciel, et c'est le sujet de ce module.

En 1986, Fred Brooks publie son célèbre essai *No Silver Bullet*, dans lequel il
distingue deux types de difficulté en développement logiciel. La difficulté
*accidentelle* vient des outils imparfaits : un langage verbeux, un compilateur
lent, un débogueur limité. Ces difficultés se résolvent avec de meilleurs
outils, et c'est en grande partie ce qu'on a vu dans le module précédent. Mais
la difficulté *essentielle* est d'une autre nature : elle vient de la complexité
intrinsèque du problème qu'on cherche à résoudre. Un système de commerce en
ligne doit gérer des utilisateurs, des produits, des inventaires, des paiements,
des livraisons, des remboursements, des notifications, de la fraude. Cette
complexité ne disparaît pas avec de meilleurs outils. Elle ne peut être que
*gérée*, et c'est précisément le défi que ce module aborde.

Au fil des décennies, les développeurs et les penseurs du génie logiciel ont
formulé un ensemble de principes pour faire face à cette complexité. Ces
principes ne sont pas des règles rigides : ce sont plutôt des heuristiques, des
guides de réflexion qui aident à prendre de meilleures décisions de conception.
Leur force tient souvent à leur simplicité. Un acronyme comme DRY (Don't Repeat
Yourself) ou YAGNI (You Ain't Gonna Need It) résume en quelques mots une
intuition profonde qui, une fois intériorisée, influence la manière dont on
pense et structure un système. Plutôt que de les présenter dans un catalogue
isolé, nous les introduirons tout au long de ce module et des suivants, dans les
contextes où ils sont le plus pertinents.

Ce module s'articule autour de trois grandes dimensions. D'abord,
l'**architecture et la modularité** : comment découper un système en parties
cohérentes, et quels sont les compromis impliqués. Ensuite, les **APIs** : le
mécanisme par lequel ces parties communiquent entre elles, et qui constitue
l'interface visible d'un système. Enfin, les **données** : comment on les
représente, comment on les stocke, et comment elles circulent à travers les
différentes couches d'un système. Ces trois dimensions sont étroitement liées,
et ensemble, elles forment l'ossature de tout logiciel qui dépasse le stade du
programme individuel.