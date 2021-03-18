#### 1. Introduction aux full text queries

##### Importer un nouvel ensemble de données

```shell
cd /path/to/data/file/directory
```

```shell
curl -H "Content-Type: application/x-ndjson" -XPOST 'http://localhost:9200/recipe/_bulk?pretty' --data-binary "@test-data.json"
```

##### Vérifier le mapping

```
GET /recipe/_mapping
```

#### 2. Matching flexible avec requête `match`

##### Requête `match`

```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": "Recipes with pasta or spaghetti"
    }
  }
}
```

Le premier élément possède deux fois le mot spaghetti et une fois le mot pasta dans le titre. Ce qui lui confère le plus haut score de pertinence.<br/>
Pour ce type de requête l'opérateur par défaut est le OR, raison pour laquelle il y a des recettes avec juste un des mots dans le titre.

![alt text](https://i.ibb.co/GMpFt3L/077-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Spécifier un opérateur booléen

Pour spécifier de manière explicite l'opérateur logique du `match`, utiliser le mot clè `operator`.

Comme tous les mots de la requêtes doivent figurer dans le titre, aucun document ne correspond à la recherche :

```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Recipes with pasta or spaghetti",
        "operator": "and"
      }
    }
  }
}
```

![alt text](https://i.ibb.co/3z0Sxqp/078-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

Supprimer tous les mots qui nuisent à la pertinence de la recherche : 

```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "pasta spaghetti",
        "operator": "and"
      }
    }
  }
}
```

![alt text](https://i.ibb.co/2Zqn2Yw/080-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 3. Match phrase

##### L'ordre des termes compte

```
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "spaghetti puttanesca"
    }
  }
}
```

Même résultat que la recherche précédente.

```
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "puttanesca spaghetti"
    }
  }
}
```

![alt text](https://i.ibb.co/wQbf2zG/081-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 4. Recherche dans différents champs

```
GET /recipe/_search
{
  "query": {
    "multi_match": {
      "query": "pasta",
      "fields": [ "title", "description" ]
    }
  }
}
```

![alt text](https://i.ibb.co/VSQ7kVV/082-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 5. Exercices

Y'a-t-il une recette de pâtes avec du parmesan et/ou des épinards ?

```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": "Pasta with parmesan and spinach"
    }
  }
}
```

![alt text](https://i.ibb.co/2y2VnZT/083-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

On veut préparer des pates carbonara.

```
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "pasta carbonara"
    }
  }
}
```
Aucun résultat.

```
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "carbonara pasta"
    }
  }
}
```

![alt text](https://i.ibb.co/mh6ZHc4/084-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

`#Yummy`

Peut être des pâtes au pesto ?

```
GET /recipe/_search
{
  "query": {
    "multi_match": {
      "query": "pasta pesto",
      "fields": [ "title", "description" ]
    }
  }
}
```

![alt text](https://i.ibb.co/8r66JT0/085-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")
