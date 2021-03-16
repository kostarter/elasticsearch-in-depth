#### 1. Création et Suppression d'index

###### Suppression d'un index

```
DELETE /pages
```

Remarqez que l'état du cluster est redevenu vert. Il n'y a plus aucun shard non assigné.

###### Quizz
- Ecrivez une requête pour vérifier le contenu de l'index "pages"
- Quel est le résultat de cette requête ?
- Peut-on récupérer les données contenues précédemment dans l'index ?

###### Creation d'un index avec paramètres

```
PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}
```

Malgré des paramètres douteux de création d'index, cela semble fonctionner :
![alt text](https://i.ibb.co/ngn1wH0/009-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

###### Quizz: 
- Que pensez-vous de la réplication dans le contexte actuel ?
- A-t-elle un sens ? Pourquoi ?

#### 2. Indexer des documents

###### Indexer un document un identifiant auto-generé

```
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}
```

En retour Elasticsearch renvoie l'identifiant généré :

![alt text](https://i.ibb.co/5B0Xmt0/010-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

###### Indexer un document en spécifiant l'identifiant

```
PUT /products/_doc/100
{
  "name": "Toaster",
  "price": 49,
  "in_stock": 4
}
```

###### Quizz: 

- Listez les avantages de chaque méthode.

#### 3. Rechercher des documents par identifiant

```
GET /products/_doc/100
```

Nous retrouvons notre produit avec le nom de l'index et le type de l'objet retrouvé :

![alt text](https://i.ibb.co/T8mXBxC/011-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

#### 4. Modifier des documents

###### Modifier un champ existant

Maintenant que nous avons quelques produits dans notre index "products", nous allons modifié les champs de certains produits.<br/>
Modifier le stock du produit d'on l'identifiant est 100 :

```
POST /products/_update/100
{
  "doc": {
    "in_stock": 3
  }
}
```

Que remarquez-vous dans la réponse ?

![alt text](https://i.ibb.co/4FxdbrR/012-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

TODO : Compléter la réponse.

**En effet, les documents dans Elasticsearch sont immutables !**

Donc ce que fait l'update c'est de :
* Retrouver le document à modifier.
* modifier les champs nécessaires.
* Stocker les données dans un nouveau document en incrémentant le numéro de version.
* Réplique la modication sur tous les shards.

###### Ajouter un nouveau champ

_Oui, la syntaxe de la requête est la même. Merci le semi-structuré. ;-)_

```
POST /products/_update/100
{
  "doc": {
    "tags": ["electronics"]
  }
}
```

#### 5. Modifer par script

Supposons qu'un produit a été vendu, il faut donc réduire la valeur de `in_stock` d'une unité sans en connaitre la valeur actuelle.
Nous allons donc envoyer une requête de type POST à l'API `_update` en spécifiant le document à mettre à jour et un objet de type `script` où sera spécifié notre logique.
```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}
```

La variable`ctx` représente le contexte d'exécution sous forme d'objet, il contient des variables locales représentant les champs de l'objet à modifier.

Le numéro de version a encore changé, c'est plutôt une bonne nouvelle :

![alt text](https://i.ibb.co/f41QKx3/013-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

Ca marche !

###### Assigner une valeur arbitraire à `in_stock`

Nous avons reçu du stock :
```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock = 10"
  }
}
```

###### Utiliser une valeur passé en paramètre

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}
```

Vérifions le résulat :

![alt text](https://i.ibb.co/RPDT7Tv/014-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

Le résultat correspond à nos attentes !

Vous pouvez tester avec un nom de paramètres qui n'existe pas pour voir comment Elasticsearch réagit.<br/>
L'eereur est très explicite :
```
      "reason" : "runtime error",
      "script_stack" : [
        "ctx._source.in_stock -= params.quantite",
        "                              ^---- HERE"
```
###### Utiliser l'opérateur `noop` sur critère conditionnel

L'opérateur `noop` permet de dire à Elasticsearch d'ignorer le document à modifier. Ici nous allons lui demander d'ignorer l'opération de mise à jour dans le ou stock est à 0 en assignant la valeur `noop` au champs `op` de la variable qui reprèsente le contexte.<br/>
Avant de lancer la prochaine requête n'oubliez pas de mettre le stock à un pour ce produit.

```
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock == 0) {
        ctx.op = 'noop';
      }
      
      ctx._source.in_stock--;
    """
  }
}
```

Première exécution :

![alt text](https://i.ibb.co/kHz3kCp/015-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

Seconde exécution :

![alt text](https://i.ibb.co/7r1VjHd/016-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

Quelle différence remarquez-vous ?

Autre question : Quelle différence entre les deux requêtes ci-dessous :

![alt text](https://i.ibb.co/L1KHxW5/016-1-Screenshot-from-2021-03-16-17-09-07.png "Learning Elasticsearch")

TODO : Compléter la réponse.

###### Mettre à jour un champ après test conditionnel

Remettons le stock à 10 pour le document à modifier et lancer la requête suivante :
```
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock > 0) {
        ctx._source.in_stock--;
      }
    """
  }
}
```

Vérifier que le champ `result` est bien à `updated`.

###### Changement d'opération

Au lieu de mettre à jour la valeur du champ `stock`, nous voulons dans certains cas supprimer le document. Pour cela il faut changer l'opération qui est en train d'être exécutée dans le script.<br/>
Par exemple supprimer un produit dont le stock est inférieur 0. N'oubliez pas de mettre stock à -1 par exemple.
```
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock < 0) {
        ctx.op = 'delete';
      }
      
      ctx._source.in_stock--;
    """
  }
}
```

Que remarquez-vous dans le résultat de la requête ?

![alt text](https://i.ibb.co/vPm08PL/017-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

#### 6. Upserts

Il existe une autre façon de modifier des documents en utilisant la méthode `upsert`. Cela signifie que le mise à jour est conditionnée par l'existence du document, si il n'existe pas dans l'index il sera créé.<br>
Ici le document sera créé sans que le script ne soit exécuté :
```
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock++"
  },
  "upsert": {
    "name": "Blender",
    "price": 399,
    "in_stock": 5
  }
}
```

Première exécution :
```
  "result" : "created"
```
Deuxième exécution, la partie script sera exécutée, le document existe déjà et le résultat est `updated` :
```
  "result" : "updated",
```

Maintenant, modifier le prix et rejouer la même requête. Vérifier le résultat et l'état du document.
```
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock++"
  },
  "upsert": {
    "name": "Blender",
    "price": 395,
    "in_stock": 5
  }
}
```

![alt text](https://i.ibb.co/5c1n6cB/018-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

#### 7. Replacing documents
C'est la même requête que pour créer un nouveau document :
```
PUT /products/_doc/100
{
  "name": "Television",
  "price": 79,
  "in_stock": 4
}
```

#### 8. Deleting documents
Pour cela utiliser le verbe HTTP Delete :
```
DELETE /products/_doc/101
```

Première exécution :
```
  "result" : "deleted"
```
Deuxième exécution :
```
  "result" : "not_found",
```
Il est également possible de supprimer des documents en utilisant des critères dans un `match`. Nous verrrons cela plus tard.

#### 9. Gestion des accès concurrents

La gestion des accès concurrents est essentiel pour éviter qu'une ancienne version d'un document n'en ecrase une plus récente par inadvertance. Comme Elasticsearch est distribué et dépend des accés réseaux, c'est un scénario qui n'est pas à exclure.<br/>

Ici un exemple de mise à jour du stock pour notifier une vente en reduisant le stock d'une unité :

![alt text](https://i.ibb.co/H2h3F0C/019-Screenshot-from-2021-03-16-19-00-10.png "Learning Elasticsearch")

L'achat du visiteur B devrait donner un stock de 4 mais dans ce cas cela donne une incohérence sans que personne ne s'en rende compte ! Cela peut produire la vente de produits qui ne sont plus en stock.<br/>
Nous voulons que le deuxième update plante si le document a entre temps été modifié.

Pour cela il y a 2 options :
* Utiliser le numéro de version du document lors de l'update :

![alt text](https://i.ibb.co/dkqjzFk/020-Screenshot-from-2021-03-16-19-02-46.png "Learning Elasticsearch")

Mais cette approche a été deprécié car elle ne couvre pas tous les cas.

* Utiliser le primary term et le numéro de séquence du document :

![alt text](https://i.ibb.co/xJy7LV4/021-Screenshot-from-2021-03-16-19-04-50.png "Learning Elasticsearch")

###### Retrouver un document et vérifier que le primary term et le sequence number sont présents
```
GET /products/_doc/100
```

###### Modifier le champ `in_stock` seulement si le document n'a pas été mis à jour entre temps
Utiliser les valeurs retrouvées lors du précendent GET :
```
POST /products/_update/100?if_primary_term=xxx&if_seq_no=xxx
{
  "doc": {
    "in_stock": 123
  }
}
```

Ce que cela veut dire c'est que le document ne sera mis à jour que si les primary term et sequence number du document correspondent a ceux spécifiées dans la requête. 

* En cas de succès :

![alt text](https://i.ibb.co/mRKfhVw/022-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

Vérifier que les primary term et sequence number ne sont plus les mêmes.

* En cas d'échec :

Essayer de modifier de nouveau en gardant les anciens primary term et sequence number.

![alt text](https://i.ibb.co/zbMCXtR/023-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

C'est la première fois qu'on se félicitera d'avoir une erreur. ;)

#### 10. Modfier avec une requête

Similaire à une requête UPDATE avec une clause WHERE en relationnel. 

###### Modifier des documents correspondant aux critères d'une requête

Pour cela il faut appeler l'API `_update_by_query`.<br/>
Il est possible de remplacer la query `match_all` par n'importe quelle autre query.

Ici tous les produits verront leur stock diminué d'une unité :
```
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}
```
Dans la réponse nous avons le nombre de documents mis à jour :

![alt text](https://i.ibb.co/S7ffLJk/024-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")


Pour vérifier les résultats, appeler l'API `_search` sans aucun critère pour avoir tous les documents de l'index prodcuts :
```
GET /products/_search
```

###### Ignorer les conflicts de versions

L'attribut `conflicts` doit être rajouter aux paramètres de la requête, ou `?conflicts=proceed`.<br/>
Cela permet de remonter le nombre de conflits de version sans que la requête ne plante.

```
POST /products/_update_by_query
{
  "conflicts": "proceed",
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}
```

#### 11. Supprimer avec une requête

Similaire à une requête DELETE avec une clause WHERE en relationnel. 

###### Supprimer des documents correspondant aux critères d'une requête

Suppression de tous les produits stockés dans l'index, la requête est très similaire à celle du `_update_by_query` :
```
POST /products/_delete_by_query
{
  "query": {
    "match_all": { }
  }
}
```

![alt text](https://i.ibb.co/mcHVgC2/025-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

###### Ignorer les conflicts de versions

Comme pour le `update_by_query`, l'attribut `conflicts` doit être rajouter aux paramètres de la requête, ou `?conflicts=proceed`.<br/>
Cela permet de remonter le nombre de conflits de version sans que la requête ne plante.

```
POST /products/_delete_by_query
{
  "conflicts": "proceed",
  "query": {
    "match_all": { }
  }
}
```

#### 12. Batch processing

L'objectif est de réaliser les opérations vues précédemment (indexer, modifier, supprimer) sur un lot de documents avec une seule requête.<br/>
Pour cela nous allons utiliser l'API Bulk. 

https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html

Cette API accepte des lignes au format Json séparées par des \n ou de \r\n, le format NDJSON du fichier étant le suivant :
```
action_and_meta_data\n
optional_source\n
action_and_meta_data\n
optional_source\n
...

```

###### Indexer des documents
Le nom de l'index n'est pas spécifié dans le path de la requête, mais dans les lignes `action_and_meta_data` :
```
POST /_bulk
{ "index": { "_index": "products", "_id": 200 } }
{ "name": "Espresso Machine", "price": 199, "in_stock": 5 }
{ "create": { "_index": "products", "_id": 201 } }
{ "name": "Milk Frother", "price": 149, "in_stock": 14 }
```
Ici le nom de l'action est `index`. Il existe aussi une autre action pour indexer des documents : `create`.<br>
La différence est que le `create` plante si un document existe déjà avec le même identifiant.

![alt text](https://i.ibb.co/hymTQwQ/026-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

###### Modifier et supprimer des documents
Le type d'opération est spécifié dans les lignes `action_and_meta_data` :
```
POST /_bulk
{ "update": { "_index": "products", "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_index": "products", "_id": 200 } }
```

###### Spécifier le nom de l'index dans le path de la requête
Cela peut être pratique si on veut charger le même fichier sur des index avec des noms différents :
```
POST /products/_bulk
{ "update": { "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_id": 200 } }
```

###### Vérifier le résultat du bulk
```
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}
```

#### 13. Importer des données avec cURL

Rappel :
* Le Content-Type doit être application/**x-ndjson**.
* Chaque ligne doit se terminer par un \n ou \r\n, dernière ligne incluse !
* Une action qui plante n'affectera pas les autres actions.
* L'API Bulk renvoie un rapport détaillé pour toutes les actions.
* L'API Bulk est plus efficace que l'envoi de plusieurs actions individuelles (réduction de traffic réseau).

###### Naviguer vers le dossier ou se trouve le fichier bulk

```
$ cd /path/to/data/file/directory
```

A noter que le nom de l'index n'est pas défini dans le fichier bulk, il faudra le spécifier dans le path de la requête HTTP.

###### Importer les données dans le cluster local

```
$ curl -H "Content-Type: application/x-ndjson" -XPOST http://localhost:9200/products/_bulk --data-binary "@products-bulk.json"
```

Il y a 1000 documents dans le fichier json. Vérifions que tout a été chargé dans l'index.

```
GET /products/_count
```

![alt text](https://i.ibb.co/Jnv6G5q/027-Screenshot-2021-03-16-Elastic-Kibana.png "Learning Elasticsearch")

###### Importer les données dans le cluster sur le cloud
```
$ curl -H "Content-Type: application/x-ndjson" -XPOST -u username:password https://elastic-cloud-endpoint.com:9243/product/_bulk --data-binary "@products-bulk.json"
```
