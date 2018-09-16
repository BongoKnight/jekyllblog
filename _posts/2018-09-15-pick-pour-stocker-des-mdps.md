---
title: Un petit gestionnaire de mot de passe
layout: post
description: Un petit gestionnaire de mots de passe pour vos projets personnels.
image: "/jekyllblog/img/pick-mdp.gif"
category: 'short'
tags:
- CLI-Tools
- security
- password

twitter_text: "Un petit utilitaire permettant de stocker des notes ou des mots de passe de façon sure: pick."
introduction: "Je vous présente dans cet article, un petit gestionnaire de mots de passe."
---
> Testé sur Ubuntu 16.04LTS en Septembre 2018

# Présentation

Pick est un gestionnaire de mots de passe, écrit en *Go* dont les sources sont disponibles sur [GitHub](https://github.com/bndw/pick), souvent dans des projets on est amené à avoir des mots de passe pour des bases de test, des comptes d'administration etc.
Il est bon de les avoir sous la main même lorsque l'on est dans son terminal. Et il est recommandé de ne pas les laisser traîner en clair. Pick est un outil qui répond tout à fait à cette problématique.

Il permet en effet :
- de générer des nouveaux mots de passe,
- de stocker les siens,
- de stocker des petites notes,
- de récupérer un mot de passe dans son presse papier.

Le tout de manière sure car tout est chiffré en utilisant un mot de passe principale comme dans tout gestionnaire de mots de passe.

# Quelques commandes

```bash
# Pour initialiser le mot de passe principal
pick init

# Ajout d'un compte
pick add work/email

# Lister ses comptes
pick ls

# Voir un mdp
pick cat work/email

# Copier un mdp
pick cp work/email
```
