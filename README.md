# CursoSQL_OracleFoundations
# Episódio 1: Tabelas

## 1.1 Heap Tables (Tabela Sem Índice Clusterizado)

**DEF.:** Os dados não são armazenados em nenhuma ordem específica e ficam gravados na ordem em que foram inseridos, sem garantia de sequência lógica.

Sem índice clusterizado, o SQL Server precisa usar o Table Scan ou o Index Seek/Scan. Apesar de garantir inserções mais rápidas, as consultas e buscas por dados são mais lentas já que é necessário realizar uma varredura completa da tabela. Além disso, operações de exclusão e atualização podem gerar **fragmentação** e **páginas não utilizadas**.

## Quando usar?

- Tabelas temporárias ou de *staging* (carga intermediária de dados)
- Cenários de inserção em massa (*bulk insert*) c/ ordenação desnecessária
- Dados processados e descartados rapidamente

```sql
-- Criando uma tabela heap (sem índice clusterizado)
CREATE TABLE ClientesHeap (
    Id INT NOT NULL,
    Nome NVARCHAR(100),
    Email NVARCHAR(100)
);
-- Nenhum índice clusterizado é criado aqui

-- Inserindo dados
INSERT INTO ClientesHeap (Id, Nome, Email)
VALUES (1, 'João Silva', 'joao@email.com');

-- Criando um índice não clusterizado opcional
CREATE NONCLUSTERED INDEX IX_ClientesHeap_Email
ON ClientesHeap (Email);
```

## 1.2 Index Organized Table (Tabela Organizada por Índice - IOT)

**DEF.:** É um tipo especial de tabela onde os dados são armazenados na própria estrutura do índice.

TABELA  →    guarda os dados

**ÍNDICE**  →    guarda ponteiros para os dados

Já numa IOT, os dados ficam organizados fisicamente pela chave primária obrigatória (*primary key*) dentro de uma B-tree, ou seja, a tabela é o índice.

## Quando usar?

- Realização de buscas pela chave primária
- Economizar espaço
- Leituras rápidas

No entanto, não é uma boa ideia se você atualiza muito a chave primária, faz muitas buscas por colunas que não são a *primary key* (PK) **ou precisa de inserções aleatórias em grande volume. Como os dados ficam organizados fisicamente pela PK, qualquer mudança grande pode causar reorganização da estrutura. 

```sql
CREATE TABLE clientes (
    id NUMBER,
    nome VARCHAR2(100),
    email VARCHAR2(100),
    CONSTRAINT clientes_pk PRIMARY KEY (id)
) ORGANIZATION INDEX;
```

## 1.3 External Tables (Tabelas Externas)

**DEF.:** Utilizada para acessar arquivos que não estão no banco de dados, usadas como áreas de teste antes do carregamento seus dados para as tabelas reais. 

Não armazena os dados dentro do banco, mas permite fazer **SELECT** como se fosse uma tabela normal. Ou seja, o banco só “aponta” para o arquivo e os dados continuam fora dele.

## Quando usar?

- Ler arquivos grandes sem fazer **INSERT**
- Processos de ETL: Usado para pegar dados de várias fontes, tratar/organizar esses dados e jogar num destino final.  (**EXTRAIR  →  TRANSFORMAR  →  CARREGAR)**
- Integração com sistemas legados
- Evitar carga temporária de daods

## Como funciona?

1º Criação de um *directory* (diretório) apontando para uma pasta no servidor

2º A *External Table* define o layout do arquivo

```sql
CREATE OR REPLACE DIRECTORY dados_ext AS '/home/oracle/arquivos';
```

```sql
CREATE TABLE vendas_externas (
    id NUMBER,
    produto VARCHAR2(100),
    valor NUMBER
)
ORGANIZATION EXTERNAL (
    TYPE ORACLE_LOADER
    DEFAULT DIRECTORY dados_ext
    ACCESS PARAMETERS (
        RECORDS DELIMITED BY NEWLINE
        FIELDS TERMINATED BY ','
    )
    LOCATION ('vendas.csv')
);
```

```sql
SELECT * FROM vendas_externas;
```

Possui limitações importante:

- Normalmente, é *read-only*
- Depende do arquivo existir
- Performance depende do I/O do servidor

## 1.4 Temporary Tables (Tabelas Temporárias)

**DEF.:** Armazenam dados privados para cada sessão, sem ninguém externo poder ver ou acessar a informação adicionada. Guardam dados/resultados intermediários temporariamente e servem de apoio para consultas complexas.

*Oracle*

```sql
CREATE GLOBAL TEMPORARY TABLE temp_vendas (
    id NUMBER,
    valor NUMBER
)
ON COMMIT DELETE ROWS;
```

*Microsoft SQL Server*

```sql
CREATE TABLE #temp_clientes (
    id INT,
    nome VARCHAR(100)
);
```

## Quando usar?

- Consultas muito complexas
- Processos ETL
- Cálculos intermediários
- Evitar *subqueries* grandes

**OBS! ETL x ELT**

→  ETL: transforma antes de carregar

→  ELT: carrega e, depois, transforma

No geral, as tabelas temporárias melhoram a organização da consulta, a perfomance e evitam poluir bancos com dados auxiliares.

## 1.5 Partitioned Table (Tabela Particionada)

**DEF.:** Divisão de tabelas em outras menores, onde as linhas são partidas de acordam com a *partition key*. Para o usuário, ela continua parecendo uma tabela só e isso ajuda muito na sua performance e organização.

## Tipos de Partição

1. Range Partitioning (por intervalos)
2. List Partitioning: divide por valores específicos
3. Hash Partitioning: Usa a função *hash*  e é bom para balanceamento de carga
4. Composite Partitioning: Mistura tipos de partições

## Vantagens

- Melhor performance
- Manutenção fácil
- Pode apagar uma partição inteira rápido
- Backup eficiente

## 1.6 Table Clusters (Grupo de Tabelas)

**DEF.:** Armazenam fisicamente dados relacionados juntos em blocos, facilita buscas e reduz I/O quando você faz **JOIN** entre elas.

```sql
SELECT *
FROM clientes c
JOIN pedidos p
ON c.id = p.cliente_id;
```

## Tipos de Clusters

1. Index Cluster: Usa um índice para organizar as linhas dentro do cluster.
2. Hash Cluster: Usa função *hash* para determinar onde armazenar as linhas, mais rápido para buscas por igualdade.

**OBS! O que é uma função *hash*?**

É um algoritmo que pega um dado de qualquer tamanho e transforma em um valor (*hash)* de tamanho fixo. O banco usa *hash*  para decidir em qual bloco guardar ou em qual partição colocar o registro.

## Quando usar?

- Acesso a duas ou mais tabelas juntas frequentemente
- Quando **JOIN** por igualdade é muito comum
- Relação 1:N

