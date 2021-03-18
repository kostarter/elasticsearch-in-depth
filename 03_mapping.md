## Mapping :

Rappel :

* Le mapping décrit la structure des documents (champs et types de données).
* Similaire au schéma d'une table en base de données relationnel.

![alt text](https://i.ibb.co/cvSPZVR/01-Screenshot-from-2021-03-18-09-40-46.png "Learning Elasticsearch")

Il existe deux types de mapping :

* Mapping explicite : défini par l'utilisateur.
* Mapping implicite : généré par Elasticsearch qui inspecte les données et essaye de leut trouver un type.

Les deux approches peuvent être combinées pour un même index.

#### 1. Mapping Dynamique

##### Récupérer le mapping

```
GET /products/_mapping
```

![alt text](https://i.ibb.co/BVp6hrW/028-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 2. Ajouter un mapping à un index existant

##### Ajouter un mapping pour le champ `discount`

Ajout d'un nouveau champ :
```
PUT /products/_mapping
{
  "properties": {
    "discount": {
      "type": "double"
    }
  }
}
```

##### Vérifier le nouveau mapping
```
GET /products/_mapping
```

![alt text](https://i.ibb.co/93zyKNX/029-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

Ceci est un exemple simple d'une approche hybride, on laisse Elasticsearch inférer le mapping, puis on affine.

#### 3. Modifier un mapping existant

* On peut imaginer un identifiant de produit qui va contenir des lettres dorénavent.
* On peut vouloir utiliser cet identifiant en tant que keyword pour des aggrégations.
* etc...

Dans Elasticsearch il n'est pas toujours possible de modifier le mapping d'un champ.
Essayons de changer le type du champ `discout` à text :
```
PUT /products/_mapping
{
  "properties": {
    "discount": {
      "type": "text"
    }
  }
}
```

![alt text](https://i.ibb.co/rxFtGsG/02-Screenshot-2021-03-18-Elastic-Kibana.png "Learning Elasticsearch")

Conclusion : La solution est de ré-indexer les documents dans un nouvel index !

##### Suppression de l'index
```
DELETE /products
```

##### Creation de l'index avec mapping
```
PUT /products
{
  "mappings": {
    "dynamic": false,
    "properties": {
      "in_stock": {
        "type": "integer"
      },
      "is_active": {
        "type": "boolean"
      },
      "price": {
        "type": "integer"
      },
      "sold": {
        "type": "long"
      }
    }
  }
}
```

Vérifions notre index :

![alt text](https://i.ibb.co/hHLJv03/030-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 4. Le multi-fields mapping

Le mapping d'un champ peut être multiple. Par exemple, un champ text peut également être défini en tant que keyword.

##### Ajouter le mapping
```
PUT /products/_mapping
{
  "properties": {
    "description": {
      "type": "text"
    },
    "name": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword"
        }
      }
    },
    "tags": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword"
        }
      }
    }
  }
}
```

##### Récupérer le mapping
```
GET /products/_mapping
```

Vérifions notre index :

![alt text](https://i.ibb.co/qgpmLZs/031-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 5. Les formats de date

Les dates peuvent spécifiées de trois manières différentes :
* Des chaines de caractères formattées.
* Timestamp en millisecondes (depuis le 1er Janvier 1970).
* Timestamp en secondes (depuis le 1er Janvier 1970).

Toutes les dates sont stockées en millisecondes (depuis le 1er Janvier 1970).

![alt text](https://i.ibb.co/8PW1r2S/03-Screenshot-from-2021-03-18-10-26-16.png "Learning Elasticsearch")

##### Mapping de date avec le format `year`

Ajouter un mapping de type date au format year :
```
PUT /products/_mapping
{
  "properties": {
    "created": {
      "type": "date",
      "format": "year"
    }
  }
}
```

![alt text](https://i.ibb.co/tX5bKdX/032-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Mapping de date avec le format `strict_year`
```
PUT /products/_mapping
{
  "properties": {
    "created": {
      "type": "date",
      "format": "strict_year"
    }
  }
}
```

![alt text](https://i.ibb.co/85sCZVw/033-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

Il faut ré-indexer !

##### Mapping explicite de date avec le format par défault
```
PUT /products/_mapping
{
  "properties": {
    "created": {
      "type": "date",
      "format": "strict_date_optional_time||epoch_millis"
    }
  }
}
```

##### Mapping explicite de date avec format de date et heure optionnelle
```
PUT /products/_mapping
{
  "properties": {
    "created": {
      "type": "date",
      "format": "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd"
    }
  }
}
```

#### 6. Récupérer des champs nouveaux sans mapping dynamique

##### Ajouter un document de test
```
PUT /products/_doc/2000
{
  "description": "Test",
  "discount": 20
}
```

##### Ajouter un mapping pour le champ `discount`
```
PUT /products/_mapping
{
  "properties": {
    "discount": {
      "type": "integer"
    }
  }
}
```

![alt text](https://i.ibb.co/Dr4pc1m/037-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Requêter le champ `description`
```
GET /products/_search
{
  "query": {
    "match": {
      "description": "Test"
    }
  }
}
```

![alt text](https://i.ibb.co/ZNYvTPC/038-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Requêter le champ `discount`
```
GET /products/_search
{
  "query": {
    "term": {
      "discount": 20
    }
  }
}
```

![alt text](https://i.ibb.co/5W8hRmq/039-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

##### Récupérer le nouveau mapping pour les documents
```
POST /products/_update_by_query?conflicts=proceed
```

Refaire la recherche précédente et vérifier si maintenant il y a des résultats.

##### Suppression du document de test
```
DELETE /products/_doc/2000
```

![alt text](https://i.ibb.co/nkZ5wH5/040-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")
