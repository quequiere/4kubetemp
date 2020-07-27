# 4Kube

Par LARIBIERE Bruno - 298261



## Installation

Voir chapitre **flux**

## Vérification

Après le déploiement automatique avec flux la node sur laquelle est déployée le front devrait exposer le site sur le port 30000.

Dans un premier temps, les logs de prestashop doivent nous confirmer la bonne prise en compte des credentials:

![](.\img\bootproof.jpg)



Nous pouvons donc ensuite aller sur l'ip de notre node (192.168.127.146:30000):

![](.\img\getnode.jpg)



Et pouvons constater que prestashop est bien en ligne avec nos credentials

![](.\img\datapannel.jpg)



Nous confirmons donc que le déploiement s'est bien effectué avec succès.

## Détails manifestes

Les manifestes se trouvent dans **./releases/prestans** 

#### 1-config.yaml

Configmap contenant des data non confidentielles comme le nom et prénom à utiliser pour le profil administrateur prestashop

#### 1-secrets.yaml

Contient l'ensemble des données privées, comme les passwords prestashop ou bien ceux de la database

#### 2-persistance.yaml

Décrit la méthode de stockage sur le host afin de pouvoir stocker ensuite les données de la database

#### 3-mariadb.yaml

Déploiement d'une DB mariadb. Nous utilisons les secrets en tant que variable d'environnements

#### 4-mariasql-svc.yaml

Expose la mariadb uniquement sur le cluster afin qu'il ne soit pas accessible depuis l'extérieur

#### 4-prestashop.yaml

Déploie prestashop. Le PRESTASHOP_HOST permet que le site fonctionne correctement avec une ip interne.

#### 5-prestashop-svc.yaml

Expose prestashop sur le nœud. 

## FluxCD 

FluxCD va s'occuper du déploiement de nos manifestes. Pour cela nous admettons par avance que le client fluxctl  a été installé sur la machine cliente.

#### Installation de flux

```
kubectl create ns flux

helm upgrade -i flux fluxcd/flux --set git.url=git@github.com:quequiere/4kubetemp --set git.defaultRef=master --namespace flux
```

Ceci nous permet d'installation flux dans le namespace flux, et de lui dire de rester à l'écoute du repository github **quequiere/4kubetemp**

### Synchronisation git

A cette étape nous devons rajouter la clef RSA qui permettra à flux d'avoir assez d'accès sur notre repository.

Plusieurs choix pour obtenir la clef

##### Regarder les logs à l'installation de flux

![](.\img\getrsakey.jpg)

##### Utiliser la commande suivante

```
fluxctl identity --k8s-fwd-ns flux
```



Une fois la clef obtenue on l'ajoute dans les accès de notre repository:

![](.\img\Gitkey.jpg)

Il ne reste plus qu'à attendre la synchronisation. Il est possible de la forcer avec la commande suivante:

```
fluxctl sync --k8s-fwd-ns flux
```

Pour suivre le bon déroulement de la synchronisation nous pouvons regarder les logs de cette manière:

```
kubectl -n flux logs -f deployment/flux
```



### Fin du déploiement

Après la synchronisation l'ensemble des fichiers vont être poussés depuis **./releases/prestans**

Le namespace sera aussi créé automatiquement vu qu'il est présent dans **./namespaces**

## Bonus

Quelques difficulté d'intégrations on fait que je n'ai pas réussi à introduire mon package helm dans le déploiement automatique avec flux.

Cependant le package helm est bien fonctionnel et peut être trouvé dans **./BONUS.HELM**

**Pour l'installation utiliser la commande**

```
helm install 4kube .
```

**Pour la suppression**

```
helm delete 4kube
```

Rien de particulier à signaler si ce n'est que bien entendu l'ensemble des valeurs est défini dans le values.yaml



## Mise à jour du cluster

Dans mon cas mon cluster a été fait mis en place sur un nœud unique. Mes screens se baseront donc là dessus.

#### Vérification de la version

Nous vérifions que la versions de kubeadm est bien en 1.16

![](.\img\version.jpg)



#### Installation de kubeadm 1.17.4

Nous installons la nouvelle version de kubeadm avec la commande suivante:

```
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.17.4-00 && \
apt-mark hold kubeadm
```

Et nous confirmons que la version a été correctement installée (1.17.4)

![](.\img\confirmversion.jpg)



#### Update du cluster

Drainage de nœud que nous voulons mettre à jour afin d'éviter l'interruption de service. 

(Non exécuté chez moi car un seul noeud)

```
kubectl drain kube4 --ignore-daemonsets
```

On prépare la procédure de migration:

```
sudo kubeadm upgrade plan
```

![](.\img\confirmv.jpg)

Tout semble ok et pret pour la migration que nous confirmons avec la commande suivante:

```
sudo kubeadm upgrade apply v1.17.4
```

![](.\img\success.jpg)

Ce message me confirme que la mise à jour s'est déroulée avec succès.