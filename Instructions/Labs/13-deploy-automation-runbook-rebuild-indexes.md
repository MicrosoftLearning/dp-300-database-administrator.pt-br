---
lab:
  title: 'Laboratório 13: implantar um runbook de automação para recompilar índices de modo automático'
  module: Automate database tasks for Azure SQL
---

# Implantar um runbook de automação para recompilar índices de modo automático

**Tempo estimado**: 30 minutos

Você foi contratado como Administrador de Banco de Dados Sênior para ajudar a automatizar operações rotineiras de administração do banco de dados. Essa automação foi criada para ajudar a garantir que os bancos de dados do AdventureWorks continuem operando com desempenho máximo, bem como fornecendo métodos para gerar alertas com base em determinados critérios. O AdventureWorks usa o SQL Server em ofertas de Infraestrutura como Serviço (IaaS) e Plataforma como Serviço (PaaS).

> &#128221; Esses exercícios podem solicitar que você copie e cole código T-SQL e use recursos SQL existentes. Verifique se o código foi copiado corretamente antes de executá-lo.

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

## Criar uma Conta de Automação

1. Na máquina virtual do laboratório, ou em seu computador local, se não tiver sido fornecida, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando as suas credenciais do Azure.

1. No portal do Azure, digite *automação* na barra de pesquisa, escolha **Contas de Automação** nos resultados da pesquisa e, em seguida, selecione **+Criar**.

1. Na página **Criar uma Conta de Automação**, insira as informações abaixo e selecione **Examinar + Criar**.

    - **Grupo de recursos:** &lt;Seu grupo de recursos&gt;
    - **Nome da conta de automação:** autoAccount
    - **Região:** use a padrão.

1. Na página Examinar, selecione **Criar**.

    > &#128221; Sua conta de automação pode levar alguns minutos para ser criada.

## Conectar-se a um Banco de Dados SQL do Azure existente

1. No portal do Azure, navegue até o banco de dados pesquisando por **bancos de dados SQL**.

1. Selecione o banco de dados SQL **AdventureWorksLT**.

1. Na seção principal da página do Banco de Dados SQL, selecione **Editor de Consultas (versão prévia)**.

1. Será solicitado que você forneça fornecer credenciais para entrar em seu banco de dados usando a conta de administrador do banco de dados e clique em **OK.**

    Uma nova guia será aberta no navegador. Clique em **Adicionar IP do cliente**, depois em **Salvar**. Depois de salvo, retorne à guia anterior e selecione **OK** novamente.

    > &#128221; Você pode receber a mensagem de erro *Não é possível abrir o servidor "your-sql-server-name" solicitado pelo logon. O cliente de endereço IP "xxx.xxx.xxx.xxx" não tem permissão para acessar o servidor.* Nesse caso, você precisará adicionar seu endereço IP público atual às regras de firewall do SQL Server.

    Se você precisar configurar as regras de firewall, siga estas etapas:

    1. selecione **Definir firewall do servidor** na barra de menus superior da página **Visão geral** do banco de dados.
    1. Selecione **Adicionar endereço IPv4 atual (xxx.xxx.xxx)** e **Salvar**.
    1. Depois de salvo, retorne à página do banco de dados **AdventureWorksLT** e selecione **Editor de Consultas (versão prévia)** novamente.
    1. Será solicitado que você forneça fornecer credenciais para entrar em seu banco de dados usando a conta de administrador do banco de dados e clique em **OK.**

1. No **Editor de consultas (versão prévia)**, selecione **Abrir consulta**.

1. Selecione o ícone da *pasta* de navegação e navegue até a pasta **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Module13**. Selecione o arquivo **usp_AdaptiveIndexDefrag.sql**, **Abrir** e, em seguida, **OK.**

1. Exclua **USE msdb** e **GO** nas linhas 5 e 6 da consulta e selecione **Executar**.

1. Expanda a pasta **Procedimentos Armazenados** para ver os procedimentos armazenados recém-criados.

## Configurar ativos da Conta de Automação

As próximas etapas consistem em configurar os ativos necessários na preparação para a criação do runbook. Depois selecione **Contas de Automação**.

1. No portal do Azure, na caixa de pesquisa superior, digite **automação** e selecione **Contas de automação**.

1. Selecione a conta de automação **autoAccount** que você criou.

1. Selecione **Módulos** na seção de **Recursos Compartilhados** da folha de Automação. Em seguida, selecione **Procurar na galeria**.

1. Pesquise **SqlServer** na Galeria.

1. Selecione **SqlServer**, o que abrirá a próxima tela, e clique no botão **Selecionar**.

1. Na página **Adicionar um módulo**, escolha a versão de runtime mais recente disponível e selecione **Importar**. Isso importará o módulo do PowerShell para sua Conta de Automação.

