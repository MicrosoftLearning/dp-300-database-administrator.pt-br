---
lab:
  title: 'Laboratório 1: provisionar o SQL Server em uma Máquina Virtual do Azure'
  module: Plan and Implement Data Platform Resources
---

# Provisionar um SQL Server em uma máquina virtual do Azure

**Tempo estimado**: 30 minutos

Os alunos vão explorar o Portal do Azure e usá-lo para criar uma VM do Azure com o SQL Server 2022 instalado. Em seguida, vão se conectar à máquina virtual por meio do Protocolo de Área de Trabalho Remota.

Você é um administrador de banco de dados da AdventureWorks. Você precisa criar um ambiente de teste para usar em uma prova de conceito. A prova de conceito usará o SQL Server em uma Máquina Virtual do Azure e um backup do banco de dados AdventureWorksDW. Você precisará configurar a Máquina Virtual, restaurar o banco de dados e consultá-lo para garantir que esteja disponível.

## Implantar o SQL Server em uma Máquina Virtual do Azure

1. Na máquina virtual do laboratório, inicie uma sessão do navegador, navegue até [https://portal.azure.com](https://portal.azure.com/) e faça login usando a conta Microsoft associada à sua assinatura do Azure.

1. Localize a barra de pesquisa na parte superior da página. Pesquise por **SQL do Azure**. Selecione o resultado da pesquisa do **SQL do Azure** que aparece nos resultados em **Serviços**.

1. Na folha do **SQL do Azure**, selecione **Criar**.

1. Na folha **Selecionar opção de implantação do SQL**, abra a caixa suspensa na guia **Máquinas virtuais do SQL**. Selecione a opção rotulada como **Licença Gratuita do SQL Server: Desenvolvedor do SQL 2022 no Windows Server 2022**. Em seguida, selecione **Criar**.

1. Na página **Criar uma máquina virtual**, insira as seguintes informações e *deixe todas as outras opções como os valores* padrão:

    - **Assinatura**: &lt;Sua assinatura&gt;
    - **Grupo de recursos**: &lt;seu grupo de recursos&gt;
    - **Nome da máquina virtual:** AzureSQLServerVM
    - **Região:**&lt;escolha a sua região local, a mesma região selecionada para o seu grupo de recursos&gt;
    - **Opções de Disponibilidade:** Sem necessidade de redundância de infraestrutura
    - **Imagem:** Licença Gratuita do SQL Server: Desenvolvedor do SQL 2022 no Windows Server 2022 — Gen2
    - **Executar com desconto spot do Azure:** Não (desmarcado)
    - **Tamanho:** Padrão *D2s_v5* (2 vCPUs, 8 GiB de memória). *Talvez você precise clicar no link "Ver todos os tamanhos" para ver essa opção.*
    - **Nome de usuário da conta de administrador:**&lt;escolha um nome para sua conta de administrador.&gt;
    - **Senha da conta de administrador:**&lt;escolha uma senha forte.&gt;
    - **Selecionar as portas de entrada:** RDP (3389)
    - **Quer usar uma licença existente do Windows Server?** Não (desmarcada)

    > &#128221; Anote o nome de usuário e a senha para usar mais tarde.

1. Navegue até a guia **Discos** e examine a configuração.

1. Navegue até a guia **Rede** e examine a configuração.

1. Navegue até a guia **Gerenciamento** e examine a configuração.

    Verifique se **Habilitar auto_shutdown** está desmarcado.

1. Navegue até a guia **Avançado** e examine a configuração.

1. Navegue até a guia **Configurações do SQL Server** e examine a configuração.

    > &#128221; Você também pode configurar o armazenamento da sua VM do SQL Server nessa tela. Por padrão, os modelos de VM do SQL Server do Azure cria um disco premium com armazenamento em cache de leitura para dados e um disco premium sem armazenamento em cache para o log de transações, além de usar o SSD local (D:\ no Windows) para o tempdb.

1. Selecione o botão **Revisar + criar**. Em seguida, selecione **Criar**.

1. Na folha de implantação, aguarde até que a implantação seja concluída. A VM levará cerca de 5 a 10 minutos para ser implantada. Após a implantação ser concluída, selecione **Ir para o recurso**.

    > &#128221 Sua implantação pode levar vários minutos para ser concluída.

1. Na página **Visão geral** da máquina virtual, explore as opções de menu para esse recurso e leia o que está disponível.

---

## Conectar-se ao SQL Server em uma Máquina Virtual do Azure

1. Na página **Visão geral** da máquina virtual, selecione o menu suspenso **Conectar** e selecione **Conectar**.

1. Na página Conectar, clique no botão **Baixar arquivo RDP**.

    > &#128221; Se vir o erro **Pré-requisito de porta não atendido**. certifique-se de selecionar o link para adicionar uma regra de entrada do grupo de segurança de rede com a porta de destino mencionada no campo *Número da Porta*.

1. Abra o arquivo RDP que acabou de ser baixado. Quando uma caixa de diálogo aparecer perguntando se você quer se conectar, selecione **Conectar**.

1. Insira o nome de usuário e a senha selecionados durante o processo de provisionamento da máquina virtual. Em seguida, selecione **OK**.

1. Quando a caixa de diálogo **Conexão da Área de Trabalho Remota** aparecer, perguntando se você quer se conectar, selecione **Sim**.

1. Selecione a barra de pesquisa ao lado do botão Iniciar do Windows e digite SSMS. Selecione **Microsoft SQL Server Management Studio** na lista.  

1. Quando o SSMS for aberto, observe que a caixa de diálogo **Conectar ao Servidor** será pré-preenchida com o nome de instância padrão. Marque a opção **Confiar em certificado do servidor** e selecione **Conectar**.

1. Feche o SSMS clicando no **X** no canto superior direito.

1. Agora você pode se desconectar da máquina virtual para fechar a sessão RDP.

O portal do Azure fornece ferramentas poderosas para gerenciar um SQL Server hospedado em uma máquina virtual. Essas ferramentas incluem controle sobre aplicação de patch automatizada, backups automatizados e uma maneira fácil de configurar a alta disponibilidade.

---

## Recursos de limpeza

Se você não estiver usando a máquina virtual para nenhuma outra finalidade, poderá limpar os recursos criados neste laboratório.

### Excluir o Grupo de Recursos

Se você criou um novo grupo de recursos para este laboratório, poderá excluir o grupo de recursos para remover todos os recursos criados neste laboratório.

1. No portal do Azure, selecione **Grupos de recursos** no painel de navegação esquerdo ou pesquise **Grupos de recursos** na barra de pesquisa e selecione-o nos resultados.

1. Vá para o grupo de recursos criado para o laboratório. O grupo de recursos conterá a máquina virtual e outros recursos criados neste laboratório.

1. Escolha **Excluir grupo de recursos** no menu superior.

1. Na caixa de diálogo **Excluir grupo de recursos**, digite o nome do grupo para confirmar e clique em **Excluir**.

1. Aguarde até que o grupo de recursos seja excluído.

1. Feche o portal do Azure.

### Excluir apenas os recursos do laboratório

Se você não criou um novo grupo de recursos para este laboratório e deseja deixar o grupo de recursos e seus recursos anteriores intactos, ainda poderá excluir os recursos criados neste laboratório.

1. No portal do Azure, selecione **Grupos de recursos** no painel de navegação esquerdo ou pesquise **Grupos de recursos** na barra de pesquisa e selecione-o nos resultados.

1. Vá para o grupo de recursos criado para o laboratório. O grupo de recursos conterá a máquina virtual e outros recursos criados neste laboratório.

1. Selecione todos os recursos prefixados com o nome da máquina virtual especificado anteriormente no laboratório.

1. Selecione **Excluir** no menu superior.

1. Na caixa de diálogo **Excluir recursos**, digite **excluir** e selecione **Excluir**.

1. Para confirmar a exclusão dos recursos, clique em excluir novamente **Excluir**.

1. Aguarde a exclusão dos recursos.

1. Feche o portal do Azure.

---

Você concluiu este laboratório.
