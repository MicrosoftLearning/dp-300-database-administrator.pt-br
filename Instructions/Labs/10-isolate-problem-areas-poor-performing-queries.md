---
lab:
  title: Laboratório 10 – Isolar áreas problemáticas em consultas com baixo desempenho em um Banco de Dados SQL
  module: Optimize query performance in Azure SQL
---

# Isolar áreas problemáticas em consultas com baixo desempenho em um Banco de Dados SQL

**Tempo estimado**: 30 minutos

Você foi contratado como administrador de banco de dados sênior para ajudar com problemas de desempenho que têm ocorrido quando os usuários consultam o banco de dados *AdventureWorks2017*. Seu trabalho é identificar problemas no desempenho das consultas e resolvê-los usando as técnicas aprendidas neste módulo.

Você executará consultas com desempenho inferior, examinará os planos de consulta e tentará fazer melhorias no banco de dados.

> &#128221; Estes exercícios pedem que você copie e cole o código T-SQL. Verifique se o código foi copiado corretamente antes de executá-lo.

## Ambiente de configuração

Se a máquina virtual do laboratório tiver sido fornecida e pré-configurada, você encontrará os arquivos de laboratório prontos na pasta **C:\LabFiles**. *Reserve um momento para verificar; se os arquivos já estiverem lá, pule esta seção*. No entanto, se você estiver usando sua própria máquina ou os arquivos de laboratório estiverem ausentes, será necessário cloná-los do *GitHub* para continuar.

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, inicie uma sessão do Visual Studio Code.

1. Abra a paleta de comandos; (Ctrl+Shift+P) e digite **Git: Clone**. Selecione a opção **Git: Clone**.

1. Cole a URL a seguir no campo **URL do repositório** e selecione **Enter**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Salve o repositório na pasta **C:\LabFiles** na máquina virtual do laboratório ou em seu computador local, se não tiver sido fornecida (crie a pasta se ela não existir).

---

## Restaurar um banco de dados

Se você já tiver o banco de dados **AdventureWorks2017** restaurado, ignore esta seção.

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, inicie uma Sessão do SQL Server Management Studio (SSMS).

1. Quando o SSMS abrir, por padrão, a caixa de diálogo **Connect to Service** será exibida. Escolha Default instance e selecione **Connect**. Talvez seja necessário marcar a caixa de seleção **Trust server certificate**.

    > &#128221; Observe que, se você estiver usando sua própria instância do SQL Server, precisará se conectar a ela usando o nome e as credenciais apropriados da instância do servidor.

1. Selecione a pasta**Bancos de Dados** e **Nova Consulta**.

