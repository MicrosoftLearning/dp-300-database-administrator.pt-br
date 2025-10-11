---
lab:
  title: Laboratório 15 – fazer backup para URL e restaurar da URL
  module: Plan and implement a high availability and disaster recovery solution
---

# Backup para a URL

**Tempo estimado**: 30 minutos

Como DBA da AdventureWorks, você precisa fazer backup de um banco de dados para um URL no Azure e restaurá-lo do Armazeno de Blobs do Azure após a ocorrência de um erro humano.

## Ambiente de configuração

Se a máquina virtual do laboratório tiver sido fornecida e pré-configurada, você encontrará os arquivos de laboratório prontos na pasta **C:\LabFiles**. *Reserve um momento para verificar; se os arquivos já estiverem lá, pule esta seção*. No entanto, se você estiver usando sua própria máquina ou os arquivos de laboratório estiverem ausentes, será necessário cloná-los do *GitHub* para continuar.

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, inicie uma sessão do Visual Studio Code.

1. Abra a paleta de comandos; (Ctrl+Shift+P) e digite **Git: Clone**. Selecione a opção **Git: Clone**.

1. Cole a URL a seguir no campo **URL do repositório** e selecione **Enter**.

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. Salve o repositório na pasta **C:\LabFiles** na máquina virtual do laboratório ou em seu computador local, se não tiver sido fornecida (crie a pasta se ela não existir).

## Restaurar o banco de dados

Se você já tiver o banco de dados **AdventureWorks2017** restaurado, ignore esta seção.

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, inicie uma Sessão do SQL Server Management Studio (SSMS).

1. Quando o SSMS abrir, por padrão, a caixa de diálogo **Connect to Service** será exibida. Escolha Default instance e selecione **Connect**. Talvez seja necessário marcar a caixa de seleção **Trust server certificate**.

    > &#128221; Observe que, se você estiver usando sua própria instância do SQL Server, precisará se conectar a ela usando o nome e as credenciais apropriados da instância do servidor.

1. Selecione a pasta**Bancos de Dados** e **Nova Consulta**.

