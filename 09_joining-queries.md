### Jointures

Dans Elasticsearch les performances des recherches sont optimisées par la dénormalisation des données.</b>

![alt text](https://i.ibb.co/wrLmgLY/01-Screenshot-from-2021-03-18-15-12-00.png "Learning Elasticsearch")

Elasticsearch ne supporte pas les jointures simples comme dans une base de données relationnelles. Mais il y a de façon de la fire différemment.<br/>
Cependant : les jointures sont très couteuses !!

##### Commençons par créer un jeu de données :
```
PUT /department
{
  "mappings": {  
    "properties": {
      "name": {
        "type": "text"
      },
      "employees": {
        "type": "nested"
      }
    }
  }
}
```
Le champs `employees` est de type `nested`, il contiendra un tableau d'objets.

Indexation des données :
```
PUT /department/_doc/1
{
  "name": "Development",
  "employees": [
    {
      "name": "Eric Green",
      "age": 39,
      "gender": "M",
      "position": "Big Data Specialist"
    },
    {
      "name": "James Taylor",
      "age": 27,
      "gender": "M",
      "position": "Software Developer"
    },
    {
      "name": "Gary Jenkins",
      "age": 21,
      "gender": "M",
      "position": "Intern"
    },
    {
      "name": "Julie Powell",
      "age": 26,
      "gender": "F",
      "position": "Intern"
    },
    {
      "name": "Benjamin Smith",
      "age": 46,
      "gender": "M",
      "position": "Senior Software Engineer"
    }
  ]
}
```

```sbtshell
PUT /department/_doc/2
{
  "name": "HR & Marketing",
  "employees": [
    {
      "name": "Patricia Lewis",
      "age": 42,
      "gender": "F",
      "position": "Senior Marketing Manager"
    },
    {
      "name": "Maria Anderson",
      "age": 56,
      "gender": "F",
      "position": "Head of HR"
    },
    {
      "name": "Margaret Harris",
      "age": 19,
      "gender": "F",
      "position": "Intern"
    },
    {
      "name": "Ryan Nelson",
      "age": 31,
      "gender": "M",
      "position": "Marketing Manager"
    },
    {
      "name": "Kathy Williams",
      "age": 49,
      "gender": "F",
      "position": "Senior Marketing Manager"
    },
    {
      "name": "Jacqueline Hill",
      "age": 28,
      "gender": "F",
      "position": "Junior Marketing Manager"
    },
    {
      "name": "Donald Morris",
      "age": 39,
      "gender": "M",
      "position": "SEO Specialist"
    },
    {
      "name": "Evelyn Henderson",
      "age": 24,
      "gender": "F",
      "position": "Intern"
    },
    {
      "name": "Earl Moore",
      "age": 21,
      "gender": "M",
      "position": "Junior SEO Specialist"
    },
    {
      "name": "Phillip Sanchez",
      "age": 35,
      "gender": "M",
      "position": "SEM Specialist"
    }
  ]
}
```

#### 1. Inner hits

Les champs de type `nested` ne peuvent être requêtés que via des requêtes `nested`. Ils sont utilisés pour les objets de type `array` pour établir des liens avec des propriètès d'autres objets (relation many-to-one).<br/>
La requête `nested` comprend deux parties :
* path : pour désigner le champ qui contient les objets.
* query : la requête qui sera exécutée sur ces objets.

Rechercher les départements où il y a des employéEs en tant qu'internes:
```
GET /department/_search
{
  "_source": false,
  "query": {
    "nested": {
      "path": "employees",
      "inner_hits": {},
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "employees.position": "intern"
              }
            },
            {
              "term": {
                "employees.gender.keyword": {
                  "value": "F"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

Les deux départements figurent dans les résultats.
la clause `inner_hits` permet de récupérer dedans que les employéEs qui répondent aux critères de la recherche.
 
#### 2. Mapping de liens entre documents

Peut-on créer des liens entre des objets sont que ceux-là soient imbriqués ?

![alt text](https://i.ibb.co/zWcx4pq/02-Screenshot-from-2021-03-18-15-43-40.png "Learning Elasticsearch")

Pour ce faire, utiliser les champs de type `join`. Ces champs définissent les relations entre les documents qui font partie d'une hiérarchie de documents.
```
PUT /department
{
  "mappings": {
    "properties": {
      "join_field": { 
        "type": "join",
        "relations": {
          "department": "employee"
        }
      }
    }
  }
}
```

Ici, dans la relation décrite `departement` est le parent de `employee`

#### 3. Ajouter des documents

##### Ajouter des departments

Dans le champ `join_field` on définit quelle relation cela représente dans le mapping.

![alt text](https://i.ibb.co/RDnMjYk/03-Screenshot-from-2021-03-18-16-11-58.png "Learning Elasticsearch")

```
PUT /department/_doc/1
{
  "name": "Development",
  "join_field": "department"
}
```

Pour une meilleure compréhension de la définition des relations il est préférable d'utiliser la syntaxe suivante : 
```
PUT /department/_doc/2
{
  "name": "Marketing",
  "join_field": {
    "name" : "department"
  }
}
```

##### Ajouter des employés par department
Dans le champ `parent` de `join_field` spécifier l'identifiant du parent.<br>
A noter que le paramètre routing est obligatoire pour que les parents et enfants soient stockés dans le même shard.
```
PUT /department/_doc/3?routing=1
{
  "name": "Bo Andersen",
  "age": 28,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
```

```
PUT /department/_doc/4?routing=2
{
  "name": "John Doe",
  "age": 44,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 2
  }
}
```

```
PUT /department/_doc/5?routing=1
{
  "name": "James Evans",
  "age": 32,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
```

```
PUT /department/_doc/6?routing=1
{
  "name": "Daniel Harris",
  "age": 52,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
```

```
PUT /department/_doc/7?routing=2
{
  "name": "Jane Park",
  "age": 23,
  "gender": "F",
  "join_field": {
    "name": "employee",
    "parent": 2
  }
}
```

```
PUT /department/_doc/8?routing=1
{
  "name": "Christina Parker",
  "age": 29,
  "gender": "F",
  "join_field": {
    "name": "employee",
    "parent": 1
  }
}
```

#### 4. Recherche par identifiant du parent
Tous les employés du département dont l'identifiant est 1 :
```
GET /department/_search
{
  "query": {
    "parent_id": {
      "type": "employee",
      "id": 1
    }
  }
}
```

![alt text](https://i.ibb.co/TRxf6y4/04-Screenshot-2021-03-18-Elastic-Kibana.png "Learning Elasticsearch")

#### 5. Recherche des enfants  par parent

##### Recherche d'enfants d'un parent répondant à des critères
Pour ce faire, utiliser la clause `has_parent` où doivent être définis le type du parent et les critères de recherche auxquels le parent doit répondre.
```
GET /department/_search
{
  "query": {
    "has_parent": {
      "parent_type": "department",
      "query": {
        "term": {
          "name.keyword": "Development"
        }
      }
    }
  }
}
```

Même résultat que la précédente recherche.

##### Incorporer le score de pertinence des documents trouvés

Par défaut, la requête ignore le score de pertinence, c'est à dire que la correspondance du parent aux critères de recherches n'a aucune incidence sur le calcul du score.<br/>
Il est cependant possible d'y remédier grâce à l'option `score`. Ainsi l'enfant qui appartient au parent correspondant le plus aux critères de la requête aura le score le plus élevé.
```
GET /department/_search
{
  "query": {
    "has_parent": {
      "parent_type": "department",
      "score": true,
      "query": {
        "term": {
          "name.keyword": "Development"
        }
      }
    }
  }
}
```

Tous les documents retournés ont le même score.

![alt text](https://i.ibb.co/xFkZP8G/05-Screenshot-2021-03-18-Elastic-Kibana.png "Learning Elasticsearch")

#### 6. Recherche de parent par critères sur enfants

##### Recherche de parents avec enfants qui répondent à une requête de type `bool`

Les départements ayant au moins un employé homme (critère facultatif) dont l'age est supérieur à 50 ans (critère obligatoire).<br/>
Pour ce faire, utiliser la clause `has_parent` où doivent figurer le type des enfants et les critères de recherche sur ces enfants.
```
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}
```

![alt text](https://i.ibb.co/3FL7Q41/06-Screenshot-2021-03-18-Elastic-Kibana.png "Learning Elasticsearch")

##### Prendre en compte le score de pertinence avec le `score_mode`
```
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "score_mode": "sum",
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}
```

![alt text](https://i.ibb.co/sjnMfTJ/07-Screenshot-2021-03-18-Elastic-Kibana.png "Learning Elasticsearch")

##### Specifier le nombre minimum et maximum d'enfants
```
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "score_mode": "sum",
      "min_children": 2,
      "max_children": 5,
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}
```

Aucun résultat.

#### 7. Relations Multi-level

![alt text](https://i.ibb.co/F4147S4/11-Screenshot-from-2021-03-18-21-31-09.png "Learning Elasticsearch")

##### Creation d'un index avec mapping

Définir le type de la relation `company` qui contient les relations en tant que parent de `department` et `supply`.<br/>
Définir également la relation entre `department` et `employee`.
```
PUT /company
{
  "mappings": {
    "properties": {
      "join_field": { 
        "type": "join",
        "relations": {
          "company": ["department", "supplier"],
          "department": "employee"
        }
      }
    }
  }
}
```

##### Ajouter une compagnie
```
PUT /company/_doc/1
{
  "name": "My Company Inc.",
  "join_field": "company"
}
```

##### Ajouter un départment
```
PUT /company/_doc/2?routing=1
{
  "name": "Development",
  "join_field": {
    "name": "department",
    "parent": 1
  }
}
```

##### Ajouter un employé
```
PUT /company/_doc/3?routing=1
{
  "name": "Bo Andersen",
  "join_field": {
    "name": "employee",
    "parent": 2
  }
}
```

##### Ajouter d'autres données de test
```
PUT /company/_doc/4
{
  "name": "Another Company, Inc.",
  "join_field": "company"
}
```

```
PUT /company/_doc/5?routing=4
{
  "name": "Marketing",
  "join_field": {
    "name": "department",
    "parent": 4
  }
}
```

```
PUT /company/_doc/6?routing=4
{
  "name": "John Doe",
  "join_field": {
    "name": "employee",
    "parent": 5
  }
}
```

##### Exemple de requête multi-level

Rechercher la compganie ayant un département où travaille un certain "John Doe", soupçonné en tant que lanceur d'alerte.
```
GET /company/_search
{
  "query": {
    "has_child": {
      "type": "department",
      "query": {
        "has_child": {
          "type": "employee",
          "query": {
            "term": {
              "name.keyword": "John Doe"
            }
          }
        }
      }
    }
  }
}
```
![alt text](https://i.ibb.co/nf6bbdJ/08-Screenshot-2021-03-18-Elastic-Kibana.png "Learning Elasticsearch")

#### 8. Parent/enfant inner hits

##### Inclure les inner hits pour les requêtes `has_child`
```
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "inner_hits": {},
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}
```
![alt text](https://i.ibb.co/cYh8Pb4/09-Screenshot-2021-03-18-Elastic-Kibana.png "Learning Elasticsearch")

##### Inclure les inner hits pour les requêtes `has_parent`

```
GET /department/_search
{
  "query": {
    "has_parent": {
      "inner_hits": {},
      "parent_type": "department",
      "query": {
        "term": {
          "name.keyword": "Development"
        }
      }
    }
  }
}
```
![alt text](https://i.ibb.co/WyFg2mD/10-Screenshot-2021-03-18-Elastic-Kibana.png "Learning Elasticsearch")

#### 9. Terms lookup

##### Ajouter des données de test
```
PUT /users/_doc/1
{
  "name": "John Roberts",
  "following" : [2, 3]
}
```

```
PUT /users/_doc/2
{
  "name": "Elizabeth Ross",
  "following" : []
}
```

```
PUT /users/_doc/3
{
  "name": "Jeremy Brooks",
  "following" : [1, 2]
}
```

```
PUT /users/_doc/4
{
  "name": "Diana Moore",
  "following" : [3, 1]
}
```

```
PUT /stories/_doc/1
{
  "user": 3,
  "content": "Wow look, a penguin!"
}
```

```
PUT /stories/_doc/2
{
  "user": 1,
  "content": "Just another day at the office... #coffee"
}
```

```
PUT /stories/_doc/3
{
  "user": 1,
  "content": "Making search great again! #elasticsearch #elk"
}
```

```
PUT /stories/_doc/4
{
  "user": 4,
  "content": "Had a blast today! #rollercoaster #amusementpark"
}
```

```
PUT /stories/_doc/5
{
  "user": 4,
  "content": "Yay, I just got hired as an Elasticsearch consultant - so excited!"
}
```

```
PUT /stories/_doc/6
{
  "user": 2,
  "content": "Chilling at the beach @ Greece #vacation #goodtimes"
}
```

##### Recherche des stories des user's suivis par un user donnés
```
GET /stories/_search
{
  "query": {
    "terms": {
      "user": {
        "index": "users",
        "id": "1",
        "path": "following"
      }
    }
  }
}
```

![alt text](https://i.ibb.co/Zd841bc/12-Screenshot-2021-03-18-Elastic-Kibana.png "Learning Elasticsearch")

Illustration :

![alt text](https://i.ibb.co/ccqRgFh/13-Screenshot-from-2021-03-18-21-50-02.png "Learning Elasticsearch")
