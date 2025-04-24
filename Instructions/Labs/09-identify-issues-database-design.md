---
lab:
  title: Laboratório 9 – Identificar problemas de design de banco de dados
  module: Optimize query performance in Azure SQL
---

# Identificar problemas de design de banco de dados

**Tempo estimado**: 15 minutos

Os alunos levarão as informações obtidas nas aulas para definir o escopo das entregas para um projeto de transformação digital no AdventureWorks. Ao examinar o portal do Azure, bem como outras ferramentas, os alunos determinarão como utilizar ferramentas nativas para identificar e resolver problemas relacionados ao desempenho. Por fim, os alunos poderão avaliar um design de banco de dados em relação a problemas com normalização, seleção de tipo de dados e design de índice.

Você foi contratado como administrador de banco de dados para identificar problemas relacionados ao desempenho e fornecer soluções viáveis para solucionar todos os problemas encontrados. A AdventureWorks vende bicicletas e peças para bicicletas diretamente para consumidores e distribuidores há mais de uma década. Seu trabalho é identificar problemas no desempenho das consultas e resolvê-los usando as técnicas aprendidas neste módulo.

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

## Examinar a consulta e identificar o problema

1. Selecione **Nova Consulta**. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    USE AdventureWorks2017

    GO
    
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

1. Clique no ícone **Incluir Plano de Execução Real** à direita do botão **Executar** antes de executar a consulta ou pressione **CTRL+M**. Isso fará com que o plano de execução seja exibido quando você executar a consulta. Clique em **Executar** para executar esta consulta.

1. Navegue até o plano de execução e selecione a guia **Plano de execução** no painel de resultados. Você notará que o operador **SELECT** tem um triângulo amarelo com um ponto de exclamação. Isso indica que há uma mensagem de aviso associada ao operador. Passe o mouse sobre o ícone de aviso para ver a mensagem e ler a mensagem de aviso.

    > &#128221; A mensagem de aviso informa que há uma conversão implícita na consulta. Isso significa que o otimizador de consulta do SQL Server teve que converter o tipo de dados de uma das colunas na consulta em outro tipo de dados para executar a consulta.

## Identificar maneiras de corrigir a mensagem de aviso

A estrutura de tabela *[HumanResources].[Employee]* é definida pela seguinte na instrução de linguagem de definição de dados (DDL). Examine os campos que foram usados na consulta SQL anterior nessa DDL, prestando atenção em seus tipos.

```sql
CREATE TABLE [HumanResources].[Employee](
     [BusinessEntityID] [int] NOT NULL,
     [NationalIDNumber] [nvarchar](15) NOT NULL,
     [LoginID] [nvarchar](256) NOT NULL,
     [OrganizationNode] [hierarchyid] NULL,
     [OrganizationLevel] AS ([OrganizationNode].[GetLevel]()),
     [JobTitle] [nvarchar](50) NOT NULL,
     [BirthDate] [date] NOT NULL,
     [MaritalStatus] [nchar](1) NOT NULL,
     [Gender] [nchar](1) NOT NULL,
     [HireDate] [date] NOT NULL,
     [SalariedFlag] [dbo].[Flag] NOT NULL,
     [VacationHours] [smallint] NOT NULL,
     [SickLeaveHours] [smallint] NOT NULL,
     [CurrentFlag] [dbo].[Flag] NOT NULL,
     [rowguid] [uniqueidentifier] ROWGUIDCOL NOT NULL,
     [ModifiedDate] [datetime] NOT NULL
) ON [PRIMARY]
```

1. De acordo com a mensagem de aviso apresentada no plano de execução, que mudança você recomendaria?

    1. Identifique qual campo está causando a conversão implícita e por quê.
    1. Se você examinar a consulta:

        ```sql
        SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
        FROM HumanResources.Employee
        WHERE NationalIDNumber = 14417807;
        ```

        Você notará que o valor comparado à coluna *NationalIDNumber* na cláusula **WHERE** é comparado como um número, já que **14417807** não é uma cadeia entre aspas.

        Após examinar a estrutura da tabela, você descobrirá que a coluna *NationalIDNumber* está usando o tipo de dados **NVARCHAR** e não um tipo de dados **INT**. Essa inconsistência faz com que o otimizador de banco de dados converta implicitamente o número em um valor *NVARCHAR*, o que gera sobrecarga adicional no desempenho da consulta e cria um plano de qualidade inferior.

Há duas abordagens que podemos implementar para corrigir o aviso de conversão implícita. Vamos investigar cada uma delas nas próximas etapas.

### Alterar o código

