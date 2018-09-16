---
title: Un blog avec Jekyll
layout: post
description: Vous aussi vous voulez un blog avec Jekyll ? C'est par ici...
image: "/jekyllblog/img/jekyll_start.png"
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
bundle install
bundle exec jekyll serve

```

Gem et Bundler sont deux gestionnaires de dépendances en ruby. Gem permet l'installation des dépendances sur la machine lorsque Bundler permet de mettre en place le nécessaire pour le lancement de l'application.

Une fois la dernière commande lancée, se rendre en 127.0.0.1:4000 pour admirer son site.

Vous devriez avoir quelque chose dans ce style :

![Jekyll Screen](/jekyllblog/img/jekyll_start.png)

# Ajout d'un template

Pour ajouter un template, rechercher un template de site qui vous plaît en ligne. L'idée est plus de comprendre l'architecture d'un template afin d'être capable de le modifier.
Pour ma part j'ai choisi le template [Jetflix](http://jekyllthemes.org/themes/jekflix/). Une fois téléchargé, j'ai supprimé les fonctionnalités dont je n'avais pas besoin.
Je vais maintenant essayer de présenter la structure d'un projet Jekyll.

## Structure du projet

- Le fichier Gemfile permet la gestion des dépendances Ruby, il est possible de le modifier en fonction des besoins.
- Le fichier \_config.yaml contient la majorité des informations sur l'application, permet de définir des liens fixes, des variables pour le site etc...

```yaml
# Variables utilisateurs
username: SVernin
user_description: Étudiant souhaitant partager ses découvertes

# Plugins
gems:
  - jemoji

# Liens dans le menu
# Pour des redirections externes ajouter : external: true
links:
  - title: Accueil
    url: /
```
- L'entête de chaque article peut permettre de déclarer de nouvelles variables yaml, en particulier pour le titre, un résumé etc.
- Le répertoire \_layouts vous permet de définir des types de pages en utilisant les variables du site :

```html
<!DOCTYPE html>
<html lang="en" itemscope itemtype="http://schema.org/WebPage">
    <body class="has-push-menu">
            <section class="content">
                { { content } }
            </section>
    </body>
</html>
```

> Ici un template par défaut, la variable *content* va être remplacée par le contenu de l'article qui utilise ce template.
>
> Les crochets entourant une variable ne sont pas séparés par un espace.
>
> Il est possible d'inclure d'autre template ou ressource statique dans un template avec le mot clef include.

- Le répertoire \_posts est celui où vous pourrez écrire vos articles en utilisant la syntaxe markdown. (dont voici un rappel [ici](https://fr.wikipedia.org/wiki/Markdown)) Le format du fichier devra être le suivant : **AAAA-MM-JJ-mon-article.md**
- Vous pouvez inclure d'autres répertoires qui seront copiés lors de la génération du site, pour y stocker vos images, CSS, JS...
- Le site sera généré dans le répertoire \_site

# Publication avec Git-pages

## Activation de Git-pages dans un projet Git.

Rendez-vous sur [GitHub](https://github.com/) sur le *repository* contenant les sources de votre site. Dans *Settings* il est possible d'activer le service GitPages pour le *repository* courant.

Cela va permettre de créer votre site sous l'URL suivante : https://nom_utilisateur.github.io/mon_repository/ .

## Paramétrage pour pouvoir tester votre site en local

Il se peut parfois qu'il y ait de légères différences entre le rendu de votre site en local et ce à quoi il ressemble une fois sur GitPages. En particulier, parfois si les liens ne sont pas correctement référencés, il se peut que des liens fonctionnant en local ne fonctionnent plus une fois votre site mis en ligne. Pour remédier à celà il suffit d'ajouter la ligne suivante à votre Gemfile :

```yaml
gem "github-pages", group: :jekyll_plugins
```

Vous pouvez ensuite lancer la commande bash afin de récupérer la dépendance manquante:
```bash
bundle update
```

Vous n'aurez alors plus de mauvaises surprises.

Pour admirer votre site en ligne, vous n'aurez plus qu'à *push* vos modifications.
