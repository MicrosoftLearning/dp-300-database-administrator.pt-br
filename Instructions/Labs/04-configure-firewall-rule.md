---
lab:
  title: Laboratório 4 - Configurar regras de firewall do Banco de Dados SQL do Azure
  module: Implement a Secure Environment for a Database Service
---

# Implementar um ambiente seguro

**Tempo estimado**: 30 minutos

Os alunos usarão as informações obtidas nas lições para configurar e, posteriormente, implementar a segurança no Portal do Azure e no banco de dados do AdventureWorks.

Você foi contratado como um Administrador de Banco de Dados Sênior para ajudar a garantir a segurança do ambiente de banco de dados. Essas tarefas se concentrarão no Banco de Dados SQL do Azure.

**Observação:** Esses exercícios solicitam que você copie e cole o código T-SQL e use os recursos existentes do SQL. Verifique se o código foi copiado corretamente antes de executá-lo.

## Configurar regras de firewall do Banco de Dados SQL do Azure

1. Na máquina virtual do laboratório, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando o **Nome de Usuário** e a **Senha** do Azure fornecidos na guia **Recursos** dessa máquina virtual do laboratório.

    ![Figura 1](../images/dp-300-module-01-lab-01.png)

1. No Portal do Azure, pesquise por "servidores SQL" na caixa de pesquisa na parte superior e clique em **servidores SQL** na lista de opções.

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-1.png)

1. Selecione o nome do servidor **dp300-lab-XXXXXXXXX** a ser levado para a página de detalhes (você pode ter um grupo de recursos e um local diferentes atribuídos ao seu servidor SQL).

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-2.png)

1. Na tela de detalhes do SQL Server, mova o mouse para a direita do nome do servidor e selecione o botão **Copiar para área de transferência**, conforme mostrado abaixo.

    ![Imagem 2](../images/dp-300-module-04-lab-3.png)

1. Selecione **Mostrar configurações de rede**.

    ![Imagem 2](../images/dp-300-module-04-lab-4.png)

1. Na página **Rede**, clique em **+ Adicionar o endereço IPv4 do cliente (seu endereço IP)** e, em seguida, clique em **Salvar**.

    ![Imagem 3](../images/dp-300-module-04-lab-5.png)

    **Observação:** O endereço IP do seu cliente foi inserido automaticamente. A inclusão do endereço IP do cliente à lista permitirá que você se conecte ao seu Banco de Dados SQL do Azure usando o SQL Server Management Studio ou qualquer outra ferramenta de cliente. **Anote o endereço IP do cliente, pois você o usará mais adiante**.

1. Abra o SQL Server Management Studio. Na caixa de diálogo Conectar ao Servidor, cole o nome do seu servidor do banco de dados SQL do Azure e entre com as credenciais abaixo:

    - **Nome do servidor:** &lt;_cole o nome do servidor do Banco de Dados SQL do Azure aqui_&gt;
    - **Autenticação:** Autenticação do SQL Server
    - **Logon de administrador do servidor:** sqladmin
    - **Senha**: P@ssw0rd01

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-6.png)

1. Clique em **Conectar**.

1. No Pesquisador de Objetos, expanda o nó do servidor e clique com o botão direito do mouse em **Bancos de Dados**. Clique em **Importar um aplicativo da camada de dados**.

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-7.png)

1. Na caixa de diálogo **Importar Aplicativo da Camada de Dados**, clique em **Avançar** na primeira tela.

1. Faça download do arquivo .bacpac localizado no **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorksLT.bacpac** para o caminho **C:\LabFiles\Secure Environment** na VM do laboratório (crie a estrutura de pastas, se ainda não existir).

1. Na tela **Configurações de Importação**, clique em **Procurar** e acesse a pasta **C:\LabFiles\Secure Environment**, clique no arquivo **AdventureWorksLT.bacpac** e clique em **Abrir**. De volta à tela **Importar Aplicativo da Camada de Dados**, clique em **Avançar**.

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-8.png)

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-9.png)

1. Na tela **Configurações do Banco de Dados**, faça as seguintes alterações:

    - **Nome do banco de dados:** AdventureWorksFromBacpac
    - **Edição do Banco de Dados SQL do Microsoft Azure**: Básico

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-10.png)

1. Clique em **Avançar**.

1. Na tela **Resumo**, clique em **Concluir**. Quando a importação for concluída, você verá os resultados abaixo. Em seguida, clique em **Fechar**.

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-11.png)

1. De volta ao SQL Server Management Studio, no **Pesquisador de Objetos**, expanda a pasta **Bancos de Dados**. Em seguida, clique com o botão direito do mouse no banco de dados **AdventureWorksFromBacpac** e, em seguida, **Nova Consulta**.

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-12.png)

1. Execute a consulta T-SQL a seguir colando o texto na janela de consulta.
    1. **Importante:** Substitua **000.000.000.00** pelo endereço IP do cliente. Clique em **Executar** ou pressione **F5**.

    ```sql
    EXECUTE sp_set_database_firewall_rule 
            @name = N'AWFirewallRule',
            @start_ip_address = '000.000.000.00', 
            @end_ip_address = '000.000.000.00'
    ```

1. Em seguida, você criará um usuário contido no banco de dados **AdventureWorksFromBacpac**. Clique em **Nova Consulta** e execute o T-SQL a seguir.

    ```sql
    USE [AdventureWorksFromBacpac]
    GO
    CREATE USER ContainedDemo WITH PASSWORD = 'P@ssw0rd01'
    ```

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-13.png)

    **Observação:** Esse comando cria um usuário contido no banco de dados **AdventureWorksFromBacpac**. Vamos testar essa credencial na próxima etapa.

1. Navegue até o **Pesquisador de Objetos**. Clique em **Conectar** e, em seguida **Mecanismo de Banco de Dados**.

    ![Figura 1960831949](../images/dp-300-module-04-lab-14.png)

1. Tente se conectar com as credenciais criadas na etapa anterior. Será preciso usar as seguintes informações:

    - **Logon:** ContainedDemo
    - **Senha**: P@ssw0rd01

     Clique em **Conectar**.

     Você receberá o erro a seguir.

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-15.png)

    **Observação:** Esse erro é gerado porque a conexão tentou entrar no banco de dados *mestre* e não no **AdventureWorksFromBacpac**, em que o usuário foi criado. Altere o contexto de conexão clicando em **OK** para sair da mensagem de erro e selecionando **Opções>>** na caixa de diálogo **Conectar-se ao Servidor**, conforme mostrado abaixo.

    ![Imagem 9](../images/dp-300-module-04-lab-16.png)

1. Na guia **Propriedades da Conexão**, digite o nome do banco de dados **AdventureWorksFromBacpac** e clique em **Conectar**.

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-17.png)

1. Observe que você conseguiu autenticar com êxito usando o usuário **ContainedDemo**. Desta vez, você foi conectado diretamente ao **AdventureWorksFromBacpac**, que é o único banco de dados ao qual o usuário recém-criado tem acesso.

    ![Figura 10](../images/dp-300-module-04-lab-18.png)

Neste exercício, você configurou regras de firewall de servidor e banco de dados para acessar um banco de dados hospedado no Banco de Dados SQL do Azure. Você também usou instruções T-SQL para criar um usuário contido e usou o SQL Server Management Studio para verificar o acesso.
