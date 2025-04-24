---
lab:
  title: Laboratório 8 – Identificar e resolver problemas de bloqueio
  module: Optimize query performance in Azure SQL
---

# Identificar e resolver problemas de bloqueio

**Tempo estimado**: 15 minutos

Os alunos levarão as informações obtidas nas aulas para definir o escopo das entregas para um projeto de transformação digital no AdventureWorks. Ao examinar o portal do Azure, bem como outras ferramentas, os alunos determinarão como utilizar ferramentas nativas para identificar e resolver problemas relacionados ao desempenho. Finalmente, os alunos serão capazes de identificar e resolver problemas de bloqueio adequadamente.

Você foi contratado como administrador de banco de dados para identificar problemas relacionados ao desempenho e fornecer soluções viáveis para solucionar todos os problemas encontrados. Você precisa investigar os problemas de desempenho e sugerir métodos para resolvê-los.

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

## Executar um relatório de consultas bloqueadas

1. Selecione **Nova Consulta**. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    USE MASTER

    GO

    CREATE EVENT SESSION [Blocking] ON SERVER 
    ADD EVENT sqlserver.blocked_process_report(
    ACTION(sqlserver.client_app_name,sqlserver.client_hostname,sqlserver.database_id,sqlserver.database_name,sqlserver.nt_username,sqlserver.session_id,sqlserver.sql_text,sqlserver.username))
    ADD TARGET package0.ring_buffer
    WITH (MAX_MEMORY=4096 KB, EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS, MAX_DISPATCH_LATENCY=30 SECONDS, MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE, TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)

    GO

    -- Start the event session 
    ALTER EVENT SESSION [Blocking] ON SERVER 
    STATE = start;

    GO
    ```

    O código T-SQL mostrado acima criará uma sessão de Evento Estendido que vai capturar eventos de bloqueio. Os dados conterão os seguintes elementos:

    - Nome do aplicativo cliente
    - O nome do host do cliente
    - ID do banco de dados
    - Nome do banco de dados
    - Nome de usuário do NT
    - ID da Sessão
    - Texto do T-SQL
    - Nome de Usuário

1. Selecione **Nova Consulta**. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    EXEC sys.sp_configure N'show advanced options', 1

    RECONFIGURE WITH OVERRIDE;

    GO
    EXEC sp_configure 'blocked process threshold (s)', 60

    RECONFIGURE WITH OVERRIDE;

    GO
    ```

    > &#128221 O comando acima especifica o limite, em segundos, no qual os relatórios de processos bloqueados são gerados. Como resultado, não somos obrigados a esperar tanto tempo para que o *blocked_process_report* seja levantado nesta lição.

1. Selecione **Nova Consulta**. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO

    BEGIN TRANSACTION
        UPDATE Person.Person 
        SET LastName = LastName;

    GO
    ```

1. Abra outra janela de consulta clicando no botão **Nova Consulta**. Copie e cole o seguinte código T-SQL na nova janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO

    SELECT TOP (1000) [LastName]
      ,[FirstName]
      ,[Title]
    FROM Person.Person
    WHERE FirstName = 'David'
    ```

    > &#128221; Esta consulta não retorna nenhum resultado e parece ser executada indefinidamente.

1. Em **Pesquisador de Objetos**, expanda **Gerenciamento** -> **Eventos Estendidos** -> **Sessões**.

    Observe que o evento estendido chamado *Bloqueio* que acabamos de criar está na lista.

1. Expanda o evento estendido *Bloqueio* e clique com o botão direito do mouse em **package0.ring_buffer**. Selecione **Exibir dados de destino**.

1. Selecione o hiperlink listado.

1. O XML mostrará quais processos estão sendo bloqueados e qual processo está causando o bloqueio. Será possível conferir as consultas que foram executadas nesse processo, bem como informações do sistema. Observe que os IDs de sessão (SPID).

1. Como alternativa, você pode executar a seguinte consulta para identificar sessões que bloqueiam outras sessões, incluindo uma lista de IDs de sessão bloqueadas por *session_id*. Abra uma janela **Nova Consulta**, copie e cole o seguinte código T-SQL nela e selecione **Executar**.

    ```sql
    WITH cteBL (session_id, blocking_these) AS 
    (SELECT s.session_id, blocking_these = x.blocking_these FROM sys.dm_exec_sessions s 
    CROSS APPLY    (SELECT isnull(convert(varchar(6), er.session_id),'') + ', '  
                    FROM sys.dm_exec_requests as er
                    WHERE er.blocking_session_id = isnull(s.session_id ,0)
                    AND er.blocking_session_id <> 0
                    FOR XML PATH('') ) AS x (blocking_these)
    )
    SELECT s.session_id, blocked_by = r.blocking_session_id, bl.blocking_these
    , batch_text = t.text, input_buffer = ib.event_info, * 
    FROM sys.dm_exec_sessions s 
    LEFT OUTER JOIN sys.dm_exec_requests r on r.session_id = s.session_id
    INNER JOIN cteBL as bl on s.session_id = bl.session_id
    OUTER APPLY sys.dm_exec_sql_text (r.sql_handle) t
    OUTER APPLY sys.dm_exec_input_buffer(s.session_id, NULL) AS ib
    WHERE blocking_these is not null or r.blocking_session_id > 0
    ORDER BY len(bl.blocking_these) desc, r.blocking_session_id desc, r.session_id;
    ```

    > &#128221; A consulta acima retornará os mesmos SPIDs que o XML.

1. Clique com o botão direito do mouse no evento estendido chamado **Bloqueio**, e selecione **Parar Sessão**.

1. Navegue de volta para a sessão de consulta que está causando o bloqueio e digite `ROLLBACK TRANSACTION` na linha abaixo da consulta. Realce `ROLLBACK TRANSACTION` e selecione **Executar**.

1. Navegue de volta para a sessão de consulta que estava sendo bloqueada. Você observará que a consulta já foi concluída.

## Habilitar nível de isolamento do instantâneo da confirmação de leitura

1. Clique no botão **Nova Consulta** do SQL Server Management Studio. Copie e cole o código T-SQL abaixo na janela de consulta. Clique no botão **Executar** para executar esta consulta.

    ```sql
    USE master

    GO
    
    ALTER DATABASE AdventureWorks2017 SET READ_COMMITTED_SNAPSHOT ON WITH ROLLBACK IMMEDIATE;

    GO
    ```

1. Execute novamente a consulta que causou o bloqueio em um novo Editor de Consultas. *Não execute o comando ROLLBACK TRANSACTION*.

    ```sql
    USE AdventureWorks2017
    GO
    
    BEGIN TRANSACTION
        UPDATE Person.Person 
        SET LastName = LastName;
    GO
    ```

1. Execute novamente a consulta que estava sendo bloqueada em um novo Editor de Consultas.

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT TOP (1000) [LastName]
     ,[FirstName]
     ,[Title]
    FROM Person.Person
    WHERE firstname = 'David'
    ```

    Por que a mesma consulta é concluída, enquanto na tarefa anterior ela foi bloqueada pelo comando update?

    O nível de isolamento Instantâneo de Leitura Confirmada é uma forma otimista de isolamento de transação, e a última consulta mostrará a última versão confirmada dos dados, em vez de ser bloqueada.

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

Neste exercício, você aprendeu a identificar sessões que estão sendo bloqueadas e a mitigar esses cenários.
