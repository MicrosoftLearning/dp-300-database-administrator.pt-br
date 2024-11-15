---
lab:
  title: Laboratório 2 – Provisionar um Banco de Dados SQL do Azure
  module: Plan and Implement Data Platform Resources
---

# Provisionar um Banco de Dados SQL do Azure

**Tempo estimado: 40 minutos**

Os alunos configurarão os recursos básicos necessários para implantar um Banco de Dados SQL do Azure com um Ponto de Extremidade de Rede Virtual. A conectividade com o Banco de Dados SQL será validada usando o Azure Data Studio da VM de laboratório.

Como administrador de banco de dados do AdventureWorks, você configurará um novo Banco de Dados SQL, incluindo um Ponto de Extremidade de Rede Virtual para aumentar e simplificar a segurança da implantação. O Azure Data Studio será usado a fim de avaliar o uso de um Notebook SQL para consulta de dados e retenção dos resultados.

## Navegar no portal do Azure

1. Na máquina virtual do laboratório, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando o **Nome de Usuário** e a **Senha** do Azure fornecidos na guia **Recursos** dessa máquina virtual do laboratório.

    ![Figura 1](../images/dp-300-module-01-lab-01.png)

1. No portal do Azure, procure "grupos de recursos" na caixa de pesquisa na parte superior e selecione **Grupos de recursos** na lista de opções.

    ![Figura 1](../images/dp-300-module-02-lab-45.png)

1. Na página **Grupo de recursos**, verifique o grupo de recursos listado (ele deve começar com *contoso-rg*), anote a **Localização** atribuída ao seu grupo de recursos, pois você a usará no próximo exercício.

    **Observação:** você pode ter um local diferente atribuído.

    ![Figura 1](../images/dp-300-module-02-lab-46.png)

## Criar uma rede virtual

1. Na página inicial do portal do Azure, selecione o menu à esquerda.  

    ![Imagem 2](../images/dp-300-module-02-lab-01_1.png)

1. No painel de navegação esquerdo, clique em **Redes Virtuais**  

1. Clique em **+ Criar** para abrir a página **Criar Rede Virtual**. Na guia **Básico**, complete as seguintes informações:

    - **Assinatura**: &lt;Sua assinatura&gt;
    - **Grupo de recursos:** começando com *contoso-rg*
    - **Nome:** lab02-vnet
    - **Região:** selecione a mesma região em que o seu grupo de recursos foi criado

1. Clique em **Revisar + Criar**, reveja as definições da nova rede virtual e, em seguida, clique em **Criar**.

1. Configure o intervalo de IP da rede virtual para o ponto de extremidade do banco de dados SQL do Azure navegando até a rede virtual criada e, no painel **Configurações**, clique em **Sub-redes**.

1. Clique no link de sub-rede **padrão**. Observe que o **intervalo de endereços de sub-rede** que você vê pode ser diferente.

1. No painel **Editar sub-rede** à direita, expanda a lista suspensa **Serviços** e marque **Microsoft.Sql**. Selecione **Salvar**.

## Provisionar um Banco de Dados SQL do Azure

1. No Portal do Azure, procure por "bancos de dados SQL" na caixa de pesquisa na parte superior e clique em **Bancos de dados SQL** na lista de opções.

    ![Imagem 5](../images/dp-300-module-02-lab-10.png)

1. No painel **Bancos de dados SQL**, selecione **+ Criar**.

    ![Imagem 6](../images/dp-300-module-02-lab-10_1.png)

1. Na página **Criar Banco de Dados SQL**, selecione as seguintes opções na guia **Básico** e clique em **Avançar: Rede**.

    - **Assinatura**: &lt;Sua assinatura&gt;
    - **Grupo de recursos:** começando com *contoso-rg*
    - **Nome do banco de dados:** AdventureWorksLT
    - **Servidor:** clique no link **Criar novo**. A página **Criar Servidor de Banco de Dados SQL** será aberta. Forneça os detalhes do servidor da seguinte maneira:
        - **Nome do servidor:** dp300-lab-&lt;suas iniciais (minúsculas)&gt; (o nome do servidor deve ser exclusivo globalmente)
        - **Local:**&lt;sua região local, igual à região selecionada para seu grupo de recursos, caso contrário, ela pode falhar&gt;
        - **Método de autenticação**: use a autenticação do SQL.
        - **Login do administrador do servidor:** dp300admin
        - **Senha:**dp300P@ssword!
        - **Confirmar senha:**dp300P@ssword!

        Sua página **Criar Servidor do Banco de Dados SQL** deve ser semelhante à página abaixo. Em seguida, clique em **OK**.

        ![Imagem 7](../images/dp-300-module-02-lab-11.png)

    -  De volta à página **Criar Banco de Dados SQL**, verifique se **Deseja usar o Pool Elástico?** está definido como **Não**.
    -  Na opção **Computação + armazenamento**, clique no link **Configurar banco de dados**. Na página **Configurar**, para a lista suspensa **Camada de serviço**, selecione **Básico** e **Aplicar**.

    **Observação:** anote este nome de servidor e suas informações de logon. Você os usará nos laboratórios seguintes.

