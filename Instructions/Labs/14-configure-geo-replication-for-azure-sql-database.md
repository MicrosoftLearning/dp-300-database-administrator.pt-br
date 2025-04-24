---
lab:
  title: 'Laboratório 14: configurar a replicação geográfica para o Banco de Dados SQL do Azure'
  module: Plan and implement a high availability and disaster recovery solution
---

# Configurar a replicação geográfica para o Banco de Dados SQL do Azure

**Tempo estimado**: 30 minutos

Como um administrador de banco de dados na AdventureWorks, você precisa habilitar a replicação geográfica para o Banco de Dados SQL do Azure e garantir que esteja funcionando corretamente. Além disso, você fará failover manualmente para outra região usando o portal.

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

## Habilitar a replicação geográfica

1.  Na máquina virtual do laboratório, ou em seu computador local, se não tiver sido fornecida, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando as suas credenciais do Azure.

1. No portal do Azure, navegue até o banco de dados pesquisando por **bancos de dados SQL**.

1. Selecione o banco de dados SQL **AdventureWorksLT**.

1. Na folha do banco de dados, na seção **Gerenciamento de dados**, selecione **Réplicas**.

1. Selecione **+ Criar réplica**.

1. Na página **Criar banco de dados SQL — Réplica geográfica**, observe que as seções **Detalhes do projeto** e **Banco de dados primário** já estão preenchidas com a assinatura, o grupo de recursos e o nome do banco de dados.

1. Na seção **Configuração de réplica**, selecione **Réplica geográfica** em *Tipo de réplica*.

1. Nos **Detalhes do banco de dados geográfico secundário**, preencha os seguintes valores:

    - **Assinatura**: &lt;o nome da sua assinatura&gt; (o mesmo do banco de dados primário).
    - **Grupo de recursos**: &lt;selecione o mesmo grupo de recursos que o banco de dados primário.&gt; 
    - **Nome do banco de dados**: o nome do banco de dados ficará esmaecido e será o mesmo que o nome do banco de dados principal.
    - **Servidor**: selecione **Criar novo**.
    - Insira os seguintes valores na página **Criar Servidor do Banco de Dados SQL**:

        - **Nome do servidor**: insira um nome exclusivo para o servidor secundário. O nome deve ser exclusivo para todos os servidores do Banco de Dados SQL do Azure.
        - **Local**: selecione uma região diferente do banco de dados primário. Talvez a sua assinatura não tenha todas as regiões disponíveis.
        - Marque a caiaxa de seleção **Permitir que os serviços do Azure acessem o servidor**. Observe que, em um ambiente de produção, talvez você queira restringir o acesso ao servidor.
        - Em autenticação, selecione **Autenticação do SQL**. Observe que, em um ambiente de produção, talvez você queira usar a autenticação **Usar somente o Microsoft Entra**. Digite **sqladmin* no nome de login do administrador e uma senha segura. A senha deve atender aos requisitos de complexidade de senha do SQL do Azure, com pelo menos 12 caracteres e conter pelo menos 1 letra maiúscula, 1 letra minúscula, 1 número e 1 caractere especial.
        - Selecione **OK** para criar a o servidor.

    - **Deseja usar o pool elástico?**: Não.
    - **Computação + armazenamento**: Uso Geral, Gen 5, 2 vCores, 32 GB de armazenamento.
    - **Redundância de armazenamento de backup**: Armazenamento com redundância local (LRS). Observe que, em um ambiente de produção, talvez você queira usar o **armazenamento com redundância geográfica (GRS)**.

1. Selecione **Examinar + criar**.

1. Selecione **Criar**. Levará alguns minutos para criar o servidor secundário e o banco de dados. Depois de concluído, o progresso mudará de **Implantação em andamento** para **Implantação concluída**.

1. Selecione **Ir para o recurso** para navegar até o banco de dados do servidor secundário para a próxima etapa.

## Fazer failover do Banco de Dados SQL para uma região secundária

Agora que a réplica do Banco de Dados SQL do Azure foi criada, você fará um failover.

1. Se ainda não estiver no banco de dados do servidor secundário, pesquise **bancos de dados sql** no portal do Azure e selecione o banco de dados SQL **AdventureWorksLT** no servidor secundário.

1. Na folha principal do banco de dados SQL, na seção **Gerenciamento de dados**, selecione **Réplicas**.

1. Observe que o link de replicação geográfica agora está estabelecido. O valor do *Estado da réplica* do banco de dados primário é **Online** e o valor do *Estado da réplica* das réplicas geográficas é **Legível**.

1. Selecione o menu **...** do servidor de réplica geográfica secundária e selecione **Failover forçado**.

    > &#128221; O failover forçado mudará o banco de dados secundário para a função de primário. Todas as sessões são desconectadas durante esta operação.

1. Quando solicitado pela mensagem de aviso, clique em **Sim**.

1. O status da réplica primária será alternado para **Pendente** e o status da secundária para **Failover**. 

     > &#128221; Como o banco de dados é pequeno, o failover será rápido. Em um ambiente de produção, esse processo pode levar alguns minutos.

1. Após a conclusão, as funções serão alternadas quando o secundário se tornar o novo primário e o primário anterior se tornar o secundário. Talvez seja necessário atualizar a página para ver o novo status.

Vimos que a réplica secundária do banco de dados pode estar na mesma região do Azure que o primário ou, o que é mais comum, em uma região diferente. Esse tipo de banco de dados secundário legível também é conhecido como secundários geográficos ou réplicas geográficas.

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

Você aprendeu a habilitar a replicação geográfica para o Banco de Dados SQL do Azure e fazer o failover manual para outra região usando o portal.
