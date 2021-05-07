# Introduction Système & Infrastructure

Thème abordé : Virtualisation, Système Unix, Services d'infrastructure

## 1. Découverte de la virtualisation
**Objectif :** Créer un environnement virtuel fonctionnel

Prérequis :
* Activer **Intel**(R) **Virtualization** Technology dans le BIOS
* VMware Workstation Pro 16
* L'export de la VM [**master_centos7**](https://auvencecom-my.sharepoint.com/:u:/g/personal/jonathan_dinh_ynov_com/ESiYWqb8WvFKjgau67fTEaQBq5m9nm_cwyy8szy_1d8y3Q?e=uIttkS)

Facultatif :
* [MobaXterm](https://mobaxterm.mobatek.net/)

### a. Configuration des réseaux virtuels
Avant de commencer la partie système, nous allons mettre en place le coté réseau virtuel pour pouvoir intégrer au futur machine virtuelle.

Il existe 3 types de réseau virtuel :
- le réseau **Host-only**,  dit **réseau privé hôte** en français, est un réseau privé isolé dans lequel l'hôte et les machines virtuelles a l'intérieur pourront communiquer entre eux.
- le réseau **NAT**, pour **Network Address Translation**, est un réseau privé pouvant accéder à internet. En effet, ce réseau est virtuel mais dispose d'une patte dans le réseau physique puisqu'il permet d'aller sur Internet.
- le réseau **Bridge**, dit **réseau ponté** en français, nous permet de se connecter au réseau physique à travers un pont virtuel vers la carte physique de l'hôte. 

Nous allons utiliser le réseau NAT tout au long de cette journée. Cela va nous permettre à nos machines d'accéder à internet pour récupérer les différents paquets nécessaires.

Sur VMware Workstation > **Edit** > **Virtual Network Editor**, récupérer les informations suivantes :
- l'adresse réseau
- le masque sous-réseau
- la passerelle du réseau

### b. Importation de la machine virtuel

Nous allons maintenant importer la VM sur VMware Workstation Pro. 
Quelques précisions sur VMware workstation Pro, c'est un hyperviseur de niveau 2.

Hyperviseur de niveau 1 vs Hyperviseur de niveau 2
![Hyperviseur niveau 1 vs Hyperviseur niveau 2](https://user.oc-static.com/upload/2019/06/14/15605121904745_fzfzefezfz.png)

Sur VMware Workstation > **Home** > **Open a Virtual Machine**, sélectionner le fichier en `.ova`

Une fois importer, faites un clone de celui-ci, il s'agit de faire de celui-ci un template sur lequel s'appuyer.

Une fois le clone créé, vérifier que l'adaptateur réseau soit bien sur le `VMnet8` puis générer une nouvelle adresse MAC.

## 2. Systèmes Unix
**Objectif :** Manipuler un système Unix

### a. Configuration de la machine virtuelle

Nous allons maintenant attribué une adresse ip fixe dans le réseau NAT à notre machine fraichement créé.

Voici le fichier de configuration votre interface réseau `/etc/sysconfig/network-scripts/ifcfg-ens33`

Dans ce fichier, renseignez les informations suivantes :
- [ ] Type d'interface
- [ ] Le type d'adresse ip
- [ ] le nom de l'interface
- [ ] l'interface que l'on configure
- [ ] L'adresse ip
- [ ] Masque sous-réseau
- [ ] Passerelle réseau
- [ ] DNS
- [ ] Démarrage de l'interface au boot

Quelques commandes utiles :

`vi <fichier>` permet d'ouvrir un fichier avec l'éditeur vi
  - `i` pour entrer en mode insertion (pour écrire)
  - `:q` pour quitter
  - `:q!` pour quitter sans sauvegarder
  - `:w` pour enregistrer
  - `:x` ou `:wq` pour enregistrer et quitter

`cat <fichier>` offre un aperçu du fichier

`ip a` liste les interfaces réseau

`ifup ens33` allume l'interface réseau `ens33`

`ifdown ens33` éteins l'interface réseau `ens33`

`yum` pour installer des paquets

> Si vous aimez pas `vi`, vous pouvez opter pour `nano` mais avant tout il vous faut internet :)

### b. Création d'un utilisateur

Créer votre utilisateur avec la commande `useradd`. Cet utilisateur doit :
- [ ] avoir un répertoire dans home
- [ ] être ajouté dans le groupe wheel
- [ ] le tout en une seul commande :)

Changer le mot de passe de votre utilisateur avec la commande `passwd`.

Quelques commandes utiles :

`man <commande>` pour ouvrir le manuel d'une commande

`cat /etc/passwd` voir tous les utilisateurs du système

`userdel -r <username>` pour supprimer un utilisateur et son home associé