No entanto, não é uma boa ideia usar se quase nunca faz **JOIN**, se há muitas atualizações e se o padrão de acesso muda muito.

# Tutorial para Criação de Tabelas

Para criar uma tabela, é necessário definir o nome, a coluna e o tipo de dados inserido.

```sql
create table <table_name> (
  <column1_name> <data_type>,
  <column2_name> <data_type>,
  <column3_name> <data_type>,
  ...
)
```

```sql
create table toys (
  toy_name varchar2(100),
  weight   number
);
```

O dicionário de dados armazena informações sobre o seu banco de dados. Você pode consultá-lo para ver quais tabelas ele contém. Existem três visualizações principais com essas informações:

- user_tables  →  todas as tabelas pertencentes ao usuário atual do banco de dados
- all_tables  →  todas as tabelas às quais o usuário do seu banco de dados tem acesso
- dba_tables  →  todas as tabelas no banco de dados, disponível apenas se você tiver privilégios de DBA

```sql
select table_name, iot_name, iot_type, external, 
       partitioned, temporary, cluster_name
from   user_tables;
```

---

## Exercício 001

```sql
create table bricks ( 
		colour varchar2(10)
		shape varchar2(10)
);
```

```sql
select table_name 
from   user_tables
where  table_name = 'BRICKS';
```

---

Por padrão, as tabelas são organizadas como *heap*. Isso significa que o banco de dados pode armazenar linhas onde houver espaço disponível. Você pode adicionar a cláusula *organization heap* se quiser ser explícito.

```sql
create table toys_heap (
  toy_name varchar2(100)
) organization heap;

select table_name, iot_name, iot_type, external,
       partitioned, temporary, cluster_name
from   user_tables
where  table_name = 'TOYS_HEAP';
```

Ao contrário de uma tabela heap, uma tabela organizada por índice (IOT) impõe ordem às linhas dentro dela. Ela armazena fisicamente as linhas ordenadas pela sua chave primária. 

```sql
create table toys_iot (
  toy_id   integer primary key,
  toy_name varchar2(100)
) organization index;
```

```sql
select table_name, iot_type
from   user_tables
where  table_name = 'TOYS_IOT';
```

---

# Exercício 002

```sql
create table bricks_iot (
  bricks_id integer primary key
) 
organization index;

select table_name, iot_type
from   user_tables
where  table_name = 'BRICKS_IOT';
```

```sql
TABLE_NAME IOT_TYPE
BRICKS_IOT IOT
```

---

Você utiliza tabelas externas para ler arquivos que não estão no banco de dados no servidor de banco de dados. Por exemplo, arquivos de valores separados por vírgula (CSV).

```sql
create or replace directory toy_dir as '/path/to/file';

create table toys_ext (
  toy_name varchar2(100)
) organization external (
  default directory tmp
  location ('toys.csv')
);
```

Quando você consulta esta tabela, ela irá ler do arquivo:  /path/to/file/toys.csv

Este arquivo deve ser acessível ao servidor de banco de dados. Você não pode usar tabelas externas para ler arquivos na sua máquina!

Tabelas temporárias armazenam dados específicos da sessão. Apenas a sessão que adiciona as linhas pode vê-las. Isso pode ser útil para armazenar dados de trabalho.

Existem dois tipos de tabela temporária no Oracle Database: global e privada. Para criar uma tabela temporária global, adicione a cláusula *global temporary* entre *create* e *table*.

```sql
create global temporary table toys_gtt (
  toy_name varchar2(100)
);
```

```sql
create private temporary table ora$ptt_toys (
  toy_name varchar2(100)
);
```

Para ambos os tipos de tabela temporária, por padrão, as linhas desaparecem quando você encerra sua transação. Você pode mudar isso para quando sua sessão terminar usando a cláusula *on commit*.

Mas, de qualquer forma, ninguém mais pode visualizar as linhas. Certifique-se de copiar os dados que você precisa para tabelas permanentes antes de sua sessão terminar!

```sql
select table_name, temporary
from   user_tables
where  table_name in ( 'TOYS_GTT', 'ORA$PTT_TOYS' );
```

O particionamento divide logicamente uma tabela em tabelas menores de acordo com a(s) coluna(s) de partição. Assim, linhas com a mesma chave de partição são armazenadas no mesmo local físico.

```sql
create table toys_range (
  toy_name varchar2(100)
) partition by range ( toy_name ) (
  partition p0 values less than ('b'),
  partition p1 values less than ('c')
);

create table toys_list (
  toy_name varchar2(100)
) partition by list ( toy_name ) (
  partition p0 values ('Sir Stripypants'),
  partition p1 values ('Miss Snuggles')
);

create table toys_hash (
  toy_name varchar2(100)
) partition by hash ( toy_name ) partitions 4;
```

Por padrão, uma tabela particionada é organizada em heap. Mas você pode combinar o particionamento com algumas outras propriedades. Por exemplo, você pode ter um IOT particionado.

```sql
create table toys_part_iot (
  toy_id integer primary key,
  toy_name varchar2(100)
) organization index
partition by hash ( toy_id ) partitions 4;
```

```sql
select table_name, partitioned
from   user_tables
where  table_name in ( 'TOYS_HASH', 'TOYS_LIST', 'TOYS_RANGE', 'TOYS_PART_IOT' );

select table_name, partition_name
from   user_tab_partitions;
```

Um cluster de tabela pode armazenar linhas de várias tabelas no mesmo local físico. Para fazer isso, primeiro você deve criar o cluster:

```sql
create cluster toy_cluster (
  toy_name varchar2(100)
);
```

```sql
create table toys_cluster_tab (
  toy_name varchar2(100)
) cluster toy_cluster ( toy_name );

create table toy_owners_cluster_tab (
  owner varchar2(20),
  toy_name varchar2(100)
) cluster toy_cluster ( toy_name );
```

```sql
select cluster_name from user_clusters;

select table_name, cluster_name
from   user_tables
where  table_name in ( 'TOYS_CLUSTER_TAB', 'TOY_OWNERS_CLUSTER_TAB' );
```

Você pode remover tabelas existentes com o comando drop table. Basta adicionar o nome da tabela que você deseja destruir:

```sql
select table_name
from   user_tables
where  table_name = 'TOYS_HEAP';
```

```sql
drop table toys_heap;
```

```sql
select table_name
from   user_tables
where  table_name = 'TOYS_HEAP';
```

Uma vez que você remove uma tabela, você não pode acessá-la. Portanto, tenha cuidado com este comando!

# Estudo Extra: Oracle Docs

```sql
CREATE TABLE
```

Esse comando serve para criar tabelas na base de dados, que são estruturas onde os dados são armazenados. 

## 1. Tipos de Tabelas

**1.1 Tabela Relacional Simples:** Usada para dados tradicionais (colunas e linhas)

