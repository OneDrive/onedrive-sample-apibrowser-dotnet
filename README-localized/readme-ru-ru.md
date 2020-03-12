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
# Пример использования OneDrive API Browser

Пример OneDriveAPIBrowser представляет собой приложение [Windows Forms](https://msdn.microsoft.com/en-us/library/dd30h2yb(v=vs.110).aspx), использующее клиентскую библиотеку [Microsoft Graph .NET Client Library](https://github.com/microsoftgraph/msgraph-sdk-dotnet) для C#/.NET. В этом примере пользователи могут просматривать файлы и папки, хранящиеся в OneDrive, а также их метаданные.

## Регистрация и настройка приложения

1. Войдите на [портал регистрации приложений](https://apps.dev.microsoft.com/) с помощью личной, рабочей или учебной учетной записи.  
2. Выберите пункт**Добавить приложение**.  
3. Введите имя приложения и выберите пункт **Создать приложение**. Откроется страница регистрации со списком свойств приложения.  
4. В разделе**Платформы**, нажмите**Добавление платформы**.  
5. Выберите пункт**Мобильное приложение**.  
6. Скопируйте значение идентификатора клиента (App Id) в буфер обмена. Вам нужно будет использовать его в примере приложения. Идентификатор приложения является уникальным.   
7. Нажмите кнопку **Сохранить**.  

## Настройка

1. Установите [Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs) со всеми доступными обновлениями. 
2. Скачайте пример OneDriveAPIBrowser из [GitHub](https://github.com/OneDrive/onedrive-sample-apibrowser-dotnet) или создайте свое ответвление репозитория.
3. В Visual Studio откройте решение **OneDriveApiBrowser.sln**.
4. Перейдите к проекту OneDriveApiBrowser в решении и просмотрите код для FormBrowser.cs.
5. Настройте пример так, чтобы использовался идентификатор клиента (идентификатор приложения), который вы зарегистрировали, задав его в качестве значения переменной `MsaClientId`:
```csharp
        private const string MsaClientId = "Insert your client ID here";
```

## Запуск примера

В Visual Studio выберите пример OneDriveAPIBrowser из списка запускаемых проектов, затем нажмите **F5** или **Запуск**, чтобы запустить пример. Пример выглядит следующим образом: 

![Пример OneDriveAPIBrowser](OneDriveApiBrowser/images/OneDriveAPIBrowser.PNG)

### Вход
Когда откроется приложение OneDrive API Browser, выберите **Файл** | **Вход...** для входа в личную учетную запись OneDrive или учетную запись OneDrive для бизнеса. Когда вы войдете в свою учетную запись Майкрософт, откроется диалоговое окно с запросом разрешения на доступ к файлам OneDrive.

![Дать этому приложению доступ к вашим данным](OneDriveApiBrowser/images/Permissions.PNG)

Нажмите кнопку **Да**.

### После входа

Элементы OneDrive отобразятся в области слева, каждый из них будет представлен эскизом. В области справа будут отображаться свойства выбранного элемента. Можно выбрать представление свойств элементов в формате JSON или в виде дерева.

Чтобы добавить файл, выберите пункт меню **Добавление**, а затем выберите **Простое – на основе пути**, чтобы добавить файл по его пути, или **Простое – на основе идентификатора**, чтобы добавить файл по идентификатору элемента.

Чтобы скачать файл, выберите файл, а затем выберите в меню пункт **Скачать**.

## Особенности API

### Получение клиента, прошедшего проверку подлинности

В этом примере показано, как получить экземпляр клиента **GraphServiceClient** Microsoft Graph и выполнить вход пользователя с использованием метода **GetAuthenticatedClient** в файле **AuthenticationHelper.cs**.
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

Файл **AuthenticationHelper.cs** также содержит метод **SignOutAsync** для простого выполнения выхода пользователя:

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

### Получение свойств элемента

В этом примере показано, как получить свойства элемента, вызвав метод **GetAsync** объекта **GraphServiceClient**:

```csharp
folder = await this.graphClient.Drive.Root.Request().Expand(expandValue).GetAsync();
```

### Добавление простого элемента

В этом примере можно с помощью Microsoft Graph добавлять элементы по их пути или по идентификатору.
Элемент добавляется по своему пути следующим образом:

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

В этом примере показано, как добавить элемент по идентификатору:
```csharp
var uploadedItem =
	await
	this.graphClient.Drive.Items[targetFolder.Id].ItemWithPath(filename).Content.Request()
                                    .PutAsync<DriveItem>(stream); 
```

## Дополнительные ресурсы

Вы можете продолжить работу с этим примером и узнать об остальных его возможностях с помощью GitHub или Visual Studio. Пример приложения Windows Universal, использующего пакет SDK Microsoft Graph для CSharp/.NET, размещен по ссылке [OneDrivePhotoBrowser](https://github.com/OneDrive/graph-sample-photobrowser-uwp). Кроме того, обязательно ознакомьтесь с официальной документацией по API Microsoft Graph по адресу [https://developer.microsoft.com/en-us/graph/](https://developer.microsoft.com/en-us/graph/). 

## Лицензия

[Лицензия](LICENSE.txt)

Этот проект соответствует [Правилам поведения разработчиков открытого кода Майкрософт](https://opensource.microsoft.com/codeofconduct/). Дополнительные сведения см. в разделе [часто задаваемых вопросов о правилах поведения](https://opensource.microsoft.com/codeofconduct/faq/). Если у вас возникли вопросы или замечания, напишите нам по адресу [opencode@microsoft.com](mailto:opencode@microsoft.com).
