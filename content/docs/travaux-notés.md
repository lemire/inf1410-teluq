---
title: "Travaux notés"
weight: 20
---

# Les travaux notés

Étant donné qu'il s'agit d'un cours de génie logiciel, nous avons cherché à
faire en sorte que les travaux soient cohérents et évolutifs. Nous avons donc
choisi un modèle de développement d'application qui comportera différents volets
reliés aux grands sujets du cours.

Nous allons vous demander de développer une application (web) d'affaire
commerciale et transactionnelle (possiblement style SaaS).

Dans un premier temps, il vous faudra imaginer la nature exacte de cette
application, pour ensuite simuler une rencontre du client imaginaire, afin d'en
extraire les spécifications et le fonctionnement exacts. Vous devrez ensuite
produire un document de conception, en utilisant une technique de modélisation
de votre choix, afin de tenter de penser d'avance au fonctionnement de votre
application, avant de vous lancer dans le développement à proprement parler.

Notez qu'il vous est possible de travailler en petites équipes (de trois
maximum), en faisant la coordination et la communication sur la plateforme
Discord du cours.

Votre projet devra comporter un dépôt géré sur Github, ainsi qu'un historique de
commits et de branches qui démontrera clairement l'évolution de votre travail.

Vous devrez utiliser une plateforme de gestion de projet, afin de faire le suivi
et la planification du développement de votre application. Il y a plusieurs
options gratuites : ClickUp, Basecamp, Jira, Trello, etc. Si vous travaillez
seul, vous devrez simuler au moins deux rôles distincts dans votre processus :
celui de chef de projet, et celui de développeur. Si vous travaillez en équipe,
vous pourrez vous distribuer naturellement ces rôles, ou toute autre formule qui
vous apparaîtra raisonnable dans le contexte.

Votre application devra gérer ses données avec une base de données relationnelle
(SQL). Elle devra également comporter obligatoirement un mécanisme
d'authentification et de création d'usagers.

Étant donné qu'il s'agit d'une application commerciale transactionnelle, votre
application devra faire usage de l'API Stripe afin de traiter le paiement en
ligne (simulé) pour les usagers.

Finalement, vous devrez mettre au point un système de déploiement automatisé de
votre application, sur une plateforme d'infonuagique en ligne: Digital Ocean,
AWS, Azure. Utiliser la conteneurisation dans ce contexte est recommandé.

Voici les livrables que vous devrez être en mesure de remettre à la fin du cours :

1. Un rapport de la rencontre (imaginaire ou non) avec le client afin d'extraire les spécifications de votre application
2. Un document de conception, selon la technique de modélisation de votre choix
3. Un lien vers la plateforme de gestion de projet que vous aurez utilisée
4. Un dépôt Github avec le code source de votre application, ainsi qu'un
   historique de commits et de branches non-trivial, qui démontrera clairement
   l'évolution de votre travail et de vos efforts
5. Un document qui expliquera votre modèle de données et les choix de conception qui y sont reliés
6. Un lien vers votre application 100% fonctionnelle, disponible en ligne
6. Un journal de bord qui détaillera l'historique complet de vos processus, sous
   la forme d'entrées datées; vos décisions et votre réflexion devront faire
   explicitement référence au contenu du cours, et expliquer en quoi elles sont
   justifiées