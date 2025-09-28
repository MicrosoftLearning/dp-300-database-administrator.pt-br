---
lab:
  title: Laboratório 2 – Provisionar um Banco de Dados SQL do Azure
  module: Plan and Implement Data Platform Resources
---

# Provisionar um Banco de Dados SQL do Azure

**Tempo estimado: 40 minutos**

Os alunos configurarão os recursos básicos necessários para implantar um Banco de Dados SQL do Azure com um Ponto de Extremidade de Rede Virtual. A conectividade com o Banco de Dados SQL será validada usando o SQL Server Management Studio na VM do laboratório, se disponível, ou na configuração do computador local.

Como administrador de banco de dados do AdventureWorks, você configurará um novo Banco de Dados SQL, incluindo um Ponto de Extremidade de Rede Virtual para aumentar e simplificar a segurança da implantação. O SQL Server Management Studio será usado a fim de avaliar o uso de um Notebook SQL para consulta de dados e retenção dos resultados.

## Navegar no portal do Azure

1. Na máquina virtual do laboratório, se fornecida, caso contrário, em seu computador local, abra uma janela do navegador.

1. Navegue até o portal Azure em [https://portal.azure.com](https://portal.azure.com/). Entre no portal do Azure com as credenciais fornecidas, se disponíveis, ou usando a da sua conta do Azure.

1. No portal do Azure, procure *grupos de recursos* na caixa de pesquisa na parte superior e selecione **Grupos de recursos** na lista de opções.

1. Na página **Grupo de recursos**, se fornecida, selecione o grupo de recursos que começa com *contoso-rg*. Se esse grupo de recursos não existir, crie um novo grupo de recursos chamado *contoso-rg* em sua região local ou use um grupo de recursos existente e anote a região em que ele está.

## Criar uma rede virtual

1. Na página inicial do portal do Azure, selecione o menu à esquerda.  

1. No painel de navegação esquerdo, clique em **Redes Virtuais**  

1. Clique em **+ Criar** para abrir a página **Criar Rede Virtual**. Na guia **Básico**, complete as seguintes informações:

    - **Assinatura**: &lt;Sua assinatura&gt;
    - **Grupo de recursos**: começando com *DP300* ou o grupo de recurso que você selecionou anteriormente
    - **Nome:** lab02-vnet
    - **Região:** selecione a mesma região em que o seu grupo de recursos foi criado

1. Clique em **Revisar + Criar**, revise as definições da nova rede virtual e, em seguida, clique em **Criar**.

## Provisionar um Banco de Dados SQL do Azure no portal do Azure

1. No Portal do Azure, procure por *bancos de dados SQL* na caixa de pesquisa na parte superior e clique em **Bancos de dados SQL** na lista de opções.

1. No painel **Bancos de dados SQL**, selecione **+ Criar**.

1. Na página **Criar Banco de Dados SQL**, selecione as seguintes opções na guia **Básico** e clique em **Avançar: Rede**.

    - **Assinatura**: &lt;Sua assinatura&gt;
    - **Grupo de recursos**: começando com *DP300* ou o grupo de recurso que você selecionou anteriormente
    - **Nome do banco de dados:** AdventureWorksLT
    - **Servidor:** selecione no link **Criar novo**. A página **Criar Servidor de Banco de Dados SQL** será aberta. Forneça os detalhes do servidor da seguinte maneira:
        - **Nome do servidor:** dp300-lab-&lt;suaus iniciais (minúsculas)&gt; e, se necessário, um número aleatório de cinco dígitos (o nome do servidor deve ser globalmente exclusivo)
        - **Local:**&lt;sua região local, igual à região selecionada para seu grupo de recursos, caso contrário, ela pode falhar&gt;
        - **Método de autenticação**: use a autenticação do SQL.
        - **Login do administrador do servidor:** dp300admin
        - **Senha:** selecione uma senha complexa e anote-a
        - **Confirme a senha:** selecione a mesma senha selecionada anteriormente
    - Clique em **OK** para retornar à página **Criar Banco de Dados SQL**.
    - **Deseja usar o pool elástico do SQL** definido como **Não**.
    - **Ambiente de carga de trabalho**: desenvolvimento
    - Na opção **Computação + Armazenamento**, clique no link **Configurar banco de dados**. Na página **Configurar**, para a lista suspensa **Camada de serviço**, selecione **Básico** e **Aplicar**.

1. Na opção **Redundância de armazenamento de backup**, mantenha o valor padrão: **Armazenamento de backup com redundância local**.

1. Em seguida, clique em **Avançar: Rede**.

1. Na guia **Rede**, na opção **Conectividade de Rede**, clique no botão de opção **Ponto de extremidade privado**.

1. Em seguida, clique no link **+ Adicionar ponto de extremidade privado** na opção **Pontos de extremidade privados**.

1. Conclua o painel direito **Criar ponto de extremidade privado** da seguinte maneira:

    - **Assinatura**: &lt;Sua assinatura&gt;
    - **Grupo de recursos**: começando com *DP300* ou o grupo de recurso que você selecionou anteriormente
    - **Local:**&lt;sua região local, igual à região selecionada para seu grupo de recursos, caso contrário, ela pode falhar&gt;
    - **Nome:** DP-300-SQL-Endpoint
    - **Sub-recurso de destino:** SqlServer
    - **Rede virtual:** lab02-vnet
    - **Sub-rede:** lab02-vnet/default (10.x.0.0/24)
    - **Integrar com a zona DNS privada**: sim
    - **Zona DNS privada:** mantenha o valor padrão
    - Revise as configurações e selecione **OK**.  

1. O novo ponto de extremidade aparecerá na lista **Pontos de extremidade privados**.

1. Clique em **Avançar: Segurança** e, em seguida, em **Avançar: Configurações adicionais**.  

1. Na página **Configurações adicionais**, selecione **Exemplo** na opção **Usar dados existentes**. Selecione **OK** se uma mensagem pop-up for exibida para o banco de dados de exemplo.

1. Selecione **Examinar + criar**.

1. Revise as configurações antes de selecionar **Criar**.

1. Após a conclusão da implantação, selecione **Ir para o recurso**.

## Habilite o acesso a um Banco de dados SQL do Azure.

1. Na página **Banco de dados SQL**, selecione a seção **Visão geral** e, em seguida, selecione o link do nome do servidor na seção superior.

1. Na folha de navegação SQL servers, selecione **Rede** na seção **Segurança**.

1. Na guia **Acesso público**, escolha **Redes selecionadas**.

1. Selecione **+ Adicionar endereço IPv4 do cliente**. Isso adicionará uma regra de firewall para permitir que seu endereço IP atual acesse o SQL Server.

1. Marque a propriedade **Permitir que os serviços e os recursos do Azure acessem este servidor**.

1. Selecione **Salvar**.

---

## Conectar-se ao Banco de Dados SQL no SQL Server Management Studio

1. No portal do Azure, selecione **Bancos de dados SQL** no painel de navegação esquerdo. Em seguida, selecione o banco de dados **AdventureWorksLT**.

1. Na página **Visão geral**, copie o valor **Nome do servidor**.

1. Inicie o SQL Server Management Studio na máquina virtual do laboratório, se fornecida, ou em seu computador local, se não foi fornecida.

1. Na caixa de diálogo **Conectar ao servidor**, cole o valor **Nome do servidor** copiado do portal do Azure.

1. Na lista suspensa **Autenticação**, selecione **Autenticação do SQL Server**.

1. No campo **Login**, insira **dp300admin**.

1. No campo **Senha**, insira a senha selecionada durante a criação do SQL Server.

1. Selecione **Conectar**.

1. O SQL Server Management Studio se conectará ao servidor do Banco de dados SQL do Azure. Você pode expandir o servidor e, em seguida, o nó **Bancos de Dados** para ver o banco de dados *AdventureWorksLT*.

## Consultar um Banco de Dados SQL do Azure com o SQL Server Management Studio

1. No SQL Server Management Studio, clique com o botão direito do mouse no banco de dados *AdventureWorksLT* e selecione **Nova Consulta**.

1. Cole a seguinte instrução SQL na janela Consulta:

    ```sql
    SELECT TOP 10 cust.[CustomerID], 
        cust.[CompanyName], 
        SUM(sohead.[SubTotal]) as OverallOrderSubTotal
    FROM [SalesLT].[Customer] cust
        INNER JOIN [SalesLT].[SalesOrderHeader] sohead
             ON sohead.[CustomerID] = cust.[CustomerID]
    GROUP BY cust.[CustomerID], cust.[CompanyName]
    ORDER BY [OverallOrderSubTotal] DESC
    ```

1. Clique no botão **Executar** na barra de ferramentas para executar a consulta.

1. Revise os resultados da consulta no painel **Resultados**.

1. Clique com o botão direito do mouse no banco de dados *AdventureWorksLT* e selecione **Nova Consulta**.

1. Cole a seguinte instrução SQL na janela Consulta:

    ```sql
    SELECT TOP 10 cat.[Name] AS ProductCategory, 
        SUM(detail.[OrderQty]) AS OrderedQuantity
    FROM salesLT.[ProductCategory] cat
        INNER JOIN [SalesLT].[Product] prod
            ON prod.[ProductCategoryID] = cat.[ProductCategoryID]
        INNER JOIN [SalesLT].[SalesOrderDetail] detail
            ON detail.[ProductID] = prod.[ProductID]
    GROUP BY cat.[name]
    ORDER BY [OrderedQuantity] DESC
    ```

1. Clique no botão **Executar** na barra de ferramentas para executar a consulta.

1. Revise os resultados da consulta no painel **Resultados**.

1. Feche o SQL Server Management Studio. Quando solicitado a salvar as alterações, selecione **Não**.

---

## Recursos de limpeza

Se você não estiver usando a máquina virtual para nenhuma outra finalidade, poderá limpar os recursos criados neste laboratório.

### Excluir o Grupo de Recursos

Se você criou um novo grupo de recursos para este laboratório, poderá excluir o grupo de recursos para remover todos os recursos criados neste laboratório.

1. No portal do Azure, selecione **Grupos de recursos** no painel de navegação esquerdo ou pesquise **Grupos de recursos** na barra de pesquisa e selecione-o nos resultados.

1. Vá para o grupo de recursos criado para o laboratório. O grupo de recursos conterá a máquina virtual e outros recursos criados neste laboratório.

1. Selecione **Excluir grupo de recursos** no menu superior.

1. Na caixa de diálogo **Excluir grupo de recursos**, digite o nome do grupo para confirmar e clique em **Excluir**.

1. Aguarde até que o grupo de recursos seja excluído.

1. Feche o portal do Azure.

### Excluir apenas os recursos do laboratório

Se você não criou um novo grupo de recursos para este laboratório e deseja deixar o grupo de recursos e seus recursos anteriores intactos, ainda poderá excluir os recursos criados neste laboratório.

1. No portal do Azure, selecione **Grupos de recursos** no painel de navegação esquerdo ou pesquise **Grupos de recursos** na barra de pesquisa e selecione-o nos resultados.

1. Vá para o grupo de recursos criado para o laboratório. O grupo de recursos conterá a máquina virtual e outros recursos criados neste laboratório.

1. Selecione todos os recursos prefixados com o nome do SQL Server especificado anteriormente no laboratório. Além disso, selecione a rede virtual e a zona DNS privada que você criou.

1. Selecione **Excluir** no menu superior.

1. Na caixa de diálogo **Excluir recursos**, digite **excluir** e selecione **Excluir**.

1. Para confirmar a exclusão dos recursos, clique em excluir novamente **Excluir**.

1. Aguarde a exclusão dos recursos.

1. Feche o portal do Azure.

---

Você concluiu este laboratório.

Neste exercício, você viu como implantar um Banco de Dados SQL do Azure com um Ponto de Extremidade de Rede Virtual. Você também foi capaz de conectar-se ao Banco de Dados SQL criado usando o SQL Server Management Studio.
