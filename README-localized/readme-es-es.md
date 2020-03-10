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
# Ejemplo de explorador de la API de OneDrive

El ejemplo OneDriveAPIBrowser es una aplicación de [Windows Forms](https://msdn.microsoft.com/en-us/library/dd30h2yb(v=vs.110).aspx) que usa la [biblioteca cliente de .NET de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para C# y .NET. En este ejemplo, los usuarios pueden examinar archivos y carpetas almacenados en OneDrive, así como ver metadatos.

## \##Registrar y configurar la aplicación

1. Inicie sesión en el [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.  
2. Seleccione **Agregar una aplicación**.  
3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**. Se muestra la página de registro, indicando las propiedades de la aplicación.  
4. En **Plataformas**, seleccione **Agregar plataforma**.  
5. Seleccione **Aplicación móvil**.  
6. Copie el valor del Id. de cliente (Id. de la aplicación) en el portapapeles. Debe usarlo en la aplicación de ejemplo. El Id. de la aplicación es un identificador único para su aplicación.   
7. Seleccione **Guardar**.  

## Configurar

1. Instale [Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs) y todas las actualizaciones disponibles si aún no lo tiene. 
2. Descargue el ejemplo OneDriveAPIBrowser de [GitHub](https://github.com/OneDrive/onedrive-sample-apibrowser-dotnet) o cree su propia bifurcación del repositorio.
3. En Visual Studio, abra la solución **OneDriveApiBrowser.sln**.
4. Vaya al proyecto OneDriveApiBrowser en la solución y vea el código de FormBrowser.cs.
5. Configure el ejemplo para usar el Id. de cliente (Id. de la aplicación) que ha registrado y convertirlo en el valor de la variable `MsaClientId`:
```csharp
        private const string MsaClientId = "Insert your client ID here";
```

## Ejecutar el ejemplo

En Visual Studio, seleccione el ejemplo OneDriveAPIBrowser en la lista del proyecto de inicio y, después, presione **F5** o haga clic en **Inicio** para ejecutar el ejemplo. El ejemplo tiene un aspecto similar al siguiente: 

![Ejemplo OneDriveAPIBrowser](OneDriveApiBrowser/images/OneDriveAPIBrowser.PNG)

### Iniciar sesión
Cuando se abra la aplicación del explorador de la API de OneDrive, elija **Archivo** | **Iniciar sesión...** para iniciar sesión en una cuenta personal de OneDrive o en una cuenta profesional de OneDrive. Cuando haya iniciado sesión en su cuenta de Microsoft, se mostrará un cuadro de diálogo que le pedirá permiso para acceder a los archivos de OneDrive.

![¿Permitir que esta aplicación tenga acceso a su información?](OneDriveApiBrowser/images/Permissions.PNG)

Haga clic en **Sí**.

### Después de iniciar sesión

Los elementos de OneDrive se mostrarán en el panel izquierdo, con cada elemento representado por una miniatura. En el panel derecho, se muestran las propiedades del elemento seleccionado. Puede elegir cómo se muestran las propiedades del elemento, independientemente de si está seleccionada la vista de árbol o la vista JSON.

Para cargar un archivo, elija **Cargar** en el menú y, después, elija **Simple: basado en ruta de acceso** para cargar por ruta de acceso o **Simple: basado en Id.** para cargar por Id. de elemento.

Para descargar un archivo, seleccione un archivo y elija **Descargar** en el menú.

## Características de la API 

### Recuperar un cliente autenticado

En este ejemplo se obtiene una instancia **GraphServiceClient** de Microsoft Graph y se inicia la sesión del usuario mediante el método **GetAuthenticatedClient** del archivo **AuthenticationHelper.cs**.
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

El archivo **AuthenticationHelper.cs** también proporciona un método **SignOutAsync** para cerrar fácilmente la sesión del usuario:

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

### Obtener propiedades de elementos

En este ejemplo se muestra cómo obtener las propiedades de un elemento llamando al método **GetAsync** del objeto **GraphServiceClient**:

```csharp
folder = await this.graphClient.Drive.Root.Request().Expand(expandValue).GetAsync();
```

### Carga de elemento simple

En este ejemplo se hace uso de la capacidad de Microsoft Graph para cargar elementos por ruta de acceso o por identificador. Aquí, se carga un elemento por la ruta de acceso:

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

En este ejemplo se muestra cómo cargar un elemento por el Id.:
```csharp
var uploadedItem =
	await
	this.graphClient.Drive.Items[targetFolder.Id].ItemWithPath(filename).Content.Request()
                                    .PutAsync<DriveItem>(stream); 
```

## Más recursos

Puede seguir examinando este ejemplo y el resto de sus características con GitHub o con Visual Studio. Para ver un ejemplo de aplicación universal de Windows que usa el SDK de Microsoft Graph para CSharp y .NET, vea [OneDrivePhotoBrowser](https://github.com/OneDrive/graph-sample-photobrowser-uwp). Asegúrese de consultar también la documentación oficial de la API de Microsoft Graph en [https://developer.microsoft.com/en-us/graph/](https://developer.microsoft.com/en-us/graph/). 

## Licencia

[Licencia](LICENSE.txt)

Este proyecto ha adoptado el [Código de conducta de código abierto de Microsoft](https://opensource.microsoft.com/codeofconduct/). Para obtener más información, vea [Preguntas frecuentes sobre el código de conducta](https://opensource.microsoft.com/codeofconduct/faq/) o póngase en contacto con [opencode@microsoft.com](mailto:opencode@microsoft.com) si tiene otras preguntas o comentarios.
