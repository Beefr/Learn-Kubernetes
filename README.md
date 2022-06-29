# Learn-Kubernetes

Plan:

1. Explications
2. Installer Minikube
3. Lancer une application
4. Quelques commandes
5. Créer une pipeline avec Jenkins
6. Jouer avec l'application

________________________________________________


# 1. Explications 

Kubernetes est un système d'orchestration de cluster. Cela veut dire qu'il permet de déployer des applications conteneurisées sur un cluster. 
  
Le besoin initial: 
- avoir des applications disponibles 24h/24, 7j/7
- pouvoir déployer de nouvelles versions de l'application plusieurs fois par jour  
C'est ce que permet Kubernetes, sans temps d'arrêt, de manière simple et rapide. Et c'est opensource.  
  
Tuto officiel de kubernetes: https://kubernetes.io/fr/docs/tutorials/kubernetes-basics/  
  
Kubernetes coordonne un groupe d'ordinateurs hautement disponibles qui sont connectés pour fonctionner comme une seule et même unité. Les abstractions de Kubernetes vous permettent de déployer des applications conteneurisées dans un cluster sans les lier spécifiquement à des ordinateurs individuels. Pour utiliser ce nouveau modèle de déploiement, les applications doivent être empaquetées de manière à les dissocier des hôtes individuels: elles doivent être conteneurisées. Les applications conteneurisées sont plus flexibles et disponibles que dans les modèles de déploiement précédents, dans lesquels les applications étaient installées directement sur des machines spécifiques sous la forme de packages profondément intégrés à l'hôte. Un cluster Kubernetes est constitué de deux types de ressources: le maître qui coordonne le cluster, et les nœuds qui sont les serveurs qui exécutent des applications.  

