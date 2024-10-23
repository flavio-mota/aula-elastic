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
Expected response from Elasticsearch:

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

