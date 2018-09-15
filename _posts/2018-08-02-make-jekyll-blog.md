---
title: Un blog avec Jekyll
layout: post
description: Vous aussi vous voulez un blog avec Jekyll ? C'est par ici...
image: '/img/jekyll_start.png'
category: 'Jekyll'
tags:
- jekyll
- blogpost
twitter_text: Comment construire son premier article avec Jekyll ?
introduction: Comment construire son premier article avec Jekyll ?
---
> Testé sur Ubuntu 16.04LTS en Septembre 2018

# Installation

Pour l’installation de Jekyll en tant que tel les informations données sur le site sont largement suffisantes :

```bash
# Installation de ruby
sudo apt-get install ruby ruby-dev build-essential
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME=$HOME/gems' >> ~/.bashrc
echo 'export PATH=$HOME/gems/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Installation de jekyll
gem install jekyll bundler

# Lancement de jekyll
jekyll new blog
cd Myblog
# ls pour vérifier la présence d'un Gemfile
bundle exec jekyll serve

```

Gem et Bundler sont deux gestionnaires de dépendances en ruby. Gem permet l'installation des dépendances sur la machine lorsque Bundler permet de mettre en place le nécessaire pour le lancement de l'application.

Une fois la dernière commande lancée, se rendre en 127.0.0.1:4000 pour admirer son site.

Vous devriez avoir quelque chose dans ce style :

![Jekyll Screen](/img/jekyll_start.png)

# Ajout d'un template

## Structure du projet
