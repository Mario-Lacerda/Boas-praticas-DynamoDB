# Desafio de Projeto - Boas práticas com DynamoDB

Trabalhando com Banco de Dados SQL e NoSQL do Bootcamp Banco PAN JAVA Developer aplicando os comandos de pesquisa (query) no banco de dados utilizando o CLI no Windows (durante a explicação do projeto utilizou-se o Linux).

No Windows, na etapa de definição da query após a --expression-atribute-values o Windows (PowerShell) não entende os atributos passados para busca ( Exemplo: --expression-atribute-values '{":artist":{"S":"Iron Maiden"}}' ). Com documentos json que podem ser excluidos após a execução da query (podem ser visualizados na pasta src), ajustei o comando de query modificando o final para ler esses arquivos --expression-atribute-values file://tempartist.json.



## Arquitetura

<h1 align=center>Boas práticas com DynamoDB</h1>

>Características Relacionais (SQL) e Não Relacionais (NoSQL) usando o mesmo banco de dados? Isso é possível? Com o DynamoDB sim! Entenda um pouco das possibilidades desse banco de dados totalmente gerenciado da AWS. Apresentação de uma série de boas práticas para o DynamoDB em um desafio totalmente prático. Sendo assim, você poderá reproduzir essa solução e, caso queira ir além, implementar suas próprias evoluções e melhorias.
>
>

<p align=center>
  <img width=80% src="https://user-images.githubusercontent.com/13790608/230742699-44aa9cfd-a0db-4d8f-99dd-a5f84d4ed866.png"/>
</p>

A imagem acima representa a arquitetura da proposta de construção desse banco de dados, tendo como referência o projeto <a href=https://github.com/cassianobrexbit/dio-live-dynamodb>dio-live-dynamodb</a> do instrutor Cassiano Peres da DIO. O projeto toma como exemplo o mundo da música, onde temos as entidades Artista, Álbum e Música.

Além das entidades, o diagrama apresenta as relações e cardinalidade entre elas:
- 1 Artista pode ter muitos Álbuns
- 1 Álbum pode ter muitas Músicas
- 1 Música pode pertencer a mais de 1 Álbum

###### 
### Serviço utilizado

- Amazon DynamoDB
- Amazon CLI para execução

### Comandos para execução do experimento:

- Criar uma tabela

```
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

- Inserir um item

```
aws dynamodb put-item \
    --table-name Music \
    --item file://itemmusic.json \
```

- Inserir múltiplos itens

```
aws dynamodb batch-write-item \
    --request-items file://batchmusic.json
```

- Criar um index global secundário baseado no título do álbum

```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no nome do artista e no título do álbum

```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=Artist,AttributeType=S \
        AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ArtistAlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"Artist\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Criar um index global secundário baseado no título da música e no ano

```
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=SongTitle,AttributeType=S \
        AttributeName=SongYear,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"SongTitleYear-index\",\"KeySchema\":[{\"AttributeName\":\"SongTitle\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"SongYear\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

- Pesquisar item por artista

```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist" \
    --expression-attribute-values  '{":artist":{"S":"Iron Maiden"}}'
```

- Pesquisar item por artista e título da música

```
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist and SongTitle = :title" \
    --expression-attribute-values file://keyconditions.json
```

- Pesquisar pelo index secundário baseado no título do álbum

```
aws dynamodb query \
    --table-name Music \
    --index-name AlbumTitle-index \
    --key-condition-expression "AlbumTitle = :name" \
    --expression-attribute-values  '{":name":{"S":"Fear of the Dark"}}'
```

- Pesquisar pelo index secundário baseado no nome do artista e no título do álbum

```
aws dynamodb query \
    --table-name Music \
    --index-name ArtistAlbumTitle-index \
    --key-condition-expression "Artist = :v_artist and AlbumTitle = :v_title" \
    --expression-attribute-values  '{":v_artist":{"S":"Iron Maiden"},":v_title":{"S":"Fear of the Dark"} }'
```

- Pesquisar pelo index secundário baseado no título da música e no ano

```
aws dynamodb query \
    --table-name Music \
    --index-name SongTitleYear-index \
    --key-condition-expression "SongTitle = :v_song and SongYear = :v_year" \
    --expression-attribute-values  '{":v_song":{"S":"Wasting Love"},":v_year":{"S":"1992"} }'
```
