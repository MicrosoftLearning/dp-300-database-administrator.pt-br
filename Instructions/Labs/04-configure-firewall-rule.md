---
lab:
  title: Laboratório 4 - Configurar regras de firewall do Banco de Dados SQL do Azure
  module: Implement a Secure Environment for a Database Service
---

# Implementar um ambiente seguro

**Tempo estimado**: 30 minutos

Os alunos usarão as informações obtidas nas lições para configurar e, posteriormente, implementar a segurança no Portal do Azure e no banco de dados do *AdventureWorksLT*.

Você foi contratado como um Administrador de Banco de Dados Sênior para ajudar a garantir a segurança do ambiente de banco de dados. Essas tarefas se concentrarão no Banco de Dados SQL do Azure.

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

## Configurar regras de firewall do Banco de Dados SQL do Azure

1. Na máquina virtual do laboratório, ou em seu computador local, se não tiver sido fornecida, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando as suas credenciais do Azure.

1. No Portal do Azure, pesquise por *servidores SQL* na caixa de pesquisa na parte superior e clique em **servidores SQL** na lista de opções.

1. Selecione o SQL Server **dp300-lab-xxxxxxxx**, em que *xxxxxxxx* é uma cadeia de caracteres numérica aleatória.

    > &#128221; Observe que, se você estiver usando seu próprio servidor SQL do Azure não criado por este laboratório, selecione o nome desse SQL Server.

1. Na tela *Visão geral* do seu servidor SQL, à direita do nome do servidor, clique no botão **Copiar para a área de transferência**.

1. Selecione **Mostrar configurações de rede**.

1. Na página **Rede**, em **Regras de firewall**, revise a lista e verifique se o endereço IP do cliente está listado. Se não estiver listado, selecione **+ Adicionar endereço IPv4 de cliente (seu endereço IP)** e **Salvar**.

    > &#128221; O endereço IP do seu cliente foi inserido automaticamente. A inclusão do endereço IP do cliente à lista permitirá que você se conecte ao seu Banco de Dados SQL do Azure usando o SSMS (SQL Server Management Studio) ou qualquer outra ferramenta de cliente. **Anote o endereço IP do cliente, pois você o usará mais adiante**.

1. Abra o SQL Server Management Studio. Na caixa de diálogo Conectar ao Servidor, cole o nome do seu servidor do Banco de Dados SQL do Azure e entre com as seguintes credenciais:

    - **Nome do servidor:** &lt;_cole o nome do servidor do Banco de Dados SQL do Azure aqui_&gt;
    - **Autenticação:** Autenticação do SQL Server
    - **Logon de administrador do servidor:** seu logon de administrador do servidor do Banco de Dados SQL do Azure
    - **Senha:** sua senha de administrador do servidor do Banco de Dados SQL do Azure

1. Selecione **Conectar**.

1. No Pesquisador de Objetos, expanda o nó do servidor e clique com o botão direito do mouse em **Bancos de Dados**. Selecione **Importar um Aplicativo da Camada de Dados**.

1. Na caixa de diálogo **Importar Aplicativo da Camada de Dados**, clique em **Avançar** na primeira tela.

1. Na tela **Importar configurações**, clique em **Procurar** e navegue até a pasta **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\04**, clique no arquivo **AdventureWorksLT.bacpac** e, em seguida, em **Abrir**. De volta à tela **Importar Aplicativo da Camada de Dados**, clique em **Avançar**.

1. Na tela **Configurações do Banco de Dados**, faça as seguintes alterações:

    - **Nome do banco de dados:** AdventureWorksFromBacpac
    - **Edição do Banco de Dados SQL do Microsoft Azure**: Básico

1. Selecione **Avançar**.

1. Na tela **Resumo**, selecione **Concluir**. Isso pode levar alguns minutos. Quando a importação for concluída, você verá os resultados abaixo. Em seguida, selecione **Fechar**.

1. De volta ao SQL Server Management Studio, no **Pesquisador de Objetos**, expanda a pasta **Bancos de Dados**. Em seguida, clique com o botão direito do mouse no banco de dados **AdventureWorksFromBacpac** e, em seguida, **Nova Consulta**.

1. Execute a consulta T-SQL a seguir colando o texto na janela de consulta.
    1. **Importante:** Substitua **000.000.000.000** pelo endereço IP do cliente. Selecione **Executar**.

    ```sql
    EXECUTE sp_set_database_firewall_rule 
            @name = N'AWFirewallRule',
            @start_ip_address = '000.000.000.000', 
            @end_ip_address = '000.000.000.000'
    ```

1. Em seguida, você criará um usuário contido no banco de dados **AdventureWorksFromBacpac**. Selecione **Nova Consulta** e execute o T-SQL a seguir.

    ```sql
    USE [AdventureWorksFromBacpac]
    GO
    CREATE USER ContainedDemo WITH PASSWORD = 'P@ssw0rd01'
    ```

    > &#128221; Esse comando cria um usuário contido no banco de dados **AdventureWorksFromBacpac**. Vamos testar essa credencial na próxima etapa.

1. Navegue até o **Pesquisador de Objetos**. Clique em **Conectar** e, em seguida **Mecanismo de Banco de Dados**.

1. Tente se conectar com as credenciais criadas na etapa anterior. Será preciso usar as seguintes informações:

    - **Logon:** ContainedDemo
    - **Senha**: P@ssw0rd01

     Clique em **Conectar**.

     Você receberá o erro a seguir.

    <span style="color:red">Falha no login do usuário "ContainedDemo". (Microsoft SQL Server, Error: 18456)</span>

    > &#128221; Este erro é gerado porque a conexão tentou entrar no banco de dados *master* e não no **AdventureWorksFromBacpac**, onde o usuário foi criado. Altere o contexto de conexão clicando em **OK** para sair da mensagem de erro e, em seguida, em **Opções >>** em **Conectar ao servidor**.

1. Na guia **Propriedades da conexão**, digite o nome do banco de dados **AdventureWorksFromBacpac** e selecione **Conectar**.

1. Observe que você conseguiu autenticar com êxito usando o usuário **ContainedDemo**. Desta vez, você foi conectado diretamente ao **AdventureWorksFromBacpac**, que é o único banco de dados ao qual o usuário recém-criado tem acesso.

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

Neste exercício, você configurou regras de firewall de servidor e banco de dados para acessar um banco de dados hospedado no Banco de Dados SQL do Azure. Você também usou instruções T-SQL para criar um usuário contido e usou o SQL Server Management Studio para verificar o acesso.
