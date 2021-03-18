#### 1. Searching for a term

##### DISCLAIMER : Certaines des requêtes de ce chapitre ne renvoient pas les résultats escomptés
En référer au formateur le cas échéant (Conflits possible de versions).

Les term level queries sont les plus utilisées pour les données structurées de type nombre ou date. Elles peuvent également être utilisées pour des labels de classification.<br/>
Par contre elles ne sont pas recommandées pour les recherches textuelles qui ne requiérent pas de correspondance exacte, elles sont inopérantes sur des champs analysés.

##### Recherche de documents avec une valeur à `true` pour le champ `is_active`

Les deux requêtes ci-dessous sont équivalentes :
```
GET /products/_search
{
  "query": {
    "term": {
      "is_active": true
    }
  }
}
```

```
GET /products/_search
{
  "query": {
    "term": {
      "is_active": {
        "value": true
      }
    }
  }
}
```

![alt text](https://i.ibb.co/M8TdZPS/058-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 2. Recherche en fonction de multiple terms

Nous avons que les tags sont stockés en tant que `keyword`.

```
GET /products/_search
{
  "query": {
    "terms": {
      "tags.keyword": [ "Soup", "Cake" ]
    }
  }
}
```

Similaire à la clause IN en SQL.

![alt text](https://i.ibb.co/SVyH7QV/059-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 3. Recherche de documents par IDs

```
GET /products/_search
{
  "query": {
    "ids": {
      "values": [ 1, 2, 3 ]
    }
  }
}
```

![alt text](https://i.ibb.co/dB9XSXb/060-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 4. Recherche de  documents avec une intervalle de valeurs

- Hint :  Une fois ces requêtes effectuées dans le dev tools, exécutez les dans le menu Discovery pour avoir un affichage tabulaire plus user friendly (demandez la démo au formateur).

##### Recherche de documents avec un champ `in_stock` entre `1` et `5` inclus

```
GET /products/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte": 1,
        "lte": 5
      }
    }
  }
}
```

![alt text](https://i.ibb.co/d2rRHmY/061-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Recherche de documents dans un intervale de dates

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01",
        "lte": "2010/12/31"
      }
    }
  }
}
```

![alt text](https://i.ibb.co/prtY8Jt/062-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Recherche de documents dans un intervale de dates, avec format de date configuré

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "01-01-2010",
        "lte": "31-12-2010",
        "format": "dd-MM-yyyy"
      }
    }
  }
}
```

![alt text](https://i.ibb.co/tX6LJ59/063-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 5. Jouons avec les dates relatives

Pour en savoir plus au sujet des opérations sur les dates dans Elasticsearch :
https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#date-math

##### Retrancher un an à partir du `2010/01/01`

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||-1y"
      }
    }
  }
}
```

![alt text](https://i.ibb.co/58FmWbZ/064-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Subtracting one year and one day from `2010/01/01`

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||-1y-1d"
      }
    }
  }
}
```

Toujours le même résultat.

##### Retrancher un an à partir de `2010/01/01` et arrondir au mois

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||-1y/M"
      }
    }
  }
}
```

Toujours le même résultat.

Gestion des arrondis par mois en finction de l'opérateur :

![alt text](https://us-ivry-natation.club/wp-content/uploads/2019/12/warning-icon-png-2753-150x150.png "Learning Elasticsearch")

##### Arrondir par mois avant de retrancher un an à partir du `2010/01/01`

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||/M-1y"
      }
    }
  }
}
```

Toujours le même résultat.

##### Arrondir par mois avant de retrancher un an à partir de la date courante

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "now/M-1y"
      }
    }
  }
}
```

Aucun résultat, élargir à 4 ou 5 ans pour avoir des résultats.

![alt text](https://i.ibb.co/kh2DjFg/065-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Recherche de documents avec le champ `created` contenant la date courante ou plus

```
GET /product/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "now"
      }
    }
  }
}
```

Aucun produit créé dans le futur. ;)

#### 6. Recherche de documents avec des valeurs non nulles

```
GET /products/_search
{
  "query": {
    "exists": {
      "field": "tags"
    }
  }
}
```

![alt text](https://i.ibb.co/kMss3r1/066-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 7. Recherche basée sur les prefixes

##### Documents contenant un tag commençant par `Vege`

```
GET /products/_search
{
  "query": {
    "prefix": {
      "tags.keyword": "Vege"
    }
  }
}
```

![alt text](https://i.ibb.co/89K8K6C/067-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 8. Recherche avec wildcards

Avertissement : Les requêtes avec wildcard sont très gourmandes en ressources et peuvent être lentes.<br/>
Par exemple ne jamais utiliser de wildcard au début du critére de recherche.

##### Ajouter un asterisque pour tous les caractères (zéro ou plus)

```
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veg*ble"
    }
  }
}
```

![alt text](https://i.ibb.co/JtSMNpk/068-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Ajouter un point d'interrogation pour tout caractère

```
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veg?ble"
    }
  }
}
```

![alt text](https://i.ibb.co/cD4GZpX/069-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

```
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veget?ble"
    }
  }
}
```

![alt text](https://i.ibb.co/7tL6WLk/070-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 9. Recherche avec expressions régulières

Pour comprendre comment fonctionnent les expressions régulières dans Elasticsearch :<br/>
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-regexp-query.html#regexp-syntax

```
GET /products/_search
{
  "query": {
    "regexp": {
      "tags.keyword": "Veg[a-zA-Z]+ble"
    }
  }
}
```

Même résultat qu'avec l'expression régulière : "Veget?ble".

#### 10. Exercices

##### Recherche de documents avec le champ `sold` inférieur à `10`

```
GET /products/_search
{
  "query": {
    "range": {
      "sold": {
        "lt": 10
      }
    }
  }
}
```

![alt text](https://i.ibb.co/Ns5L5fN/071-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Recherche de documents avec le champ `sold` entre `10` (inclusif) et `30` (exclusif)

```
GET /products/_search
{
  "query": {
    "range": {
      "sold": {
        "lt": 30,
        "gte": 10
      }
    }
  }
}
```

![alt text](https://i.ibb.co/PgLwvPP/072-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Recherche de documents contenant le tag `Meat`

```
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": "Meat"
    }
  }
}
```

![alt text](https://i.ibb.co/ZMFPbJB/073-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Recherche de documents contenant `Tomato` ou `Paste` dans le champ `name`

```
GET /products/_search
{
  "query": {
    "terms": {
      "name": [ "tomato", "paste" ]
    }
  }
}
```

![alt text](https://i.ibb.co/vZFrzLp/074-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Recherche de documents contenant `past` suivi par un caractère optionnel, pour le champ `name`

```
GET /products/_search
{
  "query": {
    "wildcard": {
      "name": "past?"
    }
  }
}
```

![alt text](https://i.ibb.co/wB4s4Mm/075-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Recherche de documents contenant un nombre dans le champ `name`

```
GET /products/_search
{
  "query": {
    "regexp": {
      "name": "[0-9]+"
    }
  }
}
```

![alt text](https://i.ibb.co/dBtbWtc/076-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")
