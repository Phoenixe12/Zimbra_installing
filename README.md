# Guide d'installation de Zimbra Collaboration Suite


## 📋 Table des matières

- [Introduction](#introduction)
- [Prérequis](#prérequis)
- [Installation](#installation)
  - [Préparation du système](#préparation-du-système)
  - [Configuration DNS](#configuration-dns)
  - [Installation de Zimbra](#installation-de-zimbra)
  - [Configuration du pare-feu](#configuration-du-pare-feu)
- [Configuration](#configuration)
  - [DKIM et authentification des emails](#dkim-et-authentification-des-emails)
  - [Configuration multi-domaines](#configuration-multi-domaines)
- [Utilisation](#utilisation)
  - [Accès à l'interface d'administration](#accès-à-linterface-dadministration)
  - [Création d'utilisateurs](#création-dutilisateurs)
- [Dépannage](#dépannage)
- [Commandes utiles](#commandes-utiles)
- [Contribution](#contribution)
- [Licence](#licence)

## 🚀 Introduction

Ce projet fournit un guide complet pour installer et configurer Zimbra Collaboration Suite sur un serveur Ubuntu. Zimbra est une solution complète de messagerie et collaboration qui inclut un serveur de messagerie, un client webmail, un calendrier, et d'autres fonctionnalités collaboratives.

## ✅ Prérequis

- Serveur Ubuntu 20.04 LTS ou plus récent
- Minimum 4 Go de RAM (8 Go recommandé)
- Au moins 50 Go d'espace disque
- Un nom de domaine configuré
- Accès root au serveur

## 📥 Installation

### Préparation du système

Mettez à jour votre système et installez les dépendances nécessaires:

```bash
# Mise à jour du système
apt update && apt upgrade -y

# Installation des dépendances
apt install -y netcat-openbsd sudo libidn11 libpcre3 libgmp10 libexpat1 libstdc++6 libaio1 resolvconf unzip pax sysstat sqlite3

# Configuration du nom d'hôte
hostnamectl set-hostname mail.votredomaine.com

# Configuration du fichier hosts
echo "VOTRE_IP_SERVEUR mail.votredomaine.com mail" >> /etc/hosts
```

### Configuration DNS

Configurez ces enregistrements DNS pour votre domaine:

| Type  | Nom                  | Valeur                               |
|-------|----------------------|--------------------------------------|
| A     | mail                 | Adresse IP de votre serveur          |
| MX    | @                    | mail.votredomaine.com (priorité 10)  |
| TXT   | @                    | v=spf1 mx ~all                       |
| PTR   | Adresse IP (reverse) | mail.votredomaine.com                |

Après l'installation, vous ajouterez:
- Enregistrement DKIM (généré par Zimbra)
- Enregistrement DMARC: `_dmarc IN TXT "v=DMARC1; p=quarantine; rua=mailto:postmaster@votredomaine.com"`

### Configuration DNS du serveur

```bash
# Configurer les serveurs DNS externes
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "nameserver 8.8.4.4" >> /etc/resolv.conf
chattr +i /etc/resolv.conf
```

### Installation de Zimbra

```bash
# Téléchargement de Zimbra
mkdir -p /opt/zimbra-install
cd /opt/zimbra-install
wget https://files.zimbra.com/downloads/8.8.15_GA/zcs-8.8.15_GA_4179.UBUNTU20_64.20211118033954.tgz

# Extraction des fichiers
tar -xzvf zcs-8.8.15_GA_4179.UBUNTU20_64.20211118033954.tgz
cd zcs-*

# Lancement de l'installation
./install.sh
```

Pendant l'installation, suivez ces étapes:
1. Acceptez la licence (Y)
2. Confirmez l'installation des paquets (Y)
3. Entrez votre nom de domaine
4. Pour configurer le mot de passe admin:
   - Tapez "7" pour menu principal
   - Tapez "4" pour configurer les mots de passe
   - Définissez un mot de passe sécurisé
   - Tapez "r" pour retourner au menu principal
   - Tapez "a" pour appliquer les changements
   - Confirmez (Y)

### Configuration du pare-feu

```bash
# Installation d'UFW
apt install -y ufw

# Ports Zimbra
ufw allow 25/tcp   # SMTP
ufw allow 80/tcp   # HTTP
ufw allow 110/tcp  # POP3
ufw allow 143/tcp  # IMAP
ufw allow 443/tcp  # HTTPS
ufw allow 465/tcp  # SMTPS
ufw allow 587/tcp  # SUBMISSION
ufw allow 993/tcp  # IMAPS
ufw allow 995/tcp  # POP3S
ufw allow 7071/tcp # Admin Console
ufw allow 8443/tcp # Proxy
ufw allow 9071/tcp # Admin Console

# SSH
ufw allow 22/tcp

# Activer UFW
ufw enable
```

## ⚙️ Configuration

### DKIM et authentification des emails

```bash
# Générer les clés DKIM
su - zimbra
/opt/zimbra/libexec/zmdkimkeyutil -a -d votredomaine.com
```

Ajoutez l'enregistrement TXT généré à votre configuration DNS.

### Configuration multi-domaines

1. Accédez à l'interface d'administration (`https://mail.votredomaine.com:7071`)
2. Allez dans "Configuration" > "Domaines"
3. Cliquez sur "Nouveau"
4. Entrez le nom du domaine supplémentaire
5. Configurez les options selon vos besoins
6. N'oubliez pas de configurer les enregistrements DNS appropriés pour ce domaine

## 🖥️ Utilisation

### Accès à l'interface d'administration
- URL: `https://mail.votredomaine.com:7071`
- Utilisateur: `admin@votredomaine.com`
- Mot de passe: celui défini lors de l'installation

### Accès au webmail
- URL: `https://mail.votredomaine.com`
- Utilisateur: adresse email complète
- Mot de passe: celui de l'utilisateur

### Création d'utilisateurs
1. Accédez à l'interface d'administration
2. Allez dans "Gérer" > "Comptes"
3. Cliquez sur "Nouveau"
4. Remplissez les informations requises

## 🔧 Dépannage

Si vous rencontrez des problèmes d'envoi/réception d'emails:

```bash
# Vérifier la résolution DNS
nslookup gmail.com
nslookup votredomaine.com

# Si nécessaire, reconfigurer les serveurs DNS
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "nameserver 8.8.4.4" >> /etc/resolv.conf
```

## 📝 Commandes utiles

```bash
# Vérifier l'état de Zimbra
su - zimbra -c "zmcontrol status"

# Redémarrer Zimbra
su - zimbra -c "zmcontrol restart"

# Vérifier la file d'attente des emails
postqueue -p

# Vider la file d'attente
postsuper -d ALL

## Installation du SSL

Si vous rencontrez des problèmes avec le certificat SSL:


# Guide d'installation Let's Encrypt pour Zimbra

Ce guide vous aidera à configurer des certificats Let's Encrypt sur votre serveur Zimbra, avec renouvellement automatique.

## Prérequis

Ce guide suppose que vous utilisez Ubuntu 20 et que vous avez configuré un nom d'hôte et un DNS corrects.

### Vérification du nom d'hôte

Vérifiez que votre configuration de nom d'hôte est correcte en exécutant les commandes suivantes en tant qu'utilisateur `zimbra` :

```bash
zimbra@le-test:~$ source ~/bin/zmshutil; zmsetvars 
zimbra@le-test:~$ zmhostname 
le-test.zimbra.tech 
zimbra@le-test:~$ hostname --fqdn 
le-test.zimbra.tech
```

Les deux commandes doivent retourner des valeurs identiques.

### Configuration DNS CAA

Assurez-vous d'avoir configuré un enregistrement DNS CAA pour que Let's Encrypt puisse émettre des certificats pour votre domaine :

```bash
zimbra@le-test:~$ sudo apt install -y net-tools dnsutils 
zimbra@le-test:~$ dig +short type257 $(hostname --d) 
0 issuewild "letsencrypt.org" 
0 issue "letsencrypt.org"
```

Vérifiez que `0 issue "letsencrypt.org"` apparaît dans la sortie.

### Vérification du port 80

Vérifiez que Zimbra n'écoute pas sur le port 80. Let's Encrypt doit pouvoir exécuter un serveur web temporaire sur ce port :

```bash
netstat -tulpn | grep ":80 "
```

Cette commande ne devrait pas produire de résultat.

### Modification du mode proxy (si nécessaire)

Si Zimbra écoute sur le port 80, modifiez le mode proxy :

```bash
sudo su zimbra - 
zmprov ms `zmhostname` zimbraReverseProxyMailMode https 
zmprov ms `zmhostname` zimbraMailMode https 
/opt/zimbra/bin/zmtlsctl https 
/opt/zimbra/libexec/zmproxyconfig -e -w -o -a 8080:80:8443:443 -x https -H `zmhostname`
```

## Installation de Certbot

La version de Certbot dans les dépôts Ubuntu est généralement trop ancienne pour être utilisée avec Zimbra. Installez une version plus récente via Python venv :

```bash
apt install -y python3 python3-venv libaugeas0 
python3 -m venv /opt/certbot/ 
/opt/certbot/bin/pip install --upgrade pip 
/opt/certbot/bin/pip install certbot 
ln -s /opt/certbot/bin/certbot /usr/local/sbin/certbot 
/usr/local/sbin/certbot certonly -d $(hostname --fqdn) --standalone --preferred-chain "ISRG Root X2" --agree-tos --register-unsafely-without-email
```

## Note sur les certificats ECDSA

La prise en charge des certificats ECDSA TLS (cryptographie à courbe elliptique ECC) est disponible à partir des versions suivantes de Zimbra :
- Version 10.0.6
- Joule-8.8.15-Patch-45 
- Kepler-9.0.0-Patch-38

Let's Encrypt Certbot utilise par défaut ECDSA secp256r1 (P-256) depuis la version 2.0.0.

Si vous utilisez une version obsolète ou si vous avez besoin de certificats RSA, consultez [l'ancienne documentation](https://wiki.zimbra.com/index.php?title=Installing_a_LetsEncrypt_SSL_Certificate&oldid=69351) à vos risques et périls.

## Certificats génériques (wildcard)

Let's Encrypt prend en charge les certificats génériques avec validation DNS manuelle :

```bash
/usr/local/sbin/certbot certonly -d *.example.com --preferred-chain "ISRG Root X2" --agree-tos --register-unsafely-without-email --preferred-challenges=dns --manual
```

## Déploiement sur Zimbra

### Création du script de déploiement

Créez le script suivant pour déployer le certificat Let's Encrypt sur Zimbra :

```bash
cat >> /usr/local/sbin/letsencrypt-zimbra << EOF 
#!/bin/bash 
/usr/local/sbin/certbot certonly -d $(hostname --fqdn) --standalone -n --preferred-chain "ISRG Root X2" --agree-tos --register-unsafely-without-email 
cp "/etc/letsencrypt/live/$(hostname --fqdn)/privkey.pem" /opt/zimbra/ssl/zimbra/commercial/commercial.key 
chown zimbra:zimbra /opt/zimbra/ssl/zimbra/commercial/commercial.key 
wget -O /tmp/ISRG-X2.pem https://letsencrypt.org/certs/isrg-root-x2.pem 
rm -f "/etc/letsencrypt/live/$(hostname --fqdn)/chainZimbra.pem" 
cp "/etc/letsencrypt/live/$(hostname --fqdn)/chain.pem" "/etc/letsencrypt/live/$(hostname --fqdn)/chainZimbra.pem" 
cat /tmp/ISRG-X2.pem >> "/etc/letsencrypt/live/$(hostname --fqdn)/chainZimbra.pem" 
chown zimbra:zimbra /etc/letsencrypt -R 
cd /tmp 
su zimbra -c '/opt/zimbra/bin/zmcertmgr deploycrt comm "/etc/letsencrypt/live/$(hostname --fqdn)/cert.pem" "/etc/letsencrypt/live/$(hostname --fqdn)/chainZimbra.pem"' 
rm -f "/etc/letsencrypt/live/$(hostname --fqdn)/chainZimbra.pem" 
EOF
```

### Configuration du renouvellement automatique

Définissez les permissions correctes, configurez une tâche cron et effectuez le premier déploiement :

```bash
chmod +rx /usr/local/sbin/letsencrypt-zimbra 
ln -s /usr/local/sbin/letsencrypt-zimbra /etc/cron.daily/letsencrypt-zimbra 
/etc/cron.daily/letsencrypt-zimbra
```

### Redémarrage de Zimbra

Redémarrez Zimbra pour charger le nouveau certificat :

```bash
sudo su zimbra -c '/opt/zimbra/bin/zmcontrol restart'
```

##Redémarrage des services off de  Zimbra 
```bash
su - zimbra

zmcontrol start mailbox
zmcontrol start zmconfigd
zmcontrol start service
zmcontrol start zimbraAdmin
zmcontrol start zimlet
zmcontrol start imapd
zmcontrol start zimbra
zmcontrol start mta
zmcontrol start stats
```

La tâche cron renouvellera automatiquement votre certificat environ un mois avant son expiration.

## Ressources supplémentaires

- [CLI zmtlsctl pour configurer le mode du serveur web](https://wiki.zimbra.com/wiki/CLI_zmtlsctl_to_set_Web_Server_Mode)
- [Configuration du proxy Zimbra et memcached](https://wiki.zimbra.com/wiki/Enabling_Zimbra_Proxy_and_memcached)
- [Installateur automatisé Zimbra](https://github.com/Zimbra/zinstaller) - Inclut également la configuration Let's Encrypt


  ## Renouvellements du SSL

# Guide de Renouvellement SSL pour Zimbra

## 📋 Vue d'ensemble

Ce guide explique comment renouveler le certificat SSL Let's Encrypt sur un serveur Zimbra déjà configuré. Il couvre le renouvellement manuel, automatique et le dépannage.

## ⚠️ Prérequis

- Serveur Zimbra fonctionnel avec certificat Let's Encrypt existant
- Accès root au serveur
- Certbot installé dans `/usr/local/sbin/certbot`
- Domaine configuré (exemple: `mail.metalafrique-it.com`)

## 🔄 Renouvellement Manuel

### Vérification de l'état du certificat

```bash
# Vérifier la date d'expiration actuelle
openssl x509 -in /opt/zimbra/ssl/zimbra/commercial/commercial.crt -text -noout | grep "Not After"

# Vérifier les certificats Let's Encrypt disponibles
/usr/local/sbin/certbot certificates
```

### Renouvellement étape par étape

#### Étape 1: Arrêter les services web
```bash
# Arrêter le proxy Zimbra pour libérer le port 80
su - zimbra -c "zmproxyctl stop"

# Vérifier que les ports sont libres
netstat -tulpn | grep ":80"
```

#### Étape 2: Renouveler le certificat
```bash
# Renouvellement avec Certbot
/usr/local/sbin/certbot renew --cert-name mail.metalafrique-it.com --force-renewal

# Ou créer un nouveau certificat si nécessaire
/usr/local/sbin/certbot certonly -d mail.metalafrique-it.com --standalone --preferred-chain "ISRG Root X2" --agree-tos --register-unsafely-without-email
```

#### Étape 3: Déployer le certificat dans Zimbra
```bash
# Copier la clé privée
cp "/etc/letsencrypt/live/mail.metalafrique-it.com/privkey.pem" /opt/zimbra/ssl/zimbra/commercial/commercial.key
chown zimbra:zimbra /opt/zimbra/ssl/zimbra/commercial/commercial.key

# Télécharger la chaîne racine ISRG
wget -O /tmp/ISRG-X2.pem https://letsencrypt.org/certs/isrg-root-x2.pem

# Créer la chaîne complète pour Zimbra
cp "/etc/letsencrypt/live/mail.metalafrique-it.com/chain.pem" "/tmp/chainZimbra.pem"
cat /tmp/ISRG-X2.pem >> "/tmp/chainZimbra.pem"

# Déployer le certificat dans Zimbra
su zimbra -c '/opt/zimbra/bin/zmcertmgr deploycrt comm "/etc/letsencrypt/live/mail.metalafrique-it.com/cert.pem" "/tmp/chainZimbra.pem"'

# Nettoyer les fichiers temporaires
rm -f /tmp/chainZimbra.pem /tmp/ISRG-X2.pem
```

#### Étape 4: Redémarrer Zimbra
```bash
# Redémarrer tous les services Zimbra
su - zimbra -c "zmcontrol restart"
```

#### Étape 5: Vérification
```bash
# Vérifier la nouvelle date d'expiration
openssl x509 -in /opt/zimbra/ssl/zimbra/commercial/commercial.crt -text -noout | grep "Not After"

# Tester l'accès HTTPS
curl -I https://mail.metalafrique-it.com

# Vérifier l'état des services
su - zimbra -c "zmcontrol status"
```
## Guide : Messages Bloqués en Queue Zimbra
# Ici vous aurez un rapport de tous les problèmes rencontrés et comment je les ai corrigés

## Problème
- 7 messages bloqués vers `xxxxxxxxx.com`
- Erreurs : Connection refused / timeout
- Queue qui ne se vide pas

## Diagnostic

### Vérifier la queue
```bash
su zimbra
/opt/zimbra/common/sbin/postqueue -p
```

### Tester la destination
```bash
telnet 168.231.76.223 25
dig xxxxxxxxx.com MX
```

**Résultat** : Pas de serveur mail configuré pour ce domaine

## Solutions

### Solution 1 : Ajouter le domaine à Zimbra
```bash
su zimbra
zmprov cd xxxxxxxxx.com
zmprov ca noreply@xxxxxxxxx.com motdepasse123
/opt/zimbra/common/sbin/postqueue -f
```


Les contributions à ce guide sont les bienvenues. Veuillez soumettre une pull request ou ouvrir une issue pour toute suggestion d'amélioration.

## Licence

Ce projet est sous licence MIT. Voir le fichier LICENSE pour plus de détails.