1. Para a opção **Redundância de armazenamento de backup**, mantenha o valor padrão: **armazenamento de backup com redundância geográfica**.

1. E clique em **Avançar: rede**.

1. Na guia **Rede**, na opção **Conectividade de Rede**, clique no botão de opção **Ponto de extremidade privado**.

    ![Imagem 8](../images/dp-300-module-02-lab-14.png)

1. Em seguida, selecione o link **+ Adicionar ponto de extremidade privado** na opção **Pontos de extremidade privados**.

    ![Imagem 9](../images/dp-300-module-02-lab-15.png)

1. Conclua o painel direito **Criar ponto de extremidade privado** da seguinte maneira:

    - **Assinatura**: &lt;Sua assinatura&gt;
    - **Grupo de recursos:** começando com *contoso-rg*
    - **Local:**&lt;sua região local, igual à região selecionada para seu grupo de recursos, caso contrário, ela pode falhar&gt;
    - **Nome:** DP-300-SQL-Endpoint
    - **Sub-recurso de destino:** SqlServer
    - **Rede virtual:** lab02-vnet
    - **Sub-rede:** lab02-vnet/default (10.x.0.0/24)
    - **Integrar com a zona DNS privada**: sim
    - **Zona DNS privada:** mantenha o valor padrão
    - Examine as configurações e clique em **OK**  

    ![Figura 10](../images/dp-300-module-02-lab-16.png)

1. O novo ponto de extremidade aparecerá na lista **Pontos de extremidade privados**.

    ![Figura 11](../images/dp-300-module-02-lab-17.png)

1. Clique em **Avançar: Segurança** e, em seguida, em **Avançar: Configurações adicionais**.  

1. Na página **Configurações adicionais**, selecione **Exemplo** na opção **Usar dados existentes**. Selecione **OK** se uma mensagem pop-up for exibida para o banco de dados de exemplo.

    ![Figura 12](../images/dp-300-module-02-lab-18.png)

1. Clique em **Examinar + Criar**.

1. Examine as configurações antes de clicar em **Criar**.

1. Quando a implantação estiver concluída, clique em **Ir para o recurso**.

## Habilite o acesso a um Banco de dados SQL do Azure.

1. Na página **Banco de dados SQL**, selecione a seção **Visão geral** e, em seguida, selecione o link para o nome do servidor na seção superior:

    ![Figura 13](../images/dp-300-module-02-lab-19.png)

1. Na folha de navegação SQL servers, selecione **Rede** na seção **Segurança**.

    ![Figura 14](../images/dp-300-module-02-lab-20.png)

1. Na guia **Acesso público**, selecione **Redes selecionadas** e marque a propriedade **Permitir que os serviços e recursos do Azure acessem este servidor**. Clique em **Save** (Salvar).

    ![Figura 15](../images/dp-300-module-02-lab-21.png)

## Conectar-se a um Banco de Dados SQL do Azure no Azure Data Studio

1. Inicie o Azure Data Studio na máquina virtual de laboratório.

    - Você pode ver esse pop-up na primeira inicialização do Azure Data Studio. Se ele for aberto, clique em **Sim (recomendado)**  

        ![Figura 16](../images/dp-300-module-02-lab-22.png)

1. Quando o Azure Data Studio for aberto, clique no botão **Conexões** no canto superior esquerdo e em **Adicionar Conexão**.

    ![Figura 17](../images/dp-300-module-02-lab-25.png)

