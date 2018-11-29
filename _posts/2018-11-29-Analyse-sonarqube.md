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

Les résultats sont les suivants :
![Résultats de SonarQube](/jekyllblog/img/sonarqube/results.png)

Ceux-ci ne sont clairement pas encourageants. Essayons de voir ce qui peut être amélioré.