1. Na janela Nova consulta, copie e cole o T-SQL abaixo. Execute a consulta para restaurar o banco de dados.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Shared\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\AdventureWorks2017_log.ldf';
    ```

    > &#128221; Você deve ter uma pasta chamada **C:\LabFiles**. Se você não tiver essa pasta, crie-a ou especifique outro local para o banco de dados e os arquivos de backup.

1. Na guia **Mensagens**, você verá uma mensagem indicando que o banco de dados foi restaurado.

## Gerar plano de execução real

Há várias maneiras de gerar um plano de execução no SQL Server Management Studio.

1. Selecione **Nova Consulta**. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

    **Observação:** use **SHOWPLAN_ALL** para ver uma versão em texto do plano de execução da consulta no painel de resultados, em vez de graficamente em uma guia separada.

    ```sql
    USE AdventureWorks2017;

    GO

    SET SHOWPLAN_ALL ON;

    GO

    SELECT BusinessEntityID
    FROM HumanResources.Employee
    WHERE NationalIDNumber = '14417807';

    GO

    SET SHOWPLAN_ALL OFF;

    GO
    ```

    No painel de resultados, você verá uma versão em texto do plano de execução, em vez dos resultados da execução da consulta da instrução **SELECT**.

1. Reserve um momento para examinar o texto na segunda linha da coluna **StmtText**:

    ```console
    |--Index Seek(OBJECT:([AdventureWorks2017].[HumanResources].[Employee].[AK_Employee_NationalIDNumber]), SEEK:([AdventureWorks2017].[HumanResources].[Employee].[NationalIDNumber]=CONVERT_IMPLICIT(nvarchar(4000),[@1],0)) ORDERED FORWARD)
    ```

    O texto acima explica que o plano de execução usa uma **Busca de Índice** na chave **AK_Employee_NationalIDNumber**. Ele também mostra que o plano de execução precisou realizar a etapa **CONVERT_IMPLICIT**.

    O otimizador de consulta conseguiu localizar um índice apropriado para buscar os registros necessários.

## Resolver plano de consulta abaixo do ideal

1. Copie e cole o código abaixo em uma nova janela de consulta.

    Selecione o ícone **Incluir Plano de Execução Real** à direita do botão Executar ou pressione <kbd>CTRL</kbd>+<kbd>M</kbd>. Execute a consulta selecionando **Executar** ou pressione <kbd>F5</kbd>. Anote o plano de execução e as leituras lógicas na guia Mensagens.

    ```sql
    SET STATISTICS IO, TIME ON;

    SELECT [SalesOrderID] ,[CarrierTrackingNumber] ,[OrderQty] ,[ProductID], [UnitPrice] ,[ModifiedDate]
    FROM [AdventureWorks2017].[Sales].[SalesOrderDetail]
    WHERE [ModifiedDate] > '2012/01/01' AND [ProductID] = 772;
    ```

    Ao revisar o plano de execução, você notará que há uma **Pesquisa de Chave**. Se você passar o mouse sobre o ícone, verá que as propriedades indicam que ela é executada para cada linha recuperada pela consulta. Você pode ver que o plano de execução está executando uma operação de **Pesquisa de Chave**.

    Anote as colunas na seção **Lista de Saída**. Como você melhoraria essa consulta?

    Para identificar qual índice precisa ser alterado para remover a pesquisa de chave, você precisa examinar a busca de índice acima dele. Passe o mouse sobre o operador de busca de índice, e as propriedades do operador serão exibidas.

1. **Pesquisas de Chave** podem ser removidas adicionando um índice de cobertura que inclua todos os campos que são retornados ou pesquisados na consulta. Neste exemplo, o índice usa somente a coluna **ProductID**. Veja a seguir a definição atual do índice. Observe que a coluna **ProductID** é a única coluna de chave que força uma **Pesquisa de chave** para recuperar as outras colunas necessárias à consulta.

    ```sql
    CREATE NONCLUSTERED INDEX [IX_SalesOrderDetail_ProductID] ON [Sales].[SalesOrderDetail]
    ([ProductID] ASC)
    ```

    Se adicionarmos os campos da **Lista de Saída** ao índice como colunas incluídas, a **Pesquisa de Chave** será removida. Como o índice já existe, você precisa usar DROP no índice e recriá-lo ou definir **DROP_EXISTING=ON** para adicionar as colunas. Observe que a coluna **ProductID** já faz parte do índice e não precisa ser adicionada como uma coluna incluída. Há outra melhoria de desempenho que podemos fazer no índice adicionando **ModifiedDate**. Abra uma janela **Nova Consulta** e execute o script a seguir para descartar e recriar o índice.

    ```sql
    CREATE NONCLUSTERED INDEX [IX_SalesOrderDetail_ProductID]
    ON [Sales].[SalesOrderDetail] ([ProductID],[ModifiedDate])
    INCLUDE ([CarrierTrackingNumber],[OrderQty],[UnitPrice])
    WITH (DROP_EXISTING = on);
    GO
    ```

1. Execute novamente a consulta da etapa 1. Anote as alterações nas leituras lógicas e as mudanças no plano de execução. O plano agora só precisa usar o índice não clusterizado que criamos.

> &#128221; Revisando o plano de execução, você notará que a **pesquisa de chave** desapareceu e estamos usando apenas o índice não clusterizado.

## Usar o Repositório de Consultas para detectar e gerenciar a regressão

Em seguida, você executará uma carga de trabalho a fim de gerar estatísticas de consultas para o Repositório de Consultas, examinará o relatório de **Consultas com Maior Consumo de Recursos** para identificar desempenho ruim e verá como forçar um plano de execução melhor.

1. Selecione **Nova Consulta**. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

    Esse script habilitará o recurso de Repositório de Consultas para o banco de dados AdventureWorks2017 e definirá o banco de dados como Nível de compatibilidade 100.

    ```sql
    USE [master];

    GO

    ALTER DATABASE [AdventureWorks2017] SET QUERY_STORE = ON;

    GO

    ALTER DATABASE [AdventureWorks2017] SET QUERY_STORE (OPERATION_MODE = READ_WRITE);

    GO

    ALTER DATABASE [AdventureWorks2017] SET COMPATIBILITY_LEVEL = 100;

    GO
    ```

    Alterar o nível de compatibilidade é como fazer o banco de dados voltar no tempo. Isso restringe os recursos que o SQL Server pode usar para aqueles que estavam disponíveis no SQL Server 2008.

1. Selecione o menu **Arquivo** > **Abrir** > **Arquivo** no SQL Server Management Studio.

1. Navegue até o arquivo **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\10\CreateRandomWorkloadGenerator.sql**.

1. Depois de aberto no SQL Server Management Studio, selecione **Executar** para executar a consulta.

1. Em um novo editor de consultas, abra o arquivo **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\10\ExecuteRandomWorkload.sql** e selecione **Executar** para executar a consulta.

1. Após a conclusão da execução, execute o script uma segunda vez para criar uma carga adicional no servidor. Deixe a guia consulta aberta para essa consulta.

1. Copie e cole o seguinte código em uma nova janela de consulta e execute-o selecionando **Executar**.

    Esse script altera o modo de compatibilidade do banco de dados para o SQL Server 2022 (**160**). Todos os recursos e aprimoramentos desde o SQL Server 2008 agora estarão disponíveis para o banco de dados.

    ```sql
    USE [master];

    GO

    ALTER DATABASE [AdventureWorks2017] SET COMPATIBILITY_LEVEL = 160;

    GO
    ```

1. Navegue de volta para a guia de consulta do arquivo **ExecuteRandomWorkload.sql** e execute-o novamente.

## Examinar o relatório de Consultas com maior consumo de recursos

1. Para exibir o nó Repositório de Consultas, você precisará atualizar o banco de dados AdventureWorks2017 no SQL Server Management Studio. Clique com o botão direito do mouse no nome do banco de dados e escolha **Atualizar**. Em seguida, você verá o nó Repositório de Consultas no banco de dados.

1. Expanda o nó **Repositório de Consultas** para exibir todos os relatórios disponíveis. Selecione o relatório **Consultas com maior consumo de recursos**.

1. Quando o relatório abrir, selecione o menu suspenso e, em seguida, selecione **Configurar** no canto superior direito do relatório.

1. Na tela de configuração, altere o filtro do número mínimo de planos de consulta para 2. Em seguida, selecione **OK**.

1. Escolha a consulta com a maior duração selecionando a barra mais à esquerda no gráfico de barras na parte superior esquerda do relatório.

    Isso mostrará o resumo da consulta e do plano para a consulta de maior duração em seu repositório de consultas. Confira o gráfico *Resumo do plano* no canto superior direito do relatório e o *plano de consulta* na parte inferior do relatório.

## Forçar um plano de execução melhor

1. Navegue até a parte de resumo do plano no relatório, como mostrado abaixo. Você notará que há dois planos de execução com durações bem diferentes.

1. Selecione a ID do plano com a menor duração (isso é indicado pela menor posição no eixo Y do gráfico) na janela superior direita do relatório. Selecione a ID do plano ao lado do gráfico Resumo do Plano.

1. Selecione **Forçar Plano** no gráfico de resumo. Na janela de confirmação exibida, selecione **Sim**.

    Quando o plano for forçado, você verá que o **Plano Forçado** está esmaecido e o plano na janela de resumo tem uma marca de seleção indicando que ele foi forçado.

    Em algumas ocasiões, o otimizador de consulta pode fazer uma escolha inadequada sobre qual plano de execução usar. Quando isso acontecer, você poderá forçar o SQL Server a usar o plano desejado se souber que ele funciona melhor.

## Usar as dicas de consulta para impactar o desempenho

A seguir, você vai executar uma carga de trabalho, alterar a consulta para usar um parâmetro, aplicar uma dica de consulta à consulta e executá-la novamente.

Antes de continuar com o exercício, feche todas as janelas de consulta atuais selecionando o menu **Janela** e **Fechar todos os documentos**. No pop-up, selecione **Não**.

1. Escolha **Nova Consulta** e selecione o ícone **Incluir Plano de Execução Real** antes de executar a consulta ou use <kbd>CTRL</kbd>+<kbd>M</kbd>.

1. Execute a consulta abaixo. Observe que o plano de execução mostra um operador de busca de índice.

    ```sql
    USE AdventureWorks2017;

    GO

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID=288;
    ```

1. Em uma nova janela de consulta, execute a próxima consulta. Comparar os dois planos de execução.

    ```sql
    USE AdventureWorks2017;
    GO

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID=277;
    ```

    A única alteração desta vez é que o valor de SalesPersonID está definido como 277. Observe a operação de Exame de Índice Clusterizado no plano de execução.

Como podemos ver, com base nas estatísticas de índice, o otimizador de consulta escolheu um plano de execução diferente devido aos valores variados na cláusula **WHERE**.

Por que temos planos diferentes se alteramos apenas o valor de *SalesPersonID*?

Essa consulta usa uma constante na cláusula **WHERE**. O otimizador vê cada uma dessas consultas como exclusiva e gera um plano de execução diferente toda vez.

## Alterar a consulta para usar uma variável e usar uma Dica de Consulta

1. Altere a consulta para usar um valor de variável para SalesPersonID.

1. Use a instrução T-SQL **DECLARE** para declarar <strong>@SalesPersonID</strong>, de modo que você possa passar um valor em vez de codificar o valor na cláusula **WHERE**. Para evitar conversão explícita, certifique-se de que o tipo de dados da variável corresponda ao tipo de dados da coluna na tabela de destino. Execute a consulta com o plano de consulta real habilitado.

    ```sql
    USE AdventureWorks2017;

    GO

    SET STATISTICS IO, TIME ON;

    DECLARE @SalesPersonID INT;

    SELECT @SalesPersonID = 288;

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID= @SalesPersonID;
    ```

    Se você examinar o plano de execução, notará que ele está usando uma verificação de índice para obter os resultados. O otimizador de consulta não pôde fazer boas otimizações, pois não consegue saber o valor da variável local até o runtime.

1. Você pode ajudar o otimizador de consulta a fazer escolhas melhores ao fornecer uma dica de consulta. Execute novamente a consulta acima com a **OPÇÃO (RECOMPILE)**:

    ```sql
    USE AdventureWorks2017

    GO

    SET STATISTICS IO, TIME ON;

    DECLARE @SalesPersonID INT;

    SELECT @SalesPersonID = 288;

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID= @SalesPersonID
    OPTION (RECOMPILE);
    ```

    Observe que o otimizador de consulta conseguiu escolher um plano de execução mais eficiente. A opção **RECOMPILE** faz com que o compilador de consulta substitua a variável pelo seu valor.

    Ao comparar as estatísticas, você pode ver na guia de mensagem que a diferença entre as leituras lógicas é **68%** maior (689 contra 409) na consulta sem a dica.

---

## Limpeza

Se você não estiver usando o banco de dados ou os arquivos de laboratório para qualquer outra finalidade, poderá limpar os objetos criados neste laboratório.

### Exclua a pasta C:\LabFiles

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, abra o **oExplorador de Arquivos**.
1. Navegue até **C:\\**.
1. Exclua a pasta **C:\LabFiles**.

### Exclua o banco de dados AdventureWorks2017

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, inicie uma Sessão do SQL Server Management Studio (SSMS).
1. Quando o SSMS abrir, por padrão, a caixa de diálogo **Connect to Service** será exibida. Escolha Default instance e selecione **Connect**. Talvez seja necessário marcar a caixa de seleção **Trust server certificate**.
1. Em **Object Explorer**, expanda a pasta **Databases**.
1. Clique com o botão direito do mouse no banco de dados **AdventureWorks2017** e selecione **Delete**.
1. Na caixa de diálogo **Delete Object**, marque a caixa de conexão **Close existing connections** 
1. Selecione **OK**.

---

Você concluiu este laboratório.

Neste exercício, você aprendeu a identificar problemas de consulta e a resolvê-los para melhorar o plano de consulta.
