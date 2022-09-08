# OpenVPN for Docker

[![Build Status](https://travis-ci.org/kylemanna/docker-openvpn.svg)](https://travis-ci.org/kylemanna/docker-openvpn)
[![Docker Stars](https://img.shields.io/docker/stars/kylemanna/openvpn.svg)](https://hub.docker.com/r/kylemanna/openvpn/)
[![Docker Pulls](https://img.shields.io/docker/pulls/kylemanna/openvpn.svg)](https://hub.docker.com/r/kylemanna/openvpn/)
[![ImageLayers](https://images.microbadger.com/badges/image/kylemanna/openvpn.svg)](https://microbadger.com/#/images/kylemanna/openvpn)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fkylemanna%2Fdocker-openvpn.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fkylemanna%2Fdocker-openvpn?ref=badge_shield)


{{{Instruction de commande de ligne}}}

Vous pouvez aussi  utiliser des fichiers de votre propre ordinateurs en utilisant les instructions ci-dessous .

git config --global user.name "nom_utilisateur"
git config --global user.email "email_utilisateur"

-*{{Créez un nouveaux dépôt}}

git clone ssh://git@git.ipeos.com:10022/spoullet/openvpn.git
cd openvpn
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master

-*{{Pousser un fichier existant}}

cd "fichier_existant"
git init
git remote add origin ssh://git@git.ipeos.com:10022/spoullet/openvpn.git
git add .
git commit -m "Initial commit"
git push -u origin master

-*{{Poussez un dépôt Git existant}} 

cd existing_repo
git remote rename origin old-origin
git remote add origin ssh://git@git.ipeos.com:10022/spoullet/openvpn.git
git push -u origin --all
git push -u origin --tags

----

-*{{{Démarrage Rapide}}}

Choisissez un nom pour le conteneur de volume de données $OVPN_DATA. Il est recommandé d'utiliser le préfixe  ovpn-data pour opérer de façon transparente  avec la référence du service systemd. Les utilisateurs sont encouragés  à remplacer l'exemple  avec un nom descriptif de leur choix.

OVPN_DATA="ovpn-data-example"

Initialiser le conteneur $OPVN_DATA qui contiendra les fichiers de configuration  et les  certificats. Le conteneur vous demandera une passe-phrase pour protéger la clé privée utilisé par le nouvellement généré certificat d’autorité.


docker volume create --name $OVPN_DATA
docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki

 -*{{ Démarrer le processus du serveur  OpenVPN server}}

docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN kylemanna/openvpn

 -*{{Générer  un certificat client sans mot de passe}}

 docker run -v $OVPN_un DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full CLIENTNAME nopass

 -*{{Récupéré la configuration du client avec les  certifcats embarqués .}}

 docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn

 -*{{Révocation des certificats clients  .}}

docker exec -it OPENVPN-CONTAINER easyrsa revoke MY-USER-CN
docker exec -it OPENVPN-CONTAINER easyrsa gen-crl

Révoque le certificat de MY-USER-CN et génére la liste de révocation de certificats (CRL)

Ou à l'aide du ovpn_revokeclientscript :

docker run --rm -it -v $OVPN_DATA:/etc/openvpn kylemanna/openvpn ovpn_revokeclient client1

 -*{{Liste des clients}}
docker run --rm -it -v $OVPN_DATA:/etc/openvpn kylemanna/openvpn ovpn_listclients

ou

sudo cat /srv/wakandaapps/ovpn/data/pki/index.txt

-*{{Astuce de débogage }} 
-* Créé une variable d'environnement avec le nom DEBUG et la valeur  1 pour déclencher  une sortie débogage ( utilisé " docker -e") 

 docker run -v $OVPN_DATA:/etc/openvpn -p 1194:1194/udp --cap-add=NET_ADMIN -e DEBUG=1 kylemanna/openvpn

-*{{Test en utilisant un client qui a openvpn installé correctement }}

$ openvpn --config CLIENTNAME.ovpn

-*{{Démarrer un  barrage de  vérification de débogage  sur le client si les choses ne marchent pas.}} 

$ ping 8.8.8.8    #  Vérifier la connectivité sans toucher à la résolution de nom.
$ dig google.com  # N'utilisera pas la directive de recherche dans resolv.conf.
$ nslookup google.com # Utilisera recherche.

Considerons la mise en place d'un service systemd pour demarrage automatique  au demarrage de la machine et un redemarrage  dans le cas ou le daemon Openvpn ou le Docker Crash. 

-*{{Comment cela fonctionne ?}} 

Initialiser le conteneur de volume en utilisant l'image  Kylemanna/openvpn  avec le script inclu pour générer automatiquement:

   -{{Les paramètres Diffie-Hellman }}
   -{{Une clé privée }}
   -{{Un auto-certificat correspondant à  la clé privée pour le serveur OpenVPN.}}
   -{{Une clé et un certificat   EasyRSA CA }}
   -{{Une clé TLS auth  de sécurité HMAC}}

Le serveur OpenVPN est démarré avec commande de démarrage par défaut de opvn_run

La configuration est localisé dans /etc/openvpn, et le fichier DOCKER déclare ce dossier comme étant un volume. Cela signifie que vous pouvez démarrer un autre conteneur avec l'argument (-v), et accéder à la configuration.Le volume contient aussi les clés PKI et certificat pour qu'il puisse être  sauvegarder.

Pour générer un certicat client , kylemanna/openvpn utilise EasyRSA via la command easyrsa dans le chemin du conteneur.La variable d'environnement EASYRSA place le PKI CA dans /etc/openvpn/pki.
Idéalement kylemanna/openvpn  viens avec un script appelé ovpn_getclient, lequel depose un fichier de  configuration OpenVPN . Ce fichier seul peut étre donné à un client pour accéder au VPN.

Pour activer l'Authentification à double facteurs pour le client(a.k.a. OTP) regarder ce document.
OpenVPN Details

On utilise le mode "{{tun}}" , car il marche avec la plus grande variété d'appareil . 
Par exemple le mode "{{tap}}" ne marche pas avec Android  sauf si l'appareil est routé .

La topologie utilisé est net30, car elle fonctionne avec la plus grande  variété de système d'exploitation. Par exemple p2p ne marche pas avec Windows

Le serveur UDP utilise 192.168.255.0/24 pour les clients dynamiques par défaut.

Le profil client spécifie la passerelle de redirection def1, ce qui signifie qu'après l'établissement de la connexion VPN, tout le trafic passera par le VPN. Cela peut poser des problèmes si vous utilisez des récurseurs DNS locaux qui ne sont pas directement accessibles, car vous essayer de les atteindre via le VPN et ils pourraient ne pas vous répondre. Si cela se produit, utilisez des résolveurs DNS publics comme ceux de Google (8.8.4.4 et 8.8.8.8) ou OpenDNS (208.67.222.222 et 208.67.220.220).
Discussion sur la sécurité



{{{Instruction de commande de ligne}}}

Vous pouvez aussi  utiliser des fichiers de votre propre ordinateurs en utilisant les instructions ci-dessous .

git config --global user.name "nom_utilisateur"
git config --global user.email "email_utilisateur"

-*{{Créez un nouveaux dépôt}}

git clone ssh://git@git.ipeos.com:10022/spoullet/openvpn.git
cd openvpn
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master

-*{{Pousser un fichier existant}}

cd "fichier_existant"
git init
git remote add origin ssh://git@git.ipeos.com:10022/spoullet/openvpn.git
git add .
git commit -m "Initial commit"
git push -u origin master

-*{{Poussez un dépôt Git existant}} 

cd existing_repo
git remote rename origin old-origin
git remote add origin ssh://git@git.ipeos.com:10022/spoullet/openvpn.git
git push -u origin --all
git push -u origin --tags

----

-*{{{Démarrage Rapide}}}

Choisissez un nom pour le conteneur de volume de données $OVPN_DATA. Il est recommandé d'utiliser le préfixe  ovpn-data pour opérer de façon transparente  avec la référence du service systemd. Les utilisateurs sont encouragés  à remplacer l'exemple  avec un nom descriptif de leur choix.

OVPN_DATA="ovpn-data-example"

Initialiser le conteneur $OPVN_DATA qui contiendra les fichiers de configuration  et les  certificats. Le conteneur vous demandera une passe-phrase pour protéger la clé privée utilisé par le nouvellement généré certificat d’autorité.


docker volume create --name $OVPN_DATA
docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki

 -*{{ Démarrer le processus du serveur  OpenVPN server}}

docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN kylemanna/openvpn

 -*{{Générer  un certificat client sans mot de passe}}

 docker run -v $OVPN_un DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full CLIENTNAME nopass

 -*{{Récupéré la configuration du client avec les  certifcats embarqués .}}

 docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn

 -*{{Révocation des certificats clients  .}}

docker exec -it OPENVPN-CONTAINER easyrsa revoke MY-USER-CN
docker exec -it OPENVPN-CONTAINER easyrsa gen-crl

Révoque le certificat de MY-USER-CN et génére la liste de révocation de certificats (CRL)

Ou à l'aide du ovpn_revokeclientscript :

docker run --rm -it -v $OVPN_DATA:/etc/openvpn kylemanna/openvpn ovpn_revokeclient client1

 -*{{Liste des clients}}
docker run --rm -it -v $OVPN_DATA:/etc/openvpn kylemanna/openvpn ovpn_listclients

ou

sudo cat /srv/wakandaapps/ovpn/data/pki/index.txt

-*{{Astuce de débogage }} 
-* Créé une variable d'environnement avec le nom DEBUG et la valeur  1 pour déclencher  une sortie débogage ( utilisé " docker -e") 

 docker run -v $OVPN_DATA:/etc/openvpn -p 1194:1194/udp --cap-add=NET_ADMIN -e DEBUG=1 kylemanna/openvpn

-*{{Test en utilisant un client qui a openvpn installé correctement }}

$ openvpn --config CLIENTNAME.ovpn

-*{{Démarrer un  barrage de  vérification de débogage  sur le client si les choses ne marchent pas.}} 

$ ping 8.8.8.8    #  Vérifier la connectivité sans toucher à la résolution de nom.
$ dig google.com  # N'utilisera pas la directive de recherche dans resolv.conf.
$ nslookup google.com # Utilisera recherche.

Considerons la mise en place d'un service systemd pour demarrage automatique  au demarrage de la machine et un redemarrage  dans le cas ou le daemon Openvpn ou le Docker Crash. 

-*{{Comment cela fonctionne ?}} 

Initialiser le conteneur de volume en utilisant l'image  Kylemanna/openvpn  avec le script inclu pour générer automatiquement:

   -{{Les paramètres Diffie-Hellman }}
   -{{Une clé privée }}
   -{{Un auto-certificat correspondant à  la clé privée pour le serveur OpenVPN.}}
   -{{Une clé et un certificat   EasyRSA CA }}
   -{{Une clé TLS auth  de sécurité HMAC}}

Le serveur OpenVPN est démarré avec commande de démarrage par défaut de opvn_run

La configuration est localisé dans /etc/openvpn, et le fichier DOCKER déclare ce dossier comme étant un volume. Cela signifie que vous pouvez démarrer un autre conteneur avec l'argument (-v), et accéder à la configuration.Le volume contient aussi les clés PKI et certificat pour qu'il puisse être  sauvegarder.

Pour générer un certicat client , kylemanna/openvpn utilise EasyRSA via la command easyrsa dans le chemin du conteneur.La variable d'environnement EASYRSA place le PKI CA dans /etc/openvpn/pki.
Idéalement kylemanna/openvpn  viens avec un script appelé ovpn_getclient, lequel depose un fichier de  configuration OpenVPN . Ce fichier seul peut étre donné à un client pour accéder au VPN.

Pour activer l'Authentification à double facteurs pour le client(a.k.a. OTP) regarder ce document.
OpenVPN Details

On utilise le mode "{{tun}}" , car il marche avec la plus grande variété d'appareil . 
Par exemple le mode "{{tap}}" ne marche pas avec Android  sauf si l'appareil est routé .

La topologie utilisé est net30, car elle fonctionne avec la plus grande  variété de système d'exploitation. Par exemple p2p ne marche pas avec Windows

Le serveur UDP utilise 192.168.255.0/24 pour les clients dynamiques par défaut.

Le profil client spécifie la passerelle de redirection def1, ce qui signifie qu'après l'établissement de la connexion VPN, tout le trafic passera par le VPN. Cela peut poser des problèmes si vous utilisez des récurseurs DNS locaux qui ne sont pas directement accessibles, car vous essayer de les atteindre via le VPN et ils pourraient ne pas vous répondre. Si cela se produit, utilisez des résolveurs DNS publics comme ceux de Google (8.8.4.4 et 8.8.8.8) ou OpenDNS (208.67.222.222 et 208.67.220.220).
Discussion sur la sécurité





## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fkylemanna%2Fdocker-openvpn.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fkylemanna%2Fdocker-openvpn?ref=badge_large)
](https://github.com/kylemanna/docker-openvpn)
