---
lab:
  title: Laboratório 9 – Identificar problemas de design de banco de dados
  module: Optimize query performance in Azure SQL
---

# Identificar problemas de design de banco de dados

**Tempo estimado**: 15 minutos

Os alunos levarão as informações obtidas nas aulas para definir o escopo das entregas para um projeto de transformação digital no AdventureWorks. Ao examinar o portal do Azure, bem como outras ferramentas, os alunos determinarão como utilizar ferramentas nativas para identificar e resolver problemas relacionados ao desempenho. Por fim, os alunos poderão avaliar um design de banco de dados em relação a problemas com normalização, seleção de tipo de dados e design de índice.

Você foi contratado como administrador de banco de dados para identificar problemas relacionados ao desempenho e fornecer soluções viáveis para solucionar todos os problemas encontrados. A AdventureWorks vende bicicletas e peças para bicicletas diretamente para consumidores e distribuidores há mais de uma década. Seu trabalho é identificar problemas no desempenho das consultas e resolvê-los usando as técnicas aprendidas neste módulo.

**Observação:** estes exercícios pedem que você copie e cole o código T-SQL. Verifique se o código foi copiado corretamente antes de executá-lo.

## Restaurar um banco de dados

1. Faça download do arquivo de backup do banco de dados localizado em **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** para o caminho **C:\LabFiles\Monitor and optimize** na máquina virtual do laboratório (crie a estrutura de pastas se ela não existir).

    ![Imagem 03](../images/dp-300-module-07-lab-03.png)

1. Selecione o botão Iniciar do Windows e digite SSMS. Selecione **Microsoft SQL Server Management Studio 18** na lista.  

    ![Imagem 01](../images/dp-300-module-01-lab-34.png)

1. Quando o SSMS for aberto, observe que a caixa de diálogo **Conectar ao Servidor** será pré-preenchida com o nome de instância padrão. Selecione **Conectar**.

    ![Imagem 02](../images/dp-300-module-07-lab-01.png)

1. Selecione a pasta**Bancos de Dados** e **Nova Consulta**.

    ![Imagem 03](../images/dp-300-module-07-lab-04.png)

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

    **Observação:** o nome e o caminho do arquivo de backup do banco de dados devem corresponder ao que você baixou na etapa 1, caso contrário, o comando falhará.

1. Uma mensagem de sucesso será exibida após a conclusão da restauração.

    ![Imagem 03](../images/dp-300-module-07-lab-05.png)

## Examinar a consulta e identificar o problema

1. Selecione **Nova Consulta**. Copie e cole o código T-SQL abaixo na janela de consulta. Clique em **Executar** para executar esta consulta.

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

1. Antes de executar a consulta, selecione o ícone **Incluir Plano de Execução Real**, como mostrado abaixo, ou pressione **CTRL+M**. Isso fará com que o plano de execução seja exibido quando você executar a consulta. Clique em **Executar** para executar esta consulta.

    ![Imagem 01](../images/dp-300-module-09-lab-01.png)

1. Navegue até o plano de execução e selecione a guia **Plano de execução** no painel de resultados. No plano de execução, passe o mouse sobre o operador `SELECT`. Você notará uma mensagem de aviso identificada por um ponto de exclamação em um triângulo amarelo, conforme mostrado abaixo. Identifique o que a mensagem de aviso informa.

    ![Imagem 02](../images/dp-300-module-09-lab-02.png)

## Identificar maneiras de corrigir a mensagem de aviso

A estrutura de tabela *[HumanResources].[Employee]* é mostrada na instrução DDL (linguagem de definição de dados) a seguir. Examine os campos que foram usados na consulta SQL anterior nessa DDL, prestando atenção em seus tipos.

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

        Você notará que o valor comparado à coluna *NationalIDNumber* na cláusula `WHERE` é comparado como um número, pois **14417807** não está em uma cadeia de caracteres entre aspas. 

        Depois de examinar a estrutura da tabela, você verá que a coluna *NationalIDNumber* está usando o tipo de dados `NVARCHAR` e não um tipo de dados `INT`. Essa inconsistência faz com que o otimizador de banco de dados converta implicitamente o número em um valor `NVARCHAR`, o que gera sobrecarga adicional no desempenho da consulta e cria um plano de qualidade inferior.

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

    ![Imagem 03](../images/dp-300-module-09-lab-03.png)

    **Observação:** a mensagem de aviso agora não aparece e o plano de consulta melhorou. A alteração da cláusula `WHERE` para que o valor comparado à coluna *NationalIDNumber* corresponda ao tipo de dados da coluna na tabela permitiu que o otimizador se livrasse da conversão implícita.

### Alterar o tipo de dados

1. Também podemos corrigir o aviso de conversão implícita alterando a estrutura da tabela.

    Para tentar corrigir o índice, copie e cole a consulta abaixo em uma nova janela de consulta a fim de alterar o tipo de dados da coluna. Tente executar a consulta selecionando **Executar** ou pressionando <kbd>F5</kbd>.

    ```sql
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;
    ```

    Alterar o tipo de dados da coluna *NationalIDNumber* para INT resolveria o problema de conversão. No entanto, essa alteração apresenta outro problema que, como administrador de banco de dados, você precisa resolver.

    ![Imagem 04](../images/dp-300-module-09-lab-04.png)

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

1. Como alternativa, você pode executar a consulta abaixo para confirmar se o tipo de dados foi alterado corretamente.

    ```sql
    SELECT c.name, t.name
    FROM sys.all_columns c INNER JOIN sys.types t
        ON (c.system_type_id = t.user_type_id)
    WHERE OBJECT_ID('[HumanResources].[Employee]') = c.object_id
        AND c.name = 'NationalIDNumber'
    ```
    
    ![Imagem 05](../images/dp-300-module-09-lab-05.png)
    
1. Agora vamos verificar o plano de execução. Execute novamente a consulta original sem as aspas.

    ```sql
    USE AdventureWorks2017
    GO

    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

    ![Imagem 06](../images/dp-300-module-09-lab-06.png)

    Examine o plano de consulta e observe que agora você pode usar um inteiro para filtrar por *NationalIDNumber* sem o aviso de conversão implícita. O otimizador de consulta do SQL agora pode gerar e executar o plano mais adequado.

Neste exercício, você aprendeu a identificar problemas de consulta causados por conversões implícitas de tipo de dados e como corrigi-los para melhorar o plano de consulta.
