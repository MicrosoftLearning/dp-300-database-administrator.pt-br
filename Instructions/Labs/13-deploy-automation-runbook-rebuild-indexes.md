---
lab:
  title: 'Laboratório: implantar um runbook de automação para recompilar índices de modo automático'
  module: Automate database tasks for Azure SQL
---

# Laboratório: implantar um runbook de automação para recompilar índices de modo automático

**Tempo estimado**: 20 minutos

Você foi contratado como Engenheiro de Dados Sênior para ajudar a automatizar operações diárias de administração do banco de dados. Essa automação foi criada para ajudar a garantir que os bancos de dados do AdventureWorks continuem operando com desempenho máximo, bem como fornecendo métodos para gerar alertas com base em determinados critérios. O AdventureWorks usa o SQL Server em ofertas de Infraestrutura como Serviço e Plataforma como Serviço.

## Criar uma Conta de Automação

1. Na máquina virtual do laboratório, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando o Nome** de Usuário e **a Senha** do Azure **fornecidos na **guia Recursos** para esta máquina virtual de laboratório.

    ![Captura de tela de página de entrada do Azure](../images/dp-300-module-01-lab-01.png)

1. No portal do Azure, digite Automação na barra de pesquisa e selecione Contas de Automação nos resultados da pesquisa.

    ![Uma captura de tela de como selecionar Contas de Automação.](../images/dp-300-module-13-lab-01.png)

1. Na página Criar uma Conta** de **Automação, insira as informações abaixo e selecione **Revisar + Criar**.

    - **Grupo de recursos: *** YOUR-RESOURCE-GROUP*
    - Nome: autoAccount
    - Local: use o padrão.

    ![Uma captura de tela da opção Adicionar Conta de Automação.](../images/dp-300-module-13-lab-02.png)

1. Na página Examinar, selecione **Criar**.

    ![Uma captura de tela da opção Adicionar Conta de Automação.](../images/dp-300-module-13-lab-29.png)

    > [!NOTE]
    > Sua conta de automação deverá ser criada em até três minutos.

## Conectar-se a um Banco de Dados SQL do Azure existente

1. No portal do Azure, navegue de volta para o banco de dados pesquisando por **bancos de dados SQL**.

    ![Uma captura de tela de como pesquisar por bancos de dados SQL existentes.](../images/dp-300-module-13-lab-03.png)

1. Selecione o banco de dados **SQL AdventureWorksLT.**

    ![Captura de tela mostrando a seleção do banco de dados AdventureWorks.](../images/dp-300-module-13-lab-04.png)

1. Na página para o seu banco de dados, selecione **Editor de consultas (versão prévia)** no menu à esquerda.

    ![Uma captura de tela de como selecionar o Editor de consultas (versão prévia).](../images/dp-300-module-13-lab-05.png)

1. Você será solicitado a fornecer credenciais para entrar no banco de dados. Usar a credencial 

    - **Login:** sqladmin
    - **Senha**: P@ssw0rd01

1. Você deve receber a seguinte mensagem de erro:

    ![Uma captura de tela da mensagem Erro ao entrar.](../images/dp-300-module-13-lab-06.png)

1. Selecione o **link Allowlist IP ...** fornecido no final da mensagem de erro mostrada acima. Isso adicionará automaticamente o IP do cliente como uma entrada de regra de firewall para o Banco de dados SQL.

    ![Captura de tela da criação da regra de firewall.](../images/dp-300-module-13-lab-07.png)

1. Retorne ao Editor de consultas, depois clique em **OK** para entrar no banco de dados.

