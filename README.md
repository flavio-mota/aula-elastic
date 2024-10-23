# aula-elastic

### Obter informações da saúde do cluster
```
GET _cluster/health
```

### Obter informações sobre os nós do cluster
```
GET _nodes/stats
```

### Consultas
Consultas retornam documentos que combinam com os critérios. 

#### Recupera a informação sobre os documentos em um ínidice

Sintaxe: 
```
GET nome_do_indice/_search
```
Exemplo: 
```
GET news_headlines/_search
```

O Elasticsearch mostra um número de hits e uma amostra de 10 resultados por padrão.  

#### Obter o número exato de hits
Para melhorar o tempo de resposta em conjuntos de dados grandes, o Elasticsearch limita a contagem total para 10.000 por padrão. Se você deseja o número total exato, utilize a consulta a seguir:

Sintaxe:
```
GET nome_do_indice/_search
{
  "track_total_hits": true
}
```
Exemplo:
```
GET news_headlines/_search
{
  "track_total_hits": true
}
```

#### Busca por dados em um intervalo de tempo específico
Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "Especifique o tipo de consulta aqui": {
      "O nome do campo aqui": {
        "gte": "O menor valor da faixa aqui",
        "lte": "O maior valor da faixa aqui"
      }
    }
  }
}
```
Exemplo:

```
GET news_headlines/_search
{
  "query": {
    "range": {
      "date": {
        "gte": "2015-06-20",
        "lte": "2015-09-22"
      }
    }
  }
}
```

### Agregações

Uma agregação resume seus dados como métricas, estatísticas e outras análises. 

#### Analisar os dados para mostrar as categorias de manchetes de notícias em nosso conjunto de dados
Sintaxe:
```
GET nome_do_indice/_search
{
  "aggs": {
    "nomeie a agregação": {
      "especifique o tipo": {
        "field": "nome do campo que deseja agregar",
        "size": informe quantos buckets você deseja retornar
      }
    }
  }
}
```
Exemplo:
```
GET news_headlines/_search
{
  "aggs": {
    "by_category": {
      "terms": {
        "field": "category",
        "size": 100
      }
    }
  }
}
```

### Uma combinação de consulta e agregação

#### Pesquisa o termo mais significativo em uma categoria

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "match": {
      "nome do campo": "valor que está procurando"
    }
  },
  "aggregations": {
    "nomeie a sua agregação": {
      "significant_text": {
        "field": "O nome do campo no qual está buscando"
      }
    }
  }
}
```
Exemplo:
```
GET news_headlines/_search
{
  "query": {
    "match": {
      "category": "ENTERTAINMENT"
    }
  },
  "aggregations": {
    "popular_in_entertainment": {
      "significant_text": {
        "field": "headline"
      }
    }
  }
}
```


### Precisão e Recall

#### Aumentando o Recall

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "match": {
      "Especifique o campo que quer procurar": {
        "query": "Adicione os termos de busca"
      }
    }
  }
}
```
Exemplo:
```
GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Khloe Kardashian Kendall Jenner"
      }
    }
  }
}
```
 

Por padrão, a consulta de correspondência usa uma lógica "OR". Se um documento contiver um dos termos de pesquisa, o Elasticsearch considerará esse documento como um acerto (hit).

A lógica "OR" resulta em um número maior de acertos, aumentando assim o Recall. No entanto, os acertos são vagamente relacionados à consulta e, como resultado, diminuem a precisão. 

#### Aumentando a Pecisão

Podemos aumentar a precisão adicionando um operador "and" à consulta. 

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "match": {
      "Especifique o campo em que quer buscar": {
        "query": "Adicione os termos de busca",
        "operator": "and"
      }
    }
  }
}
```

Exemplo: 
```
GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Khloe Kardashian Kendall Jenner",
        "operator": "and"
      }
    }
  }
}
```

O operador "AND" resultará em correspondências mais precisas, aumentando assim a precisão. No entanto, reduzirá o número de ocorrências retornadas, resultando em menor recall. 

#### minimum_should_match
Este parâmetro permite que você especifique o número mínimo de termos que um documento deve ter para ser incluído nos resultados da pesquisa.