**1.2 Tabela de Objeto:** Usa tipos definidos pelo usuário, armazenando instâncias de objetos

**1.3 Tabela de Coluna Especial:  Por exemplo, *XML Type***

Algumas opções permitem criar a tabela a partir do resultado de uma subquery (**AS SELECT**) e o total de colunas máximo que uma tabela pode ter é limitado.

É possível adicionar **constraints** (como **NOT NULL, PRIMARY KEY**) já na criação ou depois com **ALTER TABLE**. Existem também cláusulas para especificar **particionamento, armazenamento, índices, virtual columns (colunas calculadas), etc.**

---

```sql
Partitioning for Availability, Manageability, and Performance
```

Esse capítulo da documentação da **Oracle Database** explica os conceitos e benefícios de **particionamento de tabelas e índices** no banco de dados — um recurso usado para melhorar desempenho, facilitar administração e aumentar disponibilidade em sistemas com grandes volumes de dados.

### Por que usar particionamento?

O particionamento divide uma tabela ou índice em **várias partes menores (partições)**, sem mudar a forma como a aplicação vê a tabela. Isso traz vantagens como:

- **Melhor desempenho de consultas**
    
    O Oracle pode acessar apenas as partições necessárias para responder à consulta, reduzindo I/O e acelerando a execução (isso é chamado de **partition pruning**).
    
- **Facilidade de manutenção**
    
    Operações de administração (como carregar dados, excluir dados antigos ou reconstruir índices) podem ser feitas em uma partição individual, sem afetar o restante da tabela.
    
- **Maior disponibilidade**
    
    Se uma partição estiver offline ou em manutenção, as outras continuam disponíveis — isso ajuda a reduzir paradas e manter o sistema funcionando.
    

### Tópicos principais que o capítulo aborda

1. **Partition Pruning (eliminação de partições)**
    
    O otimizador pode **ignorar partições que não são necessárias** para responder a uma consulta, com base nas condições do `WHERE`, acelerando bastante a execução.
    
2. **Partition-Wise Operations (operações ao nível de partição)**
    
    Métodos como **partition-wise joins** permitem que o banco junte dados de tabelas baseando-se nas partições — o que melhora ainda mais o desempenho em joins grandes.
    
3. **Index Partitioning (particionamento de índices)**
    
    Índices também podem ser particionados, possibilitando manutenções e acessos mais eficientes, assim como prunning de índice.
    
4. **Particionamento e compressão**
    
    O banco pode aplicar **compressão de dados** em partições para reduzir espaço em disco e acelerar operações de I/O.
    
5. **Quando usar cada tipo de estratégia de particionamento**
    
    O texto dá recomendações sobre quando aplicar particionamento por intervalo (*range*), por lista (*list*), por hash (*hash*) ou combinações deles (**composite partitioning**) com base no padrão de uso dos dados.
    

---

```sql
External Tables : Querying Data From Flat Files in Oracle
```

No Oracle 9i foi introduzido o conceito de **external tables** — tabelas cujo **dados não ficam dentro do banco**, mas sim em **arquivos de texto (flat files)** no sistema de arquivos.

### Como funciona

- A definição da tabela (colunas, tipos, etc.) fica armazenada dentro do Oracle.
- Os **dados em si permanecem nos arquivos fora do banco**, e são lidos quando você faz uma consulta.
- Você pode **consultar os dados usando SQL (SELECT, joins, ORDER BY, etc.)** como se fosse uma tabela normal.

## Características principais

**Leitura apenas / somente leitura**

- Você **não pode fazer DML** (ou seja, não pode usar *INSERT*, *UPDATE* ou *DELETE* nessa tabela).
- Não dá para criar índices nesses dados externos.

**Útil para ETL ou Query temporária**

- Muito usado em processos de **ETL (Extract, Transform, Load)**, porque você **não precisa importar (“stage”) os dados antes**, você lê direto do arquivo.

**Paralelismo**

- Pode ser consultado em paralelo, o que ajuda em processamento de grandes arquivos.

⚠️ **Não recomendado para uso frequente**

- Por não ter índice nem DML, não é bom para consultas que vão acontecer várias vezes.

## Como criar uma *external table* (visão geral)

Para usar *external tables*, você precisa:

1. **Criar um objeto de diretório** no Oracle que representa o caminho onde o arquivo está no sistema operacional:
    
    ```
    CREATEOR REPLACE DIRECTORY ext_tab_dataAS'/caminho/para/os/arquivos';
    ```
    
2. **Criar a tabela externa com a sintaxe `CREATE TABLE … ORGANIZATION EXTERNAL`:**
    
    Após isso você define como os campos do arquivo estão delimitados, onde os arquivos estão, etc.
    
3. **Consultar como tabela normal:**
    
    ```
    SELECT*FROM tabela_externa;
    ```
    
    O Oracle vai **ler o arquivo quando você fizer a consulta**.
    

## O que acontece nos bastidores

- Oracle usa o **driver ORACLE_LOADER** (o mesmo do SQL *Loader*) para interpretar o formato dos arquivos e trazer os dados para a consulta.
- Quando você escreve SQL, o banco lê os arquivos externos conforme definido na tabela externa.
- 

## Recursos extras

- É possível criar **views ou sinônimos** baseados em external tables para facilitar consultas.

---

```sql
Tables and Table Clusters
```

No Oracle Database, **tabelas são os principais objetos de esquema** usados para armazenar dados. Elas podem ser de vários tipos dependendo de como e onde os dados são mantidos e como são acessados.

### 1. Tabela Relacional (Relational Tables)

- É o tipo mais comum de tabela.
- Armazena dados em **linhas e colunas** organizadas.
- Por padrão, é uma **heap table**: dados não têm uma ordem específica física.

### 2. Tabela Organizada por Índice (Index-Organized Table)

- Diferente da relacional comum, os dados são **armazenados fisicamente em uma estrutura de índice** (ordenada por chave).
- Pode melhorar performance em algumas consultas e usar menos espaço.

### 3. Tabela Externa (External Table)

- A estrutura (metadata) da tabela está no banco de dados, mas **os dados residem em arquivos externos** (como arquivos de texto/CSV).
- Útil para consultar dados externos com SQL sem carregar os dados no banco.

### 4. Tabela de Objetos (Object Table)

- Cada linha representa um **objeto Oracle criado a partir de um tipo definido pelo usuário**.
- Permite modelar dados com atributos e métodos (orientação a objeto).

## Outros conceitos importantes

### Permanente vs Temporária

- **Tabela permanente**: dados persistem entre sessões e transações.
- **Tabela temporária**: dados existem apenas por sessão ou transação (útil para armazenar resultados intermediários em cálculos/consultas).

### Colunas

- Cada tabela tem colunas com **nomes e tipos de dados** definidos no momento da criação.
- Pode incluir colunas virtuais (calculadas) ou invisíveis (não retornadas em `SELECT *`).

