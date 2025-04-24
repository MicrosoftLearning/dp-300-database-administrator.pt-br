---
lab:
  title: Laboratório 5 – Habilitar o Microsoft Defender para SQL e a Classificação de Dados
  module: Implement a Secure Environment for a Database Service
---

# Habilitar o Microsoft Defender para SQL e a Classificação de Dados

**Tempo estimado**: 30 minutos

Os alunos usarão as informações obtidas nas lições para configurar e, posteriormente, implementar a segurança no Portal do Azure e no banco de dados do AdventureWorks.

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

## Habilitar o Microsoft Defender para SQL

1. Na máquina virtual do laboratório, ou em seu computador local, se não tiver sido fornecida, inicie uma sessão do navegador e navegue até [https://portal.azure.com](https://portal.azure.com/). Conecte-se ao Portal usando as suas credenciais do Azure.

1. No Portal do Azure, pesquise por *servidores SQL* na caixa de pesquisa na parte superior e clique em **servidores SQL** na lista de opções.

1. Selecione o SQL Server **dp300-lab-xxxxxxxx**, em que *xxxxxxxx* é uma cadeia de caracteres numérica aleatória.

    > &#128221; Observe que, se você estiver usando seu próprio servidor SQL do Azure não criado por este laboratório, selecione o nome desse SQL Server.

1. Na folha *Visão geral*, selecione **Não configurado** ao lado de *Microsoft Defender para SQL*.

1. Clique no **X** no canto superior direito para fechar o painel Visão geral do *Microsoft Defender para Nuvem*.

1. Selecione **Habilitar** em *Microsoft Defender para SQL*.

1. Em um ambiente de produção, haverá várias recomendações listadas. Você deve selecionar **Exibir todas as recomendações no Defender para Nuvem** e examinar todas as recomendações do *Microsoft Defender* listadas para o servidor SQL do Azure e implementá-las conforme apropriado.

## Avaliação de Vulnerabilidade

1. Na folha principal do servidor SQL do Azure, navegue até a seção **Configurações** e selecione **Bancos de dados SQL** e selecione o nome do banco de dados chamado **AdventureWorksLT**.

1. Selecione a configuração **Microsoft Defender para Nuvem** em **Segurança**.

1. Clique no **X** no canto superior direito para fechar o painel Visão Geral do *Microsoft Defender para Nuvem* e exibir o painel do **Microsoft Defender para Nuvem** do banco de dados `AdventureWorksLT`.

1. Para começar a examinar as funcionalidades de Avaliação de Vulnerabilidade, em **Descobertas da avaliação de vulnerabilidade**, selecione **Exibir descobertas adicionais na Avaliação de Vulnerabilidade**.  

1. Selecione **Examinar** para obter os resultados mais atuais da Avaliação de Vulnerabilidade. Esse processo leva alguns minutos, enquanto a Avaliação de Vulnerabilidade examina o banco de dados.

1. Cada risco de segurança tem um nível de risco (alto, médio ou baixo) e informações adicionais. As regras em vigor se baseiam em parâmetros de comparação fornecidos pelo [Centro de Segurança da Internet](https://www.cisecurity.org/benchmark/microsoft_sql_server/?azure-portal=true). Na guia **Descobertas**, selecione uma vulnerabilidade. Anote o **ID** da vulnerabilidade, por exemplo, **VA1143** (se listado).

1. Dependendo da verificação de segurança, haverá exibições e recomendações alternativas. Examine as informações fornecidas. Para essa verificação de segurança, você pode selecionar o botão **Adicionar todos os resultados como linha de base** e, em seguida, selecionar **Sim** para definir a linha de base. Agora que uma linha de base está em vigor, essa verificação de segurança falhará em qualquer verificação futura em que os resultados são diferentes da linha de base. Selecione o **X** no canto superior direito para fechar o painel da regra específica.  

1. Vamos executar a **Varredura** novamente para confirmar se a vulnerabilidade selecionada agora está aparecendo como uma verificação de segurança *Aprovada*.

    Se selecionar a verificação de segurança aprovada anterior, você deverá ver a linha de base configurada. Se algo mudar no futuro, os exames de Avaliação de Vulnerabilidade detectarão essa mudança e a verificação de segurança falhará.  

## Proteção Avançada contra Ameaças

1. Selecione o **X** no canto superior direito para fechar o painel de Avaliação de Vulnerabilidade e retornar ao painel **Microsoft Defender para Nuvem** para o seu banco de dados. Em **Alertas de segurança e incidentes**, você não deve ver nenhum item. Isso significa que a **Proteção Avançada contra Ameaças** não detectou nenhum problema. A Proteção Avançada contra Ameaças detecta atividades anômalas que indicam tentativas incomuns e potencialmente prejudiciais de acessar ou explorar os bancos de dados.  

    > &#128221; Provavelmente, você não verá nenhum alerta de segurança nesta fase. Na próxima etapa, você executará um teste que vai disparar um alerta, para que você possa examinar os resultados na Proteção Avançada contra Ameaças.  

    É possível usar a Proteção Avançada contra Ameaças para identificar ameaças e alertar você quando suspeitar que ocorram eventos como os seguintes:  

    - Injeção de SQL
    - Vulnerabilidade de injeção de SQL
    - Exfiltração dos dados
    - Ação não segura
    - Força bruta
    - Logon de cliente anômalo

    Nesta seção, você aprenderá como um alerta de Injeção de SQL pode ser disparado por meio do SSMS. Os alertas de Injeção de SQL destinam-se a aplicativos gravados de modo personalizado e não a ferramentas padrão como o SSMS. Portanto, para disparar um alerta por meio do SSMS como um teste para uma Injeção de SQL, você precisa definir o **Nome do Aplicativo**, que é uma propriedade de conexão para clientes que se conectam ao SQL Server ou ao SQL do Azure.

1. Na máquina virtual do laboratório ou em seu computador local, se não tiver sido fornecida, abra o SQL Server Management Studio (SSMS). Na caixa de diálogo Conectar ao Servidor, cole o nome do seu servidor do Banco de Dados SQL do Azure e entre com as seguintes credenciais:

    - **Nome do servidor:** &lt;_cole o nome do servidor do Banco de Dados SQL do Azure aqui_&gt;
    - **Autenticação:** Autenticação do SQL Server
    - **Logon de administrador do servidor:** seu logon de administrador do servidor do Banco de Dados SQL do Azure
    - **Senha:** sua senha de administrador do servidor do Banco de Dados SQL do Azure

1. Selecione **Conectar**.

1. No SSMS, selecione **Arquivo** > **Novo** > **Consulta do Mecanismo de Banco de Dados** para criar uma consulta usando uma nova conexão.  

1. Na janela de logon principal, entre no banco de dados **AdventureWorksLT** como faria normalmente, com a autenticação SQL e seu nome do servidor SQL do Azure e credenciais de administrador. Antes de se conectar, selecione **Opções >>** > **Propriedades de Conexão**. Digite **AdventureWorksLT** na opção **Conectar ao banco de dados**.  

1. Selecione a guia **Parâmetros de Conexão Adicionais** e insira a seguinte cadeia de conexão na caixa de texto:  

    ```sql
    Application Name=webappname
    ```

1. Selecione **Conectar**.  

1. Na nova janela de consulta, cole a seguinte consulta e selecione **Executar**:  

    ```sql
    SELECT * FROM sys.databases WHERE database_id like '' or 1 = 1 --' and family = 'test1';
    ```

1. No portal do Azure, acesse o banco de dados **AdventureWorksLT**. No painel esquerdo, em **Segurança**, selecione **Microsoft Defender para Nuvem**.

1. Em **Alertas e incidentes de segurança**, selecione **Verificar alertas nestes recursos no Microsoft Defender para Nuvem**.  

1. Agora você pode ver os alertas de segurança gerais.  

1. Selecione **Possível injeção de SQL** para exibir alertas mais específicos e receber as etapas de investigação.

1. Selecione **Exibir detalhes completos** para exibir os detalhes do alerta.

1. Na guia **Detalhes do alerta**, observe que a *instrução Vulnerable* é exibida. Essa é a instrução SQL que foi executada para disparar o alerta. Essa também foi a instrução SQL executada no SSMS. Além disso, observe que o **Aplicativo cliente** é mostrado como **webappname**. Esse é o nome que você especificou na cadeia de conexão no SSMS.

1. Como uma etapa de limpeza, considere fechar todos os editores de consulta no SSMS e remover todas as conexões para que você não dispare alertas adicionais acidentalmente nos próximos exercícios.

## Habilitar Classificação de Dados

1. Na folha principal do servidor SQL do Azure, navegue até a seção **Configurações** e selecione **Bancos de dados SQL** e selecione o nome do banco de dados chamado **AdventureWorksLT**.

1. Na folha principal do banco de dados **AdventureWorksLT**, navegue até a seção **Segurança** e selecione **Descoberta e Classificação de Dados**.

1. Na página **Descoberta e Classificação de Dados**, você verá a seguinte mensagem informativa: **Atualmente usando a política de Proteção de Informações do SQL. Encontramos 15 colunas com recomendações de classificação**. Selecione esse link.

1. Na tela **Descoberta e Classificação de Dados**, marque a caixa de seleção ao lado de **Selecionar tudo**, selecione **Recomendações selecionadas aceitas** e **Salvar** para salvar as classificações no banco de dados.

1. De volta à tela **Descoberta e Classificação de Dados**, observe que quinze colunas foram classificadas com sucesso em cinco tabelas diferentes. Revise o *Tipo de informação* e o *Rótulo de confidencialidade* de cada uma das colunas.

## Configurar o mascaramento e a classificação de dados

1. No portal do Azure, acesse sua instância do Banco de Dados SQL do Azure **AdventureWorksLT** (não seu servidor lógico).

1. No painel à esquerda, em **Segurança**, selecione **Descoberta e Classificação de Dados**.  

1. Na tabela Cliente SalesLT, a *Descoberta e Classificação de Dados* identificou `FirstName` e `LastName` como classificados, mas não `MiddleName`. Use as listas suspensas para adicioná-lo agora. Selecione **Nome** em *Tipo de informação* e **Confidencial — GDPR** em *Rótulo de confidencialidade* e, em seguida, selecione **Adicionar classificação**.  

1. Selecione **Salvar**.

1. Confirme se a adição da classificação foi bem-sucedida exibindo a guia **Visão geral** e confirmando se agora `MiddleName` é exibido na lista de colunas classificadas no esquema SalesLT.

1. No painel à esquerda, selecione **Visão geral** para voltar para a visão geral do seu banco de dados.  

   A DDM (Máscara Dinâmica de Dados) está disponível tanto no SQL do Azure quanto no SQL Server. A DDM limita a exposição de dados mascarando dados confidenciais para usuários sem privilégios no nível do SQL Server, em vez de no nível do aplicativo, no qual você precisa codificar esses tipos de regras. O SQL do Azure recomenda itens a serem mascarados ou você pode adicionar as máscaras manualmente.

   Nas próximas etapas, serão mascaradas as colunas `FirstName`, `MiddleName` e `LastName` que foram examinadas na etapa anterior.  

1. No portal do Azure, acesse o Banco de Dados SQL do Azure. No painel esquerdo, em **Segurança**, selecione **Máscara Dinâmica de Dados** e, em seguida, selecione **Adicionar máscara**.  

1. Nas listas suspensas, selecione o esquema **SalesLT**, a tabela **Cliente** e a coluna **FirstName**. Você pode examinar as opções de mascaramento, mas a opção padrão pode ser usada nesse cenário. Selecione **Adicionar** para adicionar a regra de mascaramento.  

1. Repita a etapa anterior para **MiddleName** e **LastName** nessa tabela.  

    Agora você tem três regras de mascaramento.  

1. Selecione **Salvar**.

    > &#128221; Observe que, se o nome do servidor SQL do Azure não for composto apenas por letras minúsculas, números e traços, essa etapa falhará e você não poderá continuar com as seções de mascaramento de dados.

1. No painel à esquerda, selecione **Visão geral** para voltar para a visão geral do seu banco de dados.

## Recuperar dados que são confidenciais e mascarados

Em seguida, você simulará alguém consultando as colunas classificadas e explorará a Máscara Dinâmica de Dados em ação.

1. Acesse o SQL Server Management Studio (SSMS), conecte-se ao seu servidor SQL do Azure e abra uma nova janela de consulta.

1. Clique com o botão direito do mouse no banco de dados **AdventureWorksLT** e selecione **Nova Consulta**.  

1. Execute a consulta a seguir para retornar os dados confidenciais e, em alguns casos, as colunas marcados para mascaramento de dados. Selecione **Executar** para executar a consulta.

    ```sql
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    ```

    O seu resultado deve exibir os primeiros dez nomes, sem nenhuma máscara aplicada. Por quê? Porque você é o administrador deste servidor lógico do Banco de Dados SQL do Azure.  

1. Na consulta a seguir, você criará um novo usuário e executará a consulta anterior como esse usuário. Você também usará `EXECUTE AS` para representar `Bob`. Quando uma instrução `EXECUTE AS` é executada, o contexto de execução da sessão é alternado para o logon ou o usuário. Isso significa que as permissões são verificadas com relação ao logon ou usuário em vez da pessoa que está executando o comando `EXECUTE AS` (nesse caso, você). O `REVERT` é usado para interromper a representação do logon ou usuário.  

    Você pode reconhecer as primeiras partes dos comandos a seguir, pois eles são uma repetição de um exercício anterior. Crie uma nova consulta com os comandos a seguir e selecione **Executar** para executar a consulta e observar os resultados.

    ```sql
    -- Create a new SQL user and give them a password
    CREATE USER Bob WITH PASSWORD = 'c0mpl3xPassword!';

    -- Until you run the following two lines, Bob has no access to read or write data
    ALTER ROLE db_datareader ADD MEMBER Bob;
    ALTER ROLE db_datawriter ADD MEMBER Bob;

    -- Execute as our new, low-privilege user, Bob
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;
    ```

    Agora, o resultado deve exibir os primeiros dez nomes, mas com a máscara aplicada. Pedro não recebeu acesso ao formato sem máscara desses dados.  

    E se, por algum motivo, Pedro precisar de acesso aos nomes e obter permissão para tê-los?  

    Você pode atualizar os usuários excluídos do mascaramento no portal do Azure acessando o painel **Máscara Dinâmica de Dados** em **Segurança**, mas você também pode fazer isso usando o T-SQL.

1. Clique com o botão direito do mouse no banco de dados **AdventureWorksLT** e selecione **Nova Consulta**; em seguida, insira a consulta a seguir para permitir que Bob consulte os resultados dos nomes sem mascaramento. Selecione **Executar** para executar a consulta.

    ```sql
    GRANT UNMASK TO Bob;  
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;  
    ```

    Os resultados devem incluir os nomes completos.  

1. Você também pode tirar os privilégios de desmascaramento de um usuário e confirmar essa ação executando os seguintes comandos T-SQL em uma nova consulta:  

    ```sql
    -- Remove unmasking privilege
    REVOKE UNMASK TO Bob;  

    -- Execute as Bob
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;  
    ```

    Os resultados devem incluir os nomes mascarados.  

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

Neste exercício, você aprimorou a segurança de um Banco de Dados SQL do Azure habilitando o Microsoft Defender para SQL. Você também criou colunas classificadas com base nas recomendações do portal do Azure.
