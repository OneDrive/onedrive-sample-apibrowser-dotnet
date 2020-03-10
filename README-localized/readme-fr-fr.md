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
# Exemple de OneDriveAPIBrowser

L’exemple de OneDriveAPIBrowser est un exemple d’application [Windows Forms](https://msdn.microsoft.com/en-us/library/dd30h2yb(v=vs.110).aspx) qui utilise la [bibliothèque cliente .NET Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet) pour C#/.NET. Dans cet exemple, les utilisateurs peuvent parcourir les fichiers et dossiers qui sont stockés sur OneDrive et afficher les métadonnées.

## Inscription et configuration de l’application

1. Connectez-vous au [portail d’inscription des applications](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.  
2. Sélectionnez **Ajouter une application**.  
3. Entrez un nom pour l’application, puis sélectionnez **Créer une application**. La page d’inscription s’affiche, répertoriant les propriétés de votre application.  
4. Sous **Plateformes**, sélectionnez **Ajouter une plateforme**.  
5. Sélectionnez **Application mobile**.  
6. Copiez la valeur d’ID client (Id d’application) dans le Presse-papiers. Vous devez l’utiliser dans l’exemple d’application. L’ID d’application est un identificateur unique pour votre application.   
7. Sélectionnez **Enregistrer**.  

## Configurer

1. Installez [Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs) et toutes les mises à jour disponibles si ce n’est déjà fait. 
2. Téléchargez l’exemple OneDriveAPIBrowser à partir de [GitHub](https://github.com/OneDrive/onedrive-sample-apibrowser-dotnet) ou créez votre propre bifurcation du référentiel.
3. Dans Visual Studio, ouvrez la solution **OneDriveApiBrowser.sln**.
4. Accédez au projet OneDriveApiBrowser dans la solution et affichez le code de FormBrowser.cs.
5. Configurez l’exemple afin d’utiliser l’ID client (ID d’application) que vous avez enregistré en lui donnant la valeur de la variable `MsaClientId` :
```csharp
        private const string MsaClientId = "Insert your client ID here";
```

## Exécuter l’exemple

Dans Visual Studio, sélectionnez l’exemple de OneDriveAPIBrowser dans la liste de projets de démarrage, puis appuyez sur **F5** ou cliquez sur **Démarrer** pour exécuter l’exemple. L’exemple se présente ainsi : 

![Exemple de OneDriveAPIBrowser](OneDriveApiBrowser/images/OneDriveAPIBrowser.PNG)

### Connexion
Lorsque l’application navigateur de l’API OneDrive s’ouvre, sélectionnez **Fichier** | **Se connecter...** pour vous connecter à un compte OneDrive personnel ou professionnel. Une fois que vous êtes connecté à votre compte Microsoft, une boîte de dialogue s’affiche et vous demande les autorisations d’accès aux fichiers OneDrive.

![Autoriser cette application à accéder à vos informations](OneDriveApiBrowser/images/Permissions.PNG)

Cliquez sur **Oui**.

### Après la connexion

Vos éléments OneDrive apparaîtront dans le volet gauche, chacun d’entre eux étant représenté par une miniature. Dans le volet droit, les propriétés de l’élément sélectionné s’affichent. Vous pouvez choisir le mode d’affichage des propriétés de l’élément, au format JSON ou sous forme d’arborescence.

Pour charger un fichier, sélectionnez **Charger** dans le menu, puis choisissez **Simple : basé sur un chemin d’accès** pour effectuer un chargement par chemin d’accès ou **Simple : basé sur l’ID** pour effectuer un chargement par ID d’élément.

Pour télécharger un fichier, sélectionnez un fichier, puis choisissez **Télécharger** dans le menu.

## Fonctionnalités de l’API

### Récupération d’un client authentifié

Cet exemple récupère une instance **GraphServiceClient** de Microsoft Graph et se connecte à l’utilisateur à l’aide de la méthode **GetAuthenticatedClient** dans le fichier **AuthenticationHelper.cs**.
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

Le fichier **AuthenticationHelper.cs** fournit également une méthode **SignOutAsync** pour déconnecter facilement l’utilisateur :

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

### Obtenir les propriétés de l’élément

Cet exemple montre comment obtenir les propriétés d’un élément en appelant la méthode **GetAsync** de l’objet **GraphServiceClient** :

```csharp
folder = await this.graphClient.Drive.Root.Request().Expand(expandValue).GetAsync();
```

### Chargement d’un élément simple

Cet exemple utilise la possibilité pour Microsoft Graph de charger des éléments par chemin ou par ID.
Ici, vous allez charger un élément par chemin :

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

Cet exemple montre comment télécharger un élément par ID :
```csharp
var uploadedItem =
	await
	this.graphClient.Drive.Items[targetFolder.Id].ItemWithPath(filename).Content.Request()
                                    .PutAsync<DriveItem>(stream); 
```

## Autres ressources

Vous pouvez continuer à explorer cet exemple et le reste de ses fonctionnalités à l’aide de GitHub ou de Visual Studio. Pour afficher un exemple d’application universelle Windows qui utilise le kit de développement logiciel Microsoft Graph pour CSharp/.NET, voir [OneDrivePhotoBrowser](https://github.com/OneDrive/graph-sample-photobrowser-uwp). Veillez également à extraire la documentation officielle de l’API Microsoft Graph sur [https://developer.microsoft.com/en-us/graph/](https://developer.microsoft.com/en-us/graph/). 

## Licence

[Licence](LICENSE.txt)

Ce projet a adopté le [Code de conduite Open Source de Microsoft](https://opensource.microsoft.com/codeofconduct/). Pour en savoir plus, reportez-vous à la [FAQ relative au code de conduite](https://opensource.microsoft.com/codeofconduct/faq/) ou contactez [opencode@microsoft.com](mailto:opencode@microsoft.com) pour toute question ou tout commentaire.