![Schéma du Cluster](https://d33wubrfki0l68.cloudfront.net/283cc20bb49089cb2ca54d51b4ac27720c1a7902/34424/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)

Le maître est responsable de la gestion du cluster. Le maître coordonne toutes les activités de votre cluster, telles que la planification des applications, la gestion de l'état souhaité des applications, la mise à l'échelle des applications et le déploiement de nouvelles mises à jour.  
  
Un nœud est une machine virtuelle ou un ordinateur physique servant d’ordinateur de travail dans un cluster Kubernetes. Chaque nœud est doté d’un Kubelet, qui est un agent permettant de gérer le nœud et de communiquer avec le maître Kubernetes. Le nœud doit également disposer d'outils permettant de gérer les opérations de conteneur, telles que Docker ou rkt. Un cluster Kubernetes qui gère le trafic de production doit comporter au moins trois nœuds.

![Node overview](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

Lorsque vous déployez des applications sur Kubernetes, vous indiquez au maître de démarrer les conteneurs d'applications. Le maître planifie l'exécution des conteneurs sur les nœuds du cluster.

![Déploiement d'une application conteneurisée](https://d33wubrfki0l68.cloudfront.net/8700a7f5f0008913aa6c25a1b26c08461e4947c7/cfc2c/docs/tutorials/kubernetes-basics/public/images/module_02_first_app.svg)

Un cluster Kubernetes peut être déployé sur des machines physiques ou virtuelles. Pour démarrer avec le développement de Kubernetes, vous pouvez utiliser Minikube. Minikube est une implémentation Kubernetes légère qui crée une machine virtuelle sur votre machine locale et déploie un cluster simple contenant un seul nœud. (Ce que nous ferons.)  
  
Le déploiement instruit Kubernetes de comment créer et mettre à jour des instances de votre application. Une fois que vous avez créé un déploiement, le planificateur de Kubernetes (kube-scheduler) planifient les instanciations d'application sur des nœuds du cluster. Une fois les instances d’application créées, un contrôleur de déploiement Kubernetes surveille en permanence ces instances. Si le nœud hébergeant une instance tombe en panne ou est supprimé, le contrôleur de déploiement remplace l'instance par une instance située sur un autre nœud du cluster. Ceci fournit un mécanisme d'auto-réparation pour faire face aux pannes ou à la maintenance de la machine. Dans le monde de pré-orchestration, les scripts d'installation étaient souvent utilisés pour démarrer des applications, mais ils ne permettaient pas une récupération après une panne d'ordinateur.  
  
Un pod est un élément de kubernetes représentant un groupe de un ou plusieurs conteneurs(applications) et ressources partagées entre ces conteneurs. Les ressources incluent les volumes, les services et des informations sur comment faire tourner les conteneurs (comme l'image et les ports à utiliser). Un pod représente un élément logique, un ensemble d'éléments étroitement liés les uns aux autres. Les conteneurs dans un pod partagent une adresse ip et un espace de ports. Un pod est l'unité atomique sur kubernetes. Lors de la création d'un déploiement, celui-ci crée des pods contenant des conteneurs à l'intérieur. Chaque pod est lié au noeud qui le manage et y reste jusqu'à sa mort. S'il meurt, un autre pod prend sa place dans un autre noeud disponible. 

![Pods overview](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)

Un service est un élément de kubernetes représentant une manière d'accéder aux différents pods. Bien que chaque pod ait une adresse ip unique, ces ip ne sont pas exposées en dehors du cluster sans la présence d'un service. Vous verrez plus tard les différents types de services ainsi que ce que cela implique derrière.  

# 2. Installer Minikube

Commençons par l'installation, lancez votre vm linux, nous allons avoir besoin d'installer quelques outils. 
```
sudo apt-get update
sudo apt-get upgrade
```
### - Kubectl pour l'interface ligne de commande (CLI): 
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```
### - Docker qui est un moteur de conteneur:
```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu `lsb_release -cs` test"
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo apt autoremove
sudo usermod -aG docker $USER && newgrp docker
```
### - Minikube pour lancer et manipuler les clusters :
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 
chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```

### - On lance minikube:
```
minikube start --driver=docker
kubectl create namespace monnamespace
kubectl config set-context $(kubectl config current-context) --namespace=monnamespace
```

Faîtes: 
```
minikube status
```
Cela renvoie:
```
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

# 3. Lancer une application

A ce stade vous êtes prêt à déployer une application.

Actuellement rien ne tourne, vous pouvez vérifier avec:
```
kubectl get pods
kubectl get deploy
kubectl get svc
kubectl get pv
kubectl get pvc
```

Créons une image de python contenant nginx, flask et du code python pour pouvoir ensuite la lancer:  
(nginx est un serveur web et un reverse proxy)  
(Flask est un micro framework open-source de développement web en Python.)  
```
git clone https://github.com/xXHayabusaXx/miniGame.git
```
Il va falloir modifier légèrement le Dockerfile, remplace le tout par ceci:
```
FROM cren0/nginxflask:latest

WORKDIR /app
COPY ./app /app

CMD ["python", "/app/main.py"]
```
Et maintenant **build cette image** (Il faut se positionner dans le dossier contenant le Dockerfile):
```
minikube image build -t anog:latest -f ./Dockerfile .
```

**Et là on lance le pod:**
```
kubectl apply -f application.yaml
```

Il faut lancer quelques autres éléments aussi:
```
kubectl apply -f persistent-volume.yaml
kubectl apply -f service.yaml
kubectl apply -f database.yaml
kubectl apply -f jenkins.yaml
```

Refaîtes:
```
kubectl get pods
kubectl get deploy
kubectl get svc
kubectl get pv
kubectl get pvc
```

Dans ce tuto on ne va pas regarder comment faire les fichiers de configuration (les yaml), mais cela peut être intéressant d'aller y jeter un petit coup d'oeil. https://github.com/Beefr/kubernetes-training/blob/master/exercice-3-k8s-manifest.md  
Déjà vous pouvez voir avec les commandes précédentes qu'il y a un pvc, cela sert d'emplacement de stockage pour les données de l'application, car l'un des conteneurs est un système de gestion de bases de données et donc il faut pouvoir stocker les informations quelque part. D'ailleurs il faudra mettre en place la base de données aussi, on verra cela plus tard, cela permettra de voir une commande interessante de Kubernetes. L'autre pod c'est l'application, elle contient nginx, flask et le code python. Svc c'est pour service, c'est ce qui sert à connecter tous les éléments entre eux.

# 4. Voici quelques commandes intéressantes:

**Voir les logs** d'un des éléments, ici on regarde les logs du pod nginx, et vous voyez que flask tourne à l'intérieur, d'ailleurs vous pourrez voir plus tard chaque requête effectuée sur le pod...
```
kubectl logs anog-cont
```
Cela vous dit que il ne trouve pas le module menu, normal tu n'as pas mit le python dans le conteneur, tu verras cela avec la partie sur jenkins.

On peut **exposer un déploiement:**
```
kubectl expose deployment nameofmydeployment --port 80 --type ClusterIP
```
**ClusterIP** permet d'exposer des services entre pods du même cluster, mais ne permet pas d'y accéder depuis l'extérieur.
```
kubectl get svc
```
Ce qui affiche:
```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
nginx        ClusterIP   100.70.204.237   <none>        80/TCP    15m
```
Avec un **nodeport** il n'y a toujours pas d'adresse ip externe, par contre le port 80 est mappé sur le port 32593, cela signifie que nous pouvons accéder à notre pod via l'ip du node et le port 32593
```
kubectl expose deployment nameofmydeployment --port 80 --type NodePort
```
```
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx        NodePort    100.65.29.172  <none>        80:32593/TCP   8s
```
curl -s ipdunode:32593
Nous pouvons maintenant accéder au POD, via l'IP du Node, ce qui signifie qu'il est nécessaire de fournir l'adresse IP du Node, hors celle-ci peut changer et donc devoir être modifiée.  
Le service de type **load balancer** va permettre d'obtenir une adresse IP externe pour le cluster et ainsi d'accéder aux applications dans les pods, sans besoin de connaitre l'IP des nodes
```
kubectl expose deployment nameofmydeployment --port 80 --type LoadBalancer
```
Avec cela, il est possible d'accéder directement à une application via l'IP du cluster.  
NB : sur Minikube en local, aucun système de load balancing n'est en place, vous n'obtiendrez donc pas d'adresse IP externe (publique), si vous voulez en obtenir une il faut :  
- soit utiliser un fournisseur de cloud qui inclut ce service
- soit configurer un outil de load balancing en local (il est possible d'utiliser minikube tunnel, qui permet d'accèder à l'hote de Minikube https://minikube.sigs.k8s.io/docs/handbook/accessing/ )

### - Pour avoir des informations sur les nodes:
```
kubectl get nodes -o wide
```

### - Pour avoir des informations sur un élément:
```
kubectl describe pod anog-cont
```

### - Modifier le nombre de réplicas:
```
kubectl scale deployment nameofmydeployment --replicas=4
```

### - Mettre à jour l'image utilisé pour le déploiement:
```
kubectl set image deployment nameofmydeployment nginx=nginx:1.9.1 --record
```

### - Pour annuler (défaire) le déploiement et restaurer la version précédente :
```
kubectl rollout undo deployment nameofmydeployment
```

### - Supprimer un élément:
```
kubectl delete pod/svc/deployment nomdelaressource
```

### - Avoir des informations sur le cluster
```
kubectl cluster-info
```

### - Se connecter sur un pod:
```
kubectl exec -ti mariadb-anog -- bash
```
Et ça tombe bien, c'est un pod mariadb, donc si on veut on peut se connecter à mariadb:
```
mysql -u root -p
```
Ah, cela demande un mot de passe... Rentrez juste: "pwd".  
Du coup vous pouvez maintenant faire des requêtes SQL. On avait pas dit qu'il fallait mettre en place la base de données? Copiez tout ceci et collez le:
```
CREATE DATABASE data;
use data;

CREATE TABLE joueur(username varchar(20) PRIMARY KEY NOT NULL, password varchar(250));
CREATE TABLE equipage(username varchar(20) PRIMARY KEY NOT NULL, position varchar(20), piratesid varchar(250));
CREATE TABLE fruit(name varchar(20)  PRIMARY KEY NOT NULL, power varchar(20), allocated INT);
CREATE TABLE pirate(id INT PRIMARY KEY NOT NULL, name varchar(40), level INT, fruit varchar(20), qualite INT);
CREATE TABLE world(name varchar(20) PRIMARY KEY NOT NULL, stage INT);

INSERT INTO fruit VALUES('GumGum','[25,25,25,25]', 0);
INSERT INTO fruit VALUES('Fire','[25,50,0,25]', 0);
INSERT INTO fruit VALUES('Ice','[25,0,50,25]', 0);
INSERT INTO fruit VALUES('Electric','[50,0,0,50]', 0);

INSERT INTO world VALUES('Karugarner', 0);
INSERT INTO world VALUES('Cupcake', 1);
INSERT INTO world VALUES('Bonbons', 1);
INSERT INTO world VALUES('Bottle', 2);
INSERT INTO world VALUES('String', 2);
INSERT INTO world VALUES('Slip', 2);
INSERT INTO world VALUES('Diplodocus', 3);
INSERT INTO world VALUES('Fridge', 3);
INSERT INTO world VALUES('Montgolfiere', 3);
INSERT INTO world VALUES('Picmin', 4);
INSERT INTO world VALUES('PoissonRouge', 5);
INSERT INTO world VALUES('Gateau', 5);
INSERT INTO world VALUES('Bouton', 6);
INSERT INTO world VALUES('Fesse', 6);
INSERT INTO world VALUES('Shinsekai', 7);
INSERT INTO world VALUES('Marguerite', 8);
INSERT INTO world VALUES('Tulipe', 8);
INSERT INTO world VALUES('Serpent', 9);
INSERT INTO world VALUES('Singe', 9);
INSERT INTO world VALUES('Chien', 9);
INSERT INTO world VALUES('Dragon', 9);
INSERT INTO world VALUES('Chaise', 10);
INSERT INTO world VALUES('portefeuille', 10);
INSERT INTO world VALUES('Table', 10);
INSERT INTO world VALUES('Escalier', 10);
INSERT INTO world VALUES('Fourchette', 10);
INSERT INTO world VALUES('Voiture', 11);
INSERT INTO world VALUES('Velo', 11);
INSERT INTO world VALUES('Train', 11);
INSERT INTO world VALUES('Avion', 11);
INSERT INTO world VALUES('Etoile', 12);


```
# 5. Créer une pipeline avec Jenkins

Jenkins permet d'automatiser un certain nombre de choses, notamment les builds, au travers de pipelines. Ici on ne va pas le faire build car j'ai préféré créer une image contenant tout ce qu'il faut sauf le code, donc pas besoin de refaire d'autres images. En revanche il manque le code, et c'est là que Jenkins intervient dans notre cas. On va le configurer pour qu'il clone le code python dans le volume auquel il est connecté, et cela permettra à notre conteneur nginx d'y accéder. L'intérêt c'est que si on modifie le code il suffit de relancer la pipeline, qui retélécharge le code et pas besoin de recréer une image!   
L'intérêt aussi c'est que si on a potentiellement plusieurs applications python qui fonctionnent (plus ou moins) de la même manière alors on a besoin que d'une seule image et seulement d'adapter le backend.   
  
Passons à la configuration:  
On doit d'abord se connecter à Jenkins.  
### Pour le port:
```
kubectl get svc
```
```
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
jenkins-service        NodePort    10.110.175.157   <none>        8080:30037/TCP,50000:32754/TCP   26h
```
C'est le 30037.
### Pour l'ip:
```
kubectl get nodes -o wide
```
```
NAME       STATUS   ROLES                  AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
minikube   Ready    control-plane,master   11d   v1.23.3   192.168.49.2   <none>        Ubuntu 20.04.2 LTS   5.13.0-1031-azure   docker://20.10.12
```
L'ip est 192.168.49.2.  
Donc dans votre navigateur, vous rentrez 192.168.49.2:30037, et cela vous affiche la page d'accueil de Jenkins (après un chargement d'1 ou 2min).  
Et là ça vous demande un mot de passe, pas de panique, il suffit de rentrer dans le conteneur jenkins comme indiqué sur la page:
```
kubectl exec -ti jenkins-cont -- bash
cat var/jenkins_home/secrets/ini.....
```
Cela renvoie une chaîne de caractères que vous rentrez.  
Ensuite il faut configurer votre compte admin de Jenkins, vous mettez "admin" partout. Mettez bien "admin" sinon faudra adapter certaines choses...  
Vous le faîtes télécharger les plugins recommandés avec le bouton de gauche.  
Une fois rentré sur le dashboard vous créez un nouveau job, vous l'appelez "python-pipeline", vous sélectionnez "pipeline", puis "OK".  
Dans "Build Triggers", il faut cocher "Trigger builds remotely", vous écrivez "copyCode".  
Et dans script, vous mettez:
```
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git url: 'https://github.com/Beefr/OnePiece.git', branch: 'tutorial'

            }
        }
    }
}
```
Plus qu'à sauvegarder.

Donc on récapitule, on a une image avec nginx et flask mais c'est une coquille vide, il n'y a pas le code dedans, donc il faut lancer la pipeline pour faire fonctionner l'application. 
```
curl -I http://admin:admin@192.168.49.2:30037/job/python-pipeline/build?token=copyCode
kubectl delete pod anog-cont
```
Regardez dans Jenkins, vous pouvez voir qu'il y a eu un "build", c'est juste un git clone du code python. Il a mit ce code dans son système de fichiers et comme on y a connecté un volume, volume qui est connecté à nginx, lorsque nous relancerons le pod, le conteneur nginx aura accès au code. Dès que le git clone est fini, relancez le pod:
```
kubectl apply -f application.yaml
``` 

# 6. Tester l'application

Tu vas pouvoir tester que l'application tourne vraiment maintenant!  
Tu as besoin de trouver l'ip et le port et mettre dans le navigateur de la vm: ip:port  
Tu vas arriver sur la page de connexion de l'application, ce qui est drôle c'est que si tu vas sur ip:port/bdd tu verras que ce que tu rentres dans l'application apparaît ensuite dans la base de données

### Pour le port:
```
kubectl get svc
```
```
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
anog                   NodePort    10.106.206.133   <none>        8080:30036/TCP,3306:30944/TCP   6d4h
mariadb-anog-service   ClusterIP   10.104.239.146   <none>        3306/TCP
```
C'est le 30036.

### Pour l'ip:
Ce coup là on change de commande:
```
minikube ip
```
```
NAME       STATUS   ROLES                  AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
minikube   Ready    control-plane,master   11d   v1.23.3   192.168.49.2   <none>        Ubuntu 20.04.2 LTS   5.13.0-1031-azure   docker://20.10.12
```
C'est le 192.168.49.2



# 6. Fin du tutoriel:

Merci d'avoir suivi ce tutoriel.  
Tu peux arrêter Minikube quand tu as fini de jouer: 
```
minikube stop
```

Il y a encore énormément de choses à apprendre sur kubernetes, tout ceci ne fait qu'effleurer la surface.


