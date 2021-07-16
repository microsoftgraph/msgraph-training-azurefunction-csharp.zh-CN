---
ms.openlocfilehash: 3e6a83c19de68b6047914a68d94e66dbab95c6c4
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445953"
---
<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将完成 Azure 函数的实现 `GetMyNewestMessage` ，并更新测试客户端以调用 函数。

Azure 函数 [使用代表流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)。 此流中事件的基本顺序为：

- 测试应用程序使用交互式身份验证流来允许用户登录并授予同意。 它将返回一个作用域为 Azure 函数的令牌。 令牌不包含 **任何** Microsoft Graph作用域。
- 测试应用程序调用 Azure Function，在 标头中发送其访问 `Authorization` 令牌。
- Azure 函数验证令牌，然后将该令牌交换为包含 Microsoft 作用域的第二个Graph令牌。
- Azure 函数Graph第二个访问令牌代表用户调用 Microsoft 服务。

> [!IMPORTANT]
> 为了避免在源中存储应用程序 ID 和密码，你将使用 [.NET 密码管理器](https://docs.microsoft.com/aspnet/core/security/app-secrets) 存储这些值。 密码管理器仅供开发使用，生产应用应该使用受信任的密码管理器来存储密码。

## <a name="add-authentication-to-the-single-page-application"></a>向单页应用程序添加身份验证

首先将身份验证添加到 SPA。 这将允许应用程序获取访问令牌，以授予调用 Azure Function 的访问权限。 因为这是一个 SPA，它将授权 [代码流与 PKCE 一同使用](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)。

1. 在 **TestClient** 目录中新建一个名为 **config.js** 并添加以下代码。

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    将 `YOUR_TEST_APP_APP_ID_HERE` 替换为你在 Azure 门户中创建的应用程序 ID，Graph **Azure Function Test App**。 将 替换为从 Azure `YOUR_TENANT_ID_HERE` **(复制) 的 Directory** 租户租户 ID 值。 将 `YOUR_AZURE_FUNCTION_APP_ID_HERE` 替换为 Azure Function Graph **ID。**

    > [!IMPORTANT]
    > 如果你使用的是源代码管理（如 git），那么现在应该将 **config.js** 文件从源代码管理中排除，以避免意外泄露应用 ID 和租户 ID。

1. 在 **TestClient** 目录中新建一个名为 **auth.js** 并添加以下代码。

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    考虑此代码执行哪些功能。

    - 它使用存储在 `PublicClientApplication`config.js中的 **值进行初始化**。
    - 它 `loginPopup` 使用 Azure 函数的权限范围登录用户。
    - 它将用户的用户名存储在会话中。

    > [!IMPORTANT]
    > 由于应用使用 ，因此可能需要更改浏览器的弹出窗口阻止程序，以 `loginPopup` 允许弹出窗口 `http://localhost:8080` 。

1. 刷新页面并登录。 页面应该使用用户名进行更新，指示登录成功。

## <a name="add-authentication-to-the-azure-function"></a>向 Azure 函数添加身份验证

在此部分中，你将在 Azure 函数中实现代表流，以获得与 `GetMyNewestMessage` Microsoft Graph。

1. 在包含 **GraphTu一l.csproj** 的目录中打开 CLI 并运行以下命令，初始化 .NET 开发密码存储。

    ```Shell
    dotnet user-secrets init
    ```

1. 使用下列命令将应用程序 ID、机密和租户 ID 添加到密码存储。 将 `YOUR_API_FUNCTION_APP_ID_HERE` 替换为 Azure Function Graph **ID。** 将 替换为你在 Azure 门户中为 Azure Function Graph `YOUR_API_FUNCTION_APP_SECRET_HERE` **密码**。 将 替换为从 Azure `YOUR_TENANT_ID_HERE` **(复制) 的 Directory** 租户租户 ID 值。

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a>处理传入的 bearer 令牌

在此部分中，你将实现一个类，以验证并处理从 SPA 发送到 Azure 函数的令牌。

1. 在 **GraphTu一** l 目录中新建一个名为 Authentication **的目录**。

1. 在 **./GraphTu一l/Authentication** 文件夹中创建一个名为 **TokenValidationResult.cs** 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. 在 **./GraphTu一l/Authentication** 文件夹中创建一个名为 **TokenValidation.cs** 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

考虑此代码执行哪些功能。

- 它确保 标头中具有一个 bearer `Authorization` 标记。
- 它验证 Azure 已发布 OpenID 配置中的签名和颁发者。
- 它验证声明 (`aud` 是否) Azure Function 的应用程序 ID 匹配。
- 它分析令牌并生成 MSAL 帐户 ID，这是利用令牌缓存所需要的。

### <a name="create-an-on-behalf-of-authentication-provider"></a>创建代表身份验证提供程序

1. 在身份验证目录中新建一个名为 **OnBehalfOfAuthProvider.cs** 的文件，然后向该文件中添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

花些时间考虑 **OnBehalfOfAuthProvider.cs 中的代码** 有什么功能。

- 在 `GetAccessToken` 函数中，它首先尝试使用 从令牌缓存获取用户令牌 `AcquireTokenSilent` 。 如果此操作失败，它将使用测试应用发送到 Azure Function 的 bearer 令牌生成用户断言。 然后，它使用该用户断言获取Graph兼容的令牌 `AcquireTokenOnBehalfOf` 。
- 它实现 接口，从而允许在 的构造函数中传递此类 `Microsoft.Graph.IAuthenticationProvider` `GraphServiceClient` 以对传出请求进行身份验证。

### <a name="implement-a-graph-client-service"></a>实现Graph客户端服务

在此部分中，你将实现可注册用于依赖 [关系注入的服务](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)。 该服务将用于获取经过身份验证Graph客户端。

1. 在 **GraphTu一** l 目录中新建一个名为 Services 的 **目录**。

1. 在"服务"目录中新建一个名为 **IGraphClientService.cs** 的文件，然后向该文件中添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. 在"服务"目录中新建一个名为 **GraphClientService.cs** 的文件，然后向该文件中添加以下代码。

    ```csharp
    using GraphTutorial.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;

    namespace GraphTutorial.Services
    {
        // Service added via dependency injection
        // Used to get an authenticated Graph client
        public class GraphClientService : IGraphClientService
        {
        }
    }
    ```

1. 将以下属性添加到 `GraphClientService` 类。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. 将以下函数添加到 `GraphClientService` 类。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. 为 函数添加占位符 `GetAppGraphClient` 实现。 您将在稍后部分中实现此方案。

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    `GetUserGraphClient`函数获取令牌验证的结果，并生成一个经过身份验证 `GraphServiceClient` 的用户。

1. 打开 **./GraphTu一l/Program.cs，** 并将其内容替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="ProgramSnippet" highlight="15-23":::

    此代码将用户密码添加到配置 [，并启用](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) Azure 函数中的依赖项注入，从而公开 `GraphClientService` 服务。

### <a name="implement-getmynewestmessage-function"></a>实现 GetMyNewestMessage 函数

1. 打开 **./GraphTu一l/GetMyNewestMessage.cs，** 并将其全部内容替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a>查看 GetMyNewestMessage.cs 中的代码

花些时间考虑 **GetMyNewestMessage.cs** 中的代码有什么功能。

- 在构造函数中，它保存通过 `IConfiguration` `IGraphClientService` 依赖关系注入传入的 和 对象。
- 在 `Run` 函数中，它执行以下操作：
  - 验证对象中是否包含所需的配置 `IConfiguration` 值。
  - 验证 bearer 令牌，如果 `401` 令牌无效，则返回状态代码。
  - 从 Graph `GraphClientService` 请求的用户获取一个客户端。
  - 使用 Microsoft Graph SDK 从用户收件箱获取最新邮件，并作为响应中的 JSON 正文返回。

## <a name="call-the-azure-function-from-the-test-app"></a>从测试应用调用 Azure 函数

1. 打开 **auth.js** 并添加以下函数，获取访问令牌。

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    考虑此代码执行哪些功能。

    - 它首先尝试以静默方式获取访问令牌，而无需用户交互。 由于用户应该已经登录，因此 MSAL 的缓存中应包含用户令牌。
    - 如果失败并出现错误，指示用户需要交互，它将尝试以交互方式获取令牌。

    > [!TIP]
    > 可以在 中分析访问令牌，并确认声明是 Azure Function 的应用 ID，并且声明包含 Azure Function 的权限范围，而不是 [https://jwt.ms](https://jwt.ms) `aud` Microsoft `scp` Graph。

1. 在 **TestClient** 目录中新建一个名为 **azurefunctions.js** 并添加以下代码。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. 将 CLI 中的当前目录更改为 **./GraphTu一l** 目录，并运行以下命令以在本地启动 Azure Function。

    ```Shell
    func start
    ```

1. 如果尚未提供 SPA，请打开第二个 CLI 窗口，将当前目录更改为 **./TestClient** 目录。 运行以下命令以运行测试应用程序。

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. 打开浏览器，并导航到 `http://localhost:8080`。 登录并选择"最新 **邮件"** 导航项。 应用程序显示有关用户收件箱中最新邮件的信息。