1. Na janela Nova consulta, copie e cole o T-SQL abaixo. Execute a consulta para restaurar o banco de dados.

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Shared\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\AdventureWorks2017_log.ldf';
    ```

    > &#128221; Você deve ter uma pasta chamada **C:\LabFiles**. Se você não tiver essa pasta, crie-a ou especifique outro local para o banco de dados e os arquivos de backup.

1. Na guia **Mensagens**, você verá uma mensagem indicando que o banco de dados foi restaurado.

## Configurar o backup da URL

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, inicie uma sessão do Visual Studio Code.

1. Abra o repositório clonado em **C:\LabFiles\dp-300-database-administrator**.

1. Clique com o botão direito do mouse na pasta **Allfiles** e selecione **Abrir no Terminal Integrado**. Abrirá uma janela de terminal no local correto.

1. No terminal, digite o seguinte comando e pressione **Enter**.

    ```bash
    az login
    ```

1. Será solicitado que você abra um navegador e insira um código exclusivo. Siga as instruções para entrar na sua conta do Azure.

1. *Se você já tiver um grupo de recursos, ignore esta etapa*. Se você não tiver um grupo de recursos, crie um usando o seguinte comando no terminal. Substitua *contoso-rgXXX######* por um nome exclusivo para o grupo de recursos. O nome deve ser exclusivo em todo o Azure. Substitua seu local (-l) pelo local do seu grupo de recursos.

    ```bash
    az group create -n "contoso-rglod#######" -l eastus2
    ```

    Substitua **######** por alguns caracteres aleatórios.

1. No terminal, digite o seguinte e pressione **Enter** para criar uma conta de armazenamento. Certifique-se de usar um nome exclusivo para a conta de armazenamento. *O nome deve ter entre 3 e 24 caracteres e pode conter apenas números e letras minúsculas.* Substitua *########* por oito caracteres numéricos aleatórios. O nome deve ser exclusivo em todo o Azure. Substitua contoso-rgXXX###### pelo nome do grupo de recursos. Por fim, substitua seu local (-l) pelo local do seu grupo de recursos.

    ```bash
    az storage account create -n "dp300bckupstrg########" -g "contoso-rgXXX########" --kind StorageV2 -l eastus2
    ```

1. Em seguida, você obterá as chaves da sua conta de armazenamento, que usará nas etapas seguintes. Execute o seguinte código no terminal usando o nome exclusivo da sua conta de armazenamento e grupo de recursos.

    ```bash
    az storage account keys list -g contoso-rgXXX######## -n dp300bckupstrg########
    ```

    Sua chave de conta estará nos resultados do comando acima. Use o mesmo nome (após o **-n**) e o grupo de recursos (após o **-g**) que você usou no comando anterior. Copie o valor retornado para **key1** (sem as aspas duplas).

1. O backup de um banco de dados no SQL Server para uma URL usa um contêiner em uma conta de armazenamento. Você criará um contêiner especificamente para armazenamento de backup nessa etapa. Para fazer isso, execute os comandos abaixo.

    ```bash
    az storage container create --name "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --fail-on-exist
    ```

    Em que **dp300bckupstrg########** é o nome exclusivo da conta de armazenamento usado ao criar essa conta e **storage_key** é a chave gerada anteriormente. A saída deve retornar **true**.

1. Para verificar se os backups de contêiner foram criados corretamente, execute:

    ```bash
    az storage container list --account-name "dp300bckupstrg########" --account-key "storage_key"
    ```

    Em que **dp300bckupstrg########** é o nome exclusivo da conta de armazenamento usado ao criar essa conta e **storage_key** é a chave gerada.

1. Uma SAS (assinatura de acesso compartilhado) no nível de contêiner é necessária para segurança. No terminal, execute o seguinte comando:

    ```bash
    az storage container generate-sas -n "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --permissions "rwdl" --expiry "date_in_the_future" -o tsv
    ```

    Em que **dp300bckupstrg########-** é o nome exclusivo da conta de armazenamento usado ao criar essa conta, **storage_key** é a chave gerada e **date_in_the_future** é um momento posterior ao atual. **date_in_the_future** deve estar em UTC. Um exemplo é **2025-12-31T00:00Z**, que significa a expiração em 31 de dezembro de 2025 à meia-noite.

    A saída retornará algo semelhante ao mostrado a seguir. Copie toda a Assinatura de Acesso Compartilhado e cole-a no **Bloco de notas**, pois ela será usada na próxima tarefa.

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

## Criar credencial

Agora que a funcionalidade está configurada, você pode gerar um arquivo de backup como um blob na conta de armazenamento do Azure.

1. Inicie o **SSMS (SQL Server Management Studio)**.

1. Será exibida uma mensagem pedindo que você se conecte ao SQL Server. Verifique se a **Autenticação do Windows** está selecionada e escolha **Conectar**.

1. Selecione **Nova Consulta**.

1. Crie a credencial que será usada para acessar o armazenamento na nuvem com o Transact-SQL a seguir. Preencha os valores apropriados e selecione **Executar**.

    ```sql
    IF NOT EXISTS  
    (SELECT * 
        FROM sys.credentials  
        WHERE name = 'https://<storage_account_name>.blob.core.windows.net/backups')  
    BEGIN
        CREATE CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]
        WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
        SECRET = '<key_value>'
    END;
    GO  
    ```

    Em que as duas ocorrências de **<storage_account_name>** são o nome exclusivo da conta de armazenamento criado e **<key_value>** é o valor gerado no fim da tarefa anterior semelhante a:

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

1. Para verificar se a credencial foi criada, vá até **Segurança -> Credenciais** no Explorador de Objetos no SSMS.

1. Se você digitou algo incorretamente e precisa recriar a credencial, use o seguinte comando para corrigir o valor digitado, lembrando-se de alterar o nome da conta de armazenamento:

    ```sql
    -- Only run this command if you need to go back and recreate the credential! 
    DROP CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]  
    ```

## Backup de banco de dados em URL

1. Usando o SSMS, faça backup do banco de dados **AdventureWorks2017** no Azure com o seguinte comando no Transact-SQL:

    ```sql
    BACKUP DATABASE AdventureWorks2017   
    TO URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak';
    GO 
    ```

    Em que **<storage_account_name>** é o nome da conta de armazenamento exclusivo usado ao criá-la. 

    Se ocorrer um erro, verifique se você não digitou algo incorretamente durante a criação da credencial, e se tudo foi criado com êxito.

## Validar o backup por meio da CLI do Azure

Para ver se o arquivo está realmente no Azure, você pode usar Gerenciador de Armazenamento (versão prévia) ou o Azure Cloud Shell.

1. De volta ao terminal do Visual Studio Code, execute este comando da CLI do Azure:

    ```bash
    az storage blob list -c "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --output table
    ```

    Certifique-se de usar o mesmo nome da conta de armazenamento exclusivo (após **--account-name**) e a chave da conta (após **--account-key**) usados nos comandos anteriores.

    Podemos confirmar que o arquivo de backup foi gerado com sucesso.

## Validar o backup no Navegador de Armazenamento

1. Em uma janela do navegador, acesse o portal do Azure, pesquise e selecione **Contas de armazenamento**

1. Selecione o nome exclusivo da conta de armazenamento que você criou para os backups.

1. Na navegação à esquerda, selecione **Navegador de armazenamento**. Expanda **Contêineres de Blob**.

1. Selecione **Backups**.

1. Observe que o arquivo de backup é armazenado no contêiner.

## Restauração a partir da URL

Esta tarefa mostrará como restaurar um banco de dados a partir de um armazenamento de blob do Azure.

1. No **SSMS (SQL Server Management Studio)**, selecione **Nova Consulta**, em seguida, cole e execute a consulta a seguir.

    ```sql
    USE AdventureWorks2017;
    GO
    SELECT * FROM Person.Address WHERE AddressId = 1;
    GO
    ```

1. Execute esse comando para alterar o endereço desse cliente.

    ```sql
    UPDATE Person.Address
    SET AddressLine1 = 'This is a human error'
    WHERE AddressId = 1;
    GO
    ```

1. Execute novamente a **Etapa 1** para verificar se o endereço foi alterado. Agora, imagine se alguém tivesse alterado milhares ou milhões de linhas sem uma cláusula WHERE – ou com a cláusula WHERE errada. Uma das soluções envolve a restauração do banco de dados a partir do último backup disponível.

1. Para restaurar o banco de dados de modo que ele volte para como estava antes de o nome do cliente ter sido alterado por engano, execute o seguinte.

    > &#128221;A sintaxe **SET SINGLE_USER WITH ROLLBACK IMMEDIATE** reverterá todas as transações abertas. Isso pode evitar que a restauração falhe devido a conexões ativas.

    ```sql
    USE [master]
    GO

    ALTER DATABASE AdventureWorks2017 SET SINGLE_USER WITH ROLLBACK IMMEDIATE
    GO

    RESTORE DATABASE AdventureWorks2017 
    FROM URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak'
    GO

    ALTER DATABASE AdventureWorks2017 SET MULTI_USER
    GO
    ```

    Onde **<storage_account_name>** é o nome exclusivo da conta de armazenamento que você criou.

1. Execute novamente a **Etapa 1** para verificar se o nome do cliente foi restaurado.

É importante entender os componentes e a interação para fazer um backup ou uma restauração no serviço de Armazenamento de Blobs do Azure.

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

Se você não estiver usando o banco de dados ou os arquivos de laboratório para qualquer outra finalidade, poderá limpar os objetos criados neste laboratório.

### Exclua a pasta C:\LabFiles

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, abra o **oExplorador de Arquivos**.
1. Navegue até **C:\\**.
1. Exclua a pasta **C:\LabFiles**.

## Exclua o banco de dados AdventureWorks2017

1. Na máquina virtual do laboratório ou no computador local, se não tiver sido fornecido, inicie uma Sessão do SQL Server Management Studio (SSMS).
1. Quando o SSMS abrir, por padrão, a caixa de diálogo **Connect to Service** será exibida. Escolha Default instance e selecione **Connect**. Talvez seja necessário marcar a caixa de seleção **Trust server certificate**.
1. Em **Object Explorer**, expanda a pasta **Databases**.
1. Clique com o botão direito do mouse no banco de dados **AdventureWorks2017** e selecione **Delete**.
1. Na caixa de diálogo **Delete Object**, marque a caixa de conexão **Close existing connections** 
1. Selecione **OK**.

---

Você concluiu este laboratório.

Agora você viu que pode fazer backup de um banco de dados para uma URL no Azure e, se necessário, restaurá-lo.
