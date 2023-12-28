---
lab:
  title: Configurar regras de firewall do Banco de Dados SQL do Azure
  module: Implement a Secure Environment for a Database Service
---

# Implementar um ambiente seguro

**Tempo estimado**: 20 minutos

Os alunos pegarão as informações obtidas nas lições para configurar e, posteriormente, implementar a segurança no Portal do Azure e no banco de dados AdventureWorks.

Você foi contratado como um Administrador de Banco de Dados Sênior para ajudar a garantir a segurança do ambiente de banco de dados. Essas tarefas se concentrarão no Banco de Dados SQL do Azure.

**Nota:** Estes exercícios pedem que você copie e cole o código T-SQL. Verifique se o código foi copiado corretamente, antes de executar o código.

## Configurar regras de firewall do Banco de Dados SQL do Azure

1. Na máquina virtual do laboratório, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando o Nome** de Usuário e **a Senha** do Azure **fornecidos na **guia Recursos** para esta máquina virtual de laboratório.

    ![Figura 1](../images/dp-300-module-01-lab-01.png)

1. No Portal do Azure, procure "SQL servers" na caixa de pesquisa na parte superior e clique em **SQL servers** na lista de opções.

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-1.png)

1. Selecione o nome **do servidor dp300-lab-XXXXXXXX** a ser levado para a página de detalhes (você pode ter um grupo de recursos e um local diferentes atribuídos ao seu SQL Server).

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-2.png)

1. Na tela de detalhes do SQL Server, mova o mouse para a direita do nome do servidor e selecione o botão **Copiar para área de transferência**, conforme mostrado abaixo.

    ![Figura 2](../images/dp-300-module-04-lab-3.png)

1. Selecione **Mostrar configurações** de rede.

    ![Figura 2](../images/dp-300-module-04-lab-4.png)

1. Na página Rede, clique em + Adicionar o endereço IPv4 do **cliente (seu endereço IP)** e clique em ****Salvar**.**

    ![Figura 3](../images/dp-300-module-04-lab-5.png)

    **Nota:** O endereço IP do seu cliente foi inserido automaticamente para você. Essas configurações permitirão que você se conecte ao seu servidor do Banco de Dados SQL do Azure usando o SQL Server Management Studio ou qualquer outra ferramenta de cliente. Anote o endereço IP do cliente, pois você o usará mais adiante neste exercício.

1. Abra o SQL Server Management Studio. Cole o nome do seu servidor do banco de dados SQL do Azure e entre com estas credenciais:

    - Para o Nome do servidor, cole o nome do servidor lógico do Banco de Dados SQL do Azure.
    - Autenticação: autenticação do SQL Server
    - Logon de administrador do servidor: labadmin
    - **Senha**: P@ssw0rd01

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-6.png)

1. Clique em **Conectar**.

1. No Pesquisador de Objetos, expanda o nó de servidor e clique com o botão direito do mouse em Bancos de dados e selecione Importar Aplicativo da Camada de Dados. Clique em **Importar Aplicativo da Camada de Dados**.

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-7.png)

1. Na primeira tela da caixa de diálogo **Importar Aplicativo da Camada de Dados**, selecione **Avançar**.

1. Faça download do arquivo .bacpac localizado no ****https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorksLT.bacpac** caminho C:\LabFiles\Secure Environment** na VM do laboratório (crie a estrutura de pastas se ela não existir).

1. Na tela Configurações de Importação, selecione Procurar e acesse a pasta D:\Labfiles\Secure Environment, selecione o arquivo AdventureWorks.bacpac e escolha Abrir. De volta à **tela Importar Aplicativo** da Camada de Dados, clique em **Avançar**.

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-8.png)

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-9.png)

1. Na tela Configurações** do **Banco de Dados, faça as alterações conforme abaixo:

    - **Nome do banco de dados:** AdventureWorksFromBacpac
    - Edição do Banco de Dados SQL do Microsoft Azure:

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-10.png)

1. Clique em **Avançar**.

1. Na tela “Resumo”, clique em Concluir. Quando a importação for concluída, você verá os resultados abaixo. Em seguida, clique em **Fechar**.

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-11.png)

1. Pesquisador de Objetos do SQL Server Management Studio expandido para mostrar o banco de dados. Em seguida, clique com o botão direito do mouse no novo banco de dados, por exemplo, **AdventureWorks-copy**, e selecione **Nova Consulta**.

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-12.png)

1. Execute a consulta T-SQL a seguir colando o texto na janela de consulta e selecionando Executar ou clicando em F5.
    1. **Importante:** Substitua **000.000.000.00** pelo endereço IP do cliente. Pressione F5 ou clique em Executar.

    ```sql
    EXECUTE sp_set_database_firewall_rule 
            @name = N'AWFirewallRule',
            @start_ip_address = '000.000.000.00', 
            @end_ip_address = '000.000.000.00'
    ```

1. Em seguida, você criará um usuário contido no banco de dados AdventureWorks. Selecione **Nova Consulta** e execute o T-SQL a seguir.

    ```sql
    USE [AdventureWorksFromBacpac]
    GO
    CREATE USER ContainedDemo WITH PASSWORD = 'P@ssw0rd01'
    ```

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-13.png)

    Insira este comando para criar um usuário contido no banco de dados AdventureWorks. Vamos testar essa credencial na próxima etapa.

1. No , navegue até o pacote no Pesquisador de Objetos. Clique em Conectar** e, em seguida **, em **Mecanismo de Banco de Dados**.

    ![Figura 65342997](../images/dp-300-module-04-lab-14.png)

1. Tente se conectar com as credenciais criadas na etapa 4. Será preciso fornecer as seguintes informações:

    - Logon: containeddemo
    - **Senha**: P@ssw0rd01

     Clique em **Conectar**.

     Você recebe o seguinte erro:

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-04-lab-15.png)

    Esse erro ocorreu porque a conexão tentou entrar no banco de dados mestre e não no AdventureWorks, em que o usuário foi criado. Altere o contexto de conexão selecionando OK para sair da mensagem de erro e escolhendo Opções na caixa de diálogo Conectar-se ao Servidor, conforme mostrado abaixo.

    ![Figura 9](../images/dp-300-module-04-lab-16.png)

1. Na guia Propriedades da **Conexão, digite o nome **do banco de dados AdventureWorksFromBacpac** e clique em **Conectar****.

    ![Uma captura de tela de uma postagem de mídia social Descrição gerada automaticamente](../images/dp-300-module-04-lab-17.png)

1. Observe que você conseguiu autenticar com êxito usando o **usuário ContainedDemo** . Essa conexão ignora o banco de dados mestre e conecta você diretamente ao AdventureWorks, que é o único banco de dados ao qual o usuário recém-criado tem acesso.

    ![Figura 10](../images/dp-300-module-04-lab-18.png)

Neste exercício, você configurou regras de firewall de servidor e banco de dados para acessar um banco de dados hospedado no Banco de Dados SQL do Azure. Você também usou instruções T-SQL para criar um usuário contido e usou o SQL Server Management Studio para verificar o acesso.
