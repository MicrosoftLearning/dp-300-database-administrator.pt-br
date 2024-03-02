---
lab:
  title: 'Laboratório 14: configurar a replicação geográfica para o Banco de Dados SQL do Azure'
  module: Plan and implement a high availability and disaster recovery solution
---

# Configurar a replicação geográfica para o Banco de Dados SQL do Azure

**Tempo estimado**: 30 minutos

Como um administrador de banco de dados na AdventureWorks, você precisa habilitar a replicação geográfica para o Banco de Dados SQL do Azure e garantir que esteja funcionando corretamente. Além disso, você fará failover manualmente para outra região usando o portal.

**Observação:** Esses exercícios podem solicitar que você copie e cole código T-SQL e use recursos SQL existentes. Verifique se o código foi copiado corretamente antes de executá-lo.

## Habilitar a replicação geográfica

1. Na máquina virtual do laboratório, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando o **Nome de usuário** e a **Senha** do Azure fornecidos na guia **Recursos** desta máquina virtual de laboratório.

    ![Captura de tela da página de entrada do portal do Azure](../images/dp-300-module-01-lab-01.png)

1. No portal do Azure, navegue até o banco de dados pesquisando por **bancos de dados SQL**.

    ![Uma captura de tela de como pesquisar por bancos de dados SQL existentes.](../images/dp-300-module-13-lab-03.png)

1. Selecione o banco de dados SQL **AdventureWorksLT**.

    ![Captura de tela da seleção do banco de dados SQL AdventureWorks.](../images/dp-300-module-13-lab-04.png)

1. Na folha do banco de dados, na seção **Gerenciamento de dados**, selecione **Réplicas**.

    ![Captura de tela mostrando a seleção da replicação geográfica.](../images/dp-300-module-14-lab-01.png)

1. Selecione **+ Criar réplica**.

    ![Captura de tela mostrando a página de replicação geográfica.](../images/dp-300-module-14-lab-02.png)

1. Na página **Criar Banco de Dados SQL – Réplica geográfica** e, em **Servidor**, selecione o link **Criar**.

    ![Captura de tela mostrando o link para Criar Novo servidor.](../images/dp-300-module-14-lab-03.png)

    >[!NOTE]
    > Como estamos criando um novo servidor para hospedar nosso banco de dados secundário, podemos ignorar a mensagem de erro acima.

1. Na página **Criar Servidor de Banco de Dados SQL**, insira um **nome de servidor** exclusivo de sua preferência, um **logon de administrador de servidor** válido e uma **senha** segura. Selecione um **local** como a região de destino e selecione **OK** para criar o servidor.

    ![Captura de tela mostrando a página Criar Servidor de Banco de Dados SQL.](../images/dp-300-module-14-lab-04.png)

1. De volta à página **Criar Banco de Dados SQL – Réplica Geográfica**, selecione **Examinar + Criar**.

    ![Captura de tela mostrando a página Criar Servidor de Banco de Dados SQL.](../images/dp-300-module-14-lab-05.png)

1. Selecione **Criar**.

    ![Captura de tela mostrando a página para examinar e criar.](../images/dp-300-module-14-lab-06.png)

1. O servidor secundário e o banco de dados serão criados agora. Para verificar o status, examine o ícone de notificação na parte superior do portal. 

    ![Captura de tela mostrando a página para examinar e criar.](../images/dp-300-module-14-lab-07.png)

1. Se a criação tiver ocorrido com sucesso, ele mudará de **Implantação em andamento** para **Implantação bem-sucedida**.

    ![Captura de tela mostrando a página para examinar e criar.](../images/dp-300-module-14-lab-08.png)

## Fazer failover do Banco de Dados SQL para uma região secundária

Agora que a réplica do Banco de Dados SQL do Azure foi criada, você fará um failover.

1. Navegue até a página de servidores SQL e observe o novo servidor na lista. Selecione o servidor secundário (talvez você tenha um nome de servidor diferente).

    ![Captura de tela mostrando a página de servidores SQL.](../images/dp-300-module-14-lab-09.png)

1. Na folha do servidor SQL, na seção **Configurações**, selecione **bancos de dados SQL**.

    ![Captura de tela mostrando a opção bancos de dados SQL.](../images/dp-300-module-14-lab-10.png)

1. Na folha principal do banco de dados SQL, na seção **Gerenciamento de dados**, selecione **Réplicas**.

    ![Captura de tela mostrando a seleção da replicação geográfica.](../images/dp-300-module-14-lab-01.png)

1. Observe que o link de replicação geográfica agora está estabelecido.

    ![Captura de tela mostrando a opção Réplicas.](../images/dp-300-module-14-lab-11.png)

1. Selecione o menu **...** do servidor secundário e selecione **Failover forçado**.

    ![Captura de tela mostrando a opção de failover forçado.](../images/dp-300-module-14-lab-12.png)

    > [!NOTE]
    > O failover forçado mudará o banco de dados secundário para exercer a função de primário. Todas as sessões são desconectadas durante esta operação.

1. Quando solicitado pela mensagem de aviso, clique em **Sim**.

    ![Captura de tela mostrando uma mensagem de aviso de failover forçado.](../images/dp-300-module-14-lab-13.png)

1. O status da réplica primária será alternado para **Pendente** e o status da secundária para **Failover**. 

    ![Captura de tela mostrando uma mensagem de aviso de failover forçado.](../images/dp-300-module-14-lab-14.png)

    > [!NOTE]
    > Esse processo pode levar alguns minutos. Após a conclusão, as funções serão alternadas quando o secundário se tornar o novo primário e o primário anterior se tornar o secundário.

Vimos que a réplica secundária do banco de dados pode estar na mesma região do Azure que o primário ou, o que é mais comum, em uma região diferente. Esse tipo de banco de dados secundário legível também é conhecido como secundários geográficos ou réplicas geográficas.

Você aprendeu a habilitar a replicação geográfica para o Banco de Dados SQL do Azure e fazer o failover manual para outra região usando o portal.