Este parâmetro dá a você mais controle sobre o ajuste fino da precisão do recall da sua pesquisa.

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Adicione os termos da busca",
        "minimum_should_match": Número de documentos
      }
    }
  }
}
```
Exemplo: 
```
GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Khloe Kardashian Kendall Jenner",
        "minimum_should_match": 3
      }
    }
  }
}
```

Com o parâmetro minimum_should_match, conseguimos ajustar tanto a precisão quanto o recall!

## Consultas de texto completo
### Procurando por termos de pesquisa

A `match query` é uma consulta padrão para executar uma pesquisa de texto completo. Esta `query` recupera documentos que contêm os termos de pesquisa. Ela usa a lógica "OR" por padrão, o que significa que ela recuperará documentos que contêm qualquer um dos termos de pesquisa. A ordem e a proximidade em que os termos de pesquisa são encontrados (ou seja, frases) não são levadas em consideração.

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "match": {
      "Especifique o campo que deseja pesquisar": {
        "query": "Adicione os termos de busca"
      }
    }
  }
}
```

###  Procurando por uma frase
#### O que acontece quando você usa a `match query` para pesquisar frases?
Vamos procurar artigos sobre a música "Shape of you" de Ed Sheeran usando a `match query`.

Exemplo: 
```
GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Shape of you"
      }
    }
  }
}
```
O Elasticsearch retorna mais de 10.000 ocorrências. A ocorrência principal, assim como muitas outras nos resultados da pesquisa, contém apenas os termos de pesquisa "you" e "shape". Esses termos não são encontrados na mesma ordem ou próximos uns dos outros como os termos de pesquisa "Shape of you".

Quando a `match query` é usada para pesquisar uma frase, ela tem alto recall, mas baixa precisão, pois retorna muitos documentos vagamente relacionados.


Junto com alguns artigos sobre a música "Shape of you", ele retorna artigos sobre estar em forma ou o que o formato do seu rosto diz sobre você.

Quando a `match query` é usada para pesquisar uma frase, ela tem alta recuperação, mas baixa precisão. Ela puxa documentos mais vagamente relacionados, pois usa a lógica "OR" por padrão. Ela puxa documentos que contêm qualquer um dos termos de pesquisa no campo especificado. Além disso, a ordem e a proximidade em que os termos de pesquisa são encontrados não são levados em consideração.

#### Procurando frases usando a consulta `match_phrase`

Se a ordem e a proximidade em que os termos de pesquisa são encontrados (por exemplo, frases) forem importantes para determinar a relevância da sua pesquisa, use a `match_phrase query`.

Sintaxe: 
```
GET nome_do_indice/_search
{
  "query": {
    "match_phrase": {
      "Especifique o campo que deseja pesquisar": {
        "query": "Adicione os termos da busca"
      }
    }
  }
}
```
Exemplo: 
```
GET news_headlines/_search
{
  "query": {
    "match_phrase": {
      "headline": {
        "query": "Shape of You"
      }
    }
  }
}
```
Quando o parâmetro `match_phrase` é usado, todos os resultados devem atender aos seguintes critérios:
1. os termos de pesquisa "Shape", "of" e "you" devem aparecer no campo headline.
2. os termos devem aparecer nessa ordem.
3. os termos devem aparecer um ao lado do outro.


O parâmetro `match_phrase` produz maior precisão, mas menor recall, pois leva em consideração a ordem e a proximidade em que os termos de pesquisa são encontrados.


### Executando uma consulta de match em vários campos 

Ao projetar uma `query`, você nem sempre sabe o contexto da pesquisa de um usuário. Quando um usuário pesquisa por "Michelle Obama", o usuário pode estar pesquisando por declarações escritas por Michelle Obama ou artigos escritos sobre ela.

Para atender esses contextos, você pode escrever uma `multi_match query`, que pesquisa termos em vários campos.

A `multi_match query` executa uma `match query` em vários campos e calcula uma pontuação para cada campo. Em seguida, atribui a pontuação mais alta entre os campos ao documento.

Essa pontuação determinará a classificação do documento nos resultados da pesquisa.

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "multi_match": {
      "query": "Adicione os termos de busca",
      "fields": [
        "Liste o campo que quer pesquisar",
        "Liste o campo que quer pesquisar",
        "Liste o campo que quer pesquisar"
      ]
    }
  }
}
```
A seguinte `multi_match query` solicita ao Elasticsearch para `consultar` documentos que contenham os termos de pesquisa "Michelle" ou "Obama" nos campos headline, ou short_description, ou authors.

Exemplo:
```
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query": "Michelle Obama",
      "fields": [
        "headline",
        "short_description",
        "authors"
      ]
    }
  }
}
```
Enquanto a `multi_match query` aumentou o recall, ela diminuiu a precisão dos hits.

Por exemplo, em nossa busca por manchetes relacionadas a "Michelle Obama", o hit mais popular é uma manchete de notícias com Bernie Sanders como o tópico principal. Nesta manchete, Michelle Obama é mencionada uma vez no campo short_description.

#### Boosting por campo
Manchetes que mencionam "Michelle Obama" no campo headline têm mais probabilidade de estar relacionadas à nossa pesquisa do que as manchetes que mencionam "Michelle Obama" uma ou duas vezes no campo short_description.

Para melhorar a precisão da sua pesquisa, você pode designar um campo para ter mais peso do que os outros.

Isso pode ser feito aumentando (boosting) a pontuação do campo headline(`per-field boosting`). Isso é notado adicionando um símbolo de quilate (^) e o número 2 ao campo desejado, conforme mostrado abaixo.

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "multi_match": {
      "query": "Adicione os termos de busca",
      "fields": [
        "Liste o campo que você quer aumentar^2",
        "Liste o campo que você quer pesquisar",
        "Liste o campo que você quer pesquisar"
      ]
    }
  }
}
```

