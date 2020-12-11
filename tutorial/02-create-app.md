---
ms.openlocfilehash: 17c93f353c84ea2db28cd2e0203d30c5f320e36e
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655231"
---
<!-- markdownlint-disable MD002 MD041 -->

在本教程中，你将创建一个简单的 Azure 函数，该函数实现调用 Microsoft Graph 的 HTTP 触发器函数。 这些函数将涵盖以下方案：

- 实现 API 以使用代表流身份验证访问 [用户的](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) 收件箱。
- 使用客户端凭据授予流身份验证，实现 API 来订阅和取消订阅用户收件箱中的[通知。](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)
- 实现 Webhook 以从 Microsoft Graph [接收](https://docs.microsoft.com/graph/webhooks) 更改通知，以及使用客户端凭据授予流访问数据。

你还将在 SPA (创建简单的 JavaScript 单页) 以调用在 Azure 函数中实现的 API。

## <a name="create-azure-functions-project"></a>创建 Azure 函数项目

1. 在要创建项目的目录中 (CLI) 打开命令行界面。 运行以下命令。

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. 将 CLI 中的当前目录更改为 **GraphTu一l** 目录，并运行以下命令在项目中创建三个函数。

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. 打开 **local.settings.js，** 然后向文件添加以下内容以允许来自测试应用程序的 URL 的 `http://localhost:8080` CORS。

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. 运行以下命令以在本地运行项目。

    ```Shell
    func start
    ```

1. 如果一切正常，你将看到以下输出：

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. 打开浏览器并浏览到输出中显示的函数 URL，验证这些函数是否正常工作。 你应该在浏览器中看到以下 `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` 消息：

## <a name="create-single-page-application"></a>创建单页应用程序

1. 在要创建项目的目录中打开 CLI。 创建一个名为 **TestClient 的** 目录来保存 HTML 和 JavaScript 文件。

1. 在 **TestClient** index.htm创建一个名为index.htm **l** 的新文件，并添加以下代码。

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    这将定义应用的基本布局，包括导航栏。 它还添加以下内容：

    - [Bootstrap](https://getbootstrap.com/) 及其支持 JavaScript
    - [FontAwesome](https://fontawesome.com/)
    - [适用于 JavaScript (MSAL.js) 2.0 的 Microsoft 身份验证库](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > 该页面包含一个 `<link rel="shortcut icon" href="g-raph.png">` () 。 你可以删除此行，也可以从 [GitHub](https://github.com/microsoftgraph/g-raph) **g-raph.png** 文件。

1. 在 **TestClient** 目录中创建一个名为 **style.css** 的新文件，并添加以下代码。

    :::code language="css" source="../demo/TestClient/style.css":::

1. 在 **TestClient** **ui.js** 创建一个名为ui.js的新文件，并添加以下代码。

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    此代码使用 JavaScript 根据所选视图呈现当前页面。

### <a name="test-the-single-page-application"></a>测试单页应用程序

> [!NOTE]
> 本节包含有关使用 [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) 在开发计算机上运行简单测试 HTTP 服务器的说明。 不需要使用此特定工具。 可以使用您喜欢的任何测试服务器来为 **TestClient** 目录提供服务。

1. 在 CLI 中运行以下命令以安装 **dotnet-serve。**

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. 将 CLI 中的当前目录更改为 **TestClient** 目录，并运行以下命令以启动 HTTP 服务器。

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. 打开浏览器，并导航到 `http://localhost:8080`。 页面应呈现，但当前没有任何按钮可以正常工作。

## <a name="add-nuget-packages"></a>添加 NuGet 程序包

在继续之前，请安装一些稍后将使用的其他 NuGet 程序包。

- 在 Azure Functions 项目中启用依赖项注入的[Microsoft.Azure.Functions.Extensions。](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions)
- [Microsoft.Extensions.Config审核。用于从](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) [.NET](https://docs.microsoft.com/aspnet/core/security/app-secrets)开发密码存储读取应用程序配置的 UserSecret。
- [用于调用 Microsoft Graph 的 Microsoft.Graph。](https://www.nuget.org/packages/Microsoft.Graph/)
- 用于验证和管理令牌的[Microsoft.Identity.Client。](https://www.nuget.org/packages/Microsoft.Identity.Client/)
- 用于检索 OpenID 配置以用于令牌验证的[Microsoft.IdentityModel.Protocols.OpenIdConnect。](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect)
- [System.IdentityModel.Tokens.Jwt，](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) 用于验证发送到 Web API 的令牌。

1. 将 CLI 中的当前目录更改为 **GraphTu一l** 目录并运行以下命令。

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
