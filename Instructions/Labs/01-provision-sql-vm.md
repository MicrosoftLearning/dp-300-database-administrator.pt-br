---
lab:
  title: 'Laboratório: provisionar o SQL Server em uma Máquina Virtual do Azure'
  module: Plan and Implement Data Platform Resources
---

# Provisionar um SQL Server em uma máquina virtual do Azure

**Tempo estimado**: 20 minutos

Os alunos explorarão o Portal do Azure e o usarão para criar uma VM do Azure com o SQL Server 2019 instalado. Em seguida, eles se conectarão à máquina virtual por meio do Protocolo de Área de Trabalho Remota.

Você é um administrador de banco de dados do AdventureWorks. Você precisa criar um ambiente de teste para usar em uma prova de conceito. A prova de conceito usará o SQL Server em uma máquina virtual do Azure. Você precisa configurar a Máquina Virtual, restaurar o banco de dados e consultá-lo para garantir que ele esteja disponível.

## Implantar o SQL Server em uma Máquina Virtual do Azure

1. Na máquina virtual do laboratório, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/)e entre usando a conta da Microsoft associada à sua assinatura do Azure.

    ![Figura 1](../images/dp-300-module-01-lab-01.png)

1. Localize a barra de pesquisa na parte superior da página. Pesquise por SQL do Azure. Selecione o resultado da pesquisa para o SQL do Azure que aparece nos resultados em Serviços.

    ![Figura 9](../images/dp-300-module-01-lab-09.png)

1. Na folha **Azure Cosmos DB**, selecione **+ Criar**.

    ![Figura 10](../images/dp-300-module-01-lab-10.png)

1. Na folha Selecionar opção de implantação do SQL, abra a caixa suspensa em máquinas virtuais do SQL. Selecione **Licença Gratuita do SQL Server: Desenvolvedor do SQL 2019 no Windows Server 2019**. Em seguida, selecione **Criar**.

    ![Figura 11](../images/dp-300-module-01-lab-11.png)

1. Na página Criar uma máquina virtual, insira as seguintes informações:

    - **Assinatura**: &lt;Sua assinatura&gt;
    - Grupo de recursos: o grupo de recursos.
    - Nome da máquina virtual: azureSQLserverVM
    - **Região:** &lt;sua região local, igual à região selecionada para seu grupo de recursos&gt;
    - **Opções de disponibilidade**: sem redundância de infraestrutura necessária
    - Licença Gratuita do SQL Server: Desenvolvedor do SQL 2019 no Windows Server 2019 –
    - **Instância spot do Azure:** Não (desmarcada)
    - Padrão D2s v3 (2 vCPUs, 8 GiB de memória) Talvez seja necessário selecionar o link "Ver todos os tamanhos" para ver essa opção)
    - Nome de usuário da conta administrador: labadmin
    - **Senha da conta de administrador:** pwd! DP300lab01 (ou sua própria senha que atenda aos critérios)
    - **Selecionar portas de entrada** = RDP (3389)
    - Deseja usar uma licença existente do Windows Server

    Anote o nome de usuário e a senha para usar mais adiante.

    ![Figura 12](../images/dp-300-module-01-lab-12.png)

1. Navegue até a guia **Discos** e examine a configuração.

    ![Figura 13](../images/dp-300-module-01-lab-13.png)

1. Navegue até a guia **Rede** e examine a configuração.

    ![Figura 14](../images/dp-300-module-01-lab-14.png)

1. Navegue até a guia **Gerenciamento** e examine a configuração.

    ![Figura 15](../images/dp-300-module-01-lab-15.png)

    Verifique se **a opção Ativar auto_shutdown** está desmarcada.

1. Navegue até a guia **Avançado** e examine a configuração.

    ![Figura 16](../images/dp-300-module-01-lab-16.png)

1. Navegue até a guia **Configurações do SQL Server** e examine a configuração.

    ![Figura 17](../images/dp-300-module-01-lab-17.png)

    Você também pode configurar o armazenamento para sua VM do SQL Server nessa tela. Por padrão, o modelo de VM do Azure SQL Server cria um disco premium com cache de leitura para dados, um disco premium sem cache para o log de transações e usa o SSD local (D:\ no Windows) para o tempdb.

1. Selecione o botão **Revisar + criar**. Em seguida, selecione **Criar**.

    ![Figura 18](../images/dp-300-module-01-lab-18.png)

1. Na folha de implantação, aguarde até que a implantação seja concluída. A VM levará aproximadamente 5 a 10 minutos para ser implantada. Depois da conclusão da implantação, selecione **Ir para o recurso**.

    A implantação pode levar vários minutos para ser concluída.

    ![Figura 19](../images/dp-300-module-01-lab-19.png)

1. Na página Visão geral da máquina virtual, percorra as opções de menu do recurso para revisar o que está disponível.

    ![Figura 20](../images/dp-300-module-01-lab-20.png)

## Conectar-se ao SQL Server em uma Máquina Virtual do Azure

1. Na página de visão geral da máquina virtual, selecione o botão **Conectar** e selecione **RDP**.

    ![Figura 21](../images/dp-300-module-01-lab-21.png)

1. Na guia RDP, selecione Baixar arquivo RDP.

    ![Figura 22](../images/dp-300-module-01-lab-22.png)

    **Nota:** Se vir o erro **Pré-requisito de porta não atendido.** Certifique-se de selecionar o link para adicionar uma regra de grupo de segurança de rede de entrada com a porta de destino mencionada no campo Número* da *porta.

    ![Foto 22_1](../images/dp-300-module-01-lab-22_1.png)

1. Abra o arquivo RDP que foi baixado. Quando aparecer uma caixa de diálogo perguntando se você deseja se conectar, selecione **Conectar**.

    ![Figura 23](../images/dp-300-module-01-lab-23.png)

1. Insira o nome de usuário e a senha selecionados durante o processo de provisionamento da máquina virtual. Selecione **OK**.

    ![Figura 24](../images/dp-300-module-01-lab-24.png)

1. Quando a **caixa de diálogo Conexão de Área de Trabalho Remota** for exibida perguntando se você deseja se conectar, selecione **Sim**.

    ![Figura 26](../images/dp-300-module-01-lab-26.png)

1. Selecione o botão Iniciar do Windows e digite SSMS. Selecione **Microsoft SQL Server Management Studio 18** na lista.  

    ![Imagem 34](../images/dp-300-module-01-lab-34.png)

1. Quando o SSMS for aberto, observe que a caixa de diálogo Conectar ao Servidor** será pré-preenchida com o **nome de instância padrão. Selecione **Conectar**.

    ![Figura 35](../images/dp-300-module-01-lab-35.png)

O portal do Azure fornece ferramentas poderosas para gerenciar um SQL Server hospedado em uma máquina virtual. Essas ferramentas incluem controle sobre aplicação de patch automatizada, backups automatizados e uma maneira fácil de configurar a alta disponibilidade.
