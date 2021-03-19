#### 1. Spécifier le format du résultat

##### Retourner les résultats en YAML

```
GET /recipe?format=yaml
```

##### Retourner les résultats en JSON formatté

```
GET /recipe/_search?pretty
{
    "query": {
      "match": { "title": "vegan" }
    }
}
```
Sous Kibana le résultat est mis en forme par défaut, par contre en ligne de commande avec cURL l'utilisation du paramètre `pretty` n'est pas superflu.<br/>
Résulat sans le paramètre `pretty` en ligne de commande :

![alt text](https://i.ibb.co/sgP2NBH/01-Screenshot-from-2021-03-19-11-46-10-copy.png "Learning Elasticsearch")

Joli pavé.

#### 2. Spécifier la taille du résultat

##### En utilisant un paramètre de la requête

```
GET /recipe/_search?size=2
{
  "_source": false,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

Même si le nombre de résultat répondant à la recherche est de 9, seuls les 2 éléments ayant le plus haut score de pertinance sont affichés.

![alt text](https://i.ibb.co/8r4f31N/02-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

##### En utilisant le paramètre dans le corps de la requête

```
GET /recipe/_search
{
  "_source": false,
  "size": 2,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

#### 3. Filtrage de la Source

Par défaut tout le contenu est retourné. Il est possible de réduire la quantité d'information retournée dans source par soucis de clarté ou de réduction du flux sur le réseau, spécialement quand le résultat est très volumineux et avec beaucoup de longs textes.<br/>

##### Exclure complétement le champ `_source`
Cela peut être pertinent quand les seules informations ciblées sont les identifiants des documents.
```
GET /recipe/_search
{
  "_source": false,
  "query": {
    "match": { "title": "vegan" }
  }
}
```

Sans `_source` :

![alt text](https://i.ibb.co/0KmpCXK/03-1-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

Avec `_source` :

![alt text](https://i.ibb.co/K9cYGYf/03-2-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

##### Retourner le champ `created` uniquement

```
GET /recipe/_search
{
  "_source": "created",
  "query": {
    "match": { "title": "vegan" }
  }
}
```

![alt text](https://i.ibb.co/jRVmXRF/04-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

##### Retrouner des clés spécifiques d'un objet
Rajouter le titre de la recette pour une meilleure lisibilité.
```
GET /recipe/_search
{
  "_source": ["ingredients.name", "title"]
  "query": {
    "match": { "title": "cheese" }
  }
}
```

![alt text](https://i.ibb.co/2STM3fd/05-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

##### Retourner toutes les clés d'un objet

```
GET /recipe/_search
{
  "_source": "ingredients.*",
  "query": {
    "match": { "title": "cheese" }
  }
}
```

![alt text](https://i.ibb.co/D1L5Phf/06-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

##### Retourner l'objet `ingredients` avec toutes les clés et le champ `servings`

```
GET /recipe/_search
{
  "_source": [ "ingredients.*", "servings", "title" ],
  "query": {
    "match": { "title": "cheese" }
  }
}
```

![alt text](https://i.ibb.co/wKNtsn6/07-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

##### Inclure toutes les clés de l'objet `ingredients` object', excepté la clé `name`

```
GET /recipe/_search
{
  "_source": {
    "includes": [ "ingredients.*", "title"],
    "excludes": "ingredients.name"
  },
  "query": {
    "match": { "title": "cheese" }
  }
}
```

![alt text](https://i.ibb.co/1f4dh53/08-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

#### 4. Spécifier un offset

##### Spécifier un offset avec le paramètre `from`

```
GET /recipe/_search
{
  "_source": false,
  "size": 2,
  "from": 2,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

![alt text](https://i.ibb.co/42WZr3t/09-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

#### 5. Sorting results

##### Tri ascendant (implicite)

```
GET /recipe/_search
{
  "_source": false,
  "query": {
    "match_all": {}
  },
  "sort": [
    "preparation_time_minutes"
  ]
}
```

Malgrè l'exclusion complète de `_source`, le critère de tri est visible dans l'élément `sort`.

![alt text](https://i.ibb.co/S6qJYPg/10-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

##### Tri descendant (explicite)

```
GET /recipe/_search
{
  "_source": "created",
  "query": {
    "match_all": {}
  },
  "sort": [
    { "created": "desc" }
  ]
}
```

L'élément `created` affiché dans le `sort` n'est pas la date formattée mais la représentation sous forme de nombre de millisecondes depuis le 1er Janvier 1970.

![alt text](https://i.ibb.co/GW6XDnd/11-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

##### Tri par multiple champs

Ici les recettes sont d'abord trièes en fonction du temps de préparation (ascendant) et ensuite par la date de création (descendant).
```
GET /recipe/_search
{
  "_source": [ "preparation_time_minutes", "created" ],
  "query": {
    "match_all": {}
  },
  "sort": [
    { "preparation_time_minutes": "asc" },
    { "created": "desc" }
  ]
}
```

#### 6. Tri par multi-valeur

##### Trier par moyenne de ratings (descendant)
```
GET /recipe/_search
{
  "_source": "ratings",
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "ratings": {
        "order": "desc",
        "mode": "avg"
      }
    }
  ]
}
```

![alt text](https://i.ibb.co/Kbx5Zdj/12-Screenshot-2021-03-19-Dev-Tools-Elastic.png "Learning Elasticsearch")

Dans le résultat de la requête l'élément `sort` contient la moyenne des `ratings`, qui est le critère de tri.

#### 7. Filtres

##### Ajouter une clause `filter` à la requête `bool`
```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "pasta"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```
Seules 3 recettes de `pasta` peuvent être préparées en moins de 15 minutes.
