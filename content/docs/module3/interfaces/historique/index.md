---
title: "Perspective historique"
weight: 10
---

# Perspective historique des interfaces utilisateur

L'interface utilisateur est le point de contact entre un humain et un logiciel. C'est la partie visible, celle que l'utilisateur touche, voit et manipule. Pourtant, la manière dont on conçoit cette interaction a radicalement changé au fil des décennies. Les premiers programmeurs interagissaient avec leurs machines par des cartes perforées et des imprimantes ; aujourd'hui, on glisse du doigt sur des écrans tactiles ou on parle à des assistants vocaux. Cette évolution n'est pas qu'une question de technologie : elle reflète des changements profonds dans notre conception de ce qu'un ordinateur peut et devrait être. Comprendre cette histoire permet de mieux saisir pourquoi les interfaces modernes sont construites comme elles le sont, et quelles tensions fondamentales continuent de les façonner.

## Les terminaux texte et la ligne de commande

Avant les écrans graphiques, l'interaction avec un ordinateur passait par le texte. Dans les années 1960 et 1970, les *teletypes* (ou TTY) servaient à la fois de clavier et d'imprimante : l'utilisateur tapait une commande, et la machine répondait en imprimant du texte sur du papier. Le terme "terminal" vient de cette époque, et l'abréviation TTY survit encore aujourd'hui dans les systèmes Unix (la commande `tty` affiche le nom du terminal courant). Avec l'arrivée des écrans cathodiques, les terminaux vidéo comme le célèbre VT100 de DEC (1978) ont remplacé le papier par un affichage à l'écran, mais le modèle d'interaction restait le même : une boucle textuelle où l'utilisateur tape une commande et attend une réponse.

<!-- TODO: ajouter un vidéo YouTube de la "Mother of All Demos" de Douglas Engelbart (1968) -->