1. Abra uma guia no navegador e acesse a página do GitHub para executar [**AdaptativeIndexDefragmentation**](https://github.com/microsoft/tigertoolbox/blob/master/AdaptiveIndexDefrag/usp_AdaptiveIndexDefrag.sql). Em seguida, selecione **Adicionar**.

    ![Uma captura de tela de como selecionar a opção Bruto no GitHub.](../images/dp-300-module-13-lab-08.png)

    Isso fornecerá o código em um formato em que será possível copiá-lo. Selecione todo o texto (<kbd>CTRL</kbd> + <kbd>A</kbd>) e copie-o para a área de transferência (<kbd>CTRL</kbd> + <kbd>C</kbd>).

    >[!NOTE]
    > AdaptiveIndexDefrag é um procedimento cujo objetivo é executar uma desfragmentação inteligente em um ou mais índices, bem como a atualização das estatísticas necessárias, para um ou mais bancos de dados.

1. Feche a guia do navegador do GitHub e retorne ao portal do Azure.

1. Cole o texto copiado no painel da Consulta 1.

    ![Captura de tela mostrando a colagem do código em uma nova janela de consulta.](../images/dp-300-module-13-lab-09.png)

1. Exclua `USE msdb` e `GO` nas linhas 5 e 6 da consulta (que estão realçadas na captura de tela), depois clique em **Executar**.

1. Expanda a pasta **Procedimentos Armazenados** para conferir o que foi criado.

    ![Uma captura de tela dos novos procedimentos armazenados.](../images/dp-300-module-13-lab-10.png)

## Configurar ativos da Conta de Automação

As próximas etapas consistem em configurar os ativos necessários na preparação para a criação do runbook. Depois selecione **Contas de Automação**.

1. No portal do Azure, digite **automação** na caixa de pesquisa da parte superior da página.

    ![Uma captura de tela de como selecionar Contas de Automação.](../images/dp-300-module-13-lab-11.png)

1. Selecione a conta de automação criada.

    ![Uma captura de tela de como selecionar a conta de automação autoAccount.](../images/dp-300-module-13-lab-12.png)

1. Selecione Módulos na seção de Recursos Compartilhados da folha de Automação. Selecione **Procurar na Galeria**.

    ![Uma captura de tela de como selecionar o menu Módulos.](../images/dp-300-module-13-lab-13.png)

1. Pesquise o módulo **sqlserver** na Galeria.

    ![Uma captura de tela de como selecionar o módulo SqlServer.](../images/dp-300-module-13-lab-14.png)

1. Clique em SqlServer e você será direcionado para a próxima tela.

    ![Captura de tela da seleção de Fechar.](../images/dp-300-module-13-lab-15.png)

1. **Na página Adicionar um módulo**, selecione a versão de tempo de execução mais recente disponível e, em seguida, selecione **Importar**. Isso importará o módulo do PowerShell para sua Conta de Automação.

    ![Uma captura de tela de como clicar na opção Importar.](../images/dp-300-module-13-lab-16.png)

1. Será preciso criar uma credencial para entrar com segurança no banco de dados. Na folha Conta de Automação, navegue até a seção **Recursos Compartilhados** e selecione **Credenciais**.

    ![Captura de tela da opção Credenciais selecionadas.](../images/dp-300-module-13-lab-17.png)

1. Selecione **+ Adicionar uma credencial**, insira as informações abaixo e selecione **Criar**.

    - Nome: **SQLUser**
    - Nome de usuário: sqladmin
    - Senha: **P@ssw0rd01**
    - Confirmar senha:

    ![Uma captura de tela de como adicionar credenciais da conta.](../images/dp-300-module-13-lab-18.png)

## Criar runbook do PowerShell

1. No portal do Azure, navegue de volta para o banco de dados pesquisando por **bancos de dados SQL**.

    ![Uma captura de tela de como pesquisar por bancos de dados SQL existentes.](../images/dp-300-module-13-lab-03.png)

1. Selecione o banco de dados **SQL AdventureWorksLT.**

    ![Captura de tela mostrando a seleção do banco de dados AdventureWorks.](../images/dp-300-module-13-lab-04.png)

1. Na página Visão Geral, copie o Nome do servidor do Banco de Dados SQL do Azure, conforme mostrado abaixo. Você colará essa informação nas próximas etapas.

    ![Uma captura de tela de como copiar o nome do servidor.](../images/dp-300-module-13-lab-19.png)

1. No portal do Azure, digite **automação** na caixa de pesquisa da parte superior da página.

    ![Uma captura de tela de como selecionar Contas de Automação.](../images/dp-300-module-13-lab-11.png)

1. Selecione a conta de automação criada.

    ![Uma captura de tela de como selecionar a conta de automação autoAccount.](../images/dp-300-module-13-lab-12.png)

1. Role até a seção de Automação do Processo da folha Conta de Automação e selecione Runbooks, depois clique em +Criar um runbook.

    ![Uma captura de tela da página de Runbooks e como selecionar a opção Criar um runbook.](../images/dp-300-module-13-lab-20.png)

    >[!NOTE]
    > Como aprendemos, observe que existem dois runbooks existentes. Eles foram criados automaticamente durante a implantação da conta de automação.

1. Digite o nome do runbook como **IndexMaintenance** e um tipo de runbook do **PowerShell**. Selecione a versão de tempo de execução mais recente disponível e, em seguida, selecione **Criar**.

    ![Uma captura de tela de como criar um runbook.](../images/dp-300-module-13-lab-21.png)

1. Depois que o runbook tiver sido criado, copie e cole o trecho de código do Powershell abaixo em seu editor de runbook. Na linha 1 do arquivo, cole o Nome do servidor copiado nas etapas acima. Selecione **Salvar** e depois **Publicar**.

    **Nota:** Verifique se o código foi copiado corretamente, antes de salvar o runbook.

    ```powershell
    $AzureSQLServerName = ''
    $DatabaseName = 'AdventureWorksLT'
    
    $Cred = Get-AutomationPSCredential -Name "SQLUser"
    $SQLOutput = $(Invoke-Sqlcmd -ServerInstance $AzureSQLServerName -UserName $Cred.UserName -Password $Cred.GetNetworkCredential().Password -Database $DatabaseName -Query "EXEC dbo.usp_AdaptiveIndexDefrag" -Verbose) 4>&1

    Write-Output $SQLOutput
    ```

    ![Captura de tela de colar código recortado.](../images/dp-300-module-13-lab-22.png)

1. Se tudo correr bem, você deve receber uma mensagem de sucesso.

    ![Captura de tela de uma mensagem bem-sucedida para a criação do runbook.](../images/dp-300-module-13-lab-23.png)

## Criar um agendamento para o runbook

Em seguida, você criará um agendamento para o runbook ser executado de modo regular.

1. Em Recursos, selecione Agendamentos na barra de navegação esquerda. Depois clique em **+Adicionar um agendamento**.

    ![Uma captura de tela da página de Agendamentos e como clicar em Adicionar um agendamento.](../images/dp-300-module-13-lab-24.png)

1. Selecione **Vincular um agendamento ao runbook**.

    ![Uma captura de tela de como clicar em Vincular um agendamento ao runbook.](../images/dp-300-module-13-lab-25.png)

1. Selecione **Adicionar um agendamento**.

    ![Uma captura de tela de como criar um link de agendamento.](../images/dp-300-module-13-lab-26.png)

1. Forneça um nome descritivo para o agendamento e uma descrição, se desejado.

1. Especifique o horário de início às **04:00** do dia seguinte e no **Fuso Horário do Leste**. Configure a recorrência para executar a cada **1** dia. Não defina uma data de término.

    ![Uma captura de tela do pop-up Novo Agendamento concluído com informações de exemplo.](../images/dp-300-module-13-lab-27.png)

1. Clique em **Criar**, depois em **OK**.

1. O agendamento agora está vinculado ao runbook. Selecione **OK**.

    ![Uma captura de tela do agendamento criado.](../images/dp-300-module-13-lab-28.png)

A Automação do Azure fornece um serviço de configuração e automação baseado em nuvem que dá suporte ao gerenciamento consistente em seus ambientes, tanto do Azure quanto não Azure.

Ao concluir este exercício, você automatizou a desfragmentação de índices em um banco de dados do SQL Server, bem como agendou para todos os dias, às 4h.