### Linhas e ROWID

- Cada registro (linha) representa uma ocorrência dos dados.
- O Oracle atribui um identificador físico chamado **ROWID** que indica onde a linha está armazenada.

### Integridade de dados

- **Constraints** (como `NOT NULL`, `PRIMARY KEY`, `FOREIGN KEY`) garantem regras e integridade dos dados nas tabelas.

### Armazenamento e compressão

- Os dados ficam em segmentos dentro de tablespaces.
- O Oracle oferece **compressão de tabela** para reduzir espaço e potencialmente melhorar performance.

---

```sql
Global Temporary Tables
```

O Oracle suporta **tabelas temporárias**, que são usadas para armazenar dados temporários usados apenas por uma sessão ou durante uma transação específica. Elas existem como objeto de banco de dados, mas **os dados nelas inseridos não persistem permanentemente**.

Existem dois tipos principais:

- **Global Temporary Tables (GTT)** – disponíveis desde o Oracle 8i
- **Private Temporary Tables (PTT)** – introduzidas no Oracle 18c (não faz parte do artigo original, mas é citada)

## Global Temporary Tables (GTT)

### Comportamento dos dados

- Os dados inseridos são **privados para cada sessão**.
- Dados inseridos por um usuário **não são visíveis para outros**.

### Escopo dos dados

Ao criar uma GTT, você especifica como os dados temporários devem ser tratados no *commit*:

- **ON COMMIT DELETE ROWS**
    
    → Os dados são apagados automaticamente **quando a transação é concluída**.
    
- **ON COMMIT PRESERVE ROWS**
    
    → Os dados permanecem **até o fim da sessão** (mesmo depois de commits).
    

### Exemplo de criação

```
CREATEGLOBALTEMPORARYTABLE my_temp_table (
  id NUMBER,
  description VARCHAR2(20)
)
ONCOMMITDELETEROWS;
```

Assim, após um **COMMIT**, a tabela fica vazia.

## Características importantes

### Undo e Redo

- **Undo** ainda é gravado na tablespace de undo tradicional (antes do Oracle 12c).
- Isso significa que embora os dados sejam temporários, o Oracle ainda gera algum redo devido ao undo.
- A partir do Oracle 12c existe uma opção chamada *temporary undo* que faz o undo ser gerenciado na tablespace temporária, reduzindo redo extra.

### Armazenamento

- Os dados das GTTs são armazenados em **segmentos temporários na tablespace temporária**.
- Quando a sessão termina (ou o *commit*, dependendo da definição), esses segmentos são automaticamente liberados.

### Outros pontos

⇒  Você pode criar **índices, triggers e views** sobre GTTs.

⇒  A definição da tabela (metadados) é permanente no dicionário de dados, mesmo que os dados sejam temporários.

⇒  O arquivo de exportação/importação (Data Pump/expdp/impdp) *exporta a definição*, mas **não os dados temporários**.

## Private Temporary Tables (Oracle 18c+)

Além das GTTs, o Oracle 18c introduziu **Private Temporary Tables (PTT)** — tabelas que são automaticamente removidas quando a sessão ou transação termina (inclusive a definição da tabela).

---

```sql
Extended Data Types in Oracle Database 12c Release 1 (12.1)
```

Antes do Oracle 12c, os tipos de dados **VARCHAR2**, **NVARCHAR2** e **RAW** tinham limites relativamente pequenos:

- **VARCHAR2**: até 4 000 bytes
- **NVARCHAR2**: até 4 000 bytes
- **RAW**: até 2 000 bytes

Com os **Extended Data Types** no Oracle 12c (opcional), esses limites aumentam para:

- **VARCHAR2**: até **32 767 bytes**
- **NVARCHAR2**: até **32 767 bytes**
- **RAW**: até **32 767 bytes**

*Importante:* os valores são em **bytes**, não em número de caracteres — então a quantidade real de caracteres depende do conjunto de caracteres usado.

## Como ativar tipos de dados estendidos

Essa funcionalidade **não é ativada por padrão** no Oracle 12c. Para usá-la, você precisa:

1. **Alterar o parâmetro de inicialização `MAX_STRING_SIZE`** para `EXTENDED`.
2. Isso precisa ser feito com o banco aberto em **modo de upgrade**.
3. Depois, execute o script `utl32k.sql` (fornecido no Oracle) para recompilar objetos afetados e validar mudanças.
4. O parâmetro **não pode ser revertido** de `EXTENDED` para `STANDARD` depois de ativado.

Esse processo é uma *alteração permanente* da estrutura do banco, por isso deve ser testado com cuidado antes de aplicar em produção.

## Como funciona?

Quando você declara colunas maiores do que os limites antigos (como `VARCHAR2(32767)`), o Oracle **armazena esses dados em segmentos LOB por trás dos panos**:

- Esses campos estendidos acabam usando a tecnologia de **LOB (Large Object)** mesmo que você declare como `VARCHAR2` ou `RAW`.
- O armazenamento é interno e gerenciado automaticamente — você não precisa criar ou manipular LOBs manualmente.

Ou seja, o usuário continua escrevendo SQL normalmente, mas por trás os dados maiores são tratados como objetos LOB pelo Oracle.

## Quando usar?

- **Migrar bases de dados que vinham de outros bancos** com maiores limites de tamanho por coluna.
- Aplicações que **precisam armazenar textos longos** sem usar tipos LOB diretamente.

Mas ele também traz alguns cuidados:

- Pode gerar mais trabalho para o DBA (porque exige upgrade e recompilação de objetos).
- Colunas estendidas são internamente LOBs, então podem ter **restrições e comportamento parecido com LOBs** (por exemplo, performance diferente em alguns casos).

---

```sql
Data Types: Basic Elements of Oracle SQL
```

## 1. Lexical Units (Unidades Léxicas)

São os blocos básicos que formam qualquer comando SQL.

Incluem:

- **Palavras reservadas** (`SELECT`, `FROM`, `WHERE`)
- **Identificadores** (nomes de tabelas, colunas, índices)
- **Literais** (valores fixos como `'Maria'`, `10`, `DATE '2025-01-01'`)
- **Operadores** (`+`, , `=`, `<>`)
- **Delimitadores** (vírgula, parênteses, ponto)

## 2. Identifiers (Identificadores)

São os **nomes dos objetos do banco**.

Exemplos:

- nome de tabela
- nome de coluna
- nome de *constraint*
- nome de *view*

### Regras importantes:

- Máx. **30 caracteres** (no 12c tradicional)
- Devem começar com letra
- Podem conter letras, números, `_`, `$`, `#`
- Não podem ser palavras reservadas (a menos que use aspas)

Se você usar aspas duplas:

```
"Nome-Completo"
```

