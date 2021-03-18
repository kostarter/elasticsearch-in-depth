#### 1. Utilisation de l'API `_analyze`

Rappel :

* L'analyse dans Elasticsearch est seulement applicable aux champs textuels.
* Lorsqu'un document est indexé les valeurs textuels sont analysées.
* Le résulat est stocké dans des structures de données pour rendre la recherche efficiente.

![alt text](https://i.ibb.co/zmQtMSk/01-Screenshot-from-2021-03-18-11-00-54.png "Learning Elasticsearch")

La documentation des analyzers embarqués dans Elasticsearch :<br/>
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html

###### Découpage en tokens d'un texte avec le tokenizer de type `standard`

```
POST _analyze
{
  "tokenizer": "standard",
  "text": "I'm in the mood for drinking semi-dry red wine!"
}
```
![alt text](https://i.ibb.co/9Y79z3c/041-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

A noter que le tokenizer stocke également l'offset de chaque token.

###### Utilisation du filtre `lowercase`

Un filtre reçoit les données du tokenizer, il peut les filtrer ou les modifier.<br/>
Un analyzer peut contenir aucun ou plusieurs filtres.

```
POST _analyze
{
  "filter": [ "lowercase" ],
  "text": "I'm in the mood for drinking semi-dry red wine!"
}
```

![alt text](https://i.ibb.co/TRF52QD/042-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

###### Utilisation d'un analyzer de type `standard`

```
POST _analyze
{
  "analyzer": "standard",
  "text": "I'm in the mood for drinking semi-dry red wine!"
}
```

![alt text](https://i.ibb.co/Q96GqQW/043-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

Pour résumer l'action du standard analyzer :

![alt text](https://i.ibb.co/6ZTgZdz/02-Screenshot-from-2021-03-18-11-13-22.png "Learning Elasticsearch")

#### 2. Jouons avec les Analyzers

###### Configurer un analyzer de type `standard`

```
PUT /analyzers_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "english_stop": {
          "type": "standard",
          "stopwords": "_english_"
        }
      },
      "filter": {
        "my_stemmer": {
          "type": "stemmer",
          "name": "english"
        }
      }
    }
  }
}
```

**Rappel :** Les stopwords sont les mots qui vont être filtrés durant l'analyse de texte.<br/> 
Exemple, en anglais ce sera : "the", "on", "of", "a", etc.

###### Tester l'analyzer

```
POST /analyzers_test/_analyze
{
  "analyzer": "english_stop",
  "text": "I'm in the mood for drinking semi-dry red wine!"
}
```

![alt text](https://i.ibb.co/vmGgWk4/044-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

```
POST /analyzers_test/_analyze
{
  "tokenizer": "standard",
  "filter": [ "my_stemmer" ],
  "text": "I'm in the mood for drinking semi-dry red wines!"
}
```
Mettre un 's' à la fin de wine pour vérifier que le stemming fonctionne bien.<br>
**Rappel :** Le stemming a pour fonction de réduire les mots à leur racine. Exemple : "loved", "loves", "loving" vont converger vers "love".

![alt text](https://i.ibb.co/BqRPRKf/045-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

#### 3. Creation d'un analyzer custom

###### Ajouter un analyzer configuré

```
DELETE /analyzers_test
```

Utiliser le filtre HTML strip :<br/>
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-htmlstrip-charfilter.html

```
PUT /analyzers_test
{
  "settings": {
    "analysis": {
      "filter": {
        "my_stemmer": {
          "type": "stemmer",
          "name": "english"
        }
      },
      "analyzer": {
        "english_stop": {
          "type": "standard",
          "stopwords": "_english_"
        },
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
            "lowercase",
            "trim",
            "my_stemmer"
          ]
        }
      }
    }
  }
}
```

###### Tester l'analyzer configuré

```
POST /analyzers_test/_analyze
{
  "analyzer": "my_analyzer",
  "text": "I'm in the mood for drinking <strong>semi-dry</strong> red wine!"
}
```

![alt text](https://i.ibb.co/p2gmhLh/03-Screenshot-2021-03-18-Elastic-Kibana.png "Learning Elasticsearch")

#### 4. Utiliser des analyzers dans les mappings

###### Utiliser un analyzer configuré dans le mapping d'un champ

```
PUT /analyzers_test/_mapping
{
  "properties": {
    "description": {
      "type": "text",
      "analyzer": "my_analyzer"
    },
    "teaser": {
      "type": "text",
      "analyzer": "standard"
    }
  }
}
```

###### Ajouter un document de test

```
POST /analyzers_test/_doc/1
{
  "description": "drinking",
  "teaser": "drinking"
}
```

##### Tester le mapping

```
GET /analyzers_test/_doc/_search
{
  "query": {
    "term": {
      "teaser": {
        "value": "drinking"
      }
    }
  }
}
```

![alt text](https://i.ibb.co/hZtYK2L/048-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

```
GET /analyzers_test/_doc/_search
{
  "query": {
    "term": {
      "description": {
        "value": "drinking"
      }
    }
  }
}
```

![alt text](https://i.ibb.co/YL9hy4L/049-Screenshot-2021-03-17-Elastic-Kibana.png "Learning Elasticsearch")

Essayez avec "drink" et ça ira mieux. ;)

#### 5. TP : Construire un bon analyzer français pour Elasticsearch
(By Joli Code)


Dans un index de recherche tel qu’Elasticsearch, une recherche full-text est une simple collecte de documents, qui s’effectue via une comparaison de tokens.

Ces tokens vivent dans l’index inversé et ont été extraits du contenu de vos documents lors de l’indexation. Plus vos tokens sont proprement indexés, et plus facilement un utilisateur trouvera vos documents : c’est le rôle de l’analyse.

Ce TP va vous guider dans la conception d’un analyzer Elasticsearch pour la langue française qui soit à la fois tolérant, pertinent et rapide – et bien meilleur que l’analyzer « french » fourni par défaut dans le moteur de recherche.


L’importance de l’analyse
Prenons un document type pour commencer : le burger 🍔.
```
{
  "name": "Hamburger",
  "description": "Un hamburger, parfois hambourgeois (au Canada francophone) 
ou par aphérèse burger, est un sandwich d'origine allemande, composé de deux 
pains de forme ronde (bun) parfois garnis de viande hachée (souvent du bœuf) 
et généralement de crudités — salade, tomate, oignon, cornichon (pickles) —, 
de fromage et de sauce. C'est un plat typique de la restauration rapide, 
emblématique de la cuisine américaine."
}
```
Avec l’analyse par défaut (appelée « standard »), notre index va être constitué des mots simplement mis en minuscule. Pour n’en citer que quelques-uns, par exemple :

sandwich ;
composé ;
crudités ;
américaine ;
bœuf.
Lors d’une recherche, les termes recherchés sont analysés aussi, avec la même technique. Rechercher « Sandwichs » au pluriel donnerait le token sandwichs, qui n’existe pas dans notre index. L’utilisateur va donc devoir saisir les mots exacts : avec pluriels, accents, ligature… Cela n’est bien sûr pas acceptable !

En utilisant l’analyzer french d’Elasticsearch, les tokens seront plutôt :

sandwich ;
compos ;
crudit ;
americain ;
bœuf.

Il y a une nette amélioration pour trois tokens : composé est devenu compos, son lexème (ou racine linguistique). Cela va nous permettre de trouver un burger en cherchant n’importe quelle forme de ce mot : « composer », « compose »… 
<br>Mais quelques problèmes subsistent. Par exemple l’e dans l’o de bœuf n’est pas décomposé, et il sera donc impossible de trouver notre document en recherchant « boeuf » !

C’est grâce à l’analyse que les pluriels, les conjugaisons, la casse… peuvent être gérés. Voyons comment la construire et l’améliorer.

### Les différentes étapes de l’analyse

L’analyse menée par Elasticsearch se décompose en trois étapes successives :

1. Les Char Filter
Un char_filter permet d’appliquer des transformations sur le texte complet, avant qu’il ne soit découpé en tokens. Cette étape permet de nettoyer le contenu, remplacer certains raccourcis, enlever du HTML ou de la ponctuation mal venue, etc.

Il serait par exemple possible de remplacer « & » par « et », afin d’indexer l’esperluette.

2. Le Tokenizer
L’étape du tokenizer consiste à couper le texte en tokens. Elasticsearch utilise par défaut le standard Unicode Text Segmentation, qui va retirer la ponctuation et couper à chaque espace.

La grande majorité des espaces est gérée, mais certains caractères, comme l’invisible trait d’union conditionnel (Soft hyphen) seront conservés ! Et cela va poser de sérieux problèmes pour les étapes suivantes. Il en est de même pour le point médian !

3. Les Token Filter
C’est là que la majorité du travail de nettoyage et d’enrichissement s’effectue lors de l’analyse. Les token_filter peuvent modifier, ajouter et supprimer des tokens – leur rôle est donc multiple et leur ordre d’exécution important : il s’agit d’une chaîne de filtres.

### L’analyzer « french » revisité
L’analyzer pré-configuré dans Elasticsearch (version 5.1 à l’heure où j’écris ces lignes) est le suivant :
```
{
  "settings": {
    "analysis": {
      "filter": {
        "french_elision": {
          "type":         "elision",
          "articles_case": true,
          "articles": [
              "l", "m", "t", "qu", "n", "s",
              "j", "d", "c", "jusqu", "quoiqu",
              "lorsqu", "puisqu"
            ]
        },
        "french_stop": {
          "type":       "stop",
          "stopwords":  "_french_" 
        },
        "french_keywords": {
          "type":       "keyword_marker",
          "keywords":   [] 
        },
        "french_stemmer": {
          "type":       "stemmer",
          "language":   "light_french"
        }
      },
      "analyzer": {
        "french": {
          "tokenizer":  "standard",
          "filter": [
            "french_elision",
            "lowercase",
            "french_stop",
            "french_keywords",
            "french_stemmer"
          ]
        }
      }
    }
  }
}
```

L’utilisation du tokenizer standard est le premier problème que j’aimerais régler. En effet, ce tokenizer est très simple et ne sait pas spécialement traiter des mélanges d’écritures : il va par exemple séparer « βeta » en deux token (β et eta). Il ne sait pas non plus couper les langues non occidentales…

Il faut lui préférer le icu_tokenizer : plus efficace et tirant partie de la librairie ICU, qui a une connaissance étendue d’Unicode. Ce tokenizer est disponible via l’installation du plugin officiel analysis-icu.

Le premier filtre est french_elision, il enlève les articles pouvant précéder un mot, et donc d’origine devient origine.

Le filtre lowercase, comme son nom l’indique, permet de mettre en minuscule l’intégralité du token, il est présent par défaut dans Elasticsearch.

Arrive ensuite le filtre french_stop, qui retire les tokens tels que en, au, du, par, est… car ils sont considérés comme du bruit – présent dans l’immense majorité des documents, il était considéré peu pertinent de les conserver… Et c’est bien dommage car ils peuvent apporter du sens à une phrase, ou aider à départager deux documents ayant obtenus des scores égaux. Aujourd’hui, avec la similarité par Okapi BM25 par défaut dans Elasticsearch 5 et la clause DSL common, il n’est plus nécessaire d’utiliser ce filtre !

Pour finir, french_stemmer applique une racinisation (stemming) de nos tokens, c’est ce qui permet de supprimer les formes plurielles, les différentes conjugaisons, accord de genre sur un mot. Il existe trois algorithmes pour le français, mais nous conserverons le light_french utilisé par défaut.

Cette dernière étape va grandement améliorer notre collecte de document, car nous allons pouvoir trouver le mot « composé » en recherchant « composer » par exemple. Mais elle fait aussi perdre du sens et de la pertinence, c’est pourquoi nous allons créer deux versions de notre analyzer.

Par dessus cette bonne base de travail, nous allons ajouter un meilleur support d’Unicode via le filtre icu_folding. Ce filtre va faire plusieurs traitements très utiles :

normaliser nos textes pour s’assurer que toutes les variantes d’une lettre soient simplifiées ;
remplacer les lettres accentuées par leurs formes sans accents ;
supprimer certains caractères tels que le point médian ;
remplacer les ligatures telles que œ par leurs équivalents…

L’ajout de synonymes est aussi à considérer : il serait tout à fait intéressant que « salade » puisse être trouvé en recherchant « laitue », c’est le rôle du filtre synonym. La difficulté ici réside dans la constitution d’un dictionnaire de correspondance pertinent.

Ce dictionnaire pourra servir plusieurs objectifs :

enrichir le vocabulaire de vos documents : salade, laitue, batavia ;
donner de la signification aux acronymes : NASA (National Aeronautics and Space Administration), JS (JavaScript), UN (United Nations)… Ce dernier pose d’ailleurs souvent problème avec le mot « un », d’où l’importance de la casse !
Notre domaine ici sera la cuisine rapide, les sandwichs, et nous pouvons donc utiliser un filtre synonym dans notre analyzer.

### Voici l’analyzer complet avec nos modifications :

```
PUT french
{
  "settings": {
    "analysis": {
      "filter": {
        "french_elision": {
          "type": "elision",
          "articles_case": true,
          "articles": ["l", "m", "t", "qu", "n", "s", "j", "d", "c", "jusqu", "quoiqu", "lorsqu", "puisqu"]
        },
        "french_synonym": {
          "type": "synonym",
          "ignore_case": true,
          "expand": true,
          "synonyms": [
            "salade, laitue",
            "mayo, mayonnaise",
            "grille, toaste"
          ]
        },
        "french_stemmer": {
          "type": "stemmer",
          "language": "light_french"
        }
      },
      "analyzer": {
        "french_heavy": {
          "tokenizer": "icu_tokenizer",
          "filter": [
            "french_elision",
            "icu_folding",
            "french_synonym",
            "french_stemmer"
          ]
        },
        "french_light": {
          "tokenizer": "icu_tokenizer",
          "filter": [
            "french_elision",
            "icu_folding"
          ]
        }
      }
    }
  }
}
```
Nous avons deux versions :

- `french_heavy` qui va faire une analyse poussée, qui va fortement altérer les tokens mais qui va être très utile pour la collecte (nous aurons beaucoup de résultats) :

`hamburg, compos, pain, boeuf, salad, laitu`

- `french_light` qui altère le moins possible le contenu et va nous permettre d’augmenter la pertinence de nos résultats :

`hamburger, compose, pains, boeuf, salade`

Les tokens qui ressortent lors de l’indexation de notre Hamburger sont bien plus propres, et permettent donc des recherches moins précises mais toujours pertinentes.

Sans une bonne recherche, l’analyse n’est rien
Avoir les bons tokens ne suffit pas : vous devrez adapter votre mapping et vos recherches.

Nous allons mettre en place un mapping simple avec un Multi Field :
```
PUT /french/_mapping/sandwich
{
  "sandwich": {
    "properties": {
      "description": {
        "type": "text",
        "analyzer": "french_light",
        "fields": {
          "stemmed": {
            "type": "text",
            "analyzer": "french_heavy"
          }
        }
      }
    }
  }
}
```
Et ajouter un second sandwich à notre index :
```
{
  "name": "Burrito",
  "description": "Un burrito est une préparation culinaire remontant à la fin 
du xixe siècle originaire du Mexique. D'invention récente, le burrito n'est 
pas un plat de la cuisine traditionnelle mexicaine. Il se compose d'une tortilla 
de farine de blé garnie de divers ingrédients tels que de la viande de bœuf, 
des haricots, des tomates, des épices, du piment, de l'oignon, de la salade, etc. 
On ne frit pas la tortilla, elle ne sert que d'enveloppe à son contenu. 
S'il était frit, le burrito deviendrait une chimichanga."
}
```

Recherche simple : « tomate »
Avec cette recherche, seul le Hamburger remonte, pas de Burrito, alors que nous y mettons aussi des tomates :

```
GET /french/sandwich/_search
{
  "query": {
    "match": {
      "description": "tomate"
    }
  }
}
```

Le problème ? Nous ne recherchons que sur la version « light » ! Et dans le Burrito les tomates sont au pluriel. La solution est d’utiliser multi_match :

```
GET /french/sandwich/_search
{
  "query": {
    "multi_match": {
      "query": "tomate",
      "fields": ["description", "description.stemmed"]
    }
  }
}
```
Cette fois les deux documents sont présents, et encore mieux, Hamburger a un score plus élevé ! En effet il a le mot exact (au singulier), il est donc plus pertinent car le score des deux champs est combiné.

Recherche avec un stop word : « sandwich du canada »
Cette recherche pose problème : elle remonte le Burrito ! En effet le token du est présent dans ce document.
```
GET /french/sandwich/_search
{
  "query": {
    "multi_match": {
      "query": "sandwich du canada",
      "fields": ["description", "description.stemmed"]
    }
  }
}
```
Pour réduire l’effet de ce token du, qui va être très présent dans notre index, nous allons utiliser la clause common. Elle sépare les tokens les plus présents dans l’index des autres, et ne les utilise que pour améliorer la pertinence. Cela veut dire que si l’immense majorité de mes documents possèdent le mot « du », il n’aura plus d’impact lors de la collecte – car non pertinent.
```
GET /french/sandwich/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "common": {
            "description.stemmed": {
              "query": "sandwich du canada"
            }
          }
        }
      ],
      "should": [
        {
          "match": {
            "description": "sandwich du canada"
          }
        }
      ]
    }
  }
}
```

### Tolérance aux coquilles : « vuande »

Si vous devez supporter des fautes de saisie importante, la clause Common exposée plus haut ne sera pas d’une grande aide ; elle ne supporte pas la fuzziness.

Par contre avec la clause MultiMatch et l’option fuzziness :
```
GET /french/sandwich/_search
{
  "query": {
    "multi_match": {
      "query": "vuande",
      "fuzziness": "AUTO",
      "fields": ["description", "description.stemmed"]
    }
  }
}
```

Nous trouvons ici nos deux sandwichs !

### Conclusion

La recherche est un compromis entre la collecte et la pertinence : parfois il est préférable d’avoir peu de résultats mais qu’ils soient très précis, et d’autres fois d’en avoir un maximum.
