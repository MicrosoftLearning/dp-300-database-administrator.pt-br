---
lab:
  title: Laboratório 7 – Detectar e corrigir problemas de fragmentação
  module: Monitor and optimize operational resources in Azure SQL
---

# Detectar e corrigir problemas de fragmentação

**Tempo estimado**: 20 minutos

Os alunos levarão as informações obtidas nas aulas para definir o escopo das entregas para um projeto de transformação digital no AdventureWorks. Ao examinar o portal do Azure, bem como outras ferramentas, os alunos determinarão como utilizar ferramentas nativas para identificar e resolver problemas relacionados ao desempenho. Por fim, os alunos saberão identificar a fragmentação dentro do banco de dados e aprenderão as etapas para resolvê-la adequadamente.

Você foi contratado como administrador de banco de dados para identificar problemas relacionados ao desempenho e fornecer soluções viáveis para solucionar todos os problemas encontrados. A AdventureWorks vende bicicletas e peças para bicicletas diretamente para consumidores e distribuidores há mais de uma década. Recentemente, a empresa notou uma degradação do desempenho de seus produtos que são usados para atender às solicitações dos clientes. Você precisa usar ferramentas SQL para identificar problemas de desempenho e sugerir métodos para solucioná-los.

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

## Investigar fragmentação de índice

1. Selecione **Nova Consulta**. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO
    
    SELECT i.name Index_Name
     , avg_fragmentation_in_percent
     , db_name(database_id)
     , i.object_id
     , i.index_id
     , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
     INNER JOIN sys.indexes i ON ps.object_id = i.object_id 
     AND ps.index_id = i.index_id
    WHERE avg_fragmentation_in_percent > 50 -- find indexes where fragmentation is greater than 50%
    ```

    Essa consulta relatará índices que tenham uma fragmentação acima de **50%**. A consulta não deve retornar nenhum resultado.

1. A fragmentação do índice pode ser causada por vários fatores, incluindo os seguintes:

    - Atualizações frequentes da tabela ou do índice.
    - Inserções ou exclusões frequentes na tabela ou índice.
    - Divisões de página.

    Para aumentar o nível de fragmentação da tabela Person.Address e seus índices, você inserirá e excluirá um grande número de registros. Para fazer isso, execute a consulta a seguir.

    Selecione **Nova Consulta**. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO
    
    -- Insert 60000 records into the Address table    

    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Split Avenue ' + CAST(v1.number AS VARCHAR(10)), 
        'Apt ' + CAST(v2.number AS VARCHAR(10)), 
        'PageSplitTown', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '88' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure unique rowguid
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 300 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;
    GO
    
    -- DELETE 25000 records from the Address table
    DELETE FROM [Person].[Address] WHERE AddressID BETWEEN 35001 AND 60000;

    GO

    -- Insert 40000 records into the Address table
    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Fragmented Street ' + CAST(v1.number AS VARCHAR(10)), 
        'Suite ' + CAST(v2.number AS VARCHAR(10)), 
        'FragmentCity', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '99' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure a unique rowguid per row
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 200 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;

    GO
    ```

    Essa consulta aumentará o nível de fragmentação da tabela Person.Address e seus índices adicionando e excluindo um grande número de registros.

1. Execute a primeira consulta novamente. Agora você poderá ver quatro índices altamente fragmentados.

1. Selecione uma **Nova Consulta** e copie e cole o seguinte código T-SQL na janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    SET STATISTICS IO,TIME ON

    GO
        
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

    Clique na guia **Mensagens** no painel de resultados do SQL Server Management Studio. Anote a contagem de leituras lógicas executadas pela consulta na tabela **Endereço**.

## Recompilar índices fragmentados

1. Selecione uma **Nova Consulta** e copie e cole o seguinte código T-SQL na janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO
    
    ALTER INDEX [IX_Address_StateProvinceID] ON [Person].[Address] REBUILD PARTITION = ALL 
    WITH (PAD_INDEX = OFF, 
        STATISTICS_NORECOMPUTE = OFF, 
        SORT_IN_TEMPDB = OFF, 
        IGNORE_DUP_KEY = OFF, 
        ONLINE = OFF, 
        ALLOW_ROW_LOCKS = ON, 
        ALLOW_PAGE_LOCKS = ON)
    ```

1. Selecione uma **Nova Consulta** e execute a seguinte consulta para confirmar se o índice **IX_Address_StateProvinceID** não tem mais fragmentação maior que 50%.

    ```sql
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT i.name Index_Name
        , avg_fragmentation_in_percent
        , db_name(database_id)
        , i.object_id
        , i.index_id
        , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
        INNER JOIN sys.indexes i ON (ps.object_id = i.object_id AND ps.index_id = i.index_id)
    WHERE i.name = 'IX_Address_StateProvinceID'
    ```

    Ao comparar os resultados, podemos ver que a fragmentação **IX_Address_StateProvinceI** caiu de 88% para 0.

1. Execute novamente a instrução select da seção anterior. Anote as leituras lógicas na guia **Mensagens** do painel de **Resultados** no Management Studio. *Houve uma alteração do número de leituras lógicas encontradas antes de você recompilar o índice da tabela de endereço?*

    ```sql
    SET STATISTICS IO,TIME ON

    GO
        
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

Como o índice foi recompilado, agora ele terá a maior eficácia possível, e as leituras lógicas deverão ser reduzidas. Agora você viu que a manutenção do índice pode ter um efeito no desempenho da consulta.

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

Neste exercício, você aprendeu a recriar o índice e analisar leituras lógicas para aumentar o desempenho da consulta.