- O nome vira *case sensitive* e pode usar caracteres especiais

## 3. Data Types (Tipos de Dados)

Define que tipo de informação uma coluna pode guardar.

Principais grupos:

### 🔢 Numéricos

- `NUMBER`
- `FLOAT`

### 🔤 Caracteres

- `CHAR`
- `VARCHAR2`
- `NCHAR`
- `NVARCHAR2`

### 📅 Data e Hora

- `DATE`
- `TIMESTAMP`

### 📦 LOB (Large Objects)

- `CLOB`
- `BLOB`
- `NCLOB`

## 4. Literals (Literais)

São **valores escritos diretamente no código SQL**.

Exemplos:

```
'João'
100
DATE'2026-02-27'
```

Tipos de literais:

- String literal
- Numérico
- Data
- Timestamp
- Intervalo

## 5. NULLs

- `NULL` ≠ 0
- `NULL` ≠ string vazia (no Oracle, string vazia vira NULL)

Comparações com NULL precisam usar:

```
ISNULL
ISNOTNULL
```

## 6. Comments (Comentários)

```
-- comentário de uma linha
```

```
/* comentário
   de várias linhas */
```

## 7. Database Objects (Objetos do Banco)

- Tables
- Views
- Indexes
- Sequences
- Synonyms
- Constraints
- Esquema (schema)
- Qualificação de nome (`schema.objeto`)

## 8. Schema Object Naming Rules

Regras detalhadas para nomes de objetos:

- Sensibilidade a maiúsculas/minúsculas
- Uso de aspas
- Limites de caracteres
- Conflito com palavras reservadas

## 9. Operators (Operadores)

### Aritméticos

`+ - * /`

### Comparação

`=`

`<>`

`!=`

`<`

`>`

### Lógicos

`AND`

`OR`

`NOT`

### Concatenação

`||`

## 10. Expressions (Expressões)

Mostra como combinar:

- colunas
- literais
- operadores
- funções

Exemplo:

```
salario*1.10
```

Essa seção explica o novo recurso chamado **Extended Data Types**, que permite **aumentar o tamanho máximo de algumas colunas de dados de texto e binários** no Oracle desde a versão 12 c:

### Tipos afetados

- **VARCHAR2**
- **NVARCHAR2**
- **RAW**

Antes do Oracle 12c:

- **VARCHAR2** e **NVARCHAR2** tinham limite de **4 000 bytes**.
- **RAW** tinha limite de **2 000 bytes**.

### Com os *Extended Data Types*

Quando o banco está configurado com o parâmetro `MAX_STRING_SIZE = EXTENDED`, passa a aceitar tamanhos maiores:

- **VARCHAR2** pode ter até **32 767 bytes**
- **NVARCHAR2** também até **32 767 bytes**
- **RAW** também até **32 767 bytes**

### O que é considerado “extended”?

O trecho define que é considerado **tipo de dado estendido**:

⇒  Uma coluna `VARCHAR2` ou `NVARCHAR2` com tamanho declarado **maior que 4000 bytes**,

⇒  Ou uma coluna `RAW` com tamanho **maior que 2000 bytes**.

Essas colunas **são armazenadas internamente como um LOB** (*Large Object*) em vez de armazenarem diretamente na linha — ou seja, o Oracle usa a tecnologia de LOBs para guardar o conteúdo maior.

---

# Episódio 2: Colunas e Tipos de Dados

## 2.1  Colunas

**DEF.:** Armazena dados de um dos aspectos dos objetos que a tabela guarda.

Quando você está criando colunas, é necessário duas coisas: como chamá-las e o seu tipo.

### Tipos de Colunas

#### 2.1.1  Colunas Numéricas

ex: *integer, float, decimal, etc.*

#### 2.1.2 Datas (Calendário)

#### 2.1.3  Strings

ex: *nomes, textos, etc.*

---

# Episódio 3: Modelagem de Dados

A modelagem de dados é o processo de planejar, organizar e estruturar dados de um sistema antes de criar o banco de dados.

Durante o processo de modelagem de dados, é necessário uma conversa com o cliente para descobrir exatamente o que ele quer saber e como você pode organizar melhor os seus dados para obter isso.

Existem 3 etapas na modelagem de dados:

1. Modelagem Conceitual (Ideia)

Aqui você pensa no mundo real, sem tecnologia.

Exemplo (biblioteca):

- Livro
- Usuário
- Empréstimo

Foco: **o que existe no sistema**

2. Modelagem Lógica (estrutura)

Agora você organiza:

- Entidade → vira tabela
- Atributo → vira coluna
- Relacionamento → vira chave

Exemplo:

- Livro(id, nome, autor)
- Usuario(id, nome)
- Emprestimo(usuario_id, livro_id)

Foco: **como os dados se conectam**

3. Modelagem Física (implementação)

Aqui você cria no banco de verdade:

```
CREATETABLE Usuario (
  idINTPRIMARYKEY,
  nomeVARCHAR(100)
);
```

Foco: C**omo isso funciona no banco (Oracle, MySQL, etc.)**

## 3.1  Rational Modeling (Rational Unified Process - RUP)

DEF.: É uma forma estruturada de planejar, modelar e desenvolver sistemas, usndo modelos visuais antes de sair programando tudo direto. (ex: *UML)*

---

**OBS!** UML (Unified Modeling Language)

A UML  é uma linguagem de modelagem padronizada voltada à representação visual de sistemas complexos, especialmente no desenvolvimento de software. Criada nos anos 1990, tornou-se o principal meio de descrever, especificar e documentar arquiteturas e comportamentos de sistemas de forma independente de linguagem de programação.

### Principais fatos

- **Categorias:** Diagramas estruturais e de comportamento (14 subtipos principais).
- **Usos principais:** Engenharia de software, modelagem de banco de dados e processos de negócio.

### Estrutura e propósito

A UML serve como uma linguagem visual para projetar sistemas orientados a objetos. Ela padroniza símbolos e relações que permitem representar classes, objetos, interfaces, componentes, casos de uso e interações temporais. Ao promover uma visão unificada do sistema, reduz ambiguidades e melhora a comunicação entre equipes técnicas e não técnicas.

### Tipos de diagramas

Agrupa-se em dois grandes conjuntos:

- **Diagramas estruturais** (ex.: classe, componente, implantação, objeto, pacote) descrevem a arquitetura estática e as relações entre elementos.
- **Diagramas de comportamento** (ex.: caso de uso, sequência, atividade, máquina de estados, interação) representam fluxos e dinâmicas de execução.

### Aplicações e impacto

A UML é amplamente adotada na engenharia de software, análise de requisitos e documentação técnica. Ferramentas como **StarUML**, **Visual Paradigm**, **Lucidchart**, e **Microsoft Visio** permitem criar diagramas UML digitais integrados a fluxos de desenvolvimento. Além do código, a UML é usada para comunicação em projetos empresariais e educacionais, favorecendo padronização e entendimento de sistemas complexos.