1. Será preciso criar uma credencial para entrar com segurança no banco de dados. Na folha da *Conta de automação*, navegue até a seção **Recursos compartilhados** e selecione **Credenciais**.

1. Selecione **+ Adicionar uma Credencial**, insira as informações abaixo e selecione **Criar**.

    - Nome: **SQLUser**
    - Nome de usuário: **sqladmin**
    - Senha: &lt;Digite uma senha forte, de 12 caracteres e que contenha pelo menos 1 letra maiúscula, 1 letra minúscula, 1 número e 1 caractere especial.&gt;
    - Confirmar senha: &lt;digite novamente a senha que você digitou anteriormente.&gt;

## Criar runbook do PowerShell

1. No portal do Azure, navegue até o banco de dados pesquisando por **bancos de dados SQL**.

1. Selecione o banco de dados SQL **AdventureWorksLT**.

1. Na página **Visão Geral**, copie o **Nome do servidor** do Banco de Dados SQL do Azure (o nome do seu servidor começará com *dp300-lab*). Você colará essa informação nas próximas etapas.

1. No portal do Azure, na caixa de pesquisa superior, digite **automação** e selecione **Contas de automação**.

1. Selecione a conta de automação **autoAccount**.

1. Expanda a seção **Automação do Processo** da folha Conta de Automação e selecione **Runbooks**.

1. Selecione **+ Criar um runbook**.

    > &#128221; Como aprendemos, observe que existem dois runbooks já criados. Eles foram criados automaticamente durante a implantação da conta de automação.

1. Digite o nome do runbook como **IndexMaintenance** e um tipo de runbook do **PowerShell**. Escolha a versão de runtime mais recente disponível e, em seguida, selecione **Revisar + Criar**.

1. Na página **Criar Runbook**, selecione **Criar**.

1. Assim que o runbook for criado, copie e cole o snippet de código do PowerShell abaixo no seu editor de runbook. 

    > &#128221; Verifique se o código foi copiado corretamente antes de salvar o runbook.

    ```powershell
    $AzureSQLServerName = ''
    $DatabaseName = 'AdventureWorksLT'
    
    $Cred = Get-AutomationPSCredential -Name "SQLUser"
    $SQLOutput = $(Invoke-Sqlcmd -ServerInstance $AzureSQLServerName -UserName $Cred.UserName -Password $Cred.GetNetworkCredential().Password -Database $DatabaseName -Query "EXEC dbo.usp_AdaptiveIndexDefrag" -Verbose) 4>&1

    Write-Output $SQLOutput
    ```

    > &#128221; Observe que o código acima é um script do PowerShell que executará o procedimento armazenado **usp_AdaptiveIndexDefrag** no banco de dados **AdventureWorksLT**. O script usa o cmdlet **Invoke-Sqlcmd** para se conectar ao SQL Server e executar o procedimento armazenado. O cmdlet **Get-AutomationPSCredential** é usado para recuperar as credenciais armazenadas na Conta de automação.

1. Na primeira linha do script, cole o nome do servidor copiado nas etapas acima.

1. Selecione **Salvar** e depois **Publicar**.

1. Selecione **Sim** para confirmar a ação de publicar.

1. O runbook *IndexMaintenance* foi publicado.

## Criar um agendamento para o runbook

Em seguida, você criará um agendamento para o runbook ser executado de modo regular.

1. Em **Recursos** na navegação à esquerda do runbook **IndexMaintenance**, selecione **Agendamentos**. 

1. Selecione **+ Adicionar um agendamento**.

1. Selecione **Vincular um agendamento ao runbook**.

1. Selecione **+ Adicionar um agendamento**.

1. Insira as informações abaixo e selecione **Criar**.

    - **Nome:** DailyIndexDefrag
    - **Descrição:** desfragmentação de índice diário do banco de dados AdventureWorksLT.
    - **Início:** 4h (dia seguinte)
    - **Fuso horário:**&lt;selecione o fuso horário que corresponde à sua localização&gt;
    - **Recorrência:** Recorrente
    - **Recorrer a cada:** 1 dia
    - **Definir a validade:** Não

    > &#128221; Observe que a hora de início é definida como 4h do dia seguinte. O fuso horário foi definido para o seu fuso horário. A Recorrência foi definida para a cada uma hora. Nunca expira.

1. Clique em **Criar**, depois em **OK**.

1. O agendamento agora está criado e vinculado ao runbook. Selecione **OK**.

A Automação do Azure oferece um serviço de configuração e automação baseado em nuvem que dá suporte ao gerenciamento consistente em seus ambientes Azure e não Azure.

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

Ao concluir este exercício, você automatizou a desfragmentação de índices em um banco de dados do SQL Server para ser executada todos os dias às 4h da manhã.
