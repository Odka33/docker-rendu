# Compte rendu Docker : ESTAY, BASCOP, BEN AYOUN
## Details de l’installation :
###### Expliquez en quoi cela pourrait être dangereux d'un point de vue sécurité (c'est en rapport avec le groupe d'utilisateurs docker)
Pour utiliser docker sans être root , il suffit d’ajouter notre utilisateur au groupe docker pour lui permettre son utilisation :

`$> sudo usermod -a -G docker "monuser"`

Nous ne devons pas utiliser docker en root pour ne pas, dans le cas d’une mauvaise manipulation, faire des choses regrettables, mais aussi, n’importe quel processus lancé en root y a un accès constant, ce qui n’est pas une bonne pratique, et plutôt dangereux

![Memes](/1t53ny.jpg)

Utilisation élémentaire :

Avant tu l’allumes : 

`$> sudo systemctl start docker`

téléchargez l'image ubuntu avec docker pull

`$> sudo docker pull ubuntu`

Nous ne stipulons pas de tag car dans ce cas Docker Engine ira télécharger la dernière version par défaut.
créez un conteneur en utilisant l'image ubuntu (avec docker run)

`$> sudo docker run ubuntu`

Créez un conteneur en utilisant l'image ubuntu, faites en sorte que le conteneur reste en vie, et rentrez à l'intérieur de celui-ci avec un shell bash (vous devez donc pouvoir passer des commandes dans le conteneur)

`$> sudo docker run -it id :nom bash`

Lancez un conteneur en tâche de fond (peu importe lequel), ce conteneur doit lancer un processus facilement identifiable (par exemple sleep 999999)
 	
`$> sudo docker run -d -it ubuntu sleep 9999999`

Mettez en évidence l'utilisation des namespaces et des cgroups par ce conteneur
il existe plusieurs façons de faire ça, je vous en ai déjà montré, ça se trouve aussi sur l'ami Internet
indices en vrac : systemd pourra vous aider, le répertoire /proc aussi

`$> sudo ps -ef` (lister les processus)

`$> ls /proc (cat/proc/numéro proc/cmdline)` (liste les processus) 

`$> systemd-cgtop  ou systemd-cgls` (Afficher les principaux groupes de contrôle en fonction de leur utilisation des ressources)

`$> lsns`(Listes les namespaces)

`$> ls -al /proc/(Process ID)/ns` (A exécuter sur un process de notre hôte et sur le process d’un containers avec le même id, nous remarquons qu’ils n’ont pas le même namspaces)



![Memes](/what-process-do.jpg)


Utilisation élémentaire :
Avant tu l’allumes : sudo systemctl start docker
téléchargez l'image ubuntu avec docker pull

`$> sudo docker pull ubuntu`

Nous ne stipulons pas de tag car dans ce cas Docker Engine ira télécharger la dernière version par défaut.
Créez un conteneur en utilisant l'image ubuntu (avec docker run)

`$> sudo docker run ubuntu`

Créez un conteneur en utilisant l'image ubuntu, faites en sorte que le conteneur reste en vie, et rentrez à l'intérieur de celui-ci avec un shell bash (vous devez donc pouvoir passer des commandes dans le conteneur)

`$> sudo docker run -it id :nom bash`

Lancez un conteneur en tâche de fond (peu importe lequel), ce conteneur doit lancer un processus facilement identifiable (par exemple sleep 999999)

`$> sudo docker run -d -it ubuntu sleep 999999`

Mettez en évidence l'utilisation des namespsaces et des cgroups par ce conteneur il existe plusieurs façons de faire ça, je vous en ai déjà montré, ça se trouve aussi sur l'ami Internet. 
Indices en vrac : systemd pourra vous aier, le répertoire /proc aussi 

`$> sudo ps -ef`

`$> ls /proc (cat/proc/*numéro proc*/cmdline` Lister les processus 

`$> systemd-cgtop ou systemd-cgls`Afficher les principaux groupes de contrôle en fonction de leur utilisation des ressources

`$> lsns` Lister les namespaces

`$> ls -al /proc/(*Process ID*)/ns`A exécuter sur un process de notre hôte et sur le process d'un containers avec le même id, nous remarquons qu'ils n'ont pas le même namespaces

## Docker Focus
Cherchez où est apposée cette restriction (réfléchissez à ce que vous cherchez : on parle des droits d'accès à Docker, ou plutôt, au point d'accès pour échanger de la donnée avec Docker...) ?

> Cette restriction est apposé au /var/run/docker.sock qui correspond à la socket du démon docker .
![img](/1.png)

