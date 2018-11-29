---
title: Réalisation d'un plugin de prise de notes pour Firefox
layout: post
description: "Réalisation d'un plugin de prise de notes pour Firefox."
image: "/jekyllblog/img/addon/addon.png"
category: 'tutorial'
tags:
- Web
- tutorial

twitter_text: "Réalisation d'un plugin de prise de notes pour Firefox."
introduction: "Réalisation d'un plugin de prise de notes pour Firefox permettant la syntaxe Markdown, un stockage persistant..."
---

# Contexte et besoin

L'idée de ce projet est de faire une extension qui simplifie la prise de notes lors de la réalisation de challenges de sécurité. En effet en ayant souvent un navigateur d'ouvert, plus quelques terminaux, plus une documentation, plus une RFC, plus un plein d'autres choses plus ou moins utiles il est pratique d'avoir dans son navigateur un petit endroit pour prendre des notes du genre :
- CMS SuperCMS_nonmaintenu_v.3.0 : CVE-2018-5468435
- Champs à tester :
  - authentification
  - recherche

Puis de détailler plus tard dans un autre document ce qui a été fait. L'important est lors de la découverte d'une application Web de pouvoir tout noter rapidement sans avoir à changer d'application sans cesse pour ne rien oublier.

Je cite ici deux extensions de navigateurs qui peuvent apporter quelques petites informations précieuses et qui savent se faire discrètes :
- [Shodan.io](https://addons.mozilla.org/en-US/firefox/addon/shodan_io/) basé sur le service du même nom et qui donne un aperçu des ports ouverts sur l'hôte qui héberge le site courant. Ce qui peut renseigner sur les services qui y sont hébergés.
- [Wappalyzer](https://www.wappalyzer.com/) qui liste les différentes technos et services identifiés sur le site courant.

L'exercice est ici de comprendre la réalisation d'un plugin pour le navigateur. Ces applications sont basiquement des pages HTML particulière affichées par le navigateur. L'utilisation de l'API [WebExtension](https://developer.mozilla.org/fr/docs/Mozilla/Add-ons/WebExtensions) de Mozilla permet de dynamiser la page et d'y inclure différents contenus. La démarche ici sera de rassembler différents [exemples](https://github.com/mdn/webextensions-examples) fournit par Mozilla afin de toucher à différents composants de cette API.

# Choix

En regardant un petit peu par curiosité comment était fait un plugin navigateur, plusieurs exemples fournit par Mozilla ont retenu mon attention. Dans un premier temps :
- [QuickNote](https://github.com/mdn/webextensions-examples/tree/master/quicknote) qui permet de prendre des notes dans une PopUp accessible dans le coin du navigateur.
- [Annotate Page](https://github.com/mdn/webextensions-examples/tree/master/annotate-page) qui permet de prendre des notes persistantes sur une page donnée par l'intermédiaire d'une SideBar.

Je me suis dit que je voulait quelque chose de plus accessible que QuickNote mais qui permette aussi d'avoir plusieurs notes. Enfin un autre plugin a attiré mon attention :
- [PopUpReact](https://github.com/mdn/webextensions-examples/tree/master/react-es6-popup) qui est une simple popup affichant l'URL courante.

Quitte à apprendre, utilisons aussi un framework JS (ici React) histoire d'apprendre quelques petites choses en plus. Un court [tutoriel](https://hackernoon.com/create-a-simple-todo-app-in-react-9bd29054566b) et on est parti.

# Développement
## Reproduction du tutoriel
### Reproduction

Ici tout est assez simple. Il y a la présences de trois composants principaux :
- App.js qui est l'application principale et contient la logique,
- TodoList.js qui gère le formulaire de création,
- TodoItems.js qui gère l'affichage des items une fois ceux-ci créés.

Seule la manière de faire référence à un Item par l'intermédiaire de change:
```js
//est déprécié
ref={this.props.inputElement}
//remplacé par
ref={(input)=>{this.inputElement = input;}}

```
Et voilà une application dans laquelle on peut créer des Todos et les supprimer en cliquant dessus.
### Adaptation

On modifie l'affichage des éléments saisis dans des *div* plutôt que dans une liste *ul*, le champ d'input devient une *textarea* pour permettre des saisies plus longues.

On rajoute un bouton dans le formulaire de saisie pour permettre de supprimer la note :
```html
<form onSubmit={this.props.addItem}>
  <textarea
    placeholder="Note"
    ref={(input)=>{this.inputElement = input;}}
    value={this.props.currentItem.text}
    onChange={this.props.handleInput}
  />
<div>
  <button class="btn" type="submit"> Ajouter note </button>
</div>
</form>
<form onSubmit={this.props.deleteItem}>
  <button class="btn" type="submit"> Supprimer note </button>
</form>
```

On modifie le comportement lors d'un clic pour permettre de mettre à jour la note plutôt que de la supprimer :
```js
updateItem = key => {
  const filteredItems = this.state.items.filter(item => {
    return item.key !== key
  })
  const newItem = this.state.items.filter(item => {
    return item.key == key
  })[0]
  if (newItem.text !== '') {
    const items = filteredItems
    this.setState({
      items: items,
      currentItem: { text: newItem.text, key: newItem.key },
    })
  }
}
```

On va maintenant ajouter le support du markdown. Pour ce faire, on va utiliser [showdown](https://github.com/showdownjs/showdown) qui est une librairie permettant la conversion de markdown en HTML.

Voici à cet effet la classe complète *TodoItems.js* :
```js
import React, { Component } from 'react'
import Converter from 'showdown'
import renderHTML from 'react-render-html';

class TodoItems extends Component {
  constructor(props){
    super();
    var showdown  = require('showdown');
    this.converter = new showdown.Converter();
  }
  createTasks = item => {
      return (
        <div key={item.key} onClick={() => this.props.updateItem(item.key)}>
          {renderHTML(this.converter.makeHtml(item.text))}
        </div>
      )
    }
  render() {
    const todoEntries = this.props.entries
    const listItems = todoEntries.map(this.createTasks)

    return <ul className="theList">{listItems}</ul>
  }
}

export default TodoItems
```
> On instancie un *Converter*.
>
> On convertit le markdown en entrée et on l'affiche dans le composant.

On cherche maintenant à permettre le stockage local.

## Ajout du support du stockage local

Voici la fonction `addItem` remise au goût du jour, ainsi que le constructeur modifié de *App.js*
```js
addItem = e => {
      e.preventDefault()
      const newItem = this.state.currentItem
      if (newItem.text !== '') {
        const items = [...this.state.items, newItem]
        this.setState({
          items: items,
          currentItem: { text: '', key: '' },
        })
      }
      var gettingItem = browser.storage.local.get(newItem.key);
      gettingItem.then((result) => {

        var objTest = Object.keys(result);
        if(objTest.length < 1 && newItem.text !== '') {
          var storingNote = browser.storage.local.set({ [newItem.key] : newItem.text });
        }
      });
    }



    constructor() {
      super()
      this.state = {
        items: [],
        currentItem: {text:'', key:''},
      }
      var storedItems = []
      var gettingAllStorageItems = browser.storage.local.get();
      gettingAllStorageItems.then((results) => {

        var noteKeys = Object.keys(results);
        for (let noteKey of noteKeys) {
          var curValue = results[noteKey];
          storedItems = [...storedItems, {key:noteKey.toString(),text:curValue.toString()}];
          this.setState({items: storedItems,currentItem: {text:'', key:''}});
        }
      });

```

> La fonction `browser.storage.local.get` permet de récupérer les différents items stockés sur le disque au démarrage de l'application, puis de les stocker dans la variables d'état de l'application.
> La fonction `addItem` est modifiée pour enregistrer les changements dans le stockage local.
> Il en est de même pour les fonctions `updateItem` et `deleteItem`.


# Débogage et installation persistante

## Débogage
Dans l'onglet "about:debugging" de Firefox, il est possible d'importer temporairement un plugin et de le déboguer avec les outils intégrés.

Sinon une bonne façon de faire est de lancer les commandes suivantes dans deux terminaux distincts :
```bash
npm run build
npm start
```

Il est ensuite possible d'avoir les mêmes outils de débogage dans le navigateur que pour une application web "normale". Pour cela il suffit d'aller dans :
  - Outil de développement > Boite à outil du navigateur (Ctrl+Maj+Alt+I)

## Installation persistante

Il faut d'abord créer un fichier `.zip` des sources au bon format. Pour se faire suivre la démarche décrite [ici](https://developer.mozilla.org/fr/docs/Mozilla/Add-ons/WebExtensions/Publishing_your_WebExtension). Cette solution utilise `web-ext` qui va packager sous le bon format vos sources en se basant sur le manifest. La sortie est un zip avec de ce type : `mon-nom-du-manifest.zip`.

Se rendre sur le site de [Mozilla](https://addons.mozilla.org/en-US/developers/addon/submit/distribution), choisir une installation "On my own". Sélectionner votre `.zip`, la structure va être analysée, vous pourrez éventuellement corriger les warnings signalés. Cliquez sur "Sign add-on". Puis vous pourrez télécharger un `.xpi` qui est une archive construite comme il faut pour une extension. Téléchargez là, Firefox vous propose de l'installer, et vous êtes prêt!



# Remarques
## Sécurité
Showdown présente un [article](https://github.com/showdownjs/showdown/wiki/Markdown%27s-XSS-Vulnerability-(and-how-to-mitigate-it)) intéressant sur la gestion des failles XSS du à la nature du markdown et préconise d'utiliser une librairie de filtrage du HTML de sortie.

Récemment un code malicieux a été découvert dans une dépendance assez utilisée : `event-stream`. Il faut donc rester méfiant vis à vis de trop nombreuses dépendances dans un projet. Ici le [lien](https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident).

## Améliorations possibles

- [ ] Utilisation de [XSS](https://www.npmjs.com/package/xss) pour *"nettoyer"* le HTML généré
- [x] Avoir un bouton permettant la mise à jour pour pouvoir copier le texte du HTML sans avoir à éditer la note.
- [ ] Permettre de générer un fichier HTML à partir des notes.
- [ ] Pouvoir filtrer les notes, peut-être par nom de domaine ou par date.

# Ressources

Plugin Wappalyzer : [https://www.wappalyzer.com/](https://www.wappalyzer.com/)

Plugin Shodan.io : [https://addons.mozilla.org/en-US/firefox/addon/shodan_io/](https://addons.mozilla.org/en-US/firefox/addon/shodan_io/)

Documentation de l'API WebExtension : [https://developer.mozilla.org/fr/docs/Mozilla/Add-ons/WebExtensions](https://developer.mozilla.org/fr/docs/Mozilla/Add-ons/WebExtensions)

Exemples d'addons : [https://github.com/mdn/webextensions-examples](https://github.com/mdn/webextensions-examples)

Informations sur la publication avec WebExt : [https://developer.mozilla.org/fr/docs/Mozilla/Add-ons/WebExtensions/Publishing_your_WebExtension](https://developer.mozilla.org/fr/docs/Mozilla/Add-ons/WebExtensions/Publishing_your_WebExtension)

Publication des add-ons : [https://addons.mozilla.org/en-US/developers/addon/submit/distribution](https://addons.mozilla.org/en-US/developers/addon/submit/distribution)

Tutoriel React : [https://hackernoon.com/create-a-simple-todo-app-in-react-9bd29054566b](https://hackernoon.com/create-a-simple-todo-app-in-react-9bd29054566b)

Showdown : [https://github.com/showdownjs/showdown](https://github.com/showdownjs/showdown)

Showdown XSS : [https://github.com/showdownjs/showdown/wiki/Markdown%27s-XSS-Vulnerability-(and-how-to-mitigate-it)](https://github.com/showdownjs/showdown/wiki/Markdown%27s-XSS-Vulnerability-(and-how-to-mitigate-it))
