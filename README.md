# Projeto_BigData
Here we have a simple project using some concepts of Big Data

## Spark

Dentro dessa pasta, tem uma simples base (pequena) que contém partidas de tênis femininos. 
Foi feita uma simples análise (respondendo 2 perguntas), dentro do Colab, usando um pouco
dos conceitos do Spark.

## Hive

Nessa pasta existe um unico arquivo .csv adapatado por mim, usando o pandas com dados inventados. 
A ideia seria usar o Hive e fazer um Join com esse arquivo juntamente com o arquivo do link, que mostra dados
sobre o solo da Amazônia, colocando o passo a passo de como implementar dentro da VM / Linux.  

Link dos dados reais:
 
https://dados.gov.br/dataset/cren_pedologiaamazonialegal_250/resource/30a23fae-c13b-4c4a-8b07-de9e66582b06?inner_span=True


### Passo a passo:

Iniciando os nodes do Hadoop:

`start-dfs.sh`

`start-yarn.sh`

Verificando os nodes que iniciaram:

`jps`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/70d6ff89-8807-4d8a-a7e3-1a231cad5ab3/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/70d6ff89-8807-4d8a-a7e3-1a231cad5ab3/Untitled.png)

Verificando as pastas existentes dentro do HDFS:

`hdfs dfs -ls /`

criando uma pasta chamada "projeto" dentro do da pasta datasets do HDFS:

`hdfs dfs -mkdir /datasets/projeto`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d3c1f0e6-d216-47bb-b387-8148dcba298f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d3c1f0e6-d216-47bb-b387-8148dcba298f/Untitled.png)

Copiando os 2 arquivos necessarios para a pasta projetos que acaba de ser criada. 

Ps: Nao esquecer de entrar na pasta onde os arquivos se encontram, caso o contrario voce nao sera capaz de copialos para dentro do HDFS.

`cd Downloads`

`hdfs dfs -put join.csv /datasets/projeto`

`hdfs dfs -put PedologiaAmazoniaLegal_250.csv /datasets/projeto`

Com isso voce sera capaz de encontrar os 2 arquivos dentro do HDFS:

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/07abf8eb-aaa7-45e4-8e3d-7946c4736c92/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/07abf8eb-aaa7-45e4-8e3d-7946c4736c92/Untitled.png)

Iniciando o Apache Hive:

`beeline`

Conectando com o Hive:

`!connect jdbc:hive2://`

username e o password nesse caso é só apertar a tecla enter duas vezes para proceguir. 

Assim que se conectar com sucesso, a seguinte mensagem aparecerá:

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea61a83d-8640-47d1-8a68-12eb9b908fbe/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea61a83d-8640-47d1-8a68-12eb9b908fbe/Untitled.png)

Criando um banco chamado "projeto":

`CREATE DATABASE projeto`

Listando os bancos dentro do hive:

`show databases;`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6994992a-3de7-4f67-b697-3b2935060fd4/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6994992a-3de7-4f67-b697-3b2935060fd4/Untitled.png)

Mudando o banco a ser utilizado:

`USE projeto;`

Criando as duas tabelas externas dentro de projeto:

`CREATE EXTERNAL TABLE unir( folha string, cor string, fazendeiro string) row format delimited fields terminated by ',' stored as textfile TBLPROPERTIES( "skip.header.line.count"="1");`

`CREATE EXTERNAL TABLE solos( FID string, Folha string,Unidade_Mapeamento string, Quantidade_Componentes int, Solo_ou_Terreno string, Simbolo_Unidade string, Ordem string, Subordem string, Grande_Grupo string, Subgrupo string, Descricao string, Inclusoes string, Textura_Principal_Componente string, Horizonte_Principal_Componente string, Erosao_Principal_Componente string, Pedregosidade_Principal_Componente string, Rochosidade_Principal_Componente string, Relevo_Principal_Componente string, Area_Poligono_Km2 double, the_geom string) row format delimited fields terminated by ',' stored as textfile TBLPROPERTIES( "skip.header.line.count"="1");`

Populando as tabelas:

`LOAD DATA INPATH '/datasets/projeto/join.csv' INTO TABLE unir;`

`DESCRIBE unir`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3d6a51db-eeab-41fc-aad6-5c9d242d36f4/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3d6a51db-eeab-41fc-aad6-5c9d242d36f4/Untitled.png)

`LOAD DATA INPATH '/datasets/projeto/PedologiaAmazoniaLegal_250.csv' INTO TABLE solos;`

`DESCRIBE solos`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/195b2086-9e1c-472e-8ec0-783037c93649/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/195b2086-9e1c-472e-8ec0-783037c93649/Untitled.png)

Conferindo se as tabelas estão populadas:

`SELECT folha, quantidade_componentes, the_geom FROM solos LIMIT 10;`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7fd0e08e-b3cb-436d-8244-0cae8b07ed8c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7fd0e08e-b3cb-436d-8244-0cae8b07ed8c/Untitled.png)

`SELECT * FROM unir;`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/99186126-06a5-4810-9c49-c95720a0f2bb/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/99186126-06a5-4810-9c49-c95720a0f2bb/Untitled.png)

Construindo uma tabela com um join para analises futuras:

`SELECT solos.folha, solos.unidade_mapeamento, solos.quantidade_componentes, unir.cor, unir.fazendeiro FROM solos JOIN unir ON (solos.folha = unir.folha) LIMIT 10;`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/64bd72b4-5660-4f97-82d4-0a5677a57b53/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/64bd72b4-5660-4f97-82d4-0a5677a57b53/Untitled.png)

Salvando em uma tabela:

`CREATE TABLE juncao_dados AS SELECT solos.folha, solos.unidade_mapeamento, solos.quantidade_componentes, unir.cor, unir.fazendeiro FROM solos JOIN unir ON (solos.folha = unir.folha);`

Verificando se a tabela foi criada:

`show tables;`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4a3cc68f-1a04-41dc-bd24-69e6f428e9fa/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4a3cc68f-1a04-41dc-bd24-69e6f428e9fa/Untitled.png)

Saindo do editor do hive:

`!q`

Desligando os nodes:

`stop-yarn.sh`

`stop-dfs.sh`

FIM 🙂

