---
lab:
  title: Laboratório 3 — Autorizar o acesso ao Banco de Dados SQL do Azure com o Microsoft Entra ID
  module: Implement a Secure Environment for a Database Service
---

# Configurar a autenticação e autorização do banco de dados

**Tempo estimado: 25 minutos**

Os alunos usarão as informações obtidas nas lições para configurar e, posteriormente, implementar a segurança no Portal do Azure e no banco de dados do *AdventureWorksLT*.

Você foi contratado como um Administrador de Banco de Dados Sênior para ajudar a garantir a segurança do ambiente de banco de dados.

> &#128221; Esses exercícios solicitam que você copie e cole o código T-SQL e use os recursos existentes do SQL. Verifique se o código foi copiado corretamente antes de executá-lo.

## Ambiente de configuração

Se a máquina virtual do laboratório tiver sido fornecida e pré-configurada, você encontrará os arquivos de laboratório prontos na pasta **C:\LabFiles**. *Reserve um momento para verificar; se os arquivos já estiverem lá, pule esta seção*. No entanto, se você estiver usando sua própria máquina ou os arquivos de laboratório estiverem ausentes, será necessário cloná-los do *GitHub* para continuar.

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, inicie uma sessão do Visual Studio Code.

1. Abra a paleta de comandos; (Ctrl+Shift+P) e digite **Git: Clone**. Selecione a opção **Git: Clone**.

1. Cole a URL a seguir no campo **URL do repositório** e selecione **Enter**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Salve o repositório na pasta **C:\LabFiles** na máquina virtual do laboratório ou em seu computador local, se não tiver sido fornecida (crie a pasta se ela não existir).

## Configurar seu SQL Server no Azure

Entre no Azure e verifique se você tem uma instância existente do SQL Server do Azure em execução no Azure. *Ignore esta seção se você já tiver uma instância do SQL Server em execução no Azure*.

1. Na máquina virtual do laboratório, ou em seu computador local, se não tiver sido fornecida, inicie uma sessão do Visual Studio Code e navegue até o repositório clonado da seção anterior.

1. Clique com o botão direito do mouse na pasta **/Allfiles/Labs** e selecione **Abrir no Terminal Integrado**.

1. Vamos nos conectar ao Azure usando a CLI do Azure. Digite o comando a seguir e selecione **Enter**:

    ```bash
    az login
    ```

    > &#128221; Observe que uma janela do navegador abrirá. Use suas credenciais do Azure AD para fazer logon.

1. Depois de entrar no Azure, é hora de criar um grupo de recursos, se ele ainda não existir, e criar um SQL Server e um banco de dados nesse grupo de recursos. Digite o comando a seguir e selecione **Enter**: *O script precisará de alguns minutos para concluir*.

    ```bash
    cd ./Setup
    ./deploy-sql-database.ps1
    ```

    > &#128221; Observe que, por padrão, esse script criará um grupo de recursos chamado **contoso-rg** ou usará um recurso cujo nome comece com *contoso-rg*, se existir. Por padrão, ele também criará todos os recursos na região **Oeste dos EUA 2** (westus2). Por fim, ele irá gerar uma senha aleatória de 12 caracteres para a **senha de administrador do SQL**. Você pode alterar esses valores usando um ou mais dos parâmetros **-rgName**, **-location** e **-sqlAdminPw** com seus próprios valores. A senha terá que atender aos requisitos de complexidade de senha do SQL do Azure, com pelo menos 12 caracteres e conter pelo menos 1 letra maiúscula, 1 letra minúscula, 1 número e 1 caractere especial.

    > &#128221; Observe que o script adicionará seu endereço IP público atual às regras de firewall do SQL Server.

1. Depois que o script for concluído, ele retornará o nome do grupo de recursos, o nome do SQL Server e o nome do banco de dados e o nome de usuário e senha do administrador. Anote esses valores, pois você precisará deles mais tarde no laboratório.

---

## Autorizar o acesso ao Banco de Dados SQL do Azure com o Microsoft Entra