O exemplo a seguir aumenta a pontuação de documentos que contêm os termos de pesquisa no campo headline. Se o termo "Michelle" ou "Obama" for encontrado no campo headline de um documento, esse documento recebe uma pontuação maior e é classificado mais alto nos resultados da pesquisa.

Exemplo:
```
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query": "Michelle Obama",
      "fields": [
        "headline^2",
        "short_description",
        "authors"
      ]
    }
  }
}
```

`Per-field boosting` produz o mesmo número de hits. No entanto, ele altera a classificação dos hits. Os hits classificados mais alto na lista contêm os termos de busca "Michelle Obama" no campo aumentado, manchete.


#### Melhorando a precisão com match de tipo de frase
 
Você pode melhorar a precisão de uma `multi_match query` adicionando o "type":"phrase" à consulta.

O tipo de frase executa uma `match_phrase query` em cada campo e calcula uma pontuação para cada campo. Então, ele atribui a pontuação mais alta entre os campos ao documento.

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "multi_match": {
      "query": "Adicione os termos de busca",
      "fields": [
        "Liste o campo que você quer aumentar^2",
        "Liste o campo que você quer pesquisar",
        "Liste o campo que você quer pesquisar"
      ],
      "type": "phrase"
    }
  }
}
```
A consulta a seguir solicita que o Elasticsearch procure a frase "party planning" nos campos headline e short_description.

Usando `per field boosting`, esta consulta atribui uma pontuação mais alta a documentos que contêm a frase "party planning" no campo headline. Os documentos que incluem a frase "party planning" no campo headline serão classificados mais alto nos resultados da pesquisa.

Exemplo:
```
GET news_headlines/_search
{
  "query": {
    "multi_match": {
      "query": "party planning",
      "fields": [
        "headline^2",
        "short_description"
      ],
      "type": "phrase"
    }
  }
}
```

Os resultados que têm a frase "planejamento de festa" no título do campo impulsionado são classificados em posições mais altas nos resultados da pesquisa e apresentados no topo dos resultados da pesquisa.

## Consultas combinadas

Haverá momentos em que um usuário fará uma pergunta multifacetada que exigirá várias `consultas` para responder.

Por exemplo, um usuário pode querer encontrar manchetes políticas sobre Michelle Obama publicadas antes do ano de 2016.

Esta pesquisa é, na verdade, uma combinação de três `consultas`:

1) Consultar manchetes que contenham os termos de pesquisa "Michelle Obama" no campo headline.
2) Consultar manchetes "Michelle Obama" da categoria "POLITICS".
3) Consultar manchetes "Michelle Obama" publicadas antes do ano de 2016.

Uma das maneiras de combinar essas consultas é por meio de uma `consulta bool`.

### Bool Query
A consulta bool recupera documentos que correspondem a combinações booleanas de outras `consultas`.

Com a `bool query`, você pode combinar várias `queries` em uma solicitação e especificar ainda mais cláusulas booleanas para restringir os resultados da pesquisa.

Há quatro cláusulas para escolher:

1. must
2. must_not
3. should
4. filter

Você pode criar combinações de uma ou mais dessas cláusulas.
Cada cláusula pode conter uma ou várias consultas que especificam os critérios de cada cláusula.

Essas cláusulas são opcionais e podem ser misturadas e combinadas para atender ao seu caso de uso. A ordem em que elas aparecem também não importa!

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "bool": {
      "must": [
        {Uma ou mais consultas podem ser especificadas aqui. Um documento DEVE corresponder a todas essas consultas para ser considerado um acerto.}
      ],
      "must_not": [
        {Um documento NÃO deve corresponder a nenhuma das consultas especificadas aqui. Se corresponder, ele será excluído dos resultados da pesquisa.}
      ],
      "should": [
        {Um documento não precisa corresponder a nenhuma consulta especificada aqui. No entanto, se corresponder, esse documento recebe uma pontuação maior.}
      ],
      "filter": [
        {Esses filtros (consultas) colocam documentos na categoria sim ou não. Aqueles que se enquadram na categoria sim são incluídos nos hits.}
      ]
    }
  }
}
```
#### Uma combinação de consulta e agregação

