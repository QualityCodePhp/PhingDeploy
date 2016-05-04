# README #

Deploy permets :

* la création d'un package pour le déploiement
* le déploiement d'un package
* l'analyse statique du code

Il permet de gérer : 

* plusieurs projets
* plusieurs environnements de déploiement
* plusieurs serveurs par environnement


## Pré-requis ##

### Serveur de déploiement

* php >= 5.3
* php-ssh
* git
* composer
* installer la clef publique du serveur de déploiement sur les différents serveurs où l'on doit déployer
* ajouter la clef publique du serveur de déploiement sur bitbucket et github

### Fichiers de configuration du projet

Les fichiers de configuration du projet doivent être disponible pour chacun des environnements de déploiement.

Par exemple pour parameters.yml => parameters.yml.production

## Fonctionnement ##

### Deploy task

1. En local
    1. création du répertoire de release
    2. clonage du repository
    3. récupération du dernier tag de la branche configurée si il y a des tags ou récupération du tag passé en paramètre
    4. récupération de composer.phar
    5. copie des fichiers de configuration spécifique à l'environnement
    6. composer install
    7. exécution de la tache "package" du build.xml du projet
    8. création d'un package (fichier tgz)
2. Sur le(s) serveur(s) distant(s)
    1. création des répertoires du projet
    2. copie du package
    3. création du répertoire pour le déploiement
    4. décompression du package
    5. installation des cron
    6. exécution de la tache "deploy" du build.xml du projet
    7. mise à jour du lien current
    8. nettoyage des anciens déploiement

## Installation ##

```
#!bash
$ git clone git@github.com/flavien-metivier/phingDeploy.git
$ composer install

```
That's all !

## Utilisation ##

* ./vendor/bin/phing help                                                                            => Display this message
* ./vendor/bin/phing prepare -Dproject.name=test -Dstage.name=production                             => Prepare server
* ./vendor/bin/phing rollback -Dproject.name=test -Dstage.name=production                            => Rollback to previous deployment
* ./vendor/bin/phing package -Dproject.name=test -Dstage.name=production -Drepository.tag=1.0        => Application packaging
* ./vendor/bin/phing deploy -Dproject.name=test -Dstage.name=production -Drepository.tag=1.0         => Application deployment
* ./vendor/bin/phing qa -Dproject.name=test -Dstage.name=production -Drepository.tag=1.0             => Run qa tools
* ./vendor/bin/phing deploy-with-qa -Dproject.name=test -Dstage.name=production -Drepository.tag=1.0 => Run qa tools and deployment tasks

Si le projet est un bundle ou une library, le deployement n'a pas de sens, penser à mettre deploy.enable à false dans le fichier de 
configuration du projet.

## Structure ##

### Projet ###

```
.
├── build.xml                               => Taches de base et gestion des paramètres (deploy, help)
├── lib
│   ├── step
│   │   ├── history.xml                     => Déploiement sur les environnements distants
│   │   ├── package.xml                     => Création de l'archive
│   │   ├── remote.xml                      => Actions sur le(s) serveur(s)
│   │   ├── rollback.xml                    => Gestion du rollback
│   │   └── prepare.xml                     => Initialisation de la structure sur le(s) serveur(s)
│   └── tool
│       ├── composer.xml                    => Taches liées à composer
│       ├── git.xml                         => Taches liées à git
│       ├── cron.xml                        => Mise à jour des taches cron sur le(s) serveur(s)
│       └── qa.xml                          => Taches des outils de QA
└── properties
    ├── build.properties                    => Paramètrage général
    ├── projects                            => Définition des projets
    │   ├── swagger_bundle.properties
    │   └── wonderphoto.properties
    └── stage                               => Définition des environnements
        ├── preproduction.properties
        ├── production.properties
        ├── recette.properties
        └── recette                         => Définition des spécificités des projets pour un environnement
            ├── swagger_bundle.properties
            └── wonderphoto.properties
```

### Release ###

```
.
└── releases
    └── project_name
        └── stage_name
            └── build_uniq_id
                ├── code
                ├── package
                └── report
```

### Remote ###

```
.
└── project_name
    ├── cache
    ├── logs
    ├── packages
    ├── releases
    │   ├── previous
    │   └── current
    └── shared
```

## Taches spécifiques au projet ##

S'il y a besoin d'exécuter des taches spécifiques au projet, il suffit de placer un fichier phing build.xml à sa racine avec 
3 targets :

* pre-package   => avant la création de l'archive (par exemple : minification js et css)
* post-package  => après la création du package (par exemple : versionning du package, envoie d'un mail)
* pre-deploy    => avant le déploiement
* post-deploy   => après le déploiement
* pre-rollback  => avant le rollback
* post-rollback => après le rollback

Si la tache a besoin d'être éxécutée sur le serveur, il faut passer par la target deploy.tool.ssh

```
<phingcall target="deploy.tool.ssh">
    <property name="command" value="ma commande"/>
</phingcall>

```

## Mise en place de cron ##

Dans le projet mettre les cron dans : 

```
.
└── Ressources
    └── cron.d
        ├── project_name-__STAGE__-name1
        ├── project_name-__STAGE__-name2
        :
        └── ....
```
Les chemins des scripts dans les fichiers cron doivent être précédé par ```__PATH__```.

## Configuration ##

Paramètres et leur emplacement conseillé :

### properties/build.properties

| Nom   | Type   | Description                      | Default | Requis   |
| ----- | ------ | -------------------------------- | ------- | -------- |
| qa.phploc.path     | string | chemin complet ou relatif         | n/a | oui |
| qa.phpcpd.path     | string | chemin complet ou relatif         | n/a | oui |
| qa.phpdcd.path     | string | chemin complet ou relatif         | n/a | oui |
| qa.phpmd.path      | string | chemin complet ou relatif         | n/a | oui |
| qa.phpcs.path      | string | chemin complet ou relatif         | n/a | oui |
| qa.pdepend.path    | string | chemin complet ou relatif         | n/a | oui |
| qa.phpmetrics.path | string | chemin complet ou relatif         | n/a | oui |
| deploy.history     | entier | nombre de déploiement à conserver |  5  | non |


### properties/project/project_name.properties

| Nom   | Type   | Description                      | Default | Requis   |
| ----- | ------ | -------------------------------- | ------- | -------- |
| repository        | string  | url (http ou ssh) du dépot git         | n/a  | oui |
| repository.branch | string  | nom de la branche à récupérer          | n/a  | oui |
| source.path       | string  | chemin relatif des sources (ex: ./src) | n/a  | oui |
| deploy            | boolean | doit-on déployer le projet             | true | non |
| project.configuration.files | string | liste des fichiers de configuration (séparé par ,) | n/a | non |


### properties/stage/stage_name.properties

| Nom   | Type   | Description                      | Default | Requis   |
| ----- | ------ | -------------------------------- | ------- | -------- |
| deploy.path        | string | chemin complet sur le serveur                 | n/a | oui |
| deploy.hosts       | string | ip ou nom des serveurs (séparé par ,)         | n/a | oui |
| deploy.user        | string | utilisateur sur le serveur                    | n/a | oui |
| deploy.pubkeyfile  | string | chemin complet ou relatif de la clef publique | n/a | oui |
| deploy.privkeyfile | string | chemin complet ou relatif de la clef privée   | n/a | oui |

### properties/stage/stage_name/project_name.properties

Vous pouvez surcharger ici tous les paramètres