Você pode criar logons de contas do Microsoft Entra como um usuário de banco de dados independente usando a sintaxe T-SQL `CREATE USER [anna@contoso.com] FROM EXTERNAL PROVIDER`. Um usuário do banco de dados independente é mapeado para uma identidade no diretório do Microsoft Entra associado ao banco de dados e não tem um logon no banco de dados `master`.

Com a introdução de logons de servidor Microsoft Entra no Banco de Dados SQL do Azure, você pode criar logons de entidades de segurança do Microsoft Entra no banco de dados virtual `master` de um Banco de Dados SQL. Você pode criar logons do Microsoft Entra de *usuários, grupos e entidades de serviço* do Microsoft Entra. Para obter informações, confira [Entidades do servidor Microsoft Entra](/azure/azure-sql/database/authentication-azure-ad-logins)

Além disso, você pode usar o portal do Azure apenas para criar administradores, e as funções de controle de acesso baseadas em função do Azure não se propagam para servidores lógicos do Banco de Dados SQL do Azure. Você precisa conceder permissões adicionais de servidor e do banco de dados usando T-SQL (Transact-SQL). Vamos criar um administrador do Microsoft Entra para o SQL Server.

1. Na máquina virtual do laboratório, ou em seu computador local, se não tiver sido fornecida, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando as suas credenciais do Azure.

1. Na home page do portal do Azure, pesquise e selecione **SQL Servers**.

1. Selecione o SQL Server **dp300-lab-xxxxxxxx**, em que *xxxxxxxx* é uma cadeia de caracteres numérica aleatória.

    > &#128221; Observe que, se você estiver usando seu próprio servidor SQL do Azure não criado por este laboratório, selecione o nome desse SQL Server.

1. Na folha *Visão geral*, selecione **Não configurado** ao lado de *Administrador do Microsoft Entra*.

1. Na próxima tela, selecione **Definir administrador**.

1. Na barra lateral do **Microsoft Entra ID**, procure o nome de usuário do Azure com o qual você fez logon no portal do Azure e clique em **Selecionar**.

1. Selecione **Salvar** para concluir o processo. Isso fará com que seu nome de usuário seja o administrador do Microsoft Entra do servidor.

1. À esquerda, selecione **Visão geral** e copie o **Nome do servidor**.

1. Abra o SQL Server Management Studio (SSMS) e selecione **Conectar** > **Mecanismo de Banco de Dados**. Na pasta **Nome do servidor**, digite o nome do servidor. Altere o tipo de autenticação para **MFA do Microsoft Entra**.

1. Selecione **Conectar**.

## Gerenciar o acesso a objetos do banco de dados

Nesta tarefa, você gerenciará o acesso ao banco de dados e a seus objetos. Primeiramente, crie dois usuários no banco de dados *AdventureWorksLT*.

1. Na máquina virtual do laboratório ou em seu computador local, se não tiver sido fornecida, no SSMS, entre no banco de dados *AdventureWorksLT* usando a conta de administrador do Servidor do Azure ou a conta de administrador do Microsoft Entra.

1. Use o **Pesquisador de Objetos** e expanda **Bancos de Dados**.

1. Clique com o botão direito do mouse em **AdventureWorksLT** e selecione **Nova Consulta**.

1. Na janela Nova consulta, copie e cole o T-SQL abaixo. Execute a consulta para criar os dois usuários.

    ```sql
    CREATE USER [DP300User1] WITH PASSWORD = 'Azur3Pa$$';
    GO

    CREATE USER [DP300User2] WITH PASSWORD = 'Azur3Pa$$';
    GO
    ```

    **Observação:** Observe que esses usuários são criados no escopo do banco de dados AdventureWorksLT. Em seguida, crie uma função personalizada e adicione os usuários a ela.

1. Execute o T-SQL a seguir na mesma janela de consulta.

    ```sql
    CREATE ROLE [SalesReader];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User1];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User2];
    GO
    ```

    Em seguida, crie um novo procedimento armazenado no esquema **SalesLT**.

