---
title: Crowie, un honeypot SSH
layout: post
description: Présentation d'un honeypot
image: "/jekyllblog/img/cowrie/stats_heading.png"
category: 'Jekyll'
tags:
- SSH
- security
twitter_text: Installation et configuration d'un honeypot SSH.
introduction: Dans cet article nous allons voir ce qu'est un honeypot, comment installer Cowrie, le configurer et générer un rapport automatique.
---

# Un honeypot, c'est quoi ? Et pourquoi faire ?

## Contexte

Il y a peu j'ai décider d'acquérir un VPS, je me disais que c'était pour moi une occasion de bricoler un peu et de me pencher sur des problématique d'installation, de configuration, de protection etc. Me voilà donc "propriétaire" d'un serveur, exposé au yeux de tous. La première chose à faire est donc de le protéger, de le sécuriser un peu. J'ai donc créer un utilisateur non *root*, empêché la connexion à partir de *root*, choisi un mot de passe solide et unique. Et quelques autres choses. Je décidais ensuite de configurer *fail2ban* pour bloquer les IPs faisant de trop nombreuses tentatives de connexions.
J'ai tout d'abord bloqué les IPs à partir de 5 tentatives de connexions étonnées en 15 minutes. Plusieurs constats :

- Des IPs se sont faites bloquer en boucle toutes les 15 minutes,
- Plus d'une centaine de blocage quotidien
- En observant les logs de tentatives de connexions avant l'installation de *fail2ban* j'ai vu des IPs ayant fait plusieurs milliers de tentatives de connexion.

Bref internet est un monde hostile. Je voulais en savoir plus. Et pour comprendre les attaques, leur provenance et comprendre leur but quoi de mieux que de se faire attaquer.

## Honeypot

