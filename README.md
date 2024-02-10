# Projet student-list

Veuillez trouver la consigne en cliquant [ici](https://github.com/diranetafen/student-list.git "ici")

!["Crédit image : eazytraining.fr"](https://eazytraining.fr/wp-content/uploads/2020/04/pozos-logo.png) ![projet](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)

------------

Prénom : Adil

Nom de famille : DALAOUI

## Application

J'ai dû déployer une application nommée "*student_list*", très basique, qui permet à POZOS d'afficher la liste de certains étudiants avec leur âge.

L'application student_list comporte deux modules :

- Le premier module est une API REST (nécessitant une authentification de base) qui envoie la liste souhaitée des étudiants basée sur un fichier JSON
- Le deuxième module est une application web écrite en HTML + PHP qui permet à l'utilisateur final d'obtenir une liste d'étudiants


## Le besoin

Mon travail consistait à :
1) construire un conteneur pour chaque module
2) les faire interagir entre eux

## Mon plan

D'abord, laissez-moi vous présenter les cinq ***fichiers*** de ce projet et leur rôle

Ensuite, je vous montrerai comment j'ai ***construit*** et testé l'architecture pour justifier mes choix

La troisième et dernière partie concernera le processus de ***déploiement*** que je suggère pour cette application.


### Rôle des fichiers

Dans mon livrable, vous trouverez deux fichiers principaux : un ***Dockerfile*** et un ***docker-compose.yml*** 

- docker-compose.yml : pour lancer l'application (API et application web)
- simple_api/student_age.py : contient le code source de l'API en python
- simple_api/Dockerfile : pour construire l'image de l'API avec le code source à l'intérieur
- simple_api/student_age.json : contient le nom et l'âge des étudiants au format JSON
- index.php : page PHP où l'utilisateur final se connectera pour interagir avec le service et lister les étudiants avec leur âge.


## Construire et tester

Considérant que vous venez de cloner ce dépôt, vous devez suivre ces étapes pour préparer l'application 'student_list' :

1) Changez de répertoire et construisez l'image du conteneur api :

```bash
cd ./mini-projet-docker/simple_api
docker build . -t api.student_list.img
docker images
```
> ![1-docker images](https://user-images.githubusercontent.com/101605739/224588377-b8afa11f-33b6-41ed-9f58-6e23d2054c83.jpg)

2) Créez un réseau de type pont pour que les deux conteneurs puissent se contacter par leurs noms grâce aux fonctions dns :

```bash
docker network create student_list.network --driver=bridge
docker network ls
```
> ![2-docker network ls](https://user-images.githubusercontent.com/101605739/224588523-a842cd26-c5d5-4338-8547-2e31578655c9.jpg)


3) Revenez au répertoire racine du projet et lancez le conteneur de l'api backend avec ces arguments :

