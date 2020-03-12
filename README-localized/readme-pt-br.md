---
page_type: sample
products:
- office-onedrive
- ms-graph
languages:
- csharp
extensions:
  contentType: samples
  technologies:
  - Microsoft Graph
  services:
  - OneDrive
  createdDate: 7/6/2016 11:47:49 AM
---
# Exemplo de navegador de API do OneDrive

O exemplo OneDriveAPIBrowser é um0exemplo do aplicativo [Windows Forms](https://msdn.microsoft.com/en-us/library/dd30h2yb(v=vs.110).aspx) que utiliza a [Biblioteca do cliente do Microsoft Graph .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para C#/.net. Neste exemplo, os usuários podem procurar arquivos e pastas armazenados no OneDrive e visualizar os metadados.

## Registrar e configurar o aplicativo

1. Entre no [Portal de registro do Aplicativo](https://apps.dev.microsoft.com/) utilizando sua conta pessoal, corporativa ou de estudante.  
2. Selecione**Adicionar um aplicativo**.  
3. Insira um nome para o aplicativo e selecione **Criar aplicativo**. A página de registro será exibida, listando as propriedades do seu aplicativo.  
4. Em **Plataformas**, selecione **Adicionar plataforma**.  
5. Clique em **Aplicativo móvel**.  
6. Copie o valor da ID do Cliente (ID do aplicativo) para a área de transferência. Será necessário que você use no aplicativo do exemplo. Essa ID do aplicativo é o identificador exclusivo do seu aplicativo.   
7. Clique em **Salvar**.  

## Configure

1. Instale o [Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs) e todas as atualizações disponíveis, caso ainda não o tenha. 
2. Baixe o exemplo OneDriveAPIBrowser do](https://github.com/OneDrive/onedrive-sample-apibrowser-dotnet)GitHub [ou crie sua própria bifurcação do repositório.
3. No Visual Studio, abra a solução **OneDriveApiBrowser.sln**.
4. Vá para o projeto OneDriveApiBrowser na solução e exiba o código do FormBrowser.cs.
5. Configure o exemplo para usar a ID do cliente (ID do aplicativo) que você registrou tornando-o o valor da variável `MsaClientId`:
```csharp
        private const string MsaClientId = "Insert your client ID here";
```

## Execute o exemplo

No Visual Studio, selecione o exemplo OneDriveAPIBrowser na lista de projetos de inicialização e pressione **F5** ou clique em **Iniciar** para executar o exemplo. O exemplo é assim: 

![Exemplo OneDriveAPIBrowser](OneDriveApiBrowser/images/OneDriveAPIBrowser.PNG)

### Entrar
Quando o aplicativo do navegador da API do OneDrive for aberto, clique em **Arquivo** | **Registre-se...** para entrar em uma conta pessoal do OneDrive ou em uma conta corporativa do OneDrive. Depois de entrar na sua conta da Microsoft, uma caixa de diálogo será exibida, solicitando permissões para acessar os arquivos do OneDrive.

![Deixe que este aplicativo acesse suas informações](OneDriveApiBrowser/images/Permissions.PNG)

Clique em **Sim**.

### Depois de entrar

Seus itens do OneDrive serão exibidos no painel esquerdo com cada item representado por uma miniatura. No painel direito serão exibidas as propriedades do item selecionado. Você pode escolher como as propriedades do item serão exibidas, seja JSON ou Tree View.

Para carregar um arquivo, clique em **Carregar** no menu e clique em **Com base em um caminho simples** para carregá-lo por caminho ou **Com base em ID simples** para serem carregados pela ID do item.

Para baixar um arquivo, selecione um arquivo e clique em **Baixar** no menu.

## Recursos da API

### Recuperar um cliente autenticado

Este exemplo obtém uma instância do Microsoft Graph **GraphServiceClient** e entra no usuário utilizando o método **GetAuthenticatedClient** no arquivo **AuthenticationHelper.cs**.
```csharp

public static string[] Scopes = { "Files.ReadWrite.All" };
...
        public static GraphServiceClient GetAuthenticatedClient()
        {
            if (graphClient == null)
            {
                // Create Microsoft Graph client.
                try
                {
                    graphClient = new GraphServiceClient(
                        "https://graph.microsoft.com/v1.0",
                        new DelegateAuthenticationProvider(
                            async (requestMessage) =>
                            {
                                var token = await GetTokenForUserAsync();
                                requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", token);
                                // This header has been added to identify our sample in the Microsoft Graph service.  If extracting this code for your project please remove.
                                requestMessage.Headers.Add("SampleID", "uwp-csharp-apibrowser-sample");

                            }));
                    return graphClient;
                }

                catch (Exception ex)
                {
                    Debug.WriteLine("Could not create a graph client: " + ex.Message);
                }
            }

            return graphClient;
        } 
...
```

O arquivo **AuthenticationHelper.cs** também fornece o método **SignOutAsync** para desconectar facilmente o usuário:

```csharp
        public static void SignOut()
        {
            foreach (var user in IdentityClientApp.Users)
            {
                user.SignOut();
            }
            graphClient = null;
            TokenForUser = null;

        }
```

### Obter propriedades do item

Este exemplo demonstra como obter as propriedades de um item, chamando o método **GetAsync** do objeto **GraphServiceClient**:

```csharp
folder = await this.graphClient.Drive.Root.Request().Expand(expandValue).GetAsync();
```

### Carregamento de item simples

Este exemplo faz uso da capacidade de carregar itens por caminho ou por ID do Microsoft Graph.
Aqui, você carrega um item por caminho:

```csharp
// Since the ItemWithPath method is available only at Drive.Root, we need to strip
// /drive/root: (12 characters) from the parent path string.
string folderPath = targetFolder.ParentReference == null
	? ""
	: targetFolder.ParentReference.Path.Remove(0, 12) + "/" + Uri.EscapeUriString(targetFolder.Name);
	var uploadPath = folderPath + "/" + Uri.EscapeUriString(System.IO.Path.GetFileName(filename)); 

// Use the Microsoft Graph SDK to upload the item by path.
var uploadedItem =
	await
	this.graphClient.Drive.Root.ItemWithPath(uploadPath).Content.Request().PutAsync<DriveItem>(stream); 

```

Este exemplo mostra como carregar um item por id:
```csharp
var uploadedItem =
	await
	this.graphClient.Drive.Items[targetFolder.Id].ItemWithPath(filename).Content.Request()
                                    .PutAsync<DriveItem>(stream); 
```

## Mais recursos

Você pode continuar a explorar esse exemplo e o restante dos seus recursos usando GitHub ou Visual Studio. Para visualizar um exemplo de aplicativo universal do Windows que utiliza o SDK do Microsoft Graph para CSharp/. NET, consulte [OneDrivePhotoBrowser](https://github.com/OneDrive/graph-sample-photobrowser-uwp). Também verifique a documentação oficial da API do Microsoft Graph em [https://developer.microsoft.com/en-us/graph/](https://developer.microsoft.com/en-us/graph/). 

## Licença

[Licença](LICENSE.txt)

Este projeto adotou o [Código de Conduta de Código Aberto da Microsoft](https://opensource.microsoft.com/codeofconduct/).  Para saber mais, confira as [Perguntas frequentes sobre o Código de Conduta](https://opensource.microsoft.com/codeofconduct/faq/) ou entre em contato pelo [opencode@microsoft.com](mailto:opencode@microsoft.com) se tiver outras dúvidas ou comentários.
