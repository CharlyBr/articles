# Créer un paquet .deb à partir de binaires

Dans le cadre de la gestion de nodes [IPFS](https://ipfs.io/), j’ai été confronté à un problème de disponibilité des paquets ipfs pour debian.

IPFS est écrit en go et propose une version binaire selon l’architecture désirée dans les [releases sur github](https://github.com/ipfs/kubo/releases).

Note au passage, le projet s’appelle désormais `kubo` depuis la dernière release, il s’appelait avant `go-ipfs`.

Donc au lieu de télécharger les releases pour les installer, je me suis dit qu’il serait intéressant de faire un paquet debian pour le déployer plus facilement sur les machines. En outre cela permet de gérer des dépendances si besoin et de gérer des actions en pré ou post install.

## Structure d'un paquet

Un paquet est une archive ar contenant les fichiers à installer. Cette archive contient un fichier important, le fichier de contrôle, qui contient les meta-données du paquet.

L’archive est organisée avec la structure de fichiers telle qu’elle va être installée sur le système. Par exemple, si on install un binaire dans `/usr/local/bin/<binaire>` il sera stocké sous cette forme dans le paquet.

Les noms de paquet suivent la convention suivante :

```
<name>_<version>-<revision>_<architecture>.deb
name : nom de l’application
version : la version installée
revision : la version du paquet lui-même
architecture : l’architecture cible comme amd64 ou arm64
```

Dans notre cas, pour un paquet à destination d’une architecture amd64, le paquet sera nommé :

`kubo_0.14.0–1_amd64.deb`

## Création du paquet

On commence d’abord par créer le répertoire qui va contenir l’arborescence du paquet :

```
$ mkdir kubo_0.14.0-1_amd64
```

On crée l’arborescence désirée. Dans notre cas, kubo est distribué sous forme d’un seul binaire, que nous allons placer dans /usr/local/bin

```
$ mkdir -p kubo_0.14.0-1_amd64/usr/local/bin
```

On récupère la release de kubo pour placer le binaire dans le paquet

```
$ curl -L -O https://github.com/ipfs/kubo/releases/download/v0.14.0/kubo_v0.14.0_linux-amd64.tar.gz
$ tar -xzf kubo_v0.14.0_linux-amd64.tar.gz
$ cp kubo/ipfs kubo_0.14.0-1_amd64/usr/local/bin
```
Le binaire de kubo s’appelle toujours ipfs

## Création du fichier de contrôle

```
$ cat kubo_0.14.0-1_amd64/DEBIAN/control
Package: kubo
Version: 0.14.0-1
Architecture: amd64
Maintainer: You <hello@domain.com>
Description: binary release from kubo packaged in .deb
 This a simple package that contains the official binary release of kubo.
```

Attention à respecter la casse du répertoire DEBIAN Il faut bien que ce soit en majuscules pour les paquets binaires. On peut le trouver en minuscule pour les paquets sources.

Il existe d’autres meta-données que l’on peut indiquer dans le fichier de contrôle, vous trouverez plus d’informations dans la [documentation debian](https://www.debian.org/doc/debian-policy/ch-binary.html).

## Construction du paquet

Dernière étape maintenant, il s’agit de créer le fichier `.deb` grâce à la commande `dpkg-deb` :

```
$ dpkg-deb --build --root-owner-group -Zgzip kubo_0.14.0-1_amd64
dpkg-deb: building package 'kubo' in 'kubo_0.14.0-1_amd64.deb'.
$ ls -l kubo_0.14.0-1_amd64.deb
-rw-r--r-- 1 root root 29034410 Aug 30 09:24 kubo_0.14.0-1_amd64.deb
```

Le flag `--root-owner-group` indique que les fichiers contenus dans le paquet seront installés sous l’identité de l’utilisateur root.

Le flag `-Zgzip` permet de compresser l’archive au format gzip plutôt que xz. 
J’ai rencontré des problèmes de compatibilités de format de compression lors de la création de repository perso, depuis j’utilise gzip.

Pour consulter le contenu d’un paquet :

```
$ dpkg-deb --contents kubo_0.14.0-1_amd64.deb
drwxr-xr-x root/root         0 2022-08-26 10:07 ./
drwxr-xr-x root/root         0 2022-08-26 10:07 ./usr/
drwxr-xr-x root/root         0 2022-08-26 10:07 ./usr/local/
drwxr-xr-x root/root         0 2022-08-26 10:15 ./usr/local/bin/
-rwxr-xr-x root/root  78390320 2022-08-26 10:15 ./usr/local/bin/ipfs
```

Voilà votre paquet est prêt !

## Installation du paquet

Vous pouvez maintenant installer le paquet sur la machine cible :

```
$ dpkg -i kubo_0.14.0-1_amd64.deb
```

Il apparaît maintenant dans la liste des paquets installés sur le système :

```
$ dpkg -l kubo
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name           Version      Architecture Description
+++-==============-============-============-=========================================
ii  kubo           0.14.0-1     amd64        binary release from kubo packaged in .deb
```

Le supprimer :

```
$ dpkg -r kubo_0.14.0-1_amd64.deb
```

Liens pour aller plus loin :
* [Debian policy manual](https://www.debian.org/doc/debian-policy/index.html)
* [Kubo repository](https://github.com/ipfs/kubo)
* [Les bases du système de gestion des paquets Debian](https://www.debian.org/doc/manuals/debian-faq/pkg-basics.fr.html)


