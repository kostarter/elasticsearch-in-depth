## Introduction à la Recherche

La documentation sur les query string :<br/>
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html

#### 1. Recherche avec une requête URI

##### Matcher tous les documents

```
GET /products/_search?q=*
```

Vérifier qu'il y a bien 1000 dans "hits -> total".

##### Matcher les documents contenant le term `Lobster`

```
GET /products/_search?q=name:Lobster
```

![alt text](https://i.ibb.co/0Zcp4Bf/050-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

Les documents sont triés par pertinence.

##### Matcher les documents contenant le tag `Meat`

```
GET /products/_search?q=tags:Meat
```

![alt text](https://i.ibb.co/xgYGdN0/051-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Matcher les documents contenant le tag `Meat` _et_ nom `Tuna`

```
GET /products/_search?q=tags:Meat AND name:Tuna
```

![alt text](https://i.ibb.co/6Bv8XPK/052-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 2. Introduction au Query DSL

Il existe deux types de requêtes :
* La leaf query cherche une valeur donnée dans un champ particulier, comme les queries `match`, `term` ou `range`.
* La compound query se compose de plusieurs leaf ou compouned queries, comme `bool`.

![alt text](https://i.ibb.co/z6zVCHD/01-Screenshot-2021-03-18-Elasticsearch-Answers-The-Complete-Guide-to-Elasticsearch.png "Learning Elasticsearch")

##### Matcher tous les documents

```
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}
```

#### 3. Comprendre la pertinence du score

- Exécutez la requête suivante : 

```
GET /products/_search
{
  "query": {
    "term": {
      "name": "lobster"
    }
  }
}
```

- Relevez la valeur du champs "Max Score".
- Relevez le Nombre de Hits.
- Relevez le score de chaque Hit.
- Que constatez-vous ?

Explorons les détails du calcul du relevance score de plus près :

```
GET /products/_search
{
  "explain": true,
  "query": {
    "term": {
      "name": "lobster"
    }
  }
}
```

![alt text](https://i.ibb.co/dBR1Lnx/053-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

- Que constatez-vous ?
- Comment est fait ce calcul ?

<sup>Article très bien fait sur le fonctionnement du scoring dans Elasticsearch :</sup><br>
https://www.compose.com/articles/how-scoring-works-in-elasticsearch/

#### 4. Debugger les résultats inantendus d'une recherche

_Déprécié dans la version 7_ <br>
<sub>(Le but était d'expliciter l'erreur renvoyée)</sub>

```
GET /products/_doc/19/_explain
{
  "query": {
    "term": {
      "name": "lobster"
    }
  }
}
```

![alt text](https://i.ibb.co/sR7rNc2/054-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 5. Full text queries Vs. term level queries

##### Term level queries ne sont pas analysées

```
GET /products/_search
{
  "query": {
    "term": {
      "name": "lobster"
    }
  }
}
```

![alt text](https://i.ibb.co/R2dNXLZ/055-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

```
GET /products/_search
{
  "query": {
    "term": {
      "name": "Lobster"
    }
  }
}
```

![alt text](https://i.ibb.co/hXV4fM1/056-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Full-text queries sont analysées

```
GET /products/_search
{
  "query": {
    "match": {
      "name": "Lobster"
    }
  }
}
```

![alt text](https://i.ibb.co/RPpGh8M/057-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")
