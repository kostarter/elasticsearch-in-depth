#### 1. Inspecter le cluster

##### Vérifier l'état de santé du cluster

```
GET /_cluster/health
```

La première partie de l'URL "_cluster" est le nom de l'API, et "health" est le nom du service appelé.

Un cluster peut avoir plusieurs status : 
* green : Tout va bien !
* yellow : Warning.
* red : Il y a un problème grave et bloquant.

![alt text](https://i.ibb.co/GCnh5xd/001-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

Nous remarquons que l'état du cluster est à "green", tout va pour le mieux. :thumbsup:

##### Lister les index du cluster

```
GET /_cat/indices?v
```

![alt text](https://i.ibb.co/SwtBfQd/002-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

D'ou viennent tous ces index ?<br/>
Pas de panique, vous n'avez fait aucune manipulation, ce sont des index créés par Elasticsearch pour le monitoring des performances applicatives et Kibana pour les mét-données.

#### 2. Les noeuds et leurs roles

##### Lister les noeuds du cluster

```
GET /_cat/nodes?v
```

![alt text](https://i.ibb.co/KF5rmBT/003-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

##### Quizz

- Quelles colonnes sont présentes ?
- Combien de noeuds comptez-vous ?
- Quelle est l'adresse IP du Noeud ?

#### 3. Envoi de requêtes avec cURL

##### Vérifier l'état de santé du cluster

```
curl -XGET localhost:9200/_cluster/health?pretty
```
Pour obtenir l'équivalent de cette requête en cURL, cliquer sur la clé à molettes en face de la requête et choisir <ins>Copy as cURL</ins>. La requête en cURL est dans le presse-papier.

![alt text](https://i.ibb.co/20rGfnT/004-Screenshot-from-2021-03-16-14-36-18.png "Learning Elasticsearch")

##### Tester une requête de recherche via cURL

La requête sous Kibana :
```
GET /.kibana/_search
{
  "query": {
    "match_all": {}
  }
}
```

A transoformer en cURL :
```
A compléter...
```

##### Quizz

- Sur quelle syntaxe est basée CURL ?
- Quels sont les index requêtés ici ?
- Que retourne la dernière requête ?

#### 4. Sharding and scalabilité

![alt text](https://i.ibb.co/S6tVg1C/005-Screenshot-from-2021-03-16-14-45-00.png "Learning Elasticsearch")

![alt text](https://i.ibb.co/m50WYWd/006-Screenshot-from-2021-03-16-14-44-05.png "Learning Elasticsearch")

##### Lister les noeuds du cluster

```
GET /_cat/indices?v
```

![alt text](https://i.ibb.co/cht4SX4/007-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

##### Quizz

- Combien d'index comptez vous ?
- Listez les index que vous voyez.
- A quoi correspondent-ils ? 
- Pouvez-vous différencier des types d'index différents ?
- En quoi l'index est-il lié au Sharding et à la Scalabilité ?

**Il n'y a pas de formule magique pour déterminer le nombre de shards!!**<br/>
Cela dépend de plusieurs paramètres : Le nombre des noeuds, leurs capacités, le nombre d'index, leurs tailles futures, le nombre de requêtes, etc.

#### 5. Comprendre la réplication

![alt text](https://i.ibb.co/mFwPHXj/008-Screenshot-from-2021-03-16-15-12-14.png "Learning Elasticsearch")

Le facteur de réplication est porté par l'index, il doit être spécifié lors de la création de celui-ci.
Par défaut le facteur de réplication est de 1, c'est à dire qu'on aura une seule copie, donc un shard primaire et un secondaire.

##### Creation d'un nouvelle index

```
PUT /pages
```

##### Vérifier l'état de santé du cluster

```
GET /_cluster/health
```
Pourquoi l'état du cluster a viré au jaune ? :scream:

##### Lister les noeuds du cluster

```
GET /_cat/indices?v
```

Que voyez-vous de particulier ?

**Explication :** Lors de la création de l'index "pages" nous n'avons pas spécifié de facteur de réplication. Donc pour Elasticsearch c'est la valeur par défaut 1. Or nous n'avons qu'un seul noeud et Elasticsearch ne peut pas stocker le shard primaire et secondaire sur le même noeud. Logique.

##### Lister les shards du cluster

```
GET /_cat/shards?v
```

##### Quizz

- Que renvoie la dernière requête ?
- Que voyez de particulier concernant les shards du nouvelle index "pages" ?
- Comparer aux shards des autres index.
- Que faire pour éviter cela ?

**Encore une fois : Il n'y pas de formule magique pour déterminer le facteur de réplication !!**<br/>
Cela dépend du nombre de shards mais surtout de la sensibilité des données.