`id <username>`pour voir les informations système de l'utilisateur

`ls -al ` pour lister un dossier avec des informations avancées (droits et propriétaire), ainsi que les fichiers cachés

### c. Création d'une paire de clé SSH et utilisation de SSH

Sur votre hôte, nous allons créer une paire de clé SSH. Elle doit contenir les caractéristiques suivantes :
- [ ] Type RSA
- [ ] d'une longueur de 4096 bits
- [ ] Sans passphrase

Cependant, il faut configurer le service sshd. Le fichier de configuration se trouve à `/etc/ssh/sshd_config` :
- [ ] Permettre la connexion ssh par clé
- [ ] (Facultatif) Change le port d'écoute de ssh

Recharger la configuration de sshd.

Copier votre clé publique vers le serveur avec la commande `ssh-copy-id`

Maintenant vous pouvez vous connecter depuis votre hôte à votre système unix en ssh.

Sécuriser le tout en :
- [ ] désactivant la connexion par mot de passe en ssh
- [ ] interdisant la connexion avec le compte root en ssh

## 3. Service d'infrastructure : le partage de fichier
**Objectif :** Monter un service de partage de fichier réseau

### a. Ajouter un disque de donnée (Falcultatif)

Éteindre la machine ajouter un nouveau disque virtuel.

Faire les actions suivantes :
- [ ] Partitionner vos disques avec la commande `fdisdk`
- [ ] Créer un volume physique (PV) avec le disque partitionner.
- [ ] Créer un groupe de volume (VG).
- [ ] Créer un volume logique (LV) avec 100% de la capacité disponible.
- [ ] Formater le volume logique au format xfs.
- [ ] Créer un dossier `/data`
- [ ] Monter le volume logique sur le dossier `/data`

Indice : `fdisk -l` permet de lister les disques du système

### b. Faire le partage NFS

Installer les paquets `nfs-utils` en vous aidant de la commande `yum` cela va vous installer un serveur NFS.

Configurer le fichier `/etc/idmapd.conf` en ajoutant `localhost.localdomain` au domaine

Configurer votre export nfs dans le fichier `/etc/exports`

Démarrez le service `nfs-server`et `rpcbind`

Cloner le template et faire les manipulations du chap. 2 (user + ssh) puis monter le partage nfs.

### c. Manipuler les droits Unix

https://doc.ubuntu-fr.org/permissions

Créer les fichiers suivants depuis votre serveur NFS :
- [ ] Droit de lecture que pour root
- [ ] Droit de lecture et écriture que pour root, lecture pour wheel
- [ ] Droit de lecture et écriture pour tous le monde,
- [ ] Droit de lecture et exécution pour tout le monde
- [ ] Tous les droits pour tous les mondes

Faite des tests en vous connecter avec votre utilisateur sur le serveur puis depuis le client nfs.

## 3bis. Service d'infrastructure : le DHCP
**Objectif :** Monter un service DHCP

Prérequis :
- Désactiver le service DHCP du réseau NAT de VMware

### a. Dynamic Host Configuration Protocol

**D**ynamic **H**ost **C**onfiguration **P**rotocol (**DHCP**, protocole de configuration dynamique des hôtes) est un protocole réseau dont le rôle est d’assurer la configuration automatique des paramètres IP d’une station ou d'une machine, notamment en lui attribuant **automatiquement** une adresse IP et un masque de sous-réseau. DHCP peut aussi configurer l’adresse de la passerelle par défaut et des serveurs de noms DNS.

### b. Installation et configuration du serveur DHCP

Installer les paquets `dhcp` en vous aidant de la commande `yum` cela va vous installer un serveur DHCP.

Configurer le serveur DHCP (`/etc/dhcp/dhcpd.conf`) en vous appuyant sur les critères donnés :

- [ ] Bail par défaut de 1h
- [ ] Bail maximal de 6h
- [ ] Plage d'adresse entre 25 et 50 clients
- [ ] doit définir la passerelle par défaut à fournir aux clients.
- [ ] doit définir le serveur DNS par défaut à fournir aux clients.

### c. Vérification DHCP

Clonez une seconde fois le master_centos7 et faites les mêmes manipulations que le 1.b. (adresse mac, etc, etc)

Cette fois au lieu de lui attribuer une adresse ip fixe, cette machine doit récupérer une adresse ip depuis le serveur DHCP.

Une fois l'adresse ip récupérer vous devez entre en mesure de ping votre serveur DHCP mais aussi de pouvoir aller sur internet.

## 4. Pour aller plus loin...
**Objectif :** Mettre en place d'autres services et sécuriser les flux entrant/sortant du serveur

Mettre en place des règles Firewall qui vont bien avec iptables.

Mettre en place un serveur DNS.
