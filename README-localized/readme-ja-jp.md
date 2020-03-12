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
# OneDrive API ブラウザー サンプル

OneDrive API ブラウザー サンプルは、C#/.NET. 用の [Microsoft Graph .NET クライアント ライブラリ](https://github.com/microsoftgraph/msgraph-sdk-dotnet)を使用する [Windows Forms](https://msdn.microsoft.com/en-us/library/dd30h2yb(v=vs.110).aspx) アプリのサンプルです。このサンプルでは、ユーザーは OneDrive に保存されているファイルやフォルダーを参照して、メタデータを表示します。

## アプリケーションを登録して構成する

1. 個人用アカウントか職場または学校アカウントのいずれかを使用して、[アプリ登録ポータル](https://apps.dev.microsoft.com/)にサインインします。  
2. \[**アプリの追加**] を選択します。  
3. アプリの名前を入力して、\[**アプリケーションの作成**] を選択します。登録ページが表示され、アプリのプロパティが一覧表示されます。  
4. \[**プラットフォーム**] で、\[**プラットフォームの追加**] を選択します。  
5. \[**モバイル アプリケーション**] を選択します。  
6. クライアント ID (アプリ ID) の値をクリップボードにコピーします。この値はサンプル アプリで使用する必要があります。アプリ ID は、アプリの一意識別子です。   
7. \[**保存**] を選択します。  

## セットアップ

1. [Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs) と利用可能なすべての更新プログラムを、まだインストールしていない場合はインストールします。 
2. [GitHub](https://github.com/OneDrive/onedrive-sample-apibrowser-dotnet) から OneDriveAPIBrowser サンプルをダウンロードするか、リポジトリの独自のフォークを作成します。
3. Visual Studio から、**OneDriveApiBrowser.sln** ソリューションを開きます。
4. ソリューションの OneDriveApiBrowser プロジェクトに移動し、FormBrowser.cs のコードを表示します。
5. `MsaClientId` 変数の値にすることで、登録したクライアント ID (アプリ ID) を使用するようにサンプルを構成します:
```csharp
        private const string MsaClientId = "Insert your client ID here";
```

## サンプルの実行

Visual Studio で、スタートアップ プロジェクトの一覧からサンプル OneDriveAPIBrowser を選択し、**F5** キーを押すか、\[**開始**] をクリックしてサンプルを実行します。このサンプルは次のようになっています。 

![OneDriveAPIBrowser サンプル](OneDriveApiBrowser/images/OneDriveAPIBrowser.PNG)

### サインイン
OneDrive API ブラウザー アプリが開いたら、\[**ファイル**] | \[**サインイン...**] を選択して、個人用 OneDrive アカウントまたはビジネス OneDrive アカウントにサインインします。Microsoft アカウントにサインインすると、ダイアログが表示され、OneDrive ファイルにアクセスするためのアクセス許可を求められます。

![このアプリがあなたの情報にアクセスすることを許可します](OneDriveApiBrowser/images/Permissions.PNG)

\[**はい**] をクリックします。

### サインイン後

OneDrive のアイテムが左側のウィンドウに表示され、各アイテムがサムネイルで表示されます。右側のウィンドウに、選択したアイテムのプロパティが表示されます。JSON であってもツリー ビューであっても、アイテム プロパティの表示方法を選択できます。

ファイルをアップロードするには、メニューから \[**アップロード**] を選択し、\[**シンプル - パスベース**] を選択してパスでアップロードするか、\[**シンプル - ID ベース**] を選択してアイテム ID でアップロードします。

ファイルをダウンロードするには、ファイルを選択し、メニューから \[**ダウンロード**] を選択します。

## API の機能

### 認証済みクライアントを取得する

このサンプルは、Microsoft Graph **GraphServiceClient** インスタンスを取得し、**AuthenticationHelper.cs** ファイルの **GetAuthenticatedClient** メソッドを使用してユーザーをサインインさせます。
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

**AuthenticationHelper.cs** ファイルには、ユーザーを簡単にサインアウトさせる **SignOutAsync** メソッドも用意されています。

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

### アイテム プロパティを取得する

このサンプルでは、**GraphServiceClient** オブジェクトの **GetAsync** メソッドを呼び出して、アイテムのプロパティを取得する方法を示します。

```csharp
folder = await this.graphClient.Drive.Root.Request().Expand(expandValue).GetAsync();
```

### シンプルなアイテムのアップロード

このサンプルでは、パスまたは ID でアイテムをアップロードする Microsoft Graph の機能を利用しています。
ここでは、アイテムをパスでアップロードします。

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

この例では、アイテムを ID でアップロードする方法を示しています:
```csharp
var uploadedItem =
	await
	this.graphClient.Drive.Items[targetFolder.Id].ItemWithPath(filename).Content.Request()
                                    .PutAsync<DriveItem>(stream); 
```

## その他のリソース

GitHub または Visual Studio を使用して、このサンプルおよびその機能の残りの部分を引き続き探索できます。CSharp/.NET 用の Microsoft Graph SDK を使用する Windows ユニバーサル アプリのサンプルを表示するには、[OneDrivePhotoBrowser](https://github.com/OneDrive/graph-sample-photobrowser-uwp) を参照してください。[https://developer.microsoft.com/en-us/graph/](https://developer.microsoft.com/en-us/graph/) にある Microsoft Graph API の公式ドキュメントも確認してください。 

## ライセンス

[ライセンス](LICENSE.txt)

このプロジェクトでは、[Microsoft オープン ソース倫理規定](https://opensource.microsoft.com/codeofconduct/) が採用されています。詳細については、「[Code of Conduct の FAQ (倫理規定の FAQ)](https://opensource.microsoft.com/codeofconduct/faq/)」を参照してください。また、その他の質問やコメントがあれば、[opencode@microsoft.com](mailto:opencode@microsoft.com) までお問い合わせください。