### Relevância atual

Mesmo com o avanço de métodos ágeis e modelagens alternativas, a UML permanece uma ferramenta essencial para visualizar e alinhar arquitetura e comportamento de software, facilitando manutenção, escalabilidade e interoperabilidade de sistemas.

---

O Rational Modeling segue 4 ideias principais:

1. **Iterativo**
    
    → Não faz tudo de uma vez, vai evoluindo em ciclos
    
2. **Baseado em arquitetura**
    
    → Pensa primeiro na estrutura do sistema
    
3. **Orientado a casos de uso**
    
    → Foca no que o usuário precisa fazer
    
4. **Gerenciado por riscos**
    
    → Resolve primeiro o que pode dar problema
    

O RUP divide o projeto em 4 fases:

1. **Inception (Iniciação)** → ideia geral do projeto
2. **Elaboration (Elaboração)** → arquitetura e planejamento
3. **Construction (Construção)** → desenvolvimento
4. **Transition (Transição)** → entrega e ajustes

---

# Episódio 4: SELECT e WHERE

Para conseguir os dados que você precisa, basta escrever: 

```sql
SELECT * FROM name_table
// O asterisco * traz TODAS as colunas da tabela
// Não é uma boa prática em sistemas reais de bancos de dados
```

```sql
SELECT nome, email
FROM usuarios;

// Da sua tabela, você filtra as informações para receber apenas o nome e o email
// mas de todos os usuários.
```

```sql
SELECT DISTINCT cidade
FROM usuarios
WHERE idade > 18;

// Esse tipo de SELECT remove duplicatas
```

```sql
SELECT COUNT(*)
FROM usuarios
WHERE cidade = 'Niteroi';

// Conta quantas linhas existem dentro de usuários que moram em Niterói
// Retorna apenas a quantidade de linhas que passarem pelo filtro "cidade"
```

```sql
SELECT preco * 0.9 AS preco_com_desconto
FROM produtos
WHERE preco > 100;

// Esse SELECT nome * x pode modificar valores antes de mostrá-los
// O AS apenas nomeia o resultado do que foi selecionado
```

Com esse comando, ele vai retornar todo o conteúdo da sua tabela. Como normalmente queremos apenas uma ou duas linhas da tabela, seria melhor limitar a busca para analisar as linhas e colunas do seu interesse. 

Para isso, podemos usar o WHERE - usado para aplicar condições para as colunas da sua tabela. É possível combinar condições em diferentes colunas.

```sql
// * -> todas as colunas de usuarios com nome Ana e com mais de 40 anos

SELECT *
FROM usuarios
WHERE nome = 'Ana' and idade > 40;
```

```sql
// Apenas nome e email dos usuários que moram no Rio ou em Niterói 
// (filtragem de COLUNAS).

SELECT nome, email
FROM usuarios
WHERE cidade = 'Rio' or cidade = 'Niterói';
```

Para facilitar, podemos escrever como:

```sql
SELECT nome, email
FROM usuarios
WHERE cidade in ('Rio', 'Niterói') ;
```

Para procurar em um intervalo de valores, use uma inequação.

```sql
SELECT coluna
FROM tabela
WHERE coluna BETWEEN valor1 AND valor2;

// A ordem importa !!
```

```sql
SELECT *
FROM pedidos
WHERE data BETWEEN '2024-01-01' AND '2024-12-31';
```

```sql
SELECT *
FROM usuarios
WHERE nome BETWEEN 'A' AND 'M';
```

O uso de % para a identificação de padrões para números ou strings é bem importante também.

```sql
SELECT *
FROM usuarios
WHERE nome LIKE 'A%';
```

```sql
SELECT *
FROM usuarios
WHERE nome LIKE '%a';
```

```sql
SELECT *
FROM usuarios
WHERE nome LIKE '%an%';
```

Também temos o  _ (underscore), que representa apenas 1 caractere.

```sql
SELECT *
FROM usuarios
WHERE nome LIKE '_a';
```

CONDIÇÕES DE NEGAÇÃO (*not*)

```sql
SELECT * 
FROM colors
WHERE NOT color = 'purple' 
```

```sql
SELECT *
FROM usuarios
WHERE nome <> 'Ana';
```

```sql
SELECT *
FROM usuarios
WHERE nome NOT LIKE 'A%';
```

```sql
SELECT *
FROM produtos
WHERE categoria NOT IN ('eletronico', 'roupa');
```

```sql
SELECT *
FROM produtos
WHERE preco NOT BETWEEN 50 AND 100;
```

```sql
SELECT *
FROM usuarios
WHERE telefone IS NOT NULL;
```

---

# Episódio 5: JOIN no SQL

O JOIN serve pra combinar dados de duas ou mais tabelas com base em uma relação entre elas.

ex: 

Tabela  `clientes`

| id | nome |
| --- | --- |
| 1 | Ana |
| 2 | João |

Tabela  `pedidos`

| id | cliente_id | produto |
| --- | --- | --- |
| 1 | 1 | Pizza |
| 2 | 2 | Hambúrguer |
| 3 | 1 | Suco |

```sql
SELECT clientes.nome, pedidos.produto
FROM clientes
INNER JOIN pedidos
ON clientes.id = pedidos.cliente_id;

// INNER JOIN - só combinações que batem
```

### O que o SQL entende:

1. Começa com a tabela `clientes`
2. Junta com `pedidos`
3. Cria uma **tabela temporária combinada**
4. Aí sim aplica o `SELECT`

## Tipos de Join

| Tipo | O que faz |
| --- | --- |
| INNER JOIN | Só o que bate nas duas |
| LEFT JOIN  (outer) | Tudo da esquerda |
| RIGHT JOIN (outer) | Tudo da direita |
| FULL JOIN (outer) | Tudo de ambas |

A parte mais importante é o `ON` , já que ele define como as tabelas se conectas. Sem isso, vira um CROSS JOIN.

**OBS!** O que é um CROSS JOIN?

```sql
SELECT * FROM clientes, pedidos;
```

```sql
SELECT clientes.nome, produtos.produto
FROM clientes
CROSS JOIN produtos;
```

→  Seleção das duas tabelas simultaneamente, juntando todas as combinações possíveis.
→  Sem `WHERE` vira CROSS JOIN

Como resultado: 

| nome | produto |
| --- | --- |
| Ana | Pizza |
| Ana | Suco |
| João | Pizza |
| João | Suco |

É útil quando você quer realmente todas as combinações

**OBS!** *Outer Join*

O OUTER JOIN serve pra trazer dados mesmo quando não existe correspondência entre as tabelas, diferentemente do INNER JOIN que só mostra o que “bate”. Existem dois tipos: LEFT e RIGHT JOIN

