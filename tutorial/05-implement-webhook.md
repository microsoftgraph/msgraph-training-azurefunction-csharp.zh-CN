---
ms.openlocfilehash: 181afe9acfe45ff619a50cf874669228b421b475
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445974"
---
<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将完成实现 Azure 函数和 ，并更新测试应用程序以订阅和取消订阅用户 `SetSubscription` `Notify` 收件箱中的更改。

- 函数将充当 API，允许测试应用创建或删除用户收件箱中更改 `SetSubscription` 的订阅[](https://docs.microsoft.com/graph/webhooks)。
- `Notify`函数将充当接收订阅生成的更改通知的 Webhook。

这两个函数都将使用[客户端凭据授予](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)流获取仅应用令牌，以调用 Microsoft Graph。 由于管理员授予了对所需权限范围的管理员同意，因此无需用户交互来获取令牌。

## <a name="add-client-credentials-authentication-to-the-azure-functions-project"></a>将客户端凭据身份验证添加到 Azure Functions 项目

在此部分中，你将在 Azure Functions 项目中实现客户端凭据流，以获得与 Microsoft Graph。

1. 在包含 **GraphTu一一l.csproj 的目录中打开 CLI。**

1. 使用下列命令将 webhook 应用程序 ID 和密码添加到密码存储。 将 `YOUR_WEBHOOK_APP_ID_HERE` 替换为 Azure Function **Webhook Graph应用程序 ID。** 将 替换为你在 Azure 门户中为 Azure Function `YOUR_WEBHOOK_APP_SECRET_HERE` **Webhook Graph密码**。

    ```Shell
    dotnet user-secrets set webHookId "YOUR_WEBHOOK_APP_ID_HERE"
    dotnet user-secrets set webHookSecret "YOUR_WEBHOOK_APP_SECRET_HERE"
    ```

### <a name="create-a-client-credentials-authentication-provider"></a>创建客户端凭据身份验证提供程序

1. 在名为 **ClientCredentialsAuthProvider.cs** 的 **./GraphTu一l/Authentication** 目录中创建新文件并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/ClientCredentialsAuthProvider.cs" id="AuthProviderSnippet":::

花些时间考虑 **ClientCredentialsAuthProvider.cs** 中的代码有什么功能。

- 在构造函数中，它从程序包初始化 **ConfidentialClientApplication。** `Microsoft.Identity.Client` 它使用 `WithAuthority(AadAuthorityAudience.AzureAdMyOrg, true)` 和 `.WithTenantId(tenantId)` 函数将登录访问群体限制为仅指定Microsoft 365访问群体。
- 在 `GetAccessToken` 函数中，它 `AcquireTokenForClient` 调用 获取应用程序的令牌。 客户端凭据令牌流始终非交互。
- 它实现 接口，从而允许在 的构造函数中传递此类 `Microsoft.Graph.IAuthenticationProvider` `GraphServiceClient` 以对传出请求进行身份验证。

## <a name="update-graphclientservice"></a>更新 GraphClientService

1. 打开 **GraphClientService.cs，** 将以下属性添加到 类。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

1. 将现有的 `GetAppGraphClient` 函数替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientFunctions":::

## <a name="implement-notify-function"></a>实现 Notify 函数

在此部分中，你将实现 `Notify` 函数，它将用作更改通知的通知 URL。

1. 在 **GraphTu一ls** 目录中新建一个名为 Models **的目录**。

1. 在 Models 目录中新建 **一个名为****ResourceData.cs 的文件** 并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/ResourceData.cs" id="ResourceDataSnippet":::

1. 在 Models 目录中新建一个名为 **ChangeNotificationPayload.cs** 的文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/ChangeNotificationPayload.cs" id="ChangeNotificationSnippet":::

1. 在 Models 目录中新建 **一个名为****NotificationList.cs 的文件** 并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/NotificationList.cs" id="NotificationListSnippet":::

1. 打开 **./GraphTu一l/Notify.cs，** 并将其全部内容替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/Notify.cs" id="NotifySnippet":::

花些时间考虑 **Notify.cs** 中的代码功能。

- `Run`该函数检查是否存在 `validationToken` 查询参数。 如果该参数存在，它会将请求作为 [验证请求处理](https://docs.microsoft.com/graph/webhooks#notification-endpoint-validation)，并相应地做出响应。
- 如果请求不是验证请求，JSON 有效负载会反作用于 `ChangeNotificationCollection` 。
- 检查列表中的每个通知的预期客户端状态值，并进行处理。
- 触发通知的邮件会通过 Microsoft Graph。

## <a name="implement-setsubscription-function"></a>实现 SetSubscription 函数

在此部分中，您将实现 SetSubscription 函数。 此函数将充当由测试应用程序调用以在用户收件箱上创建或删除订阅的 API。

1. 在 Models 目录中新建一个名为 **SetSubscriptionPayload.cs** 的文件并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/SetSubscriptionPayload.cs" id="SetSubscriptionPayloadSnippet":::

1. 打开 **./GraphTu一l/SetSubscription.cs，** 并将其全部内容替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/SetSubscription.cs" id="SetSubscriptionSnippet":::

花些时间考虑 **SetSubscription.cs** 中的代码有什么功能。

- 函数读取 POST 请求中发送的 JSON 有效负载，以确定请求类型 (订阅或取消订阅) 、要订阅的用户 ID 以及要取消订阅的 `Run` 订阅 ID。
- 如果请求是订阅请求，它将使用 Microsoft Graph SDK 在指定用户的收件箱中创建新订阅。 订阅将在创建或更新邮件时通知。 新订阅在响应的 JSON 有效负载中返回。
- 如果请求是取消订阅请求，它将使用 Microsoft Graph SDK 删除指定的订阅。

## <a name="call-setsubscription-from-the-test-app"></a>从测试应用调用 SetSubscription

在此部分中，你将实现用于创建和删除测试应用中的订阅的函数。

1. 打开 **./TestClient/azurefunctions.js** 并添加以下函数。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="createSubscriptionSnippet":::

    此代码调用 Azure 函数来订阅并将新订阅添加到会话中 `SetSubscription` 的订阅数组。

1. 将以下函数添加到 **azurefunctions.js**。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="deleteSubscriptionSnippet":::

    此代码调用 `SetSubscription` Azure 函数取消订阅并从会话中的订阅数组中删除订阅。

1. 如果没有运行 ngrok，请运行 ngrok `ngrok http 7071` () 复制 HTTPS 转发 URL。

1. 通过运行以下命令，将 ngrok URL 添加到用户密码存储。

    ```Shell
    dotnet user-secrets set ngrokUrl "YOUR_NGROK_URL_HERE"
    ```

    > [!IMPORTANT]
    > 如果重新启动 ngrok，则需要重复此命令以更新 ngrok URL。

1. 将 CLI 中的当前目录更改为 **./GraphTu一l** 目录，并运行以下命令以在本地启动 Azure Function。

    ```Shell
    func start
    ```

1. 刷新 SPA 并选择" **订阅"** 导航项。 输入组织中具有邮箱的用户Microsoft 365用户EXCHANGE ONLINE ID。 这可以是来自 Microsoft (`id` 的用户Graph) 或用户的 `userPrincipalName` 。 单击 **订阅**。

1. 页面将刷新，在表中显示新订阅。

1. 向用户发送电子邮件。 过一小段时间， `Notify` 应调用 函数。 可以在 ngrok Web 界面 () Azure Function 项目的调试输出 `http://localhost:4040` 中对此进行验证。

    ```Shell
    ...
    [7/8/2020 7:33:57 PM] The following message was created:
    [7/8/2020 7:33:57 PM] Subject: Hi Megan!, ID: AAMkAGUyN2I4N2RlLTEzMTAtNDBmYy1hODdlLTY2NTQwODE2MGEwZgBGAAAAAAA2J9QH-DvMRK3pBt_8rA6nBwCuPIFjbMEkToHcVnQirM5qAAAAAAEMAACuPIFjbMEkToHcVnQirM5qAACHmpAsAAA=
    [7/8/2020 7:33:57 PM] Executed 'Notify' (Succeeded, Id=9c40af0b-e082-4418-aa3a-aee624f30e7a)
    ...
    ```

1. 在测试应用中，单击 **订阅** 的表行中的"删除"。 页面将刷新，并且订阅不再位于表中。
