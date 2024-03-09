---
lab:
  title: Laboratório 5 – Habilitar o Microsoft Defender para SQL e a Classificação de Dados
  module: Implement a Secure Environment for a Database Service
---

# Habilitar o Microsoft Defender para SQL e a Classificação de Dados

**Tempo estimado**: 20 minutos

Os alunos usarão as informações obtidas nas lições para configurar e, posteriormente, implementar a segurança no Portal do Azure e no banco de dados do AdventureWorks.

Você foi contratado como um Administrador de Banco de Dados Sênior para ajudar a garantir a segurança do ambiente de banco de dados. Essas tarefas se concentrarão no Banco de Dados SQL do Azure.

**Observação:** Esses exercícios solicitam que você copie e cole o código T-SQL e use os recursos existentes do SQL. Verifique se o código foi copiado corretamente antes de executá-lo.

## Habilitar o Microsoft Defender para SQL

1. Na máquina virtual do laboratório, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando o **Nome de Usuário** e a **Senha** do Azure fornecidos na guia **Recursos** dessa máquina virtual do laboratório.

    ![Figura 1](../images/dp-300-module-01-lab-01.png)

1. No Portal do Azure, pesquise por "servidores SQL" na caixa de pesquisa na parte superior e clique em **servidores SQL** na lista de opções.

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-1.png)

1. Selecione o nome do servidor **dp300-lab-XXXXXXXXX** a ser levado para a página de detalhes (você pode ter um grupo de recursos e um local diferentes atribuídos ao seu servidor SQL).

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-2.png)

1. No painel principal do servidor SQL do Azure, navegue até a seção **Segurança** e selecione **Microsoft Defender para Nuvem**.

    ![Captura de tela da opção Microsoft Defender para Nuvem selecionada](../images/dp-300-module-05-lab-01.png)

    Na página **Microsoft Defender para Nuvem**, selecione **Habilitar o Microsoft Defender para SQL**.

1. A mensagem de notificação a seguir será exibida depois que o Azure Defender para SQL for habilitado com sucesso.

    ![Captura de tela da seleção da opção Configurar](../images/dp-300-module-05-lab-02_1.png)

1. Na página **Microsoft Defender para Nuvem**, selecione o link **Configurar** (talvez seja necessário atualizar a página para ver essa opção)

    ![Captura de tela da seleção da opção Configurar](../images/dp-300-module-05-lab-02.png)

1. Na página **Configurações do servidor**, observe que a opção de alternância em **MICROSOFT DEFENDER PARA SQL** está definida como **ATIVADA**.

## Habilitar Classificação de Dados

1. Na folha principal do servidor SQL do Azure, navegue até a seção **Configurações** e selecione **Bancos de dados SQL** e selecione o nome do banco de dados.

    ![Captura de tela que mostra a seleção do banco de dados AdventureWOrksLT](../images/dp-300-module-05-lab-04.png)

1. Na folha principal do banco de dados **AdventureWorksLT**, navegue até a seção **Segurança** e selecione **Descoberta e Classificação de Dados**.

    ![Captura de tela que mostra a Descoberta e Classificação de Dados](../images/dp-300-module-05-lab-05.png)

1. Na página **Descoberta e Classificação de Dados**, você verá a seguinte mensagem informativa: **Atualmente usando a política de Proteção de Informações do SQL. Encontramos 15 colunas com recomendações de classificação**. Selecione esse link.

    ![Captura de tela que mostra as recomendações de classificação](../images/dp-300-module-05-lab-06.png)

1. Na tela **Descoberta e Classificação de Dados**, marque a caixa de seleção ao lado de **Selecionar tudo**, selecione **Recomendações selecionadas aceitas** e **Salvar** para salvar as classificações no banco de dados.

    ![Captura de tela que mostra o botão Aceitar recomendações selecionadas](../images/dp-300-module-05-lab-07.png)

1. De volta à tela **Descoberta e Classificação de Dados**, observe que quinze colunas foram classificadas com sucesso em cinco tabelas diferentes.

    ![Captura de tela que mostra o botão Aceitar recomendações selecionadas](../images/dp-300-module-05-lab-08.png)

Neste exercício, você aprimorou a segurança de um Banco de Dados SQL do Azure habilitando o Microsoft Defender para SQL. Você também criou colunas classificadas com base nas recomendações do portal do Azure.
