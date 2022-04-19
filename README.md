# ETL com Pentaho

## Objetivo 

Projeto desenvolvido como solução ao desafio proposto no segundo módulo (módulo de armazenamento de dados) do bootcamp de Engenheiro de Dados do IGTI (Instituto de Gestão e Tecnologia da Informação). O objetivo principal deste exercício é fazer um processo de ETL simplificado no Pentaho a partir de arquivos do tipo csv.

## Enunciado do Desafio adaptado

Modelar um Data Warehouse simples através de diferentes arquivos ("Categoria.csv", "Cliente.csv", "Funcionario.csv", "Produto.xlsx", "Regiao.csv", "Territorio.csv", "Vendas.xlsx") que servirão como dados de origens para o DW. Estes dados seguem o seguinte modelo:

![image-20220419150018215](./imgs/Img1.png)

A partir dos dados de origem, o objetivo é modelar um Star Schema como esquema abaixo:

![image-20220419150137237](./imgs/Img2.png)

### Utilizando o MySQL como Staging e DW:

Utilizar o MySQL via Workbench como repositório para a origem dos dados.

Serão necessários dois schemas diferentes, um para a Staging e outro para o DW. Crie os schemas no MySQL manualmente ou via comando sql:

Exemplo: 

​	CREATE SCHEMA IF NOT EXISTS 'stage' DEFAULT CHARACTER SET utf8 ;

![image-20220419150506425](./imgs/Img3.png)

#### Criando um repositório para o projeto do desafio.

1. Clique no botão "connect" do Pentaho:

![image-20220419150847558](./imgs/Img4.png)

2. Em seguida, clique em "Other Repositories":

![image-20220419150913833](./imgs/Img5.png)

3. Selecione "File Repository" e clique em "Get Started". 

   ![image-20220419151143920](./imgs/Img6.png)

4. Configure com o diretório de sua preferência para mandar o projeto salvo.

5. Copie as transformações anexadas ao desafio para o diretório criado que servirá como repositório:

   * Origem.ktr
   * Dimensao_Data.ktr
   * Dimensao_Produto.ktr

6. Crie as conexões com os bancos MySQL:

   ![image-20220419151224830](./imgs/Img7.png)

   Obs.: Vamos utilizar o arquivo disponível no desafio chamado "origem.ktr" para trabalhar com as transformações, trazendo os dados da origem para a stage. Abra esse arquivo no Pentaho.

7. Faça as seguintes alterações:

   a. Altere os steps "CSV file input" para que apontem para o arquivo de origem.

![image-20220419151509176](./imgs/Img8.png)

​		b. Altere os steps Table Output para que:

​			i. Ajuste a conexão de forma que funcione conectada ao MySQL Workbench.

​			ii. Aponte para o schema que receberá as tabelas da Staging.

​		c. O nome da tabela a ser criada no banco de dados é o nome que está no campo Target table. Portanto, é só clicar no botão SQL que será possível criar a tabela. 

![image-20220419151705690](./imgs/Img9.png)

Obs.: Não é necessário criar as tabelas da Stage e do DW nos bancos MySQL de forma manual. Isso pode ser feito através do componente table output do Pentaho.

![image-20220419151811619](./imgs/Img10.png)

Portanto, a transformação ficará assim:

![image-20220419151827779](./imgs/Img11.png)

#### Carga para Dimensão Produto.

1. Abrir no Pentaho o arquivo "Dimensao_Produto.ktr";
2. Alterar as conexões dos steps Table Input para Produto e Categoria;
3. Avaliar as configurações de cada step presente;
4. Rodar a transformação.

![image-20220419152245594](./imgs/Img12.png)

### Carga para Dimensão Tempo

1. Abrir no Pentaho o arquivo "Dim_Tempo.ktr";
2. Alterar as conexões do Table Output chamado DIM_Tempo;
3. Avaliar as configurações de cada step presente;
4. Rodar a transformação.

![image-20220419152446859](./imgs/Img13.png)

### Carga para Dimensão Funcionário

Fazer a transformação baseada no seguinte modelo.

![image-20220419152540370](./imgs/Img14.png)


### Carga para Dimensão Cliente

Fazer a transformação baseada no seguinte modelo.

![image-20220419152617035](./imgs/Img15.png)

### Carga da tabela Fato Venda

Configurar a transformação conforme o modelo abaixo.

![image-20220419152715286](./imgs/Img16.png)

Obs.: Os steps "lookup" devem ser configurados conforme exemplo do Database lookup-SK-Cliente:

![image-20220419152758316](./imgs/Img17.png)

Repare, pelo "preview", que nas últimas linhas da tabela fato a sk_produto está nula. Isso não deu erro porque a tabela fato não está com as surrogate keys das dimensões setada como chave.

Vamos fazer um ajuste e ver o que acontece.

Rode o comando sql no banco de dados do schema onde está o DW.

​	ALTER TABLE 'fato_vendas' 

​		ADD PRIMARY KEY ('sk_produto', 'sk_cliente', 'sk_funcionario', 'sk_data');

![image-20220419153048003](./imgs/Img18.png)

![image-20220419153117562](./imgs/Img19.png)

Agora que ligamos as chaves primárias na fato, tente rodar a carga da fato novamente. Você vai obter o erro: "Column 'sk_funcionario' cannot be null".

![image-20220419153215113](./imgs/Img20.png)

Isso será resolvido trocando o step "table output" por um step "insert update".

![image-20220419153243871](./imgs/Img21.png)

Vamos configurar quais são as chaves na tabela, e isso vai permitir fazer um update em caso de insert sem sucesso.

Repare que as dimensões do primeiro registro tem a **Surrogate Key = 1 e demais registros vazios**. Ele serve para você utilizá-lo em caso de inconsistência. Portanto, vamos utilizar o atributo SK = 1 para o caso de erro na carga.

![image-20220419153425473](./imgs/Img22.png)

Teremos que inserir um step "**if field is null**". Configure conforme figura abaixo.

![image-20220419153458283](./imgs/Img23.png)

No caso de termos valores nulos para as surrogate keys, vamos atribuir um valor = 1 para eles.

Lembrando que o Pentaho reserva o primeiro registro com a SK = 1. Os campos são nulos nessa linha e ele serve exatamente para tratar erros como acima.

Configure o step "insert / update" conforme figura abaixo.

![image-20220419153624367](./imgs/Img24.png)

Rode a carga e depois confira o resultado no MySQL. Verifique que as últimas linhas tem sk_funcionario com valores = 1. Ou seja, há dados na tabela fato referenciando dados nas dimensões, porém esses dados na dimensão estão ausentes.

![image-20220419153815723](./imgs/Img25.png)

Se tentar rodar novamente a carga da tabela fato, como ela faz um processo de insert, ocorrerá erro na carga por conta da chave primária. 

![image-20220419153815764](./imgs/Img26.png)

Isso pode ser resolvido pelo uso do step Dummy (do nothing):

![image-20220419153842877](./imgs/Img27.png)

Ou então pode ser utilizado um step Insert/update ao invés do step Table output.

![image-20220419153925185](./imgs/Img28.png)

Desabilite o Hop que vai para o step Table output (Fato Vendas) e rode duas vezes para ver se dá erro.

![image-20220419154002648](./imgs/Img29.png)

FIM.