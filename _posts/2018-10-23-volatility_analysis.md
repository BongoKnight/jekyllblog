---
title: Annalyse avec Volatility
layout: post
description: "Retrouver diverses informations avec Volatility..."
image: "/jekyllblog/img/security.png"
category: 'ticket'
tags:
- security
- Forensics

twitter_text: "Analyses d'un dump de RAM avec Volatility"
introduction: "Essayons de retrouver des mots de passe issus d'un password manager : PasswdSafe"
---

# Contexte

Nous utilisons ici une VM Xubuntu 17.04 avec la version 1.06 BETA de PasswordSafe : https://sourceforge.net/projects/passwordsafe/files/Linux-BETA/

L'idée est d'arriver à récupérer un maximum d'information à partir du dump de RAM de la VM, peut être les mot de passe, et sinon des informations contenues dans le navigateur. En particulier, celles contenues dans la page d’accueil de Gmail.

## Installation de Password PasswdSafe

Un package .deb est disponible à l'adresse mentionnée ci-dessus.

## Dump de RAM à partir de VBox

```bash
VBoxManage debugvm MyVm dumpvmcore --filename=MyVM.elf

readelf --program-headers MyVM.elf

Type de fichier ELF est CORE (fichier core)
Point d'entrée 0x0
Il y a 11 en-têtes de programme, débutant à l'adresse de décalage 64

En-têtes de programme :
  Type           Décalage           Adr.virt           Adr.phys.
                 Taille fichier     Taille mémoire      Fanion Alignement
  NOTE           0x00000000000002a8 0x0000000000000000 0x0000000000000000
                 0x0000000000002280 0x0000000000002280  R      0
  LOAD           0x0000000000002528 0x0000000000000000 0x0000000000000000
                 0x0000000080000000 0x0000000080000000  R      0
  LOAD           0x0000000080002528 0x0000000000000000 0x00000000e0000000
                 0x0000000001000000 0x0000000001000000  R      0
  LOAD           0x0000000081002528 0x0000000000000000 0x00000000f0000000
                 0x0000000000000000 0x0000000000020000  R      0
  LOAD           0x0000000081002528 0x0000000000000000 0x00000000f0400000
                 0x0000000000400000 0x0000000000400000  R      0
  LOAD           0x0000000081402528 0x0000000000000000 0x00000000f0800000
                 0x0000000000004000 0x0000000000004000  R      0
  LOAD           0x0000000081406528 0x0000000000000000 0x00000000f0804000
                 0x0000000000000000 0x0000000000001000  R      0
  LOAD           0x0000000081406528 0x0000000000000000 0x00000000f0806000
                 0x0000000000000000 0x0000000000002000  R      0
  LOAD           0x0000000081406528 0x0000000000000000 0x00000000fec00000
                 0x0000000000000000 0x0000000000001000  R      0
  LOAD           0x0000000081406528 0x0000000000000000 0x00000000fee00000
                 0x0000000000000000 0x0000000000001000  R      0
  LOAD           0x0000000081406528 0x0000000000000000 0x00000000ffff0000
                 0x0000000000010000 0x0000000000010000  R      0

# Le premier load nous intéresse :
# 0x0000000000002528 est le début du dump
# 0x0000000080000000 est la taille du dum

size=0x80000000;off=0x2528;head -c $(($size+$off)) test.elf|tail -c +$(($off+1)) > MyVM.raw

```
## Installation du profil Linux

Récupération du bon profil sur GitHub :

https://github.com/volatilityfoundation/profiles/blob/master/Linux/Ubuntu/x64/Ubuntu1604.zip
L'installation se fait en déposant le .zip, sous le chemin volatility/plugins/overlays/linux/Ubuntu1604.zip .

```bash
# Vérification de l'installation du profil
volatility --info | grep Ubuntu

# Lister les commandes disponibles
volatility -f MyVM.raw --profile=LinuxUbuntu1604x64 --help  --cache

```

Cependant ici le profil Linux n'est pas disponible, il faut alors créer son propre profil en suivant la procédure décrite [ici](https://github.com/volatilityfoundation/volatility/wiki/Linux) .

# Analyse

## Récupération des processus :


## Ressources

https://medium.com/@m4lv0id/and-i-did-oscp-589babbfea19
