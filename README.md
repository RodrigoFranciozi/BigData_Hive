# Projeto_BigData

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


## Passo a passo (Hive):

### Iniciando os nodes do Hadoop:

`start-dfs.sh`

`start-yarn.sh`


Verificando os nodes que iniciaram:

`jps`

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/nodes.png)


Verificando as pastas existentes dentro do HDFS:

`hdfs dfs -ls /`


Criando uma pasta chamada "projeto" dentro do da pasta datasets do HDFS:

`hdfs dfs -mkdir /datasets/projeto`

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/criacao_pasta_projeto.png)


Copiando os 2 arquivos necessarios para a pasta projetos que acaba de ser criada. 

**(Ps: Nao esquecer de entrar na pasta onde os arquivos se encontram, caso o contrario voce nao sera capaz de copialos para dentro do HDFS.)**

`cd Downloads`

`hdfs dfs -put join.csv /datasets/projeto`

`hdfs dfs -put PedologiaAmazoniaLegal_250.csv /datasets/projeto`


Com isso voce sera capaz de encontrar os 2 arquivos dentro do HDFS:

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/encontrar_arquivo_hdfs.png)


### Iniciando o Apache Hive:

`beeline`


Conectando com o Hive:

`!connect jdbc:hive2://`

username e o password nesse caso é só apertar a tecla enter duas vezes para proceguir. 

Assim que se conectar com sucesso, a seguinte mensagem aparecerá:

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/sucesso_conexao.png)


Criando um banco chamado "projeto":

`CREATE DATABASE projeto`


Listando os bancos dentro do hive:

`show databases;`

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/mostrando_databases.png)


Mudando o banco a ser utilizado:

`USE projeto;`


Criando as duas tabelas externas dentro de projeto:

`CREATE EXTERNAL TABLE unir( folha string, cor string, fazendeiro string) row format delimited fields terminated by ',' stored as textfile TBLPROPERTIES( "skip.header.line.count"="1");`

`CREATE EXTERNAL TABLE solos( FID string, Folha string,Unidade_Mapeamento string, Quantidade_Componentes int, Solo_ou_Terreno string, Simbolo_Unidade string, Ordem string, Subordem string, Grande_Grupo string, Subgrupo string, Descricao string, Inclusoes string, Textura_Principal_Componente string, Horizonte_Principal_Componente string, Erosao_Principal_Componente string, Pedregosidade_Principal_Componente string, Rochosidade_Principal_Componente string, Relevo_Principal_Componente string, Area_Poligono_Km2 double, the_geom string) row format delimited fields terminated by ',' stored as textfile TBLPROPERTIES( "skip.header.line.count"="1");`


Populando as tabelas:

`LOAD DATA INPATH '/datasets/projeto/join.csv' INTO TABLE unir;`

`DESCRIBE unir`

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/descrevendo_unir.png)

`LOAD DATA INPATH '/datasets/projeto/PedologiaAmazoniaLegal_250.csv' INTO TABLE solos;`

`DESCRIBE solos`

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/descrevendo_solos.png)


Conferindo se as tabelas estão populadas:

`SELECT folha, quantidade_componentes, the_geom FROM solos LIMIT 10;`

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/select_solos_teste.png)

`SELECT * FROM unir;`

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/select_unir_teste.png)


Construindo uma tabela com um join para analises futuras:

`SELECT solos.folha, solos.unidade_mapeamento, solos.quantidade_componentes, unir.cor, unir.fazendeiro FROM solos JOIN unir ON (solos.folha = unir.folha) LIMIT 10;`

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/construindo_join.png)


Salvando em uma tabela:

`CREATE TABLE juncao_dados AS SELECT solos.folha, solos.unidade_mapeamento, solos.quantidade_componentes, unir.cor, unir.fazendeiro FROM solos JOIN unir ON (solos.folha = unir.folha);`


Verificando se a tabela foi criada:

`show tables;`

![](https://github.com/RodrigoFranciozi/projeto_BigData/blob/main/Hive/imgs/verificando_join_criado.png)


Saindo do editor do hive:

`!q`


Desligando os nodes:

`stop-yarn.sh`

`stop-dfs.sh`


FIM 🙂
