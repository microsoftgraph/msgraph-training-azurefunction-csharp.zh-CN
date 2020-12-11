---
ms.openlocfilehash: eb227079656e2a57550511c3abfacb49935fe46a
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655238"
---
<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将完成实现 Azure 函数和 ，并更新测试应用程序以订阅和取消订阅用户收件箱 `SetSubscription` `Notify` 中的更改。

- 该 `SetSubscription` 函数将充当 API，允许测试应用创建或删除用户收件箱中更改的[](https://docs.microsoft.com/graph/webhooks)订阅。
- 该 `Notify` 函数将充当接收订阅生成的更改通知的 webhook。

这两个函数都将 [使用客户端凭据授予流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) 获取仅应用令牌来调用 Microsoft Graph。 由于管理员已授予管理员对所需权限范围的许可，因此无需用户交互获取令牌。

## <a name="add-client-credentials-authentication-to-the-azure-functions-project"></a>将客户端凭据身份验证添加到 Azure Functions 项目

在此部分中，你将在 Azure Functions 项目中实现客户端凭据流，以获得与 Microsoft Graph 兼容的访问令牌。

1. 在包含 **GraphTu一l.csproj 的目录中打开 CLI。**

1. 使用下列命令将 Webhook 应用程序 ID 和密码添加到密码存储。 替换为 Graph Azure 函数 `YOUR_WEBHOOK_APP_ID_HERE` **Webhook 的应用程序 ID。** 替换为你在 Azure 门户中为 `YOUR_WEBHOOK_APP_SECRET_HERE` Graph Azure **Function Webhook 创建的应用程序密码**。

    ```Shell
    dotnet user-secrets set webHookId "YOUR_WEBHOOK_APP_ID_HERE"
    dotnet user-secrets set webHookSecret "YOUR_WEBHOOK_APP_SECRET_HERE"
    ```

### <a name="create-a-client-credentials-authentication-provider"></a>创建客户端凭据身份验证提供程序

1. 在名为 ClientCredentialsAuthProvider.cs 的 **./GraphTu一l/Authentication** **目录中** 创建新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/ClientCredentialsAuthProvider.cs" id="AuthProviderSnippet":::

花些时间考虑代码中的代码 **ClientCredentialsAuthProvider.cs。**

- 在构造函数中，它初始化包中的 **ConfidentialClientApplication。** `Microsoft.Identity.Client` 它使用 `WithAuthority(AadAuthorityAudience.AzureAdMyOrg, true)` and `.WithTenantId(tenantId)` 函数将登录访问群体限制为仅指定的 Microsoft 365 组织。
- 在 `GetAccessToken` 函数中，它调用 `AcquireTokenForClient` 获取应用程序的令牌。 客户端凭据令牌流始终是非交互的。
- 它实现接口 `Microsoft.Graph.IAuthenticationProvider` ，允许此类传入的构造函数中对 `GraphServiceClient` 传出请求进行身份验证。

## <a name="update-graphclientservice"></a>更新 GraphClientService

1. 打开 **GraphClientService.cs，** 然后向类中添加以下属性。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

1. 将现有的 `GetAppGraphClient` 函数替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientFunctions":::

## <a name="implement-notify-function"></a>实现 Notify 函数

在此部分中，你将实现函数，该函数 `Notify` 将用作更改通知的通知 URL。

1. 在 **GraphTu一** ls 目录中新建一个名为 Models **的目录**。

1. 在 Models 目录中新建 **一个名为****ResourceData.cs** 文件并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/ResourceData.cs" id="ResourceDataSnippet":::

1. 在 Models 目录中新建 **一个名为****ChangeNotification.cs** 并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/ChangeNotification.cs" id="ChangeNotificationSnippet":::

1. 在 Models 目录中新建 **一个名为****NotificationList.cs** 文件并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/NotificationList.cs" id="NotificationListSnippet":::

1. 打开 **./GraphTu一l/Notify.cs，** 并将其全部内容替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/Notify.cs" id="NotifySnippet":::

花些时间考虑代码中的代码 **Notify.cs** 功能。

- `Run`该函数检查是否存在查询 `validationToken` 参数。 如果该参数存在，它将请求作为验证请求 [处理](https://docs.microsoft.com/graph/webhooks#notification-endpoint-validation)，并相应地响应。
- 如果请求不是验证请求，JSON 有效负载将反初始化为 `NotificationList` 。
- 检查列表中的每个通知是否具有预期的客户端状态值，并进行处理。
- 触发通知的消息使用 Microsoft Graph 进行检索。

## <a name="implement-setsubscription-function"></a>实现 SetSubscription 函数

在此部分中，将实现 SetSubscription 函数。 此函数将充当测试应用程序调用的 API，以在用户收件箱上创建或删除订阅。

1. 在 Models 目录中创建 **一个名为****SetSubscriptionPayload.cs** 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/SetSubscriptionPayload.cs" id="SetSubscriptionPayloadSnippet":::

1. 打开 **./GraphTu一l/SetSubscription.cs，** 并将其全部内容替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/SetSubscription.cs" id="SetSubscriptionSnippet":::

花些时间考虑代码中的代码 **SetSubscription.cs** 功能。

- 该函数读取 POST 请求中发送的 JSON 负载，以确定请求类型 (订阅或取消订阅) 、要订阅的用户 ID 以及取消订阅的 `Run` 订阅 ID。
- 如果请求是订阅请求，它将使用 Microsoft Graph SDK 在指定用户的收件箱中创建新订阅。 订阅将在邮件创建或更新时通知。 新订阅在响应的 JSON 有效负载中返回。
- 如果请求是取消订阅请求，它将使用 Microsoft Graph SDK 删除指定的订阅。

## <a name="call-setsubscription-from-the-test-app"></a>从测试应用调用 SetSubscription

在此部分中，你将实现在测试应用中创建和删除订阅的函数。

1. 打开 **./TestClient/azurefunctions.js** 并添加以下函数。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="createSubscriptionSnippet":::

    此代码调用 `SetSubscription` Azure 函数以订阅并将新订阅添加到会话中的订阅数组。

1. 将以下 **函数添加到** azurefunctions.js。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="deleteSubscriptionSnippet":::

    此代码调用 Azure 函数以取消订阅并从会话中的 `SetSubscription` 订阅数组中删除订阅。

1. 如果没有运行 ngrok，请运行 ngrok `ngrok http 7071` () 复制 HTTPS 转发 URL。

1. 通过运行以下命令将 ngrok URL 添加到用户密码存储。

    ```Shell
    dotnet user-secrets set ngrokUrl "YOUR_NGROK_URL_HERE"
    ```

    > [!IMPORTANT]
    > 如果重新启动 ngrok，则需要重复此命令以更新您的 ngrok URL。

1. 将 CLI 中的当前目录更改为 **./GraphTu一l** 目录，并运行以下命令以在本地启动 Azure 函数。

    ```Shell
    func start
    ```

1. 刷新 SPA 并选择 **订阅** 导航项。 输入具有 Exchange Online 邮箱的 Microsoft 365 组织中用户的用户 ID。 这可以是用户从 Microsoft Graph (`id` 的用户) 用户。 `userPrincipalName` 单击 **"订阅"。**

1. 页面将刷新，在表中显示新订阅。

1. 向用户发送电子邮件。 过一小会， `Notify` 应调用该函数。 可以在 ngrok Web 界面 () Azure Function 项目的调试输出中验证 `http://localhost:4040` 这一点。

    ```Shell
    ...
    [7/8/2020 7:33:57 PM] The following message was created:
    [7/8/2020 7:33:57 PM] Subject: Hi Megan!, ID: AAMkAGUyN2I4N2RlLTEzMTAtNDBmYy1hODdlLTY2NTQwODE2MGEwZgBGAAAAAAA2J9QH-DvMRK3pBt_8rA6nBwCuPIFjbMEkToHcVnQirM5qAAAAAAEMAACuPIFjbMEkToHcVnQirM5qAACHmpAsAAA=
    [7/8/2020 7:33:57 PM] Executed 'Notify' (Succeeded, Id=9c40af0b-e082-4418-aa3a-aee624f30e7a)
    ...
    ```

1. 在测试应用程序中，单击 **订阅** 的表行中的"删除"。 页面将刷新，并且订阅不再位于表中。
