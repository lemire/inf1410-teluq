---
title: "Principes et idées clés"
weight: 15
---

# Principes et idées clés du génie logiciel

Cette page rassemble les principes et idées importantes qui traversent
l'ensemble du cours. Chaque principe est traité en profondeur dans le module où
il est le plus pertinent ; les liens ci-dessous pointent vers ces sections.

*(Cette page sera enrichie au fur et à mesure que les principes seront
développés dans leurs sections respectives.)*

## Gestion de la complexité

- **Complexité essentielle vs accidentelle** (Fred Brooks, *No Silver Bullet*, 1986) → [Module 3, Introduction]({{< ref "/docs/module3" >}})

## Principes de conception

- **Information hiding** (David Parnas, 1972) : chaque module cache une décision de conception susceptible de changer → [Module 3, Architecture et modularité]({{< ref "/docs/module3/architecture" >}})
- **DRY** (Don't Repeat Yourself, Hunt & Thomas, *The Pragmatic Programmer*, 1999) : chaque connaissance doit avoir une représentation unique → [Module 3, Architecture et modularité]({{< ref "/docs/module3/architecture" >}})
- **KISS** (Keep It Simple Stupid)
- **YAGNI** (You Ain't Gonna Need It, Kent Beck, Extreme Programming) : ne construis pas d'abstraction pour un besoin qui n'existe pas encore → [Module 3, Architecture et modularité]({{< ref "/docs/module3/architecture" >}})
- **Separation of Concerns**
- **SOLID** (Robert C. Martin, *Clean Code*, 2008) : cinq principes de conception OO (S, O, L, I, D) → [Module 3, Architecture et modularité]({{< ref "/docs/module3/architecture" >}})
- **Law of Demeter**
- **Composition over inheritance** (Gang of Four, *Design Patterns*, 1994) : favoriser l'assemblage d'objets plutôt que l'héritage de classes → [Module 3, Architecture et modularité]({{< ref "/docs/module3/architecture" >}})
- **Loi de Conway** (Melvin Conway, 1967) : la structure d'un système reflète la structure de communication de l'organisation qui le produit → [Module 3, Architecture et modularité]({{< ref "/docs/module3/architecture" >}})
- **Principle of Least Astonishment (POLA)**
- **Idempotence** : une opération qu'on peut exécuter plusieurs fois avec le même résultat, propriété cruciale pour les APIs réseau → [Module 3, Les APIs]({{< ref "/docs/module3/apis" >}})

## Principes de pratique

- **Boy Scout Rule** : "Always leave the campground cleaner than you found it."

## Lectures et publications de référence

Voir la page [Lectures et publications de référence]({{< ref "/docs/lectures" >}}).