Uma `consulta bool` pode ajudar você a responder perguntas multifacetadas. Antes de passarmos pelas quatro cláusulas da consulta bool, precisamos primeiro entender que tipo de perguntas podemos fazer sobre Michelle Obama.

Vamos primeiro descobrir quais manchetes foram escritas sobre ela.

Uma maneira de entender isso é pesquisando categorias de manchetes que mencionam Michelle Obama.

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "Insira correspondência ou frase_correspondência aqui": { "Digite o nome do campo": "Digite o valor que você está procurando" }
  },
  "aggregations": {
    "Dê um nome à sua agregação": {
      "Especifique o tipo de agregação": {
        "field": "Nomeie o campo que deseja agregar",
        "size": Informe quantos buckets deseja retornar
      }
    }
  }
}
```
A consulta a seguir pede ao Elasticsearch para consultar todos os dados que têm a frase "Michelle Obama" no título. Em seguida, execute agregações nos dados consultados e recupere até 100 categorias que existem nos dados consultados.

Exemplo:
```
GET news_headlines/_search
{
  "query": {
    "match_phrase": {
      "headline": "Michelle Obama"
    }
  },
  "aggregations": {
    "category_mentions": {
      "terms": {
        "field": "category",
        "size": 100
      }
    }
  }
}
```
Ao minimizar o campo hits, você verá um relatório de agregações chamado category_mentions. Este relatório exibe uma matriz de todas as categorias que existem nos dados consultados e o número de manchetes que foram escritas sobre cada categoria.

Vemos que muitas manchetes de notícias sobre Michelle Obama foram escritas em categorias como "POLÍTICA", "VOZES NEGRAS", "CRIAÇÃO DE FILHOS", "GOSTO" e até mesmo "CASAMENTOS"!
("POLITICS", "BLACK VOICES", "PARENTING", "TASTE", "WEDDINGS")

**Agora vamos voltar para a consulta bool!**

Com a `consulta bool`, você pode combinar várias `consultas` em uma solicitação e especificar ainda mais cláusulas booleanas para restringir seus resultados de pesquisa.

Há quatro cláusulas para escolher:

- must
- must_not
- should
- filter

#### A cláusula must
A `cláusula must` define todas as `queries` (critérios) que um documento DEVE corresponder para ser retornado como hits. Esses critérios são expressos na forma de uma ou múltiplas `queries`.

Todas as `queries` na `cláusula must` devem ser satisfeitas para que um documento seja retornado como um hit. Como resultado, ter mais `queries` na `cláusula must` aumentará a precisão da sua `query`. 

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "Digite a correspondência ou a frase_de_correspondência aqui": {
        "Digite o nome do campo": "Digite o valor que você está procurando"
        }
    },
    {
        "Digite a correspondência ou a frase_de_correspondência aqui": {
        "Digite o nome do campo": "Digite o valor que você está procurando"
    }
    }
    ]
    }
  }
}
```
A seguir está uma `consulta bool` que usa a `cláusula must`. Esta `consulta` especifica que todos os resultados devem corresponder à frase "Michelle Obama" no título do campo e corresponder ao termo "POLITICS" na categoria do campo.

Exemplo:
```
GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "match_phrase": {
          "headline": "Michelle Obama"
         }
        },
        {
          "match": {
            "category": "POLITICS"
          }
        }
      ]
    }
  }
}
```
Todos os documentos contêm a frase "Michelle Obama" no título do campo e o termo "POLÍTICA" na categoria do campo.

