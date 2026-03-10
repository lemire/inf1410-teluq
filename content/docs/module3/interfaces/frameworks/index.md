---
title: "Les frameworks JavaScript"
weight: 30
---

# Les frameworks JavaScript

La section précédente a décrit les grandes architectures des applications web : SSR, SPA, hybride. Mais ces architectures ne se construisent pas à la main. Manipuler directement le HTML d'une page depuis JavaScript pour créer une application interactive est techniquement possible, mais rapidement ingérable dès que l'interface devient complexe. C'est pourquoi, depuis le milieu des années 2000, une succession de bibliothèques et de frameworks JavaScript ont émergé pour structurer le développement côté client. Leur histoire est traversée par une tension fondamentale entre deux approches : l'approche *impérative* (dire au navigateur *comment* modifier l'interface, étape par étape) et l'approche *déclarative* (décrire *ce que* l'interface devrait être, et laisser le framework se charger des modifications). Pour comprendre cette tension, il faut d'abord comprendre ce qu'est le DOM.

## Le DOM (Document Object Model)

Quand un navigateur reçoit un document HTML, il ne travaille pas directement avec le texte. Il le transforme en une structure de données arborescente appelée le **DOM** (Document Object Model) : chaque balise HTML devient un *noeud* dans l'arbre, avec des relations parent-enfant qui reflètent l'imbrication des balises. Un `<div>` qui contient un `<p>` qui contient du texte devient trois noeuds liés dans l'arbre. Le DOM n'est pas le HTML : c'est la *représentation vivante* de la page en mémoire. C'est sur cet arbre que le navigateur travaille pour afficher la page, et c'est cet arbre que JavaScript peut manipuler pour modifier ce qui est affiché.

Le DOM expose une API que JavaScript peut utiliser pour lire, modifier, ajouter ou supprimer des noeuds. Par exemple, `document.getElementById("mon-bouton")` retrouve un noeud dans l'arbre, et `element.textContent = "Nouveau texte"` en modifie le contenu. Chaque modification du DOM déclenche potentiellement un recalcul de la mise en page et un redessin de la page par le navigateur, ce qui peut être coûteux en performance. Cette caractéristique du DOM va devenir un enjeu central dans la conception des frameworks : comment minimiser les manipulations du DOM pour garder l'interface rapide et fluide ?

## jQuery : simplifier l'impératif

Au début des années 2000, manipuler le DOM directement avec l'API native du navigateur est pénible. Chaque navigateur (Internet Explorer, Firefox, Safari) implémente l'API différemment, ce qui oblige les développeurs à écrire du code conditionnel pour gérer ces incompatibilités. En 2006, John Resig publie **jQuery**, une bibliothèque qui résout ces deux problèmes d'un coup : elle fournit une API unifiée qui fonctionne sur tous les navigateurs, et elle simplifie drastiquement la syntaxe de manipulation du DOM. Sélectionner un élément, modifier son contenu, réagir à un clic : tout devient concis et lisible.

jQuery incarne l'approche impérative dans sa forme la plus pure. Le développeur dit exactement *quoi faire* au DOM, étape par étape : "trouve ce bouton, quand on clique dessus, cache ce paragraphe, puis ajoute cette classe CSS à cet autre élément". Pour une page avec quelques interactions, c'est efficace et intuitif. Mais quand l'application grandit, le code jQuery devient un enchevêtrement de callbacks qui modifient le DOM à différents endroits, sans vue d'ensemble de l'état de l'interface. On appelle parfois ça le *spaghetti jQuery* : quand l'utilisateur clique ici, quel est l'état de ce bouton là-bas ? Qui a modifié ce paragraphe, et quand ? Il n'y a pas de modèle de données centralisé, pas de structure claire. Le DOM lui-même devient la source de vérité de l'application, ce qui rend le code difficile à raisonner, à tester et à maintenir. C'est cette limite qui va motiver l'émergence d'une nouvelle génération d'outils.

## Angular : le framework qui structure tout

En 2010, Google publie **AngularJS**, le premier framework à proposer une architecture complète pour construire des SPA. Là où jQuery est une boîte à outils pour manipuler le DOM, AngularJS est un *cadre de travail* qui impose une structure : le code est organisé en contrôleurs, services et vues, selon un modèle inspiré de MVC. L'innovation centrale d'AngularJS est le **two-way data binding** (liaison de données bidirectionnelle) : quand l'utilisateur modifie un champ de formulaire, le modèle de données JavaScript est automatiquement mis à jour, et inversement, quand le code modifie les données, l'interface se met à jour sans manipulation explicite du DOM. Le développeur décrit la relation entre les données et l'interface, et le framework se charge de la synchronisation. C'est un virage vers le déclaratif.

En 2016, Google publie **Angular** (sans le "JS"), une réécriture complète du framework en TypeScript. Le changement est si radical qu'il s'agit en pratique d'un nouveau framework : pas de migration automatique possible depuis AngularJS. Angular adopte une architecture à base de composants (plutôt que de contrôleurs), intègre TypeScript comme langage principal, et fournit une solution intégrée pour chaque aspect du développement : routage, formulaires, requêtes HTTP, tests. Cette approche "batteries included" plaît aux grandes entreprises qui apprécient la structure et la prévisibilité, mais elle impose une courbe d'apprentissage abrupte et une certaine rigidité. Angular est un framework *opinionné* : il a une façon de faire les choses, et le développeur est invité à la suivre.

{{< hint info >}}
**TypeScript** est un langage développé par Microsoft (2012) qui ajoute un système de types statiques à JavaScript. Concrètement, TypeScript est un *surensemble* de JavaScript : tout code JavaScript valide est aussi du TypeScript valide, mais TypeScript permet en plus de déclarer les types des variables, des paramètres et des valeurs de retour. Un compilateur vérifie ces types avant l'exécution et produit du JavaScript standard comme sortie. L'idée est similaire à ce que font les type hints en Python avec mypy (qu'on a vus dans le module 2) : détecter les erreurs de type avant l'exécution plutôt qu'au runtime. TypeScript est devenu quasi incontournable dans l'écosystème des frameworks JavaScript modernes.
{{< /hint >}}

## React : le retour du déclaratif

En 2013, Facebook open-source **React**, une bibliothèque qui propose une approche radicalement différente de la construction d'interfaces. L'idée centrale de React est simple mais puissante : plutôt que de manipuler le DOM morceau par morceau (comme jQuery) ou de synchroniser les données et l'interface par des mécanismes complexes (comme Angular), le développeur écrit une *fonction* qui, étant donné un état, retourne une description de ce que l'interface devrait afficher. Quand l'état change, React rappelle la fonction, compare le résultat avec ce qui est actuellement affiché, et applique uniquement les modifications nécessaires au DOM. C'est le principe du **DOM virtuel** (*virtual DOM*) : React maintient une copie légère de l'arbre DOM en mémoire, calcule les différences (*diff*) entre l'ancien et le nouveau, et ne touche au vrai DOM que pour les noeuds qui ont changé.

Ce modèle est profondément déclaratif. Le développeur ne dit plus "quand l'utilisateur clique, modifie tel élément" ; il dit "voici à quoi l'interface ressemble quand l'état est X". React se charge de la transition entre les états. C'est un retour conceptuel à la simplicité du HTML (qui est lui-même déclaratif : on décrit la structure, le navigateur s'occupe du rendu), mais appliqué à des interfaces dynamiques.

L'autre idée structurante de React est le **composant**. La notion de composant d'interface n'est pas nouvelle : on la retrouve dans tous les toolkits graphiques, des widgets Tk aux contrôles Visual Basic en passant par les composants Swing. L'idée est toujours la même : un morceau d'interface autonome et réutilisable (un bouton, un formulaire, une carte de profil, une barre de navigation) qui encapsule à la fois son apparence et son comportement. Cette encapsulation crée une correspondance naturelle avec la programmation orientée objet : un bouton est un objet, avec un état interne (pressé ou non, activé ou non), des propriétés (son texte, sa couleur) et des méthodes (réagir au clic). C'est d'ailleurs ainsi que la plupart des toolkits implémentent leurs composants, et c'est aussi l'approche qu'Angular utilise avec ses classes TypeScript. React, cependant, a pris une direction différente. Plutôt que de modéliser les composants comme des objets avec de l'héritage et des méthodes, React les définit comme des *fonctions pures* : une fonction qui reçoit des propriétés (*props*) en entrée et retourne une description de l'interface en sortie. L'état est géré non pas par des attributs d'objet, mais par des *hooks* (`useState`, `useEffect`), des fonctions qui "accrochent" du comportement à un composant fonctionnel. Cette approche, plus proche de la programmation fonctionnelle que de l'OOP, rend les composants plus faciles à composer, à tester et à raisonner, au prix d'un modèle mental inhabituel pour les développeurs habitués aux objets.

Pour définir ce que chaque composant affiche, React utilise **JSX**, une extension syntaxique qui permet d'écrire du HTML directement dans le code JavaScript :

```jsx
function Compteur() {
  const [compte, setCompte] = useState(0);
  return (
    <div>
      <p>Vous avez cliqué {compte} fois</p>
      <button onClick={() => setCompte(compte + 1)}>
        Cliquer
      </button>
    </div>
  );
}
```

Le composant `Compteur` est une fonction JavaScript qui retourne du JSX, un mélange de HTML et d'expressions JavaScript (entre accolades `{}`). L'état du compteur est géré par `useState`, un *hook* de React : quand l'utilisateur clique sur le bouton, `setCompte` met à jour l'état, React rappelle la fonction, et l'interface se redessine avec la nouvelle valeur. Le développeur n'a jamais manipulé le DOM directement.

JSX a provoqué une controverse à sa sortie : il mélange le HTML et la logique dans le même fichier, ce qui semble violer le principe de Separation of Concerns incarné par la trinité HTML/CSS/JS. Mais l'argument de React est que la séparation pertinente n'est pas entre les *technologies* (HTML d'un côté, JavaScript de l'autre), mais entre les *responsabilités* (chaque composant est responsable de son propre rendu et de sa propre logique). C'est un changement de perspective sur ce que "séparer les préoccupations" signifie réellement. React est aujourd'hui le framework le plus utilisé pour le développement d'interfaces web, et ses idées (composants, déclaratif, DOM virtuel) ont influencé tous les frameworks qui l'ont suivi.

## Vue.js : l'approche progressive

En 2014, **Evan You**, un ancien développeur de Google qui avait travaillé sur AngularJS, publie **Vue.js** avec une philosophie différente. Là où Angular impose un cadre complet et où React se présente comme une bibliothèque minimale qu'il faut compléter avec d'autres outils, Vue se positionne comme un framework *progressif* : on peut l'adopter graduellement, en commençant par ajouter de la réactivité à une page existante, puis en structurant progressivement l'application en composants, en ajoutant un routeur, un gestionnaire d'état, etc. Cette approche "à la carte" rend Vue particulièrement accessible aux développeurs qui viennent du HTML/CSS/jQuery et qui veulent adopter un framework moderne sans tout réécrire d'un coup.

Vue emprunte des idées à la fois à Angular (les templates HTML avec des directives comme `v-if` et `v-for`) et à React (les composants, le flux de données unidirectionnel). Son système de **réactivité** est au coeur du framework : quand une donnée change, Vue détecte automatiquement quels composants en dépendent et ne met à jour que ceux-là. Contrairement à React qui utilise un DOM virtuel et un algorithme de diff, Vue utilise un système de suivi des dépendances plus fin, inspiré des *observables*, qui peut être plus efficace pour certains types de mises à jour. Vue a connu un succès particulier en Asie et dans les communautés francophones, et reste le deuxième framework le plus populaire après React.

## Svelte : la disparition du framework

En 2016, **Rich Harris**, journaliste devenu développeur au *New York Times*, publie **Svelte** avec une idée provocatrice : et si le framework n'existait pas au runtime ? React, Angular et Vue embarquent tous un moteur d'exécution (*runtime*) dans le navigateur : du code JavaScript qui gère le DOM virtuel, le suivi des dépendances ou la synchronisation des données. Ce runtime a un coût : il faut le télécharger, l'analyser et l'exécuter avant que l'application ne devienne interactive. Svelte prend l'approche inverse : c'est un **compilateur**. Au moment du build, Svelte analyse les composants écrits par le développeur et génère du JavaScript natif et optimisé qui manipule le DOM directement, sans couche d'abstraction intermédiaire. Il n'y a pas de DOM virtuel, pas de diff, pas de runtime : juste le code minimal nécessaire pour faire fonctionner l'interface.

Le résultat est des applications plus légères et souvent plus rapides, surtout sur les appareils modestes (téléphones d'entrée de gamme, connexions lentes). La syntaxe de Svelte est aussi remarquablement concise : la réactivité est intégrée au langage lui-même (une simple assignation `count = count + 1` suffit à déclencher une mise à jour de l'interface), sans avoir besoin d'appeler `setState` ou d'utiliser des hooks. Svelte reste moins adopté que React ou Vue dans l'industrie, mais son approche par compilation influence de plus en plus le domaine. Son meta-framework **SvelteKit** (qu'on a mentionné dans la section précédente) offre le même type de fonctionnalités full-stack que Next.js ou Nuxt.

