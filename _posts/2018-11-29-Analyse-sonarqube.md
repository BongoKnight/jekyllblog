---
title: Vérification du code avec SonarQube
layout: post
description: "Vérification de code avec SonarQube."
image: "/jekyllblog/img/sonarqube/sonarqube.png"
category: 'tutorial'
tags:
- Web
- tutorial

twitter_text: "Vérification de code avec SonarQube."
introduction: "Vérification de code avec SonarQube."
---

# Contexte

Dans cet article nous allons nous servir de SonarQube pour analyser le code qui a été fait dans l'article sur la réalisation d'un [add-on Firefox](../make-firefox-addon/index.html).
Le but ici n'est pas de se focaliser sur l'installation de SonarQube mais plus de regarder et de corriger le code qui a été fait.

# Installation de SonarQube

SonarQube est un outil d'analyse de la qualité d'un code. Il regarde différentes métriques comme le niveau de couverture par les tests et par la documentation. Mais aussi permet de créer ses propres règles d'analyse de code. Enfin un accent est mis sur les failles de l'[OWASP Top10](https://www.owasp.org/index.php/Top_10-2017_Top_10).

Pour l'installer, il suffit de se rendre sur leur [site](https://www.sonarqube.org/), de télécharger la version communautaire puis de suivre le guide de démarrage: [https://docs.sonarqube.org/7.4/setup/get-started-2-minutes/](https://docs.sonarqube.org/7.4/setup/get-started-2-minutes/)

> Il est important de souligner que comme préciser dans ce guide, l'installation de ce guide est limitée à un usage interne, pour tester vos projets en local. Pour une utilisation "industrielle", il est nécessaire de suivre le [guide d'installation](https://docs.sonarqube.org/7.4/setup/install-server/).

# Analyse du code

## Lancement de l'analyse

Après avoir lancé l'interface, se rendre sur : `localhost:9000` et se connecter avec les identifiants de base `admin:admin`.
Suivre les consignes pour créer un nouveau projet et lancer le scan.

## Résultats

### Résultats généraux

Les résultats sont les suivants :
![Résultats de SonarQube](/jekyllblog/img/sonarqube/results.png)

Ceux-ci ne sont clairement pas encourageants. Essayons de voir ce qui peut être amélioré.

Jetons d'abord un œil à l'origine des différents *code smells* et vulnérabilités.

### Considérations sur NodeJS

L'utilisation de *node* et et de *react* entraîne qu'une bonne partie du code écrit pour l'application n'est pas écrit par nous. Pour se concentrer sur la partie du code qui est à nous il est nécessaire de ne regarder que dans les sources.

La majorité du code dupliqué et des beugs détecté est présent dans le fichier *panel.js* qui est généré au moment du *build* de l'application. En se rendant dans l'onglet `Code` on peut sélectionner seulement un répertoire qui nous intéresse et afficher les problématiques seulement liées à ce dossier.

Voici les résultats :

![Résultats uniquement pour le code source du module](/jekyllblog/img/sonarqube/results_cleaned.png)

### Analyse des problèmes restants

#### Duplication de code

![Exemple de code dupliqué](/jekyllblog/img/sonarqube/duplicate.png)

Deux portions de code sont marquées comme étant dupliquée, une l'est dans le fichier "de sortie" `dist/panel.js` ce qui n'est pas nécessairement étonnant. L'autre l'est dans un sous-dossier des sources dans lequel les sources du plugins de démonstration `react-to-do` est présent. S'étant inspiré de ce code ainsi que de deux autres plugins il n'est pas étonnant de le retrouver ici. Cependant dans le "code final" (panel.js), ce code n'est pas dupliqué.

#### Code smells

Pour la partie *code smells*  plusieurs petites choses à corriger :
- des imports inutiles (ça m'apprendra à ne pas m'être servi d'un IDE)
- des assignations inutiles : `var storingNote = browser.storage.local.set({ [updatedItem.key] : updatedItem.text });` sans avoir besoin de se resservir de `storingNote` plus loin.


#### Vulnérabilités

Les vulnérabilités déclarées lors de l'analyse m'interrogent tout de même quand à leur origine, penchons nous dessus :
- L'utilisation de `alert()` est déconseillé, l'appel à cette fonction permet ici, il me semble d'alerter l'utilisateur sur la non compatibilité de son navigateur. C'est un code "rajouté" par React.

#### Remarques

Pour chaque portions de code à réviser, il y a en cliquant sur `...` un petit texte explicatif de l'erreur faite, un exemple de "bon" et de "mauvais" code. Et éventuellement un lien vers des articles pour plus d'informations.

Il est possible de définir des métriques personnelles mais je reviendrais dessus dans un autre article, ou plus tard ici.


# COnclusion

La découverte de ce logiciel est pour moi une bonne surprise. Il a l'air assez complet l'interface est bien faite. L'accompagnement à la résolution des problèmes est intelligemment pensée. Je l'ai essayé rapidement juste pour voir à quoi cela mais à l'occasion faire le test sur un code plus *industriel* me plairait. Je garde donc tout ça dans un coin de la tête.