```bash
cd ..
docker run --rm -d --name=api.student_list --network=student_list.network -v ./simple_api/:/data/ api.student_list.img
docker ps
```
> ![3-docker ps](https://user-images.githubusercontent.com/101605739/224589378-abcc3f7d-d5c6-4a81-ba28-767cb6cd7b7c.jpg)

Comme vous pouvez le voir, le conteneur backend api écoute sur le port 5000.
Ce port interne peut être atteint par un autre conteneur du même réseau, donc j'ai choisi de ne pas l'exposer.

J'ai aussi dû monter le répertoire local `./simple_api/` dans le répertoire interne `/data/` du conteneur pour que l'api puisse utiliser la liste `student_age.json`


> ![4-./simple_api/:/data/](https://user-images.githubusercontent.com/101605739/224589839-7a5d47e6-fdff-40e4-a803-99ebc9d70b03.png)


4) Mettez à jour le fichier `index.php` :

Vous devez mettre à jour la ligne suivante avant de lancer le conteneur du site web pour que ***api_ip_or_name*** et ***port*** correspondent à votre déploiement
   ` $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';`

Grâce aux fonctions dns de notre réseau de type pont, nous pouvons facilement utiliser le nom du conteneur api avec le port que nous avons vu juste avant pour adapter notre site web

```bash
sed -i s\<api_ip_or_name:port>\api.student_list:5000\g ./website/index.php
```
> ![5-api.student_list:5000](https://user-images.githubusercontent.com/101605739/224590958-49c2ce64-c9a0-4655-93da-552f27f78b2f.png)


5) Lancez le conteneur de l'application web frontale :

Le nom d'utilisateur et le mot de passe sont fournis dans le code source `.simple_api/student_age.py`

> ![6-id/passwd](https://user-images.githubusercontent.com/101605739/224590363-0fdd56ae-9fb9-45e7-8912-64a6789faa9e.png)

```bash
docker run --rm -d --name=webapp.student_list -p 80:80 --network=student_list.network -v ./website/:/var/www/html -e USERNAME=toto -e PASSWORD=python php:apache
docker ps
```
> ![7-docker ps](https://user-images.githubusercontent.com/101605739/224591443-344fd2cd-ddbc-4780-bbc5-7cc0bdac156f.jpg)


6) Testez l'api via le frontend :

6a) En utilisant la ligne de commande :

La commande suivante demandera au conteneur frontal de requêter l'api backend et de vous montrer la sortie.
Le but est de tester à la fois si l'api fonctionne et si le frontend peut obtenir la liste des étudiants.

```bash
docker exec webapp.student_list curl -u toto:python -X GET http://api.student_list:5000/pozos/api/v1.0/get_student_ages
```
> ![8-docker exec](https://user-images.githubusercontent.com/101605739/224593842-23c7f3a5-e5bc-4840-a6af-2eda0f622710.png)


6b) En utilisant un navigateur web `IP:80` :

- Si vous exécutez l'application sur un serveur distant ou une machine virtuelle (par exemple, provisionnée par le fichier vagrant d'eazytraining), veuillez trouver votre adresse IP en tapant `hostname -I`
> ![9-hostname -I](https://user-images.githubusercontent.com/101605739/224594393-841a5544-7914-4b4f-91fd-90ce23200156.jpg)

- Si vous travaillez sur PlayWithDocker, ouvrez simplement le port 80 sur l'interface graphique
- Sinon, tapez `localhost:80`

Cliquez sur le bouton

> ![10-check webpage](https://user-images.githubusercontent.com/101605739/224594989-0cb5bcb7-d033-4969-a12e-0b2aa9953a97.jpg)


7) Nettoyez l'espace de travail :

Grâce à l'argument `--rm` que nous avons utilisé en démarrant nos conteneurs, ils seront supprimés dès qu'ils s'arrêteront.
Supprimez le réseau précédemment créé.


```bash
docker stop api.student_list
docker stop webapp.student_list
docker network rm student_list.network
docker network ls
docker ps
```
> ![11-clean-up](https://user-images.githubusercontent.com/101605739/224595124-3ea15f42-e6d5-462a-92a0-52af7c73c17a.jpg)


## Déploiement

Comme les tests ont réussis, nous pouvons maintenant "composeriser" notre infrastructure en mettant les paramètres `docker run` au format ***infrastructure as code*** dans un fichier `docker-compose.yml`.

1) Exécuter l'application (API + webapp) :

Comme nous avons déjà créé l'image de l'application, il vous suffit maintenant d'exécuter :

```bash
docker-compose up -d
```

Docker-compose permet de choisir quel conteneur doit démarrer en premier grâce au paramètre `depends_on:`.
Ici, le conteneur de l'API démarrera en premier.
> ![12-dépend de](https://user-images.githubusercontent.com/101605739/224595564-e010cc3f-700b-4b3e-9251-904dafbe4067.png)

------------

# Cela conclut mon rapport d'exécution du mini-projet Docker.

Tout au long de ce projet, j'ai eu l'occasion de créer une image Docker personnalisée, de configurer des réseaux et des volumes, et de déployer des applications en utilisant docker-compose. Dans l'ensemble, ce projet a été une expérience enrichissante qui m'a permis de renforcer mes compétences techniques et de mieux comprendre les principes des microservices. Je suis maintenant mieux équipé pour aborder des projets similaires à l'avenir et contribuer à améliorer les processus de conteneurisation et de déploiement au sein de mon équipe et de mon organisation.

![octocat](https://myoctocat.com/assets/images/base-octocat.svg)
