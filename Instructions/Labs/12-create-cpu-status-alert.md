---
lab:
  title: 'Laboratório: criar um alerta de status de CPU para um SQL Server'
  module: Automate database tasks for Azure SQL
---

# Laboratório: criar um alerta de status de CPU para um SQL Server

**Tempo estimado**: 20 minutos

Você foi contratado como Engenheiro de Dados Sênior para ajudar a automatizar operações diárias de administração do banco de dados. Essa automação foi criada para ajudar a garantir que os bancos de dados do AdventureWorks continuem operando com desempenho máximo, bem como fornecendo métodos para gerar alertas com base em determinados critérios.

## Criar um alerta quando uma CPU exceder a média de 80 por cento

1. Na barra de pesquisa na parte superior do portal do Azure, digite **SQL** e clique em **Banco de dados SQL**. Selecione o nome do **banco de dados AdventureWorksLT** listado.

    ![Captura de tela da seleção de um banco de dados SQL.](../images/dp-300-module-12-lab-01.png)

1. No painel principal do banco de dados exemplo-db-with-tde, navegue até a seção monitoramento. Selecione **Alertas**.

    ![Captura de tela da seleção Alertas na página Visão geral do banco de dados SQL.](../images/dp-300-module-12-lab-02.png)

1. Selecione **Criar regra de alerta**.

    ![Captura de tela da seleção Nova regra de alerta.](../images/dp-300-module-12-lab-03.png)

1. Em Selecionar um deslizamento de sinal **, selecione **Porcentagem** da **CPU.

    ![Captura de tela de seleção da porcentagem de CPU.](../images/dp-300-module-12-lab-04.png)

1. No slide **Configurar lógica de sinal**, selecione **Estático** para a propriedade **Limite**. Verifique se o operador é Maior que, o tipo de agregação é Média. Em seguida, em **Valor limite**, insira um valor de **80**. Selecione **Concluído**.

    ![Captura de tela da inserção de 80 e a seleção de Concluído.](../images/dp-300-module-12-lab-05.png)

1. Selecione a guia **Ações**.

    ![Captura de tela da seleção do link Selecionar grupo de ações.](../images/dp-300-module-12-lab-06.png)

1. Na guia **Ações**, selecione **Criar um grupo de ações**.

    ![Captura de tela da seleção Criar grupo de ações.](../images/dp-300-module-12-lab-07.png)

1. Na tela grupo de ações, digite grupo de email no campo Nome do grupo de ações.

    ![Captura de tela da inserção de um grupo de email e seleção de Avançar: notificações.](../images/dp-300-module-12-lab-08.png)

1. Na guia **Pagamento à vista**, insira as seguintes informações:

    - Tipo de notificação: mensagem de email/SMS/Push/voz
        - **Nota:** Quando você seleciona essa opção, um submenu E-mail/Mensagem SMS/Push/Voz será exibido. Verifique a propriedade Email e digite o nome de usuário do Azure com o qual você entrou.
    - Nome: DemoLab

    ![Captura de tela da página Criar grupo de ações com as informações adicionadas.](../images/dp-300-module-12-lab-09.png)

1. Selecione **Examinar + criar**e **Criar**.

    ![Captura de tela da página Criar regra de alerta selecionando Criar regra de alerta.](../images/dp-300-module-12-lab-10.png)

    **Nota:** Antes de selecionar Criar **, você também pode selecionar ****Grupo de ações de teste (visualização)** para testar o Alerta.

1. Um email como esse é enviado para o endereço de email que você inseriu, assim que a regra é criada.

    ![Captura de tela do email de confirmação.](../images/dp-300-module-12-lab-11.png)

    Com o alerta em vigor, se o uso da CPU exceder a média de 80%, um email como este será enviado.

    ![Captura de tela do email de aviso.](../images/dp-300-module-12-lab-12.png)

Os alertas podem enviar a você um email ou chamar um webhook quando alguma métrica (por exemplo, tamanho do banco de dados ou uso da CPU) atinge o limite. Você acabou de ver como pode configurar facilmente alertas para os Bancos de Dados SQL do Azure.