1. Na barra lateral **Conexão**, preencha a seção **Detalhes da Conexão** com as informações para a conexão ao banco de dados SQL criado anteriormente.

    - Tipo de Conexão: **Microsoft SQL Server**
    - Servidor: insira o nome do SQL Server criado anteriormente. Por exemplo: **dp300-lab-xxxxxxxx.database.windows.net** (onde 'xxxxxxxx' é um número aleatório)
    - Tipo de Autenticação: **Logon do SQL**
    - Nome de usuário: **dp300admin**
    - Senha: **dp300P@ssword!**
    - Expanda a lista suspensa Banco de dados para selecionar **AdventureWorksLT.** 
        - **OBSERVAÇÃO:** você pode receber uma solicitação para adicionar uma regra de firewall que permite o acesso do seu IP do cliente a este servidor. Se receber a solicitação para adicionar uma regra de firewall, clique em **Adicionar conta** e faça logon na sua conta do Azure. Na tela **Criar nova regra de firewall**, clique em **OK**.

        ![Figura 18](../images/dp-300-module-02-lab-26.png)

        Como alternativa, você pode criar manualmente uma regra de firewall para seu SQL Server no portal do Azure navegando até o SQL Server, selecionando **Rede** e escolhendo **+ Adicionar o endereço IPv4 do cliente (seu endereço IP)**

        ![Figura 18](../images/dp-300-module-02-lab-47.png)

    De volta à barra lateral Conexão, continue preenchendo os detalhes da conexão:  

    - O grupo de servidores permanecerá como **&lt;padrão&gt;**
    - O Nome (opcional) pode ser preenchido com um nome amigável do banco de dados, se desejado
    - Revise as configurações e clique em **Conectar**  

    ![Figura 19](../images/dp-300-module-02-lab-27.png)

1. O Azure Data Studio se conectará ao banco de dados e mostrará algumas informações básicas sobre o banco, incluindo uma lista parcial de objetos.

    ![Figura 20](../images/dp-300-module-02-lab-28.png)

## Consultar um Banco de Dados SQL do Azure com um Notebook SQL

1. No Azure Data Studio, conectado ao banco de dados AdventureWorksLT deste laboratório, clique no botão **Novo Notebook**.

    ![Figura 21](../images/dp-300-module-02-lab-29.png)

1. Clique no link **+Texto** para adicionar uma nova caixa de texto no notebook  

    ![Figura 22](../images/dp-300-module-02-lab-30.png)

**Observação:** no notebook, você pode inserir texto sem formatação para explicar consultas ou conjuntos de resultados.

1. Insira o texto **Dez principais clientes por subtotal de pedidos**, em negrito, se desejar.

    ![Uma captura de tela de um celular

Descrição gerada automaticamente](../images/dp-300-module-02-lab-31.png)

1. Clique no botão **+ Célula** e em seguida**Célula de código** para adicionar uma nova célula de código no fim do notebook.  

    ![Figura 23](../images/dp-300-module-02-lab-32.png)

5. Cole a instrução SQL a seguir na nova célula:

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

1. Clique no círculo azul com a seta para executar a consulta. Observe como os resultados são incluídos na célula com a consulta.

1. Clique no botão **+ Texto** para adicionar uma nova célula de texto.

1. Insira o texto **Dez principais categorias de produto ordenadas**, em negrito, se desejar.

1. Clique em **+ Código** novamente para adicionar uma nova célula e cole a instrução SQL a seguir na célula:

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

1. Clique no círculo azul com a seta para executar a consulta.

1. Para executar todas as células no notebook e apresentar os resultados, clique no botão **Executar todas** na barra de ferramentas.

    ![Figura 17](../images/dp-300-module-02-lab-33.png)

1. No Azure Data Studio, salve o notebook a partir do menu Arquivo (Salvar ou Salvar como) no caminho **C:\Labfiles\Deploy Azure SQL Database** (crie a estrutura de pastas se ela não existir). Certifique-se de que a extensão do arquivo é **.ipynb**

1. Feche a guia do Notebook dentro do Azure Data Studio. No menu Arquivo, selecione Abrir Arquivo e abra o notebook que você acabou de salvar. Observe que os resultados da consulta foram salvos com as consultas no notebook.

Neste exercício, você viu como implantar um Banco de Dados SQL do Azure com um Ponto de Extremidade de Rede Virtual. Você também foi capaz de conectar-se ao Banco de Dados SQL criado usando o SQL Server Management Studio.