1. Execute o T-SQL abaixo na janela Consulta.

    ```sql
    CREATE OR ALTER PROCEDURE SalesLT.DemoProc
    AS
    SELECT P.Name, Sum(SOD.LineTotal) as TotalSales ,SOH.OrderDate
    FROM SalesLT.Product P
    INNER JOIN SalesLT.SalesOrderDetail SOD on SOD.ProductID = P.ProductID
    INNER JOIN SalesLT.SalesOrderHeader SOH on SOH.SalesOrderID = SOD.SalesOrderID
    GROUP BY P.Name, SOH.OrderDate
    ORDER BY TotalSales DESC
    GO
    ```

    Em seguida, use a sintaxe `EXECUTE AS USER` para testar a segurança. Isso permite que o mecanismo de banco de dados execute uma consulta no contexto do usuário.

1. Execute o T-SQL a seguir.

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

    Ocorrerá uma falha com a mensagem:

    <span style="color:red">Msg 229, Level 14, State 5, Procedure SalesLT.DemoProc, Line 1 [Batch Start Line 0]  The EXECUTE permission was denied on the object 'DemoProc', database 'AdventureWorksLT', schema 'SalesLT'.</span>

1. Em seguida, conceda permissões à função para possibilitar que ela execute o procedimento armazenado. Execute o T-SQL abaixo.

    ```sql
    REVERT;
    GRANT EXECUTE ON SCHEMA::SalesLT TO [SalesReader];
    GO
    ```

    O primeiro comando reverte o contexto de execução para o proprietário do banco de dados.

1. Reexecute o T-SQL anterior.

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

---

## Recursos de limpeza

Se você não estiver usando o SQL Server do Azure para nenhuma outra finalidade, poderá limpar os recursos criados neste laboratório.

### Excluir o Grupo de Recursos

Se você criou um novo grupo de recursos para este laboratório, poderá excluir o grupo de recursos para remover todos os recursos criados neste laboratório.

1. No portal do Azure, selecione **Grupos de recursos** no painel de navegação esquerdo ou pesquise **Grupos de recursos** na barra de pesquisa e selecione-o nos resultados.

1. Vá para o grupo de recursos criado para o laboratório. O grupo de recursos conterá o SQL Server do Azure e outros recursos criados neste laboratório.

1. Escolha **Excluir grupo de recursos** no menu superior.

1. Na caixa de diálogo **Excluir grupo de recursos**, digite o nome do grupo para confirmar e clique em **Excluir**.

1. Aguarde até que o grupo de recursos seja excluído.

1. Feche o portal do Azure.

### Excluir apenas os recursos do laboratório

Se você não criou um novo grupo de recursos para este laboratório e deseja deixar o grupo de recursos e seus recursos anteriores intactos, ainda poderá excluir os recursos criados neste laboratório.

1. No portal do Azure, selecione **Grupos de recursos** no painel de navegação esquerdo ou pesquise **Grupos de recursos** na barra de pesquisa e selecione-o nos resultados.

1. Vá para o grupo de recursos criado para o laboratório. O grupo de recursos conterá o SQL Server do Azure e outros recursos criados neste laboratório.

1. Selecione todos os recursos prefixados com o nome do SQL Server especificado anteriormente no laboratório.

1. Selecione **Excluir** no menu superior.

1. Na caixa de diálogo **Excluir recursos**, digite **excluir** e selecione **Excluir**.

1. Para confirmar a exclusão dos recursos, clique em excluir novamente **Excluir**.

1. Aguarde a exclusão dos recursos.

1. Feche o portal do Azure.

### Exclua a pasta LabFiles

Se você criou uma nova pasta LabFiles para este laboratório e não precisa mais dela, pode excluir a pasta LabFiles para remover todos os arquivos criados neste laboratório.

1. Na máquina virtual do laboratório ou em seu computador local, se não tiver sido fornecida, abra o explorador de arquivos e navegue até a unidade **C:\\**.
1. Clique com o botão direito do mouse na pasta **LabFiles** e selecione **Excluir**.
1. Selecione **Sim** para confirmar a exclusão da pasta.

---

Você concluiu este laboratório.

Neste exercício, você viu como pode usar o Microsoft Entra ID para conceder acesso às credenciais do Azure a um SQL Server hospedado no Azure. Você também usou a instrução T-SQL para criar novos usuários de banco de dados e concedeu a eles permissões para executar procedimentos armazenados.