#### A cláusula must_not
A cláusula `must_not` define `consultas` (critérios) que um documento NÃO DEVE corresponder para ser incluído nos resultados da pesquisa.

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "Digite a correspondência ou match_phrase aqui": {
        "Digite o nome do campo": "Digite o valor que você está procurando"
        }
    },
    "must_not":[
    {
    "Digite a correspondência ou match_phrase aqui": {
    "Digite o nome do campo": "Digite o valor que você está procurando"
    }
    }
    ]
    }
  }
}
```
E se você quiser todas as manchetes de Michelle Obama, exceto aquelas que pertencem à categoria "WEDDINGS"?

A seguinte `consulta bool` especifica que todos os resultados devem conter a frase "Michelle Obama" no campo título. No entanto, os resultados `must_not` contêm o termo "WEDDINGS" na categoria do campo.

Exemplo:

```
GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": {
        "match_phrase": {
          "headline": "Michelle Obama"
         }
        },
       "must_not":[
         {
          "match": {
            "category": "WEDDINGS"
          }
        }
      ]
    }
  }
}
```
Esta `query` aumenta o recall. Ela retorna todos os hits que contêm a frase "Michelle Obama" no campo headline. Entre os hits, o Elasticsearch exclui todos os documentos que contêm o termo "WEDDINGS" na categoria do campo.


#### A cláusula should

A cláusula `should` adiciona `queries` (critérios) "bom ter". Os documentos não precisam corresponder às `queries` "bom ter" para serem considerados como acertos. No entanto, os que corresponderem receberão uma pontuação maior para que apareçam mais alto nos resultados da pesquisa.

Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "Digite a correspondência ou match_phrase aqui": {
        "Digite o nome do campo": "Digite o valor que você está procurando"
        }
    },
    "should":[
    {
    "Digite a correspondência ou match_phrase aqui": {
    "Digite o nome do campo": "Digite o valor que você está procurando"
    }
    }
      ]
    }
  }
```

Vamos falar sobre um cenário em que podemos usar a cláusula `should`. Durante o Mês da História Negra, é possível que o usuário esteja procurando por "Michelle Obama" no contexto da categoria "BLACK VOICES" em vez de no contexto das categorias "WEDDINGS", "TASTE" ou "STYLE".

Para atender esse cenário, você pode escrever uma `query` em que todos os resultados DEVEM conter "Michelle Obama" no título do campo. Não é necessário ter a frase "BLACK VOICES" na categoria. No entanto, se um documento contiver a frase "BLACK VOICES" na categoria do campo, esse documento deve receber uma pontuação mais alta e deve ser colocado em uma posição mais alta nos resultados da pesquisa.

Para isso, você escreveria a seguinte `bool query`. Ela especifica que todos os resultados devem corresponder à frase "Michelle Obama" no título do campo. `Se` um resultado corresponder à frase "BLACK VOICES" na categoria de campo, esse resultado receberá uma pontuação mais alta e será exibido em uma posição mais alta nos resultados da pesquisa.

Exemplo:
```
GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "match_phrase": {
          "headline": "Michelle Obama"
          }
         }
        ],
       "should":[
         {
          "match_phrase": {
            "category": "BLACK VOICES"
          }
        }
      ]
    }
  }
}
```

Ainda devemos obter o mesmo número de ocorrências, pois a cláusula `should` não adiciona ou exclui mais ocorrências. No entanto, você notará que a classificação dos documentos foi alterada. Os documentos com a frase "BLACK VOICES" na categoria de campo agora são apresentados no topo dos resultados da pesquisa.


#### A cláusula filter
A `cláusula filter` contém `consultas` de filtro que colocam documentos na categoria "sim" ou "não".

Por exemplo, digamos que você esteja procurando por manchetes publicadas dentro de um determinado intervalo de tempo. Alguns documentos se enquadrarão nesse intervalo (sim) ou não se enquadrarão nesse intervalo (não).

A `cláusula filter` inclui apenas documentos que se enquadram na categoria sim.
Sintaxe:
```
GET nome_do_indice/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "Insira correspondência ou frase_correspondência aqui": {
          "Digite o nome do campo": "Insira o valor que você está procurando" 
         }
        }
        ],
       "filter":{
          "range":{
             "date": {
               "gte": "Insira o menor valor do intervalo aqui",
               "lte": "Insira o valor mais alto do intervalo aqui"
          }
        }
      }
    }
  }
}
```
Digamos que queremos recuperar hits que devem incluir a frase "Michelle Obama" no campo headline. Entre esses hits, você quer incluir documentos publicados dentro do intervalo de datas "2014-03-25" e "2016-03-25".

Sua `consulta bool` será parecida com esta.

Exemplo:
```
GET news_headlines/_search
{
  "query": {
    "bool": {
      "must": [
        {
        "match_phrase": {
          "headline": "Michelle Obama"
          }
         }
        ],
       "filter":{
          "range":{
             "date": {
               "gte": "2014-03-25",
               "lte": "2016-03-25"
          }
        }
      }
    }
  }
}
```

Todos os hits contêm a frase "Michelle Obama" no campo headline. Todos os hits foram publicados entre o intervalo de datas que especificamos na `cláusula filter`.