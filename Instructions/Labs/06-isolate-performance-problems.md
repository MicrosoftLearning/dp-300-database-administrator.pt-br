---
lab:
  title: 'Laboratório 6: isolar problemas de desempenho por meio do monitoramento'
  module: Monitor and optimize operational resources in Azure SQL
---

# Isolar problemas de desempenho por meio do monitoramento

**Tempo estimado**: 30 minutos

Os alunos levarão as informações obtidas nas aulas para definir o escopo das entregas de um projeto de transformação digital no AdventureWorksLT. Examinando o portal do Azure, bem como outras ferramentas, os alunos determinarão como utilizar ferramentas para identificar e resolver problemas relacionados ao desempenho.

Você foi contratado como administrador de banco de dados para identificar problemas relacionados ao desempenho e fornecer soluções viáveis para solucionar todos os problemas encontrados. Você precisa usar o portal do Azure para identificar os problemas de desempenho e sugerir métodos para resolvê-los.

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

1. Depois que o script for concluído, ele retornará o nome do grupo de recursos, o nome do SQL Server e o nome do banco de dados e o nome de usuário e senha do administrador. *Anote esses valores, pois você precisará deles mais tarde no laboratório*.

---

## Examinar a utilização da CPU no portal do Azure

1. Na máquina virtual do laboratório, ou em seu computador local, se não tiver sido fornecida, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando as suas credenciais do Azure.

1. No Portal do Azure, pesquise por *servidores SQL* na caixa de pesquisa na parte superior e clique em **servidores SQL** na lista de opções.

1. Selecione o SQL Server **dp300-lab-xxxxxxxx**, em que *xxxxxxxx* é uma cadeia de caracteres numérica aleatória.

    > &#128221; Observe que, se você estiver usando seu próprio servidor SQL do Azure não criado por este laboratório, selecione o nome desse SQL Server.

1. Na página principal do servidor SQL do Azure, em **Segurança**, selecione **Sistema de rede**.

1. Na página **Rede**, verifique se o seu IP público atual já foi adicionado à lista **Regras de firewall**. Se não foi, selecione **+ Adicionar seu endereço IPv4 do cliente (seu endereço IP)** para adicioná-lo e, em seguida, selecione **Salvar**.

1. Na folha principal do seu servidor SQL do Azure SQL, navegue até a seção **Configurações**, selecione **Bancos de Dados SQL** e, em seguida, selecione o banco de dados **AdventureWorksLT**.

1. Na barra de navegação esquerda, escolha **Editor de consultas (versão prévia)**.

    **Observação:** Esta funcionalidade está em versão prévia.

1. Selecione o nome de usuário administrador do SQL Server e insira a senha ou suas credenciais do Microsoft Entra, se atribuídas, para se conectar ao banco de dados.
    - **Nome do servidor:** &lt;_cole o nome do servidor do Banco de Dados SQL do Azure aqui_&gt;
    - **Autenticação:** Autenticação do SQL Server
    - **Logon de administrador do servidor:** seu logon de administrador do servidor do Banco de Dados SQL do Azure
    - **Senha:** sua senha de administrador do servidor do Banco de Dados SQL do Azure

1. Em **Consulta 1**, digite a consulta a seguir e escolha **Executar**:

    ```sql
    DECLARE @Counter INT 
    SET @Counter=1
    WHILE ( @Counter <= 10000)
    BEGIN
        SELECT 
             RTRIM(a.Firstname) + ' ' + RTRIM(a.LastName)
            , b.AddressLine1
            , b.AddressLine2
            , RTRIM(b.City) + ', ' + RTRIM(b.StateProvince) + '  ' + RTRIM(b.PostalCode)
            , CountryRegion
            FROM SalesLT.Customer a
            INNER JOIN SalesLT.CustomerAddress c 
                ON a.CustomerID = c.CustomerID
            RIGHT OUTER JOIN SalesLT.Address b
                ON b.AddressID = c.AddressID
        ORDER BY a.LastName ASC
        SET @Counter  = @Counter  + 1
    END
    ```

1. Aguarde a conclusão da consulta.

1. Execute novamente a consulta mais *duas* vezes para gerar carga de CPU no banco de dados.

1. Na folha do banco de dados **AdventureWorksLT**, selecione o ícone **Métricas**na seção **Monitoramento**.

    Se a mensagem *Suas alterações não salvas serão descartadas* for exibida, selecione **OK**.

1. Altere a opção de menu **Métrica** para refletir a **Porcentagem de CPU** e, em seguida, selecione uma **Agregação** de **média**. Isso mostrará a porcentagem média da CPU para o período escolhido.

1. Observe a média da CPU ao longo do tempo. Você observará um pico na utilização da CPU no final do gráfico quando a consulta estava em execução.

## Identificar consultas de alto consumo de CPU

1. Encontre o ícone de **Análise de Desempenho de Consultas** na seção **Desempenho Inteligente** da folha do banco de dados **AdventureWorksLT**.

1. Escolha **Redefinir configurações**.

1. Selecione a consulta na grade abaixo do gráfico. Se você não vir a consulta que executamos anteriormente várias vezes, aguarde de dois a cinco minutos e selecione **Atualizar**.

    > &#128221; Se houver mais de uma consulta listada, selecione cada uma para observar os resultados. Observe a grande quantidade de informações disponíveis para cada consulta.

1. Na consulta que você executou anteriormente, observe que a duração total foi superior a um minuto e que ela foi executada cerca de trinta mil vezes.

1. Revisando o texto SQL na página **Detalhes da consulta** em relação à consulta que você executou, você observará que os **Detalhes da consulta** incluem apenas a instrução **SELECT** e não o loop **WHILE** ou outra instrução. Isso acontece porque a **Análise de Desempenho de Consultas** depende de dados do **Repositório de Consultas**, que rastreia apenas instruções de Linguagem de Manipulação de Dados (DML), como **SELECT, INSERT, UPDATE, DELETE, MERGE** e **BULK INSERT**, ignorando as instruções de Linguagem de Definição de Dados (DDL).

Nem todos os problemas de desempenho estão relacionados a uma alta utilização da CPU por uma única execução de consulta. Nesse caso, a consulta foi executada milhares de vezes, o que também pode resultar em alta utilização da CPU.

---

## Recursos de limpeza

Se você não estiver usando o SQL Server do Azure para nenhuma outra finalidade, poderá limpar os recursos criados neste laboratório.

### Excluir o Grupo de Recursos

Se você criou um novo grupo de recursos para este laboratório, poderá excluir o grupo de recursos para remover todos os recursos criados neste laboratório.

1. No portal do Azure, selecione **Grupos de recursos** no painel de navegação esquerdo ou pesquise **Grupos de recursos** na barra de pesquisa e selecione-o nos resultados.

1. Vá para o grupo de recursos criado para o laboratório. O grupo de recursos conterá o SQL Server do Azure e outros recursos criados neste laboratório.

1. Selecione **Excluir grupo de recursos** no menu superior.

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

Neste exercício, você aprendeu a explorar os recursos do servidor para um Banco de Dados SQL do Azure e identificar possíveis problemas de desempenho de consulta por meio da Análise de Desempenho de Consultas.