1. Como você alteraria o código para resolver a conversão implícita? Altere o código e execute a consulta novamente.

    Lembre-se de ativar a opção **Incluir Plano de Execução Real** (**CTRL+M**) se ainda não tiver ativada. 

    Nesse cenário, basta adicionar um único símbolo de aspas em cada lado do valor para alterá-lo de um número para um formato de caractere. Mantenha a janela de consulta aberta para essa consulta.

    Execute a consulta SQL atualizada:

    ```sql
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = '14417807';
    ```

    > &#128221; Observe que a mensagem de aviso sumiu e o plano de consulta melhorou. Alterar a cláusula *WHERE* para que o valor comparado à coluna *NationalIDNumber* corresponda ao tipo de dados da coluna na tabela, o otimizador conseguiu se livrar da conversão implícita e gerar um plano mais otimizado.

### Alterar o tipo de dados

1. Também podemos corrigir o aviso de conversão implícita alterando a estrutura da tabela.

    Para tentar corrigir o índice, copie e cole a consulta abaixo em uma nova janela de consulta a fim de alterar o tipo de dados da coluna. Tente executar a consulta selecionando **Executar** ou pressionando <kbd>F5</kbd>.

    ```sql
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;
    ```

    Alterar o tipo de dados da coluna *NationalIDNumber* para INT resolveria o problema de conversão. No entanto, essa alteração apresenta outro problema que, como administrador de banco de dados, você precisa resolver. A execução da consulta acima resultará na seguinte mensagem de erro:

    <span style="color:red">Msg 5074, Level 16, Sate 1, Line1  The index 'AK_Employee_NationalIDNumber' is dependent on column 'NationalIDNumber  Msg 4922, Level 16, State 9, Line 1  ALTER TABLE ALTER COLUMN NationalIDNumber failed because one or more objects access this column</span>

    A coluna *NationalIDNumber* faz parte de um índice não clusterizado já existente. O índice precisa ser recriado para que o tipo de dados seja alterado. **Isso pode gerar um tempo de inatividade estendido na produção, o que realça a importância da escolha dos tipos de dados certos em seu design.**

1. Para resolver esse problema, copie e cole o código abaixo na janela de consulta e execute-o selecionando **Executar**.

    ```sql
    USE AdventureWorks2017

    GO
    
    --Dropping the index first
    DROP INDEX [AK_Employee_NationalIDNumber] ON [HumanResources].[Employee]

    GO

    --Changing the column data type to resolve the implicit conversion warning
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;

    GO

    --Recreating the index
    CREATE UNIQUE NONCLUSTERED INDEX [AK_Employee_NationalIDNumber] ON [HumanResources].[Employee]( [NationalIDNumber] ASC );

    GO
    ```

1. Execute a consulta abaixo para confirmar se o tipo de dados foi alterado corretamente.

    ```sql
    SELECT c.name, t.name
    FROM sys.all_columns c INNER JOIN sys.types t
        ON (c.system_type_id = t.user_type_id)
    WHERE OBJECT_ID('[HumanResources].[Employee]') = c.object_id
        AND c.name = 'NationalIDNumber'
    ```

1. Agora vamos verificar o plano de execução. Execute novamente a consulta original sem as aspas.

    ```sql
    USE AdventureWorks2017
    GO

    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

     Examine o plano de consulta e observe que agora você pode usar um inteiro para filtrar por *NationalIDNumber* sem o aviso de conversão implícita. O otimizador de consulta do SQL agora pode gerar e executar o plano mais adequado.

>&#128221; Embora alterar o tipo de dados de uma coluna possa resolver problemas de conversão implícita, nem sempre é a melhor solução. Nesse caso, alterar o tipo de dados da coluna *NationalIDNumber* para um tipo de dados **INT** teria causado tempo de inatividade na produção, pois o índice nessa coluna teria que ser descartado e recriado. É importante considerar o impacto da alteração do tipo de dados de uma coluna em consultas e índices existentes antes de fazer qualquer alteração. Além disso, pode haver outras consultas que dependem da coluna *NationalIDNumber* ser um tipo de dados **NVARCHAR**, portanto, alterar o tipo de dados pode interromper essas consultas.

---

## Limpeza

Se você não estiver usando o banco de dados ou os arquivos de laboratório para qualquer outra finalidade, poderá limpar os objetos criados neste laboratório.

### Exclua a pasta C:\LabFiles

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, abra o **oExplorador de Arquivos**.
1. Navegue até **C:\\**.
1. Exclua a pasta **C:\LabFiles**.

## Exclua o banco de dados AdventureWorks2017

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, inicie uma Sessão do SQL Server Management Studio (SSMS).
1. Quando o SSMS abrir, por padrão, a caixa de diálogo **Connect to Service** será exibida. Escolha Default instance e selecione **Connect**. Talvez seja necessário marcar a caixa de seleção **Trust server certificate**.
1. Em **Object Explorer**, expanda a pasta **Databases**.
1. Clique com o botão direito do mouse no banco de dados **AdventureWorks2017** e selecione **Delete**.
1. Na caixa de diálogo **Delete Object**, marque a caixa de conexão **Close existing connections** 
1. Selecione **OK**.

---

Você concluiu este laboratório.

Neste exercício, você aprendeu a identificar problemas de consulta causados por conversões implícitas de tipo de dados e como corrigi-los para melhorar o plano de consulta.