Un honneypot ou pot de miel en français, est un dispositif qui sert à attirer un attaquant et à lui faire croire qu'il s'introduit dans un dispositif non contrôlé alors qu'en réalité ses actions sont conservées et analysées. Ce leurre permet de saisir l'étendue d'une menace, de comprendre le comportement d'un attaquant. Ce dispositif peut simuler différents applicatifs, d'un site web sous wordpress à un serveur Elastic ou des bases de données plus conventionnelles ou encore des équipement physique comme des boxs internets ou des routeurs d'entreprises. Par exemple, le compte Twitter [Bad Packets Reports](https://twitter.com/bad_packets) se sert de nombreux honeypots pour alerter sur les campagnes malveillantes en cours.

> Our honeypots recently observed an interesting scan targeting Orange Livebox ADSL modems. A flaw exists in these modems that allow remote unauthenticated users to obtain the device’s SSID and WiFi password.
>
> Bad Packets le 28/12/2018 : https://twitter.com/bad_packets/status/1076797149524811777

Par ailleurs une liste de différents honeypot est disponible sur ce repository : [Awesome Honeypot](https://github.com/paralax/awesome-honeypots). Vous pourrez y voir la diversité de ce qui est fait!

## Cowrie

[Cowrie](https://github.com/cowrie/cowrie) est un projet disponible sur Github qui est actif. C'est un honeypot qui mime le comportement d'une session SSH ou telnet.

Le principe est le suivant : on peut définir des couples d'identifiants mots de passe interdits ou tolérés, lors d'une tentative de connexion, Cowrie laissera par conséquent l'utilisateur interagir avec ce qui semble être un *shell*. En réalité, Cowrie se contente de mimer un certain nombre de commandes : *wget*,*uname*,*whoami*,*ls*... L'attaquant aura alors l'impression d'être sur une vraie machine et tentera peut être d'effectuer des informations qui nous intéressent. Dans le pire des cas, celui où l'attaquant se rend compte de la supercherie, on aura tout de même récupéré un couple identifiant/mot de passe courant.

 L'installation est plutôt facile, pour ma part je me suis basé sur cet [article](https://null-byte.wonderhowto.com/how-to/use-cowrie-ssh-honeypot-catch-attackers-your-network-0181600/) plutôt bien fait et assez clair. Cependant je vais en résumer les différentes étapes.

# Installation

```bash
# Mise à jour
sudo apt-get update && sudo apt-get upgrade
# Installation de dépendances nécessaires
sudo apt-get install git python-virtualenv libssl-dev libffi-dev build-essential libpython-dev python2.7-minimal authbind
# Modification de la configuration ssh
sudo vim /etc/ssh/sshd_config
sudo systemctl restart ssh
# Installation de cowrie
sudo adduser --disabled-password cowrie
sudo su - cowrie
git clone https://github.com/micheloosterhof/cowrie
cd cowrie
virtualenv cowrie-env
source cowrie-env/bin/activate
pip install --upgrade pip
pip install --upgrade -r requirements.txt
```
Dans la deuxième étapes, *python 2.7* est nécessaire car Cowrie tourne dans cette version. *python-virtualenv* est nécessaire pour simplifier la gestion des dépendances et limiter le risque de conflit.

La troisième étape consiste à changer la configuration de base de SSH afin de laisser le port 22 inoccupé pour Cowrie. Il faut donc changer la ligne **PORT 22** par **PORT xxxx**. Ne pas oublier de redémarrer le service ensuite.

> Remarque : Le simple fait de changer le port par défaut a, pour ma part, fait drastiquement changer la quantité de bannissements faite par *fail2ban*, je suis actuellement à moins d'une dizaine par jour. Comme quoi ne pas faire tourner ses services sur les ports "classiques" ça a du bon.

La dernière étape consiste à créer un utilisateur non sudoer sans mot de passe qui va lancer Cowrie. Ensuite, on crée un environnement virtuel et on installe les différentes dépendances nécessaires via *pip*.

# Configuration et démarrage
La configuration passe par différents fichiers :

- *etc/cowrie.cfg* : on y change le *hostname* pour que ce ne soit pas trop évident que ce soit un honeypot, et le port par défaut pour le mettre sur le port 22.
- *etc/userdb.txt* dans lequel on peut préciser identifiants acceptés ou non.

Pour démarrer Cowrie, à partir du dossier source :
```bash
bin/cowrie start
bin/cowrie stop
```

# Monitoring

Je souhaitais mettre en place un rapport quotidien sur les tentatives de connexion de la journée. Cowrie dans le répertoire *var/log/cowrie* stocke des logs au format JSON. J'ai décidé d'en faire la matière première du reporting.

```JSON
{"eventid": "cowrie.login.failed", "username": "root", "timestamp": "2019-01-29T01:59:00.786441Z", "message": "login attempt [root/root] failed", "src_ip": "46.191.196.158", "session": "9aa1d86a2b2b", "password": "root", "sensor": "monsuperVPS"}
{"eventid": "cowrie.login.success", "username": "root", "timestamp": "2019-01-29T01:59:01.869901Z", "message": "login attempt [root/admin] succeeded", "src_ip": "46.191.196.158", "session": "9aa1d86a2b2b", "password": "admin", "sensor": "monsuperVPS"}
{"eventid": "cowrie.session.params", "timestamp": "2019-01-29T01:59:02.509301Z", "sensor": "monsuperVPS", "src_ip": "46.191.196.158", "session": "9aa1d86a2b2b", "arch": "linux-x64-lsb", "message": []}
{"eventid": "cowrie.command.input", "timestamp": "2019-01-29T01:59:02.510292Z", "message": "CMD: /ip cloud print", "src_ip": "46.191.196.158", "session": "9aa1d86a2b2b", "input": "/ip cloud print", "sensor": "monsuperVPS"}
{"eventid": "cowrie.command.failed", "timestamp": "2019-01-29T01:59:02.511336Z", "message": "Command not found: /ip cloud print", "src_ip": "46.191.196.158", "session": "9aa1d86a2b2b", "input": "/ip cloud print", "sensor": "monsuperVPS"}
{"eventid": "cowrie.log.closed", "shasum": "b846225e0081fa9151eb29ac62be1dea60bb9c567dba6c3ca3b1c6169b6d750d", "timestamp": "2019-01-29T01:59:12.588415Z", "message": "Closing TTY Log: var/lib/cowrie/tty/b846225e0081fa9151eb29ac62be1dea60bb9c567dba6c3ca3b1c6169b6d750d after 10 seconds", "ttylog": "var/lib/cowrie/tty/b846225e0081fa9151eb29ac62be1dea60bb9c567dba6c3ca3b1c6169b6d750d", "src_ip": "46.191.196.158", "duration": 10.079437971115112, "duplicate": true, "session": "9aa1d86a2b2b", "sensor": "monsuperVPS", "size": 30}
{"eventid": "cowrie.session.params", "timestamp": "2019-01-29T01:59:13.378634Z", "sensor": "monsuperVPS", "src_ip": "46.191.196.158", "session": "9aa1d86a2b2b", "arch": "linux-x64-lsb", "message": []}
{"eventid": "cowrie.command.input", "timestamp": "2019-01-29T01:59:13.379810Z", "message": "CMD: ifconfig", "src_ip": "46.191.196.158", "session": "9aa1d86a2b2b", "input": "ifconfig", "sensor": "monsuperVPS"}
{"eventid": "cowrie.log.closed", "shasum": "1d6f385dd0e7ccc3ada3e24e973fd850470dbb222547ea0c1cb7c9f6d9e1dc5e", "timestamp": "2019-01-29T01:59:13.463266Z", "message": "Closing TTY Log: var/lib/cowrie/tty/1d6f385dd0e7ccc3ada3e24e973fd850470dbb222547ea0c1cb7c9f6d9e1dc5e after 0 seconds", "ttylog": "var/lib/cowrie/tty/1d6f385dd0e7ccc3ada3e24e973fd850470dbb222547ea0c1cb7c9f6d9e1dc5e", "src_ip": "46.191.196.158", "duration": 0.08481001853942871, "duplicate": true, "session": "9aa1d86a2b2b", "sensor": "monsuperVPS", "size": 902}
```

## Écriture d'un script en python

J'ai alors écrit un petit script python qui est en charge de lire les logs, de les enregistrer dans une base SQLite puis de générer un rapport.
La base contient les données des trois derniers mois. J'ai retenu un certain nombre de métriques calculées sur les derniers moi et sur les données du jour :

- l'origine géographique des connexions (ajoutée grâce au paquet maxmind-geolite),
- les mots de passe les plus utilisés,
- l’enchaînement de commande le plus long, pour savoir ce que cherche à faire un attaquant et éventuellement pouvoir ajouter au faux système de fichier des fichiers supplémentaires,
- les couples identifiant / mot de passe les plus en vogue,
- les commandes les plus utilisées

Ces métriques sont enregistrées sous forme de graphique ou de tableaux :
```python
# Un exemple de graphique

def pie_graph(sql, alternate_text, savefile, limit=5, db='stats.db'):
    conn = sqlite3.connect(db)
    c = conn.cursor()
    c.execute(sql)
    nbs = []
    datas = []
    for row in c:
        nbs+=[row[1]]
        datas+=[row[0]]
    nbsClean = nbs
    datasClean = datas
    if len(nbs) > limit :
        nbsClean = nbs[0:limit] + [sum(nbs[limit:])]
        datasClean = [datas[i].replace("$","\\$") for i in range(0,limit)] + ["Autres (" + str(len(nbs[limit:])) + " " + alternate_text + ")" ]
    fig1, ax1 = plt.subplots()
    ax1.pie(nbsClean, labels=datasClean, autopct='%1.1f%%',
            shadow=True, startangle=90)
    ax1.axis('equal')  # Equal aspect ratio ensures that pie is drawn as a circle.

    fig1.savefig(savefile)
    conn.close()


# Et un exemple de tableau
def couple_used(db='stats.db'):
    sql= "SELECT username, password, count(1) as nb from (SELECT username, password from failed union all select username, password from success) group by username, password order by nb desc LIMIT 10"
    conn = sqlite3.connect(db)
    c = conn.cursor()
    c.execute(sql)
    text = "\n\nLes couples \"utilisateurs / mots de passes\" les plus utilisées sont :\n\n| Utilisateurs | Mot de passe | Occurences |\n|-------|--------|------|"
    for row in c:
        text+='\n|' + str(row[0]) + "|" +  str(row[1]) + "|" + str(row[2]) + "|"

    text+='\n```\n\n'
    conn.close()
    return text

```


## Envoi du rapport

Le script python écrit un rapport au format markdown. Il faut ensuite avant de l'envoyer s'occuper de le convertir dans un format plus joli. Le convertisseur que j'utilise couramment s'appelle [*pandoc*](https://pandoc.org/) et permet de générer un rapport sous différents formats : PDF, HTML, DOCX, slides... Des convertisseurs plus légers existent mais celui ci est très flexible et en modifiant peu le markdown écrit par le script, je peux adapter le document de sortie facilement.

> [Pandoc](https://pandoc.org/) peut paraître un peu gadget mais une fois maîtrisé, il est très pratique pour prendre des notes de cours ou même écrire des rapport sans s'embêter, des notes rapides et à la fin une belle mise en page qui donne envie de les relire, le tout sans efforts.

Pour envoyer le mail, une ligne de commande suffit :
```bash
sudo apt-get install sendemail
sendemail -f adresse@gmail.com -t adresse@gmail.com -u "Honeypot SSH" -m "SumUp Honeypot, here the datas of the day! :)" -a /path/to/rapport.pdf
```

## Automatisation

Pour automatiser le traitement, un petit script bash, lançant le script python puis envoyant le mail suffit. On y fait ensuite appel depuis le planificateur de tâches cron de l'utilisateur *cowrie*.

```bash
sudo su - cowrie
crontab -e
```

Il faut alors rajouter la ligne `59 23 * * * /home/cowrie/crowieStats/report.sh` au fichier.
Ainsi tous les soirs à 11h59 (juste avant l'historisation des logs), les données seront chargées en base et analysées!


# Conclusion

J’espère que cet article a pu clarifié quelques petites choses et a pu donner des idées ou des pistes de réflexions. Voici un [lien](https://github.com/BongoKnight/crowieStats) vers le repository avec le code source, à l'avenir peut être que le rapport sera complété avec d'autres métriques ou que je ferais une configuration pour permettre de générer les métriques voulues. L'idée ici est surtout de tester un honeypot et d'avoir un retour sur les données récoltées. Et ici un lien vers un [rapport](/jekyllblog/img/cowrie/rapport.pdf) généré.
