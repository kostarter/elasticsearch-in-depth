#### 1. Requêtes avec logique booléenne

* **must**

La clause (query) `must` apparait dans les documents qui matchent et va contribuer au score.

* **filter**

La clause (query) `filter` apparait dans les documents qui matchent. Cependant, et contrairement au `must`, le score de la clause est ignoré.<br/>
Les clauses `filter` sont exécutés dans le _filter context_, ce qui veut dire que le scoring est ignoré et que les clauses sont gérées par le cache.

* **should**

La clause (query) `should` apparait dans les documents qui matchent.

* **must_not**

La clause (query) `must_not` apparait dans les documents qui matchent. Le clause sont exécutés dans le _filter context_, ce qui veut dire que le scoring est ignoré et que les clauses sont gérées par le cache. Comme le scoring est ignoré, un score de 0 est assigné à tous les documents retournés.


##### Adding query clauses to the `must` key

```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
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

![alt text](https://i.ibb.co/q1LwrjC/086-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Déplacer le `range` query dans le `filter`

```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
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

![alt text](https://i.ibb.co/9gmG441/087-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Ajouter une clause `match` à la clé `must_not`

```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": "tuna"
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

![alt text](https://i.ibb.co/j6M6c8f/088-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Ajouter une clause query à la clé `should`

Rappel : 

* **must** signifie que la clause (query) `must` apparait dans les documents qui matchent. Ces clauses doivent matcher, comme le ET logique.

* **should** signifie qu'au moins une de ces clauses doit matcher, comme le OU logique.

Pour résumer : ils sont utilisés comme les opérateurs logiques ET et OU.

Dans une requête de type `bool` :

* **must** signifie que les clauses qui doivent matcher pour le document à inclure.

* **should** signifie que si les clauses matchent, ils augmentent le `_score`, sinon ils sont sans effet sur le score. Ils sont seulement utilisés pour affiner le score pour chaque document.

```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": "tuna"
          }
        }
      ],
      "should": [
        {
          "match": {
            "ingredients.name": "parsley"
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

![alt text](https://i.ibb.co/vxgZckF/089-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Le comportement de la clause `should`

```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "pasta"
          }
        }
      ],
      "should": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ]
    }
  }
}
```

![alt text](https://i.ibb.co/McpFHQC/090-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

Ici comme il n'y a qu'un seul critère, avec la clause `must` le résultat serait le même.
```
GET /recipe/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ]
    }
  }
}
```

![alt text](https://i.ibb.co/h81wgpR/091-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 2. Debugger les requêtes `bool` avec des critères de requêtes labélisées

```
GET /recipe/_search
{
    "query": {
        "bool": {
          "must": [
            {
              "match": {
                "ingredients.name": {
                  "query": "parmesan",
                  "_name": "parmesan_must"
                }
              }
            }
          ],
          "must_not": [
            {
              "match": {
                "ingredients.name": {
                  "query": "tuna",
                  "_name": "tuna_must_not"
                }
              }
            }
          ],
          "should": [
            {
              "match": {
                "ingredients.name": {
                  "query": "parsley",
                  "_name": "parsley_should"
                }
              }
            }
          ],
          "filter": [
            {
              "range": {
                "preparation_time_minutes": {
                  "lte": 15,
                  "_name": "prep_time_filter"
                }
              }
            }
          ]
        }
    }
}
```

On voit dans le bas de chaque document retourné une section `matched_queries` où sont listées les critères auxquels le document répond positivement. 

![alt text](https://i.ibb.co/rHS5nSP/092-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 3. Comment les requêtes `match` fonctionnent

##### Les deux requêtes suivantes sont equivalentes

Nous avons déjà vu que l'opérateur logique par défaut est le OU.
```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": "pasta carbonara"
    }
  }
}
```

La clause `should` correspond à l'opérateur logique OU.
```
GET /recipe/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "title": "pasta"
          }
        },
        {
          "term": {
            "title": "carbonara"
          }
        }
      ]
    }
  }
}
```

![alt text](https://i.ibb.co/f86YZY7/093-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Les deux requêtes suivantes sont equivalentes
```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "pasta carbonara",
        "operator": "and"
      }
    }
  }
}
```

La clause `must` correspond à l'opérateur logique ET.
```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "title": "pasta"
          }
        },
        {
          "term": {
            "title": "carbonara"
          }
        }
      ]
    }
  }
}
```

![alt text](https://i.ibb.co/zPztQXT/094-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")