Ce modèle d'interaction textuel a donné naissance aux *shells* : le Bourne Shell (1979), puis Bash (1989), qui sont encore omniprésents aujourd'hui. On associe souvent le shell à Linux, mais macOS, qui est une variante de Unix, inclut aussi un shell par défaut (Bash jusqu'en 2019, puis Zsh), et Windows a développé ses propres environnements en ligne de commande, de l'ancien `cmd.exe` au plus moderne PowerShell. La ligne de commande (CLI) n'a jamais disparu, et pour cause. Elle offre une puissance de composition que les interfaces graphiques peinent à égaler : on peut enchaîner des commandes avec des pipes, automatiser des tâches avec des scripts, et interagir avec des machines distantes par SSH. Comme on l'a vu dans la section sur l'architecture, la philosophie Unix de programmes simples qui communiquent par des flux de texte (pipes and filters) repose entièrement sur ce modèle textuel. La CLI reste l'interface privilégiée des développeurs, des administrateurs système, et de quiconque a besoin de contrôle précis sur une machine.

## Xerox PARC, Smalltalk et la naissance du GUI

Au début des années 1970, au centre de recherche Xerox PARC en Californie, une équipe dirigée par Alan Kay imagine une vision radicalement différente de l'interaction humain-machine. Plutôt que de taper des commandes textuelles, l'utilisateur manipulerait des objets visuels à l'écran à l'aide d'un dispositif de pointage : la souris, inventée par Douglas Engelbart en 1968. Le Xerox Alto (1973), qui n'a jamais été commercialisé à grande échelle, est le premier ordinateur à intégrer une interface graphique (GUI) avec des fenêtres, des icônes et des menus. La métaphore du bureau (*desktop metaphor*) y prend forme : les fichiers deviennent des icônes, les dossiers s'ouvrent dans des fenêtres, et la corbeille permet de "jeter" des documents.

C'est aussi à Xerox PARC que naît Smalltalk (1972-1980), un langage de programmation orienté objet conçu par Alan Kay et son équipe. Smalltalk n'est pas qu'un langage : c'est un environnement complet où tout est objet, y compris les éléments de l'interface. L'utilisateur interagit directement avec les objets à l'écran, et le programmeur construit l'interface dans le même environnement. C'est dans ce contexte que Trygve Reenskaug invente le patron MVC (Model-View-Controller) en 1979, une idée architecturale qu'on a déjà rencontrée dans la section sur l'architecture. La tragédie de Xerox, souvent racontée, est que l'entreprise n'a pas su commercialiser ces innovations. C'est Steve Jobs qui, après une visite au PARC en 1979, en saisit le potentiel et lance le Macintosh en 1984 : le premier ordinateur personnel grand public doté d'une interface graphique. Microsoft suivra avec Windows 1.0 en 1985, puis Windows 3.0 en 1990, qui popularisera véritablement le GUI sur PC.

## Le modèle événementiel

L'apparition du GUI a entraîné un changement profond dans la manière de programmer. Dans un programme en ligne de commande, le flux d'exécution est séquentiel et prévisible : le programme demande une entrée, l'utilisateur répond, le programme continue. Avec une interface graphique, l'utilisateur peut cliquer n'importe où, à n'importe quel moment : un bouton, un menu, une barre de défilement. Le programme ne peut plus dicter l'ordre des opérations. Il doit plutôt *réagir* aux actions de l'utilisateur.

C'est le principe de la **boucle d'événements** (*event loop*) : le programme tourne en boucle, attend qu'un événement se produise (un clic de souris, une touche pressée, un redimensionnement de fenêtre), puis appelle la fonction appropriée pour le traiter. Ces fonctions, qu'on appelle des *callbacks* ou des *handlers*, sont enregistrées à l'avance pour chaque type d'événement. Ce modèle est souvent appelé "l'inversion du contrôle" (*inversion of control*) : ce n'est plus le programme qui contrôle le flux, c'est l'utilisateur (ou plus précisément, le framework graphique) qui décide quand chaque morceau de code s'exécute.

Pour illustrer la différence, comparons un programme CLI classique et son équivalent GUI. D'abord, la version séquentielle :

```python
# Programme CLI : le flux est contrôlé par le programme
nom = input("Votre nom : ")
print(f"Bonjour, {nom} !")
```

Et maintenant, la version événementielle avec Tkinter, le toolkit graphique inclus avec Python :

```python
import tkinter as tk

def saluer():
    nom = champ.get()
    label_resultat.config(text=f"Bonjour, {nom} !")

# Construction de l'interface
fenetre = tk.Tk()
fenetre.title("Salutation")

champ = tk.Entry(fenetre)
champ.pack()

bouton = tk.Button(fenetre, text="Saluer", command=saluer)
bouton.pack()

label_resultat = tk.Label(fenetre, text="")
label_resultat.pack()

# La boucle d'événements : le programme attend et réagit
fenetre.mainloop()
```

La différence est structurelle. Dans la version CLI, le code se lit de haut en bas. Dans la version GUI, on *déclare* des éléments visuels (un champ de texte, un bouton, un label), on *associe* une fonction callback au bouton (`command=saluer`), puis on cède le contrôle à `mainloop()`. À partir de ce moment, c'est Tkinter qui décide quand appeler `saluer()`, en réponse au clic de l'utilisateur.

## Visual Basic et les toolkits desktop

L'exemple Tkinter ci-dessus illustre une approche purement programmatique : chaque élément d'interface est créé par du code. En 1991, Microsoft lance Visual Basic, qui propose une approche radicalement différente. Le programmeur *dessine* son interface en glissant-déposant des composants (boutons, champs de texte, listes déroulantes) sur un formulaire visuel, puis écrit le code des callbacks directement associés à chaque composant. C'est le paradigme du *RAD* (Rapid Application Development) : construire une interface fonctionnelle en quelques minutes plutôt qu'en quelques heures. Visual Basic a démocratisé le développement d'applications graphiques en le rendant accessible à des non-spécialistes. On moque parfois la qualité du code qu'il produisait, mais son impact a été immense : des millions d'applications d'entreprise ont été construites avec cet outil dans les années 1990.

En parallèle, le monde du développement GUI se structure autour de *toolkits* : des bibliothèques qui fournissent les composants visuels de base (boutons, menus, fenêtres) et la boucle d'événements. Chaque plateforme développe le sien : Win32 et MFC pour Windows, Cocoa pour macOS, GTK pour l'environnement GNOME sur Linux. Tk, qu'on a vu dans l'exemple Python, est l'un des plus anciens (1991) et reste disponible aujourd'hui. Java, lancé par Sun Microsystems en 1995 avec le slogan "write once, run anywhere", a tenté de résoudre le problème multiplateforme à la racine : son toolkit AWT, puis Swing, permettaient d'écrire une interface graphique qui fonctionnait (en théorie) sur n'importe quel système d'exploitation grâce à la machine virtuelle Java. En pratique, les interfaces Swing avaient un aspect générique qui ne correspondait jamais tout à fait à l'apparence native de chaque plateforme. Qt, développé à partir de 1995 par la société norvégienne Trolltech, a pris une approche différente : un toolkit C++ multiplateforme qui compilait en code natif et s'adaptait à l'apparence de chaque système. Le choix d'un toolkit était une décision lourde de conséquences : il liait l'application à une plateforme, un langage, et un style visuel. Cette tension entre la spécificité de chaque plateforme et le désir d'écrire du code une seule fois est un thème récurrent qu'on retrouvera tout au long de cette section.

## Flash et le web interactif

À la fin des années 1990, le web est encore essentiellement constitué de pages statiques : du texte, des images, des liens hypertexte. Les formulaires HTML permettent une interaction limitée, mais rien qui s'approche de la richesse d'une application desktop. C'est dans ce vide que s'engouffre Flash, développé par Macromedia (racheté par Adobe en 2005). Flash permet de créer des animations, des jeux, des lecteurs vidéo et des interfaces interactives complexes, le tout exécuté dans un plugin intégré au navigateur. Avec son langage ActionScript, Flash offre aux développeurs un environnement de programmation événementiel complet, similaire à ce qu'offraient les toolkits desktop, mais accessible via le web.

Pendant près d'une décennie, Flash domine le web interactif. YouTube l'utilise pour son lecteur vidéo, des sites entiers sont construits en Flash, et une industrie florissante de jeux Flash émerge. Mais Flash a des défauts structurels : il est propriétaire (contrôlé par Adobe), gourmand en ressources, et pose des problèmes de sécurité récurrents. En 2010, Steve Jobs publie sa lettre ouverte "Thoughts on Flash", où il annonce qu'Apple ne supportera jamais Flash sur l'iPhone et l'iPad. C'est le début de la fin. La montée de HTML5, qui intègre nativement la vidéo, le canvas et les animations CSS, rend Flash progressivement obsolète. Adobe cessera officiellement le support de Flash en 2020. L'épisode Flash illustre un principe important : une technologie propriétaire, aussi dominante soit-elle, est vulnérable face à des standards ouverts qui finissent par la rattraper.

L'histoire de Flash s'inscrit dans un mouvement plus large qu'on a appelé le **Web 2.0**, un terme popularisé par Tim O'Reilly à partir de 2004. Le Web 2.0 ne désigne pas une technologie particulière, mais un changement de paradigme : le web passe d'un média de consultation (des pages qu'on lit) à un média de participation (des applications qu'on utilise). Wikipédia, Flickr, YouTube, Facebook, Google Maps : ces services transforment l'utilisateur en contributeur et l'interface web en quelque chose qui ressemble de plus en plus à une application desktop. C'est dans ce contexte que la technique AJAX (Asynchronous JavaScript and XML) émerge en 2005, permettant à une page web de communiquer avec le serveur en arrière-plan sans rechargement complet. Google Maps, avec son défilement fluide de la carte, est la démonstration éclatante de ce que le web peut devenir quand on combine JavaScript, AJAX et une interface bien conçue. Le terrain est prêt pour que le web devienne une véritable plateforme d'interfaces utilisateur.

## Le web comme plateforme UI

En l'espace de quinze ans, le web a effectué une transformation remarquable. Ce qui était au départ un système de documents hypertexte, conçu par Tim Berners-Lee en 1991 pour partager des articles scientifiques, est devenu la plateforme d'interfaces utilisateur la plus universelle de l'histoire de l'informatique. Le navigateur, initialement un simple lecteur de documents, s'est mué en un environnement d'exécution complet, capable de rivaliser avec les applications desktop.

Cette transformation repose sur trois piliers : HTML pour la structure du contenu, CSS pour sa présentation visuelle, et JavaScript pour le comportement interactif. Ce trio, parfois appelé la "holy trinity" du développement web, incarne un découpage qui rappelle le principe de Separation of Concerns : chaque technologie a sa responsabilité propre. En pratique, cette séparation s'est avérée plus complexe qu'elle ne le paraît, et les frameworks modernes la remettent parfois en question, comme on le verra dans la section sur les frameworks JavaScript.

Le web présente un avantage décisif sur les toolkits desktop : le déploiement. Une application web n'a pas besoin d'être installée, mise à jour ou distribuée. L'utilisateur ouvre un URL et obtient toujours la dernière version. C'est cet avantage qui a poussé de plus en plus d'applications à migrer vers le web, des courriels (Gmail, 2004) aux suites bureautiques (Google Docs, 2006), en passant par les outils de design (Figma, 2016). Mais cette migration vers le web a aussi soulevé de nouvelles questions d'architecture : comment organiser le code d'une application web complexe ? Où s'exécute la logique, sur le serveur ou dans le navigateur ? Comment gérer l'état de l'interface ? Ces questions sont l'objet des sections qui suivent.