→ LEFT JOIN: Retorna todas as linhas da esquerda da tabela. 

→ RIGHT JOIN: Retorna todas as linhas da direita da tabela

**FULL OUTER JOIN: Retorna todas as linhas de ambas tabelas (direita e esquerda)**

```sql
SELECT *
FROM teddies t
FULL OUTER JOIN bricks b
ON t.colour = b.colour
```

Existe também o SELF JOIN, que é quando uma tabela se junta com ela mesma, e o BAND JOIN - join por INTERVALO.

```sql
FROM funcionarios f1, funcionarios f2
// SELF JOIN
```

```sql
salario BETWEEN faixa.min AND faixa.max
// BAND JOIN
```

---

# Episódio 6: AGGREGATES e GROUP BY

## 6.1 Aggregate Functions

Essas funções combinam valores de várias colunas e retornam uma única linha, permitindo que você possa saber quantas linhas você tem, encontre a menor

| Função | O que faz |
| --- | --- |
| `COUNT()` | conta linhas |
| `SUM()` | soma valores |
| `AVG()` | média |
| `MAX()` | maior valor |
| `MIN()` | menor valor |

A maioria das funções aceita a coluna como um argumento ou asterisco para oegar a informação total de todas as linhas da tabela, não apenas de uma coluna específica.

## 6.2 Group By

Com ele, é possível separar os resultados de acordo com os valores das colunas em grupo. No seu output, você consegue uma linha para cada valor diferente. 

### **ROLLUP**

Ao colocar isso no seu *group by*, é útil para criar totais e subtotais automaticamente

```sql
SELECT ano, mes, SUM(valor)
FROM vendas
GROUP BY ROLLUP(ano, mes);
```

| GROUP BY | ROLLUP |
| --- | --- |
| só grupos | grupos + subtotais |
| sem total automático | total automático |

## 6.3 Having Clause

Essa cláusula filtra os dados com o resultado das suas funções agregadas. Ou seja, ele filtra GRUPOS, não LINHAS.

ex:  Tabela `pedidos`

| cliente | valor |
| --- | --- |
| Ana | 50 |
| Ana | 30 |
| João | 20 |

SEM HAVING

```sql
SELECT cliente, SUM(valor)
FROM pedidos
GROUP BY cliente;
```

| cliente | total |
| --- | --- |
| Ana | 80 |
| João | 20 |

COM HAVING

```sql
SELECT cliente, SUM(valor)
FROM pedidos
GROUP BY cliente
HAVING SUM(valor) > 50;
```

| cliente | total |
| --- | --- |
| Ana | 80 |

Qual a diferença entre WHERE e HAVING?

| WHERE | HAVING |
| --- | --- |
| filtra **linhas** | filtra **grupos** |
| antes do GROUP BY | depois do GROUP BY |
| não usa agregação | usa agregação |

---

# Episódio 7: Insert e Commit

## 7.1  Insert

O  `insert` é usado para adicionar novas linhas em um tabela do banco de dados e tem duas formas: *Single row* ou *Multi row*. 

### 7.1.1  Single row

Nesse tipo de *insert*, é possível adicionar dados a sua tabela uma linha de cada vez.

```sql
INSERT INTO nome_da_tabela
VALUES (valor1, valor2, valor3);
```

Aqui, você deve associar um valor para cada coluna existente dessa linha na mesma ordem em que elas estão listadas. Se colocar os dados na ordem errada, dará um erro se os tipo de dados forem incompatíveis, apesar do *input*  ter dado certo.

Para evitar essa confusão, você pode explicitamente listar as colunas depois do `nome_da_tabela` .

```sql
INSERT INTO nome_da_tabela (coluna1, coluna2)
VALUES (valor1, valor2);

// Nesse exemplo, você pode escolher quais colunas quer preencher.
```

*Single row inserts* são ótimos se você quer adicionar apenas algumas linhas, mas podem levar muito tempo se for um número muito grande de linhas a serem adicionadas.

### 7.1.2  Multi row

É uma forma de inserir várias linhas de dados em uma ou mais tabelas usando um único comando  `insert` , permitindo copiar dados de uma tabela para outra de forma eficiente (*query)*.

Existem dois tipos de *multi row inserts*: Conditional e Unconditional 

1. **Unconditional Multi-Row Insert**

Nesse caso, todas as linhas retornadas pelo SELECT são inseridas nas tabelas especificadas, sem nenhuma condição.

```sql
INSERT ALL
  INTO tabela1 VALUES (...)
  INTO tabela2 VALUES (...)
SELECT ... FROM ...;
```

Aqui:

- `INSERT ALL` → indica múltiplas inserções
- `INTO` → especifica cada linha que será inserida
- `SELECT * FROM dual` → usado no Oracle para executar o comando

Resultado: **Duas linhas são inseridas em uma única execução**.

b.  C**onditional Multi-Row Insert**

Os dados inseridos dependem de condições WHEN.

```sql
INSERT ALL
  WHEN condição THEN
    INTO tabela1 VALUES (...)
  WHEN outra_condição THEN
    INTO tabela2 VALUES (...)
SELECT ... FROM ...;
```

ex:

```sql
INSERT ALL
  WHEN salario < 3000 THEN
    INTO funcionarios_junior VALUES (id, nome, salario)
  WHEN salario >= 3000 THEN
    INTO funcionarios_senior VALUES (id, nome, salario)
SELECT id, nome, salario
FROM funcionarios;
```

- funcionários com **salário < 3000** vão para `funcionarios_junior`
- funcionários com **salário ≥ 3000** vão para `funcionarios_senior`

Existe também o **INSERT FIRST**, que executa a primeira condição verdadeira.

```sql
INSERT FIRST
  WHEN condição1 THEN
    INTO tabela1 VALUES (...)
  WHEN condição2 THEN
    INTO tabela2 VALUES (...)
SELECT ... FROM ...;
```

Diferença:

- **ALL** → pode inserir em várias tabelas
- **FIRST** → insere apenas na **primeira condição satisfeita**

| Tipo | Característica |
| --- | --- |
| `INSERT ALL` | insere em várias tabelas ou várias vezes |
| `WHEN` | define condições |
| `INSERT FIRST` | executa apenas a primeira condição verdadeira |
| `SELECT ... FROM dual` | usado no Oracle para gerar linhas |

## 7.2  Commit

É um comando usdo para confirmar permanentemente as alterações feitas no banco de dados dentro de uma transação. Em outras palavras, ele salva definitivamente operações como INSERT, UPDATE e DELETE.

O  `commit`  salva as mudanças feitas nas tabelas e restringe o acesso às mudanças realizadas apenas à pessoa responsável por elas. Logo, outras pessoas não conseguem ver as alterações feitas nas tabelas por você até que você faça o COMMIT das suas mudanças.

