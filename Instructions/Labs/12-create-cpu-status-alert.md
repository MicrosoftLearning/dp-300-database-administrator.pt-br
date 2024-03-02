---
lab:
  title: 'Laboratório 12: criar um alerta de status de CPU para um SQL Server'
  module: Automate database tasks for Azure SQL
---

# Criar um alerta de status de CPU para um SQL Server no Azure

**Tempo estimado**: 30 minutos

Você foi contratado como Engenheiro de Dados Sênior para ajudar a automatizar operações diárias de administração do banco de dados. Essa automação foi criada para ajudar a garantir que os bancos de dados do AdventureWorks continuem operando com desempenho máximo, bem como fornecendo métodos para gerar alertas com base em determinados critérios.

**Observação:** Esses exercícios podem solicitar que você copie e cole código T-SQL e use recursos SQL existentes. Verifique se o código foi copiado corretamente antes de executá-lo.

## Criar um alerta quando uma CPU exceder a média de 80 por cento

1. Na barra de pesquisa na parte superior do portal do Azure, digite **SQL** e clique em **Banco de dados SQL**. Selecione o nome do banco de dados **AdventureWorksLT** listado.

    ![Captura de tela da seleção de um banco de dados SQL](../images/dp-300-module-12-lab-01.png)

1. No painel principal do banco de dados **AdventureWorksLT**, navegue até a seção de monitoramento. Selecione **Alertas**.

    ![Captura de tela da seleção Alertas na página Visão geral do banco de dados SQL](../images/dp-300-module-12-lab-02.png)

1. Selecione **Criar regra de alerta**.

    ![Captura de tela da seleção Nova regra de alerta](../images/dp-300-module-12-lab-03.png)

1. No slide **Selecionar um sinal**, selecione a **Porcentagem da CPU**.

    ![Captura de tela de seleção de porcentagem da CPU](../images/dp-300-module-12-lab-04.png)

1. No slide **Configurar sinal**, selecione **Estático** para a propriedade **Limite**. Em seguida, verifique se a propriedade **Operador** é **Maior que** e o tipo de **Agregação** é **Média**. Em seguida, em **Valor limite**, insira um valor de **80**. Selecione **Concluído**.

    ![Captura de tela da inserção de 80 e a seleção de Concluído](../images/dp-300-module-12-lab-05.png)

1. Selecione a guia **Ações**.

    ![Captura de tela da seleção do link Selecionar grupo de ações](../images/dp-300-module-12-lab-06.png)

1. Na guia **Ações**, selecione **Criar grupo de ações**.

    ![Captura de tela da seleção Criar grupo de ações](../images/dp-300-module-12-lab-07.png)

1. Na tela **Grupo de ações**, digite **grupo_de_email** no campo **Nome do grupo de ações** e selecione **Avançar: Notificações**.

    ![Captura de tela da inserção de um grupo de email e seleção de Avançar: Notificações](../images/dp-300-module-12-lab-08.png)

1. Na guia **Notificações**, insira as seguintes informações:

    - **Tipo de notificação:** Email/Mensagem SMS/Push/Voz
        - **Observação:** Ao selecionar essa opção, um submenu com as opções Email/SMS/Push/Voz será exibido. Verifique a propriedade Email e digite o nome de usuário do Azure com o qual você entrou.
    - **Nome:** DemoLab

    ![Captura de tela da página Criar grupo de ações com as informações adicionadas](../images/dp-300-module-12-lab-09.png)

1. Selecione **Examinar + criar**e **Criar**.

    ![Captura de tela da página Criar regra de alerta selecionando Criar regra de alerta](../images/dp-300-module-12-lab-10.png)

    **Observação:** Antes de selecionar **Criar**, você também pode selecionar **Testar grupo de ações (versão prévia)** para testar o Alerta.

1. Um email como esse é enviado para o endereço de email que você inseriu, assim que a regra é criada.

    ![Captura de tela do email de confirmação](../images/dp-300-module-12-lab-11.png)

    Com o alerta em vigor, se o uso da CPU exceder a média de 80%, um email como este será enviado.

    ![Captura de tela do email de aviso](../images/dp-300-module-12-lab-12.png)

Os alertas podem enviar um email ou chamar um webhook quando alguma métrica (por exemplo, tamanho do banco de dados ou uso da CPU) atingir um limite definido. Você acabou de ver como é possível configurar alertas facilmente para bancos de dados SQL do Azure.
