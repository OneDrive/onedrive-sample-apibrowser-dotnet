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
# OneDrive API 浏览器示例

OneDriveAPIBrowser 示例是一个 [Windows 窗体](https://msdn.microsoft.com/en-us/library/dd30h2yb(v=vs.110).aspx)应用示例，它使用面向 C# /.NET 的 [Microsoft Graph .NET 客户端库](https://github.com/microsoftgraph/msgraph-sdk-dotnet)。在本示例中，用户可以浏览存储在 OneDrive 上的文件和文件夹，并查看元数据。

## 注册和配置应用程序

1. 使用个人或工作或学校帐户登录到[应用注册门户](https://apps.dev.microsoft.com/)。  
2. 选择“**添加应用**”。  
3. 为应用输入名称，并选择“**创建应用程序**”。将显示注册页，其中列出应用的属性。  
4. 在“**平台**”下，选择“**添加平台**”。  
5. 选择“**移动应用程序**”。  
6. 将客户端 ID（应用 ID）值复制到剪贴板。你将需要在示例应用程序中使用它。应用 ID 是应用的唯一标识符。   
7. 选择“**保存**”。  

## 设置

1. 如果没有安装，安装 [Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs) 和所有可用更新。 
2. 从 [GitHub](https://github.com/OneDrive/onedrive-sample-apibrowser-dotnet) 中下载 OneDriveAPIBrowser 示例或为存储卡创建分支。
3. 从 Visual Studio 中，打开 **OneDriveApiBrowser.sln** 解决方案。
4. 转到解决方案中的 OneDriveApiBrowser 项目并查看 FormBrowser.cs 代码。
5. 配置示例，以使用通过使 `MsaClientId` 值为变量而注册的客户端 ID (App Id)：
```csharp
        private const string MsaClientId = "Insert your client ID here";
```

## 运行示例

在 Visual Studio 中，从启动项目中选择示例 OneDriveAPIBrowser ，随后按下 **F5** 或单击**开始**来运行示例。示例如下所示： 

![OneDriveAPIBrowser 示例](OneDriveApiBrowser/images/OneDriveAPIBrowser.PNG)

### 登录
OneDrive API 浏览器应用程序打开后，选择 **文件** | **登录..** 以登录 OneDrive 个人账户或 OneDrive 商业账户。登录至 Microsoft 账户后，系统显示对话框，要求提供访问 OneDrive 文件的权限。

![允许此应用访问你的信息](OneDriveApiBrowser/images/Permissions.PNG)

单击“**是**”。

### 登录后

OneDrive 项将在左窗格上显示，每项按缩略图显示。在右窗格中，显示所选项的属性。可选择采用 JSON 还是树形视图来显示项目的属性。

若要上传文件，从菜单中选择“**上传**”，随后选择“**简单 - 基于路径**”以按路径上传，或选择“**简单 - 基于ID**”以按项 ID 上传。

如果要下载文件，选择文件，然后从菜单中选择“**下载**”。

## API 功能

### 检索经过身份验证的客户端

此项目获取 Microsoft Graph **GraphServiceClient** 实例并使用 **AuthenticationHelper.cs** 文件中的 **GetAuthenticatedClient** 方法来登录。
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

**AuthenticationHelper.cs** 文件还提供了 **SignOutAsync** 方法，可轻松注销用户：

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

### 获取项目属性

此项目演示如何通过调用 **GraphServiceClient** 对象的 **GetAsync** 方法来获取项目的属性：

```csharp
folder = await this.graphClient.Drive.Root.Request().Expand(expandValue).GetAsync();
```

### 简单项目上传

此示例利用 Microsoft Graph 的功能来按路径或按 ID 上传项目。
这里按照路径上传项目：

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

此例显示如何按照 ID 上传项目：
```csharp
var uploadedItem =
	await
	this.graphClient.Drive.Items[targetFolder.Id].ItemWithPath(filename).Content.Request()
                                    .PutAsync<DriveItem>(stream); 
```

## 更多资源

可继续使用 GitHub 或 Visual Studio 浏览此示例和其它功能。如果要查看使用适用于 CSharp/.NET 的 Microsoft Graph SDK 的 Windows 通用应用示例，参见 [OneDrivePhotoBrowser](https://github.com/OneDrive/graph-sample-photobrowser-uwp)。另外确保访问 [https://developer.microsoft.com/en-us/graph/](https://developer.microsoft.com/en-us/graph/) 查看 Microsoft Graph API 的官方文档。 

## 许可证

[许可证](LICENSE.txt)

此项目已采用 [Microsoft 开放源代码行为准则](https://opensource.microsoft.com/codeofconduct/)。有关详细信息，请参阅[行为准则常见问题解答](https://opensource.microsoft.com/codeofconduct/faq/)。如有其他任何问题或意见，也可联系 [opencode@microsoft.com](mailto:opencode@microsoft.com)。