Caso queira desfazer alguma mudança, use  `rollback`  e, assim, será possível reverter todas as alterações desde o seu último *commit*. Depois do *commit*, não da pra usar ROLLBACK !!

```sql
INSERT INTO alunos (nome, idade)
VALUES ('Ana', 20);

COMMIT;
```

**OBS!** O **`AUTOCOMMIT`** é um modo do banco de dados em que **cada comando SQL que altera dados é automaticamente confirmado**, sem precisar usar `COMMIT` manualmente.

```sql
SET AUTOCOMMIT ON;
```

```sql
SET AUTOCOMMIT OFF;
```

| Modo | Comportamento |
| --- | --- |
| **Autocommit ON** | cada comando é confirmado automaticamente |
| **Autocommit OFF** | alterações ficam na transação até `COMMIT` |
| **ROLLBACK** | só funciona antes do commit |

---

# Episódio 8: Update e Transactions

# 8.1  Update

É o comando usado para modificar dados que já existem em uma tabela, alterando valores de uma ou mais colunas em registro específicos.

```sql
UPDATE nome_da_tabela
SET coluna1 = valor1, coluna2 = valor2
WHERE condição;
```

**Partes do comando:**

- `UPDATE` → tabela que será alterada
- `SET` → colunas e novos valores
- `WHERE` → define **quais linhas serão atualizadas**

ex:

Tabela `alunos`

| id | nome | idade |  |
| --- | --- | --- | --- |
| 1 | Ana | 20 |  |
| 2 | Carlos | 22 |  |

```sql
UPDATE alunos
SET nome = 'Ana Souza',
    idade = 22
WHERE id = 1;

// Aqui, duas colunas são modificadas ao mesmo tempo
```

| id | nome | idade |
| --- | --- | --- |
| 1 | Ana | 21 |
| 2 | Carlos | 22 |

Se você **não usar `WHERE`**, **todas as linhas da tabela serão modificadas**.

```
UPDATE alunos
SET idade = 18;
```

Isso mudaria a idade **de todos os alunos** para 18. Esse tipo de uso também gastaria muito tempo e apenas uma pessoa consegue fazer essas alterações por *update*.

| Comando | Função |
| --- | --- |
| `UPDATE` | altera dados existentes |
| `SET` | define novos valores |
| `WHERE` | seleciona quais registros serão alterados |

**OBS!**  O **`SELECT ... FOR UPDATE`** é um comando SQL usado para **selecionar linhas de uma tabela e ao mesmo tempo bloqueá-las para atualização** dentro de uma transação. É usado principalmente para evtar conflitos entre transações simultâneas.

Ou seja, ele **impede que outros usuários modifiquem essas mesmas linhas até que a transação termine (`COMMIT` ou `ROLLBACK`)**.

```sql
SELECT colunas
FROM tabela
WHERE condição
FOR UPDATE;
```

Esse comando:

1. seleciona as linhas
2. **coloca um lock (bloqueio)** nelas

ex:

```sql
SELECT saldo
FROM contas
WHERE id = 1
FOR UPDATE;

UPDATE contas
SET saldo = saldo - 100
WHERE id = 1;

COMMIT;
```

**OBS!** O **Optimistic Locking (bloqueio otimista)** é uma técnica de controle de concorrência usada em bancos de dados para **evitar conflitos quando várias transações tentam modificar o mesmo dado**.  A ideia é **assumir que conflitos são raros**, então o banco **não bloqueia o registro quando ele é lido**. Em vez disso, ele **verifica se o dado mudou antes de salvar a atualização**.

- Não bloqueia dados ao ler
- Verifica conflito **na hora de atualizar**
- Usa geralmente **coluna de versão ou timestamp**
- 

No **optimistic locking**:

1. Um usuário **lê o registro**
2. Outro usuário também pode ler o mesmo registro
3. Quando alguém tenta **atualizar**, o sistema verifica se o registro mudou
4. Se mudou → ocorre **conflito** e a operação falha

O **`version number` (número de versão)** é uma técnica usada principalmente em **Optimistic Locking** para **controlar concorrência em atualizações de dados**.

Ele funciona como um **contador que aumenta toda vez que um registro é alterado**. Assim, o sistema consegue verificar se **outra transação modificou o registro antes de você salvar suas alterações**.

```sql
UPDATE funcionarios
SET salario = 3500,
    version = version + 1
WHERE id = 1
AND version = 1;
```

| Tipo | Funcionamento |
| --- | --- |
| **Pessimistic Locking** | bloqueia o registro quando alguém acessa (`SELECT FOR UPDATE`) |
| **Optimistic Locking** | não bloqueia; verifica conflito apenas no `UPDATE` |

---

# Episódio 9: Delete e Truncate

# 9.1  Delete

Esse comando deleta informações de uma tabela de acordo com a WHERE clause. Uma vez que você deu o *commit*, o dado não será mais acessado. Um DELETE sem WHERE vai apagar tudo das tabelas, podendo demorar bastante tempo.

```sql
DELETE FROM tabela
WHERE condição;
```

### Características

- Pode usar **`WHERE`** para escolher as linhas
- Remove **linha por linha**
- Pode ser **desfeito com `ROLLBACK`** (se estiver em transação)
- Gera **logs de cada linha removida**

Se não usar `WHERE`:

```sql
DELETE FROM tabela;
```

Remove **todas as linhas**, mas a tabela continua existindo.

# 9.2 Truncate

O comando  `truncate`  reseta sua tabela instantaneamente, marcando-a como **vazia**. Não é possível fazer uma filtragem da ação desse comando e sempre deletará tudo, não permitindo desfazer tais alterações.

```sql
TRUNCATE TABLE tabela;
```

Resultado:

- **todos os registros são removidos**
- a tabela continua existindo

### Características

- **muito mais rápido que DELETE**
- não permite **WHERE**
- geralmente **não pode usar ROLLBACK**

# 9.3 Soft Delete

Não remove o registro do banco de dados, apenas marca o registro como deletado. Normalmente, é feito com uma coluna como: `deleted` , `is_deleted` , `deleted_at` , `status` , etc.

ex:

| id | nome | deleted |
| --- | --- | --- |
| 1 | Ana | false |

```sql
UPDATE alunos
SET deleted = true
WHERE id = 1;
```

O registro **continua no banco**, mas o sistema ignora ele nas consultas. É importante verificar o estado do *is_deleted* ao fazer um SELECT.

### Vantagens

- permite **recuperar dados**
- mantém **histórico**
- evita perda permanente

| Comando | Remove dados | WHERE | Pode desfazer | Velocidade |
| --- | --- | --- | --- | --- |
| DELETE | sim | sim | sim | mais lento |
| Soft Delete | não (apenas marca) | sim | sim | normal |
| TRUNCATE | sim (todos) | não | geralmente não | muito rápido |
