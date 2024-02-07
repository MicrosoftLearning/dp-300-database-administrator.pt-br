---
lab:
  title: 'Laboratório: detectar e corrigir problemas de fragmentação'
  module: Monitor and optimize operational resources in Azure SQL
---

# Detectar e corrigir problemas de fragmentação

**Tempo estimado**: 15 minutos

Os alunos levarão as informações obtidas nas aulas para definir o escopo dos resultados de um projeto de transformação digital dentro da AdventureWorks. Examinando o portal do Azure, bem como outras ferramentas, os alunos determinarão como utilizar ferramentas nativas para identificar e resolver problemas relacionados ao desempenho. Finalmente, os alunos serão capazes de identificar a fragmentação dentro do banco de dados, bem como aprender etapas para resolvê-la adequadamente.

Você foi contratado como administrador de banco de dados para identificar problemas relacionados ao desempenho e fornecer soluções viáveis para solucionar todos os problemas encontrados. A AdventureWorks vende bicicletas e peças para bicicletas diretamente para clientes finais e distribuidores há mais de uma década. Recentemente, a empresa notou uma degradação de desempenho em seus produtos que são usados para atender às solicitações dos clientes. Você precisa usar ferramentas locais para identificar problemas de desempenho e sugerir métodos para solucioná-los.

**Nota:** Estes exercícios pedem que você copie e cole o código T-SQL. Verifique se o código foi copiado corretamente, antes de executar o código.

## Restaurar um banco de dados

1. Faça download do arquivo de backup do banco de dados localizado em ****https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** C:\LabFiles\Monitor e otimize** o caminho na máquina virtual do laboratório (crie a estrutura de pastas se ela não existir).

    ![Foto 03](../images/dp-300-module-07-lab-03.png)

1. Selecione o botão Iniciar do Windows e digite SSMS. Selecione **Microsoft SQL Server Management Studio 18** na lista.  

    ![Figura 48](../images/dp-300-module-01-lab-34.png)

1. Quando o SSMS for aberto, observe que a caixa de diálogo Conectar ao Servidor** será pré-preenchida com o **nome de instância padrão. Selecione **Conectar**.

    ![Foto 02](../images/dp-300-module-07-lab-01.png)

1. Selecione a **pasta Bancos de Dados** e Nova **Consulta**.

    ![Foto 03](../images/dp-300-module-07-lab-04.png)

1. Na janela Nova consulta, copie e cole o T-SQL abaixo. Execute a consulta para restaurar o banco de dados.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017_log.ldf';
    ```

    **Nota:** O nome e o caminho do arquivo de backup do banco de dados devem corresponder ao que você baixou na etapa 1, caso contrário, o comando falhará.

1. Você verá uma mensagem bem-sucedida após a conclusão da restauração.

    ![Foto 03](../images/dp-300-module-07-lab-05.png)

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

    Essa consulta relatará quaisquer índices que tenham uma fragmentação acima de 50%. A consulta não deve retornar nenhum resultado.

1. Selecione **Nova Consulta**. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    USE AdventureWorks2017
    GO
        
    INSERT INTO [Person].[Address]
        ([AddressLine1]
        ,[AddressLine2]
        ,[City]
        ,[StateProvinceID]
        ,[PostalCode]
        ,[SpatialLocation]
        ,[rowguid]
        ,[ModifiedDate])
        
    SELECT AddressLine1,
        AddressLine2, 
        'Amsterdam',
        StateProvinceID, 
        PostalCode, 
        SpatialLocation, 
        newid(), 
        getdate()
    FROM Person.Address;
    
    GO
    ```

    Essa consulta aumentará o nível de fragmentação da tabela Person.Address e seus índices adicionando um grande número de registros novos.

1. Execute a primeira consulta novamente. Agora você deve ser capaz de ver quatro índices altamente fragmentados.

    ![Foto 03](../images/dp-300-module-07-lab-06.png)

1. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

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

    Clique na guia **Mensagens** no painel de resultados do SQL Server Management Studio. Anote a contagem de leituras lógicas executadas pela consulta.

    ![Foto 03](../images/dp-300-module-07-lab-07.png)

## Recompilar índices fragmentados

1. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

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

1. Confirme se o índice de **AK_Address_StateProvinceID** não tem mais fragmentação maior que 50%.

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

    Comparando os resultados, podemos ver que a fragmentação caiu de 81% para 0.

1. Execute novamente a instrução select da seção anterior. Anote as leituras lógicas na guia **Mensagens** do painel de **Resultados** no Management Studio. Houve uma alteração do número de leituras lógicas encontradas antes de você recompilar o índice?

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

Neste exercício, você aprendeu a recriar o índice e analisar leituras lógicas para aumentar o desempenho da consulta.