Changer la configuration de base du démon docker dockerd. Utilisez un répertoire de travail (l'habituel /var/lib/docker) sur une partition LVM à la racine (/data par exemple) ?

> J’ai ajouter un volume physique de 3 go nous pouvons constater qu’il est bien présent à l’aide de la commande :
![img](/2.png)

> Dans notre cas c'est SDC qui fait 3 Go.

> Ont à ensuite crée un volume groupe LVM ou l'on a rajouter la nouvelle partition: 
![img](/3.png)

> Par la suite nous allons crée les volumes logique à partir des volumes physiques:
![img](/4.png)

> Ont peut prouver que le volume à été crée avec lvdisplay pour afficher les volumes logiques :
![img](/5.png)

> Ont va ensuite crée le filesystem de la partition logique en ext4 :
![img](/6.png)


> Ensuite nous allons crée un dossier à la racine appelé  “/Data”.
![img](/7.png)

> Pour finir nous allons monter “/dev/mvg/Vol1” dans le répertoire “/Data” à la racine .
![img](/8.png)


> Il est possible de vérfié la configuration:
![img](/9.png)


> Pour commencer stoper le service docker:
![img](/10.png)

> Ont va rajouter une ligne “graph” qui permet de relocaliser les fichier “/var/lib/docker”:
![img](/11.png)


> Ensuite on vas déplacer le contenus de “/var/lib/docker” dans “/Data/docker” :
![img](/12.png)

> Et redémarrer le démon docker 
![img](/13.png)

> Il est possible de vérifier la configuration avec docker info:
![img](/14.png)

## Changez le OOM score du démon (plus il est haut et plus il y a des chances qu'il se fasse détruire en premier) ?


> J’ai configurer l’oom score dans le répertoire “/etc/docker/” en modifiant daemon.json j’y est ajouter un score OOM par défaut qui seras appliquer a tout les conteneur:
![img](/15.png)

## Créer une autre unité systemd docker-tcp.service ?

> Nous allons crée une unité systemd appelée “docker-tcp.service” dans le répertoire “/etc/systemd/system/docker-tcp.service.d/“ à l’aide du squelette présent dans “/usr/lib/systemd/system/”
![img](/16.png)

> L’ ip ou sera mappé le démon et changer pour être accessible à l’adresse “192.168.56.99:2375”.


## Lancez une deuxième instance du démon Docker, accessible à travers un socket TCP (à travers le réseau donc) ?

> Ont peut prouver que la nouvelle instance docker est accessible via l’hote docker :

![img](/17.png)

## Testez cette configuration avec TCP (depuis votre machine hôte, une deuxième VM, ou en local en forçant une connexion TCP) ?

> Mais aussi via l’ordinateur hote de centos et de docker qui peut effectuer des requête sur le démon: 
![img](/18.png)


## Quelles autres options du démon docker (dockerd) peuvent être utilisées pour le rendre plus sécurisé et/ou robuste et/ou restreint ?

> Il peut être implémenter un proxy.
> Empêcher des apelles systèmes sur la machine hôte.
> Choix du protocole réseaux.
> l’implémentation de driver de stockage.
> Possibilité de restreindre la mémoire des conteneurs.
> IPV4 et IPV6

## Basique docker run

> J’atteste pour mon groupe et moi même que nous passont cette partie du cours car nous avons effectuer la moitié des actions demandé dans les question précédente

## Création de Dockerfile

![img](/memes2.jpg)






### HTTP API

> lancez un conteneur et récupérez son IP depuis une requête curl

`docker run -d alpine touch /helloworld`

`curl --unix-socket /var/run/docker.sock\`

`curl --unix-socket /var/run/docker.sock "http:/v1.24/containers/ca5f55cdb/logs?stdout=1"{`
 
récupérez la liste des imageso

`curl --unix-socket /var/run/docker.sock http:/v1.24/images/json`

récupérez la liste des conteneurs actifs

`curl --unix-socket /var/run/docker.sock http:/v1.24/containers/json`


###  Docker : configuration avancée du démon

## Votre démon Docker doit utilisez la politique seccomp recommandée par le projet Moby ?

> La politique Seccomp et par défaut activer sur docker depuis l’intégration du projet moby à docker .
On peut vérifier que cette politique est active avec la commande suivante.
![img](/19.png)

## Expliquez brièvement, avec vos mots, ce qu’est seccomp et le rapport avec docker ?

> seccomp est une fonctionnalité de sécurité informatique du noyau Linux. L’utilisation de Seccomp dans docker permet de restreindre les actions disponible (appelle système) dans le conteneur c’est une whitelist.

## Expliquez l'utilité d'utiliser ce driver de stockage (la réponse n'est PAS juste "c'est sécurisé") ?

> Device Mapper est un framework permettant de mapper un bloc de donnée sur un autre bloc elle permet des fonctionnalité telle que l’instantané (snapshot) et le chiffrement de disque.



## Mettez en place une configuration valide avec le driver device-mapper ?

>il existe 2 configuration possible afin de mettre en place le driver “device-mapper”.

>Pour m’as part j’ai choisie “CONFIGURE DIRECT-LVM MODE MANUALLY”.

>Pour commencer par sauvegarder vos conteneur car ils seront alors compromis dans l’opération.

>Ensuite il vous faudras crée une partition LVM sur le disque à l’aide de la commande dd ou autre.

> Arrêtez docker avec la commande :

`$ sudo systemctl stop docker.`

> Installer les pacquet suivant device-mapper-persistent-data, lvm2.

`yum install -y device-mapper-persistent-data, lvm2`


> Ensuite il faut crée un volume physique  à partir de la phériphérique a bloc.
![img](/20.png)


> Créez un docker groupe de volumes sur le même périphérique à l'aide de la vgcreate commande
![img](/21.png)

> Créez deux volumes logiques nommés thinpoolet en thinpoolmeta utilisant la lvcreate commande. Le dernier paramètre spécifie la quantité d’espace disponible pour permettre l’extension automatique des données ou des métadonnées si l’espace est faible, en tant qu’interruption temporaire.
![img](/22.png)

> Convertissez les volumes en un thin pool et un emplacement de stockage pour les métadonnées du thin pool, à l'aide de la lvconvert commande.
![img](/23.png)

> Configurez l'auto-extension des pools minces via un lvm .
![img](/24.png)

> Ensuite on y ajoute les ligne suivantes afin d’étendre les pool.
![img](/25.png)

> Appliquez le profil LVM en utilisant la lvchange commande.
![img](/26.png)


> Activez la surveillance des volumes logiques sur votre hôte. Sans cette étape, l'extension automatique ne se produit pas même en présence du profil LVM
![img](/27.png)

> Pour finir il faut configurer le démon docker en ajoutant le code suivant:
![img](/28.png)


> Rédemarrage de docker
![img](/29.png)

> Puis vérification des configuration
![img](/30.png)
 
### Déploiement d'application n-tiers avec compose
> Je lance simplement mon conteneur Nginx en le faisant écouter sur le port 80
 
        	docker run -d --name "test-nginx" -p 8080:80 -v
        	$(pwd):/usr/share/nginx/html:ro nginx:latest
 
 
on le test pour vérifier le bon fonctionnement de celui çi :
 
        	docker inspect test-nginx
 
> nous connectons ensuite nos conteneurs Redis et Python avec un docker compose , nous installons docker compose avec :
	sudo pip install docker-compose 
> puis nous écrivons notre docker compose :

```
version: '3'
services:
	web:
    	build:
        	context: ./app
    	ports:
        	- "80:5000"
    	networks:
        	- frontend
    	depends_on:
        	- redisdb
    	environment:
        	- REDIS_HOST=db
        	- REDIS_PORT=6379
	redisdb:
    	image: redis
    	ports:
        	- "6379"
    	networks:
        	- frontend
    	volumes:
        	- redis_data:/data
networks:
	frontend:
volumes:
	redis_data:
 ```
 
> nous nommons notre hôte redis db comme demandé , et nous le faison écouter sur le port 6379 , qui est joingnable donc avec notre conteneur python crée au préalable.

### Utilisation d'un GUI avec Portainer

## Déployez Portainer sur votre hôte existant ?

> docker volume create portainer_data création d’un volume « portainer data »
![img](/31.png)

> Démarrage du conteneur possédant un apps web mapper sur le port 9000 avec la commande « docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer  »
![img](/32.png)

> Suite au démarrage du conteneur il nous est demander de choisir sur qu’elle environnement lancer portainer . Nous choisissons un environnement local.

> Nous accédons enfin a l’interface ou il est possible de manager image, registre événement volume network container . Et d’avoir tout informations sur un conteneur précis.
![img](/33.png)


## Créez un deuxième hôte Docker et le piloter depuis Portainer (socket TCP) ?

> Ont lance alors un autre container débian.
![img](/34.png)

> Ont peut constater qu’il est présent et monitorer par portainer.


# FIN
Docker c'est de l'eau

![img](/fin.jpeg)
