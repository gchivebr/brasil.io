Esse README explicita os passos para importar o dataset de sócios para o Neo4J.

### Populando o Postgres
Para importar os dados, é necessário antes ter o dataset carregado no postgres. Para isso é necessário [baixar o dump](https://brasil.io/dataset/socios-brasil) e depois executar o seguinte comando:

```
$ python manage.py import_data socios-brasil socios-brasil.csv.xz
```

E relaxa que demora **mesmo**! Se quiser agilizar, dá para comentar a [criação dos índices de busca](https://github.com/turicas/brasil.io/blob/develop/core/management/commands/import_data.py#L114) do commando `import_data`.

### Populando o Neo4J
Com os dados no Postgres, basta executar

```
$ python manage.py import_socios_to_graph
```

Na minha máquina rodou em **142 minutos**:
![screenshot from 2018-04-26 22-24-40](https://user-images.githubusercontent.com/238223/39340934-426abae0-49a7-11e8-893c-bb1355626526.png)

Para marcar as empresas mãe, também temos um comando do `manage.py` que é **muito** mais rápido:

```
$ python manage.py build_company_groups_network
Atualizando nós que são empresas mães com a query:

            MATCH (e:PessoaJuridica)-[:TEM_SOCIEDADE]->(:PessoaJuridica)
            WITH DISTINCT e as empresa, COLLECT(e) as companies
            WHERE ALL(c in companies WHERE NOT (:PessoaJuridica)-[:TEM_SOCIEDADE]->(c))
            SET empresa :EmpresaMae
            RETURN COUNT(empresa)
        
Importação realizada com sucesso terminada.
  + 112714 empresas consideradas empresas mãe.
  + Finalizado em 16 segundos
```

### Acessando
Para acessar os dados é necessário ter o docker pro Neo4j de pé e acessar http://localhost:39002/browser/. Terá uma página de login e para acessar você deve somete alterar o **host** para `bolt://localhost:39003`. Não é necessário entrar com usuário ou senha.


### Sobre o Grafo

O grafo é composto por um único tipo de relacionamento chamado `TEM_SOCIEDADE` e os seguintes labels pros nós:
- `PessoaFisica`
- `PessoaJuridica`
- `NomeExterior`

As relações são sempre de qualquer um desses 3 labels para um outro nó `PessoaJuridica`.

![screenshot from 2018-04-26 23-20-20](https://user-images.githubusercontent.com/238223/39341207-6f0d8ca2-49a8-11e8-9a3d-3949010f1dc8.png)

Por fim, a query a seguir mostra que o Neo4J tem o mesmo número de relacionamentos (entradas) que no banco postgres:

![screenshot from 2018-04-26 22-25-44](https://user-images.githubusercontent.com/238223/39341218-76e848fe-49a8-11e8-8242-00db6f8ebc63.png)


### API

Exemplos para os métodos da API de grafos (rodando localmente):

- Detalhes do nó TIM BRASIL S/A: [/api/especiais/grafo/no](http://localhost:8000/api/especiais/grafo/no?tipo=1&identificador=04214266000198)
- Grafo de societário da TIM BRASIL S/A: [/api/especiais/grafo/sociedades](http://localhost:8000/api/especiais/grafo/sociedades?tipo=1&identificador=04214266000198)
- Caminhos entre Odebrecht e a Rossi: [/api/especiais/grafo/sociedades/caminhos](http://localhost:8000/api/especiais/grafo/sociedades/caminhos?tipo1=1&identificador1=15102288000182&tipo2=1&identificador2=61065751000180)
- Todas a árvore de empresas sócias partindo da Odebrecht: [/api/especiais/grafo/sociedades/subsequentes](http://localhost:8000/api/especiais/grafo/sociedades/subsequentes?identificador=15102288000182)
- Caminhos para as empresas mãe partindo do CONSORCIO ANDRADE GUTIERREZ / CBPO LOTE 3: [/api/especiais/grafo/sociedades/empresas-mae](http://localhost:8000/api/especiais/grafo/sociedades/empresas-mae?identificador=02447044000190)
