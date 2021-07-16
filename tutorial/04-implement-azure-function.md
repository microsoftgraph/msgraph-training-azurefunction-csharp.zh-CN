---
ms.openlocfilehash: 3e6a83c19de68b6047914a68d94e66dbab95c6c4
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445953"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="05c99-101">在此练习中，你将完成 Azure 函数的实现 `GetMyNewestMessage` ，并更新测试客户端以调用 函数。</span><span class="sxs-lookup"><span data-stu-id="05c99-101">In this exercise you will finish implementing the Azure Function `GetMyNewestMessage` and update the test client to call the function.</span></span>

<span data-ttu-id="05c99-102">Azure 函数 [使用代表流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)。</span><span class="sxs-lookup"><span data-stu-id="05c99-102">The Azure Function uses the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span></span> <span data-ttu-id="05c99-103">此流中事件的基本顺序为：</span><span class="sxs-lookup"><span data-stu-id="05c99-103">The basic order of events in this flow are:</span></span>

- <span data-ttu-id="05c99-104">测试应用程序使用交互式身份验证流来允许用户登录并授予同意。</span><span class="sxs-lookup"><span data-stu-id="05c99-104">The test application uses an interactive auth flow to allow the user to sign in and grant consent.</span></span> <span data-ttu-id="05c99-105">它将返回一个作用域为 Azure 函数的令牌。</span><span class="sxs-lookup"><span data-stu-id="05c99-105">It gets back a token that is scoped to the Azure Function.</span></span> <span data-ttu-id="05c99-106">令牌不包含 **任何** Microsoft Graph作用域。</span><span class="sxs-lookup"><span data-stu-id="05c99-106">The token does **NOT** contain any Microsoft Graph scopes.</span></span>
- <span data-ttu-id="05c99-107">测试应用程序调用 Azure Function，在 标头中发送其访问 `Authorization` 令牌。</span><span class="sxs-lookup"><span data-stu-id="05c99-107">The test application invokes the Azure Function, sending its access token in the `Authorization` header.</span></span>
- <span data-ttu-id="05c99-108">Azure 函数验证令牌，然后将该令牌交换为包含 Microsoft 作用域的第二个Graph令牌。</span><span class="sxs-lookup"><span data-stu-id="05c99-108">The Azure Function validates the token, then exchanges that token for a second access token that contains Microsoft Graph scopes.</span></span>
- <span data-ttu-id="05c99-109">Azure 函数Graph第二个访问令牌代表用户调用 Microsoft 服务。</span><span class="sxs-lookup"><span data-stu-id="05c99-109">The Azure Function calls Microsoft Graph on the user's behalf using the second access token.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="05c99-110">为了避免在源中存储应用程序 ID 和密码，你将使用 [.NET 密码管理器](https://docs.microsoft.com/aspnet/core/security/app-secrets) 存储这些值。</span><span class="sxs-lookup"><span data-stu-id="05c99-110">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="05c99-111">密码管理器仅供开发使用，生产应用应该使用受信任的密码管理器来存储密码。</span><span class="sxs-lookup"><span data-stu-id="05c99-111">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

## <a name="add-authentication-to-the-single-page-application"></a><span data-ttu-id="05c99-112">向单页应用程序添加身份验证</span><span class="sxs-lookup"><span data-stu-id="05c99-112">Add authentication to the single page application</span></span>

<span data-ttu-id="05c99-113">首先将身份验证添加到 SPA。</span><span class="sxs-lookup"><span data-stu-id="05c99-113">Start by adding authentication to the SPA.</span></span> <span data-ttu-id="05c99-114">这将允许应用程序获取访问令牌，以授予调用 Azure Function 的访问权限。</span><span class="sxs-lookup"><span data-stu-id="05c99-114">This will allow the application to get an access token granting access to call the Azure Function.</span></span> <span data-ttu-id="05c99-115">因为这是一个 SPA，它将授权 [代码流与 PKCE 一同使用](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)。</span><span class="sxs-lookup"><span data-stu-id="05c99-115">Because this is a SPA, it will use the [authorization code flow with PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span></span>

1. <span data-ttu-id="05c99-116">在 **TestClient** 目录中新建一个名为 **config.js** 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="05c99-116">Create a new file in the **TestClient** directory named **config.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    <span data-ttu-id="05c99-117">将 `YOUR_TEST_APP_APP_ID_HERE` 替换为你在 Azure 门户中创建的应用程序 ID，Graph **Azure Function Test App**。</span><span class="sxs-lookup"><span data-stu-id="05c99-117">Replace `YOUR_TEST_APP_APP_ID_HERE` with the application ID you created in the Azure portal for the **Graph Azure Function Test App**.</span></span> <span data-ttu-id="05c99-118">将 替换为从 Azure `YOUR_TENANT_ID_HERE` **(复制) 的 Directory** 租户租户 ID 值。</span><span class="sxs-lookup"><span data-stu-id="05c99-118">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span> <span data-ttu-id="05c99-119">将 `YOUR_AZURE_FUNCTION_APP_ID_HERE` 替换为 Azure Function Graph **ID。**</span><span class="sxs-lookup"><span data-stu-id="05c99-119">Replace `YOUR_AZURE_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="05c99-120">如果你使用的是源代码管理（如 git），那么现在应该将 **config.js** 文件从源代码管理中排除，以避免意外泄露应用 ID 和租户 ID。</span><span class="sxs-lookup"><span data-stu-id="05c99-120">If you're using source control such as git, now would be a good time to exclude the **config.js** file from source control to avoid inadvertently leaking your app IDs and tenant ID.</span></span>

1. <span data-ttu-id="05c99-121">在 **TestClient** 目录中新建一个名为 **auth.js** 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="05c99-121">Create a new file in the **TestClient** directory named **auth.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    <span data-ttu-id="05c99-122">考虑此代码执行哪些功能。</span><span class="sxs-lookup"><span data-stu-id="05c99-122">Consider what this code does.</span></span>

    - <span data-ttu-id="05c99-123">它使用存储在 `PublicClientApplication`config.js中的 **值进行初始化**。</span><span class="sxs-lookup"><span data-stu-id="05c99-123">It initializes a `PublicClientApplication` using the values stored in **config.js**.</span></span>
    - <span data-ttu-id="05c99-124">它 `loginPopup` 使用 Azure 函数的权限范围登录用户。</span><span class="sxs-lookup"><span data-stu-id="05c99-124">It uses `loginPopup` to sign the user in, using the permission scope for the Azure Function.</span></span>
    - <span data-ttu-id="05c99-125">它将用户的用户名存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="05c99-125">It stores the user's username in the session.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="05c99-126">由于应用使用 ，因此可能需要更改浏览器的弹出窗口阻止程序，以 `loginPopup` 允许弹出窗口 `http://localhost:8080` 。</span><span class="sxs-lookup"><span data-stu-id="05c99-126">Since the app uses `loginPopup`, you may need to change your browser's pop-up blocker to allow pop-ups from `http://localhost:8080`.</span></span>

1. <span data-ttu-id="05c99-127">刷新页面并登录。</span><span class="sxs-lookup"><span data-stu-id="05c99-127">Refresh the page and sign in.</span></span> <span data-ttu-id="05c99-128">页面应该使用用户名进行更新，指示登录成功。</span><span class="sxs-lookup"><span data-stu-id="05c99-128">The page should update with the user name, indicating that the sign in was successful.</span></span>

## <a name="add-authentication-to-the-azure-function"></a><span data-ttu-id="05c99-129">向 Azure 函数添加身份验证</span><span class="sxs-lookup"><span data-stu-id="05c99-129">Add authentication to the Azure Function</span></span>

<span data-ttu-id="05c99-130">在此部分中，你将在 Azure 函数中实现代表流，以获得与 `GetMyNewestMessage` Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="05c99-130">In this section you'll implement the on-behalf-of flow in the `GetMyNewestMessage` Azure Function to get an access token compatible with Microsoft Graph.</span></span>

1. <span data-ttu-id="05c99-131">在包含 **GraphTu一l.csproj** 的目录中打开 CLI 并运行以下命令，初始化 .NET 开发密码存储。</span><span class="sxs-lookup"><span data-stu-id="05c99-131">Initialize the .NET development secret store by opening your CLI in the directory that contains **GraphTutorial.csproj** and running the following command.</span></span>

    ```Shell
    dotnet user-secrets init
    ```

1. <span data-ttu-id="05c99-132">使用下列命令将应用程序 ID、机密和租户 ID 添加到密码存储。</span><span class="sxs-lookup"><span data-stu-id="05c99-132">Add your application ID, secret, and tenant ID to the secret store using the following commands.</span></span> <span data-ttu-id="05c99-133">将 `YOUR_API_FUNCTION_APP_ID_HERE` 替换为 Azure Function Graph **ID。**</span><span class="sxs-lookup"><span data-stu-id="05c99-133">Replace `YOUR_API_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span> <span data-ttu-id="05c99-134">将 替换为你在 Azure 门户中为 Azure Function Graph `YOUR_API_FUNCTION_APP_SECRET_HERE` **密码**。</span><span class="sxs-lookup"><span data-stu-id="05c99-134">Replace `YOUR_API_FUNCTION_APP_SECRET_HERE` with the application secret you created in the Azure portal for the **Graph Azure Function**.</span></span> <span data-ttu-id="05c99-135">将 替换为从 Azure `YOUR_TENANT_ID_HERE` **(复制) 的 Directory** 租户租户 ID 值。</span><span class="sxs-lookup"><span data-stu-id="05c99-135">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span>

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a><span data-ttu-id="05c99-136">处理传入的 bearer 令牌</span><span class="sxs-lookup"><span data-stu-id="05c99-136">Process the incoming bearer token</span></span>

<span data-ttu-id="05c99-137">在此部分中，你将实现一个类，以验证并处理从 SPA 发送到 Azure 函数的令牌。</span><span class="sxs-lookup"><span data-stu-id="05c99-137">In this section you'll implement a class to validate and process the bearer token sent from the SPA to the Azure Function.</span></span>

1. <span data-ttu-id="05c99-138">在 **GraphTu一** l 目录中新建一个名为 Authentication **的目录**。</span><span class="sxs-lookup"><span data-stu-id="05c99-138">Create a new directory in the **GraphTutorial** directory named **Authentication**.</span></span>

1. <span data-ttu-id="05c99-139">在 **./GraphTu一l/Authentication** 文件夹中创建一个名为 **TokenValidationResult.cs** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="05c99-139">Create a new file named **TokenValidationResult.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. <span data-ttu-id="05c99-140">在 **./GraphTu一l/Authentication** 文件夹中创建一个名为 **TokenValidation.cs** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="05c99-140">Create a new file named **TokenValidation.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

<span data-ttu-id="05c99-141">考虑此代码执行哪些功能。</span><span class="sxs-lookup"><span data-stu-id="05c99-141">Consider what this code does.</span></span>

- <span data-ttu-id="05c99-142">它确保 标头中具有一个 bearer `Authorization` 标记。</span><span class="sxs-lookup"><span data-stu-id="05c99-142">It ensure there is a bearer token in the `Authorization` header.</span></span>
- <span data-ttu-id="05c99-143">它验证 Azure 已发布 OpenID 配置中的签名和颁发者。</span><span class="sxs-lookup"><span data-stu-id="05c99-143">It verifies the signature and issuer from Azure's published OpenID configuration.</span></span>
- <span data-ttu-id="05c99-144">它验证声明 (`aud` 是否) Azure Function 的应用程序 ID 匹配。</span><span class="sxs-lookup"><span data-stu-id="05c99-144">It verifies that the audience (`aud` claim) matches the Azure Function's application ID.</span></span>
- <span data-ttu-id="05c99-145">它分析令牌并生成 MSAL 帐户 ID，这是利用令牌缓存所需要的。</span><span class="sxs-lookup"><span data-stu-id="05c99-145">It parses the token and generates an MSAL account ID, which will be needed to take advantage of token caching.</span></span>

### <a name="create-an-on-behalf-of-authentication-provider"></a><span data-ttu-id="05c99-146">创建代表身份验证提供程序</span><span class="sxs-lookup"><span data-stu-id="05c99-146">Create an on-behalf-of authentication provider</span></span>

1. <span data-ttu-id="05c99-147">在身份验证目录中新建一个名为 **OnBehalfOfAuthProvider.cs** 的文件，然后向该文件中添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="05c99-147">Create a new file in the **Authentication** directory named **OnBehalfOfAuthProvider.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

<span data-ttu-id="05c99-148">花些时间考虑 **OnBehalfOfAuthProvider.cs 中的代码** 有什么功能。</span><span class="sxs-lookup"><span data-stu-id="05c99-148">Take a moment to consider what the code in **OnBehalfOfAuthProvider.cs** does.</span></span>

- <span data-ttu-id="05c99-149">在 `GetAccessToken` 函数中，它首先尝试使用 从令牌缓存获取用户令牌 `AcquireTokenSilent` 。</span><span class="sxs-lookup"><span data-stu-id="05c99-149">In the `GetAccessToken` function, it first attempts to get a user token from the token cache using `AcquireTokenSilent`.</span></span> <span data-ttu-id="05c99-150">如果此操作失败，它将使用测试应用发送到 Azure Function 的 bearer 令牌生成用户断言。</span><span class="sxs-lookup"><span data-stu-id="05c99-150">If this fails, it uses the bearer token sent by the test app to the Azure Function to generate a user assertion.</span></span> <span data-ttu-id="05c99-151">然后，它使用该用户断言获取Graph兼容的令牌 `AcquireTokenOnBehalfOf` 。</span><span class="sxs-lookup"><span data-stu-id="05c99-151">It then uses that user assertion to get a Graph-compatible token using `AcquireTokenOnBehalfOf`.</span></span>
- <span data-ttu-id="05c99-152">它实现 接口，从而允许在 的构造函数中传递此类 `Microsoft.Graph.IAuthenticationProvider` `GraphServiceClient` 以对传出请求进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="05c99-152">It implements the `Microsoft.Graph.IAuthenticationProvider` interface, allowing this class to be passed in the constructor of the `GraphServiceClient` to authenticate outgoing requests.</span></span>

### <a name="implement-a-graph-client-service"></a><span data-ttu-id="05c99-153">实现Graph客户端服务</span><span class="sxs-lookup"><span data-stu-id="05c99-153">Implement a Graph client service</span></span>

<span data-ttu-id="05c99-154">在此部分中，你将实现可注册用于依赖 [关系注入的服务](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)。</span><span class="sxs-lookup"><span data-stu-id="05c99-154">In this section you'll implement a service that can be registered for [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span></span> <span data-ttu-id="05c99-155">该服务将用于获取经过身份验证Graph客户端。</span><span class="sxs-lookup"><span data-stu-id="05c99-155">The service will be used to get an authenticated Graph client.</span></span>

1. <span data-ttu-id="05c99-156">在 **GraphTu一** l 目录中新建一个名为 Services 的 **目录**。</span><span class="sxs-lookup"><span data-stu-id="05c99-156">Create a new directory in the **GraphTutorial** directory named **Services**.</span></span>

1. <span data-ttu-id="05c99-157">在"服务"目录中新建一个名为 **IGraphClientService.cs** 的文件，然后向该文件中添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="05c99-157">Create a new file in the **Services** directory named **IGraphClientService.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. <span data-ttu-id="05c99-158">在"服务"目录中新建一个名为 **GraphClientService.cs** 的文件，然后向该文件中添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="05c99-158">Create a new file in the **Services** directory named **GraphClientService.cs** and add the following code to that file.</span></span>

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

1. <span data-ttu-id="05c99-159">将以下属性添加到 `GraphClientService` 类。</span><span class="sxs-lookup"><span data-stu-id="05c99-159">Add the following properties to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. <span data-ttu-id="05c99-160">将以下函数添加到 `GraphClientService` 类。</span><span class="sxs-lookup"><span data-stu-id="05c99-160">Add the following functions to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. <span data-ttu-id="05c99-161">为 函数添加占位符 `GetAppGraphClient` 实现。</span><span class="sxs-lookup"><span data-stu-id="05c99-161">Add a placeholder implementation for the `GetAppGraphClient` function.</span></span> <span data-ttu-id="05c99-162">您将在稍后部分中实现此方案。</span><span class="sxs-lookup"><span data-stu-id="05c99-162">You will implement that in later sections.</span></span>

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    <span data-ttu-id="05c99-163">`GetUserGraphClient`函数获取令牌验证的结果，并生成一个经过身份验证 `GraphServiceClient` 的用户。</span><span class="sxs-lookup"><span data-stu-id="05c99-163">The `GetUserGraphClient` function takes the results of token validation and builds an authenticated `GraphServiceClient` for the user.</span></span>

1. <span data-ttu-id="05c99-164">打开 **./GraphTu一l/Program.cs，** 并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="05c99-164">Open **./GraphTutorial/Program.cs** and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="ProgramSnippet" highlight="15-23":::

    <span data-ttu-id="05c99-165">此代码将用户密码添加到配置 [，并启用](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) Azure 函数中的依赖项注入，从而公开 `GraphClientService` 服务。</span><span class="sxs-lookup"><span data-stu-id="05c99-165">This code will add user secrets to the configuration, and enable [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in your Azure Functions, exposing the `GraphClientService` service.</span></span>

### <a name="implement-getmynewestmessage-function"></a><span data-ttu-id="05c99-166">实现 GetMyNewestMessage 函数</span><span class="sxs-lookup"><span data-stu-id="05c99-166">Implement GetMyNewestMessage function</span></span>

1. <span data-ttu-id="05c99-167">打开 **./GraphTu一l/GetMyNewestMessage.cs，** 并将其全部内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="05c99-167">Open **./GraphTutorial/GetMyNewestMessage.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a><span data-ttu-id="05c99-168">查看 GetMyNewestMessage.cs 中的代码</span><span class="sxs-lookup"><span data-stu-id="05c99-168">Review the code in GetMyNewestMessage.cs</span></span>

<span data-ttu-id="05c99-169">花些时间考虑 **GetMyNewestMessage.cs** 中的代码有什么功能。</span><span class="sxs-lookup"><span data-stu-id="05c99-169">Take a moment to consider what the code in **GetMyNewestMessage.cs** does.</span></span>

- <span data-ttu-id="05c99-170">在构造函数中，它保存通过 `IConfiguration` `IGraphClientService` 依赖关系注入传入的 和 对象。</span><span class="sxs-lookup"><span data-stu-id="05c99-170">In the constructor, it saves the `IConfiguration` and `IGraphClientService` objects passed in via dependency injection.</span></span>
- <span data-ttu-id="05c99-171">在 `Run` 函数中，它执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="05c99-171">In the `Run` function, it does the following:</span></span>
  - <span data-ttu-id="05c99-172">验证对象中是否包含所需的配置 `IConfiguration` 值。</span><span class="sxs-lookup"><span data-stu-id="05c99-172">Validates the required configuration values are present in the `IConfiguration` object.</span></span>
  - <span data-ttu-id="05c99-173">验证 bearer 令牌，如果 `401` 令牌无效，则返回状态代码。</span><span class="sxs-lookup"><span data-stu-id="05c99-173">Validates the bearer token and returns a `401` status code if the token is invalid.</span></span>
  - <span data-ttu-id="05c99-174">从 Graph `GraphClientService` 请求的用户获取一个客户端。</span><span class="sxs-lookup"><span data-stu-id="05c99-174">Gets a Graph client from the `GraphClientService` for the user that made this request.</span></span>
  - <span data-ttu-id="05c99-175">使用 Microsoft Graph SDK 从用户收件箱获取最新邮件，并作为响应中的 JSON 正文返回。</span><span class="sxs-lookup"><span data-stu-id="05c99-175">Uses the Microsoft Graph SDK to get the newest message from the user's inbox and returns it as a JSON body in the response.</span></span>

## <a name="call-the-azure-function-from-the-test-app"></a><span data-ttu-id="05c99-176">从测试应用调用 Azure 函数</span><span class="sxs-lookup"><span data-stu-id="05c99-176">Call the Azure Function from the test app</span></span>

1. <span data-ttu-id="05c99-177">打开 **auth.js** 并添加以下函数，获取访问令牌。</span><span class="sxs-lookup"><span data-stu-id="05c99-177">Open **auth.js** and add the following function to get an access token.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    <span data-ttu-id="05c99-178">考虑此代码执行哪些功能。</span><span class="sxs-lookup"><span data-stu-id="05c99-178">Consider what this code does.</span></span>

    - <span data-ttu-id="05c99-179">它首先尝试以静默方式获取访问令牌，而无需用户交互。</span><span class="sxs-lookup"><span data-stu-id="05c99-179">It first attempts to get an access token silently, without user interaction.</span></span> <span data-ttu-id="05c99-180">由于用户应该已经登录，因此 MSAL 的缓存中应包含用户令牌。</span><span class="sxs-lookup"><span data-stu-id="05c99-180">Since the user should already be signed in, MSAL should have tokens for the user in its cache.</span></span>
    - <span data-ttu-id="05c99-181">如果失败并出现错误，指示用户需要交互，它将尝试以交互方式获取令牌。</span><span class="sxs-lookup"><span data-stu-id="05c99-181">If that fails with an error that indicates the user needs to interact, it attempts to get a token interactively.</span></span>

    > [!TIP]
    > <span data-ttu-id="05c99-182">可以在 中分析访问令牌，并确认声明是 Azure Function 的应用 ID，并且声明包含 Azure Function 的权限范围，而不是 [https://jwt.ms](https://jwt.ms) `aud` Microsoft `scp` Graph。</span><span class="sxs-lookup"><span data-stu-id="05c99-182">You can parse the access token at [https://jwt.ms](https://jwt.ms) and confirm that the `aud` claim is the app ID for the Azure Function, and that the `scp` claim contains the Azure Function's permission scope, not Microsoft Graph.</span></span>

1. <span data-ttu-id="05c99-183">在 **TestClient** 目录中新建一个名为 **azurefunctions.js** 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="05c99-183">Create a new file in the **TestClient** directory named **azurefunctions.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. <span data-ttu-id="05c99-184">将 CLI 中的当前目录更改为 **./GraphTu一l** 目录，并运行以下命令以在本地启动 Azure Function。</span><span class="sxs-lookup"><span data-stu-id="05c99-184">Change the current directory in your CLI to the **./GraphTutorial** directory and run the following command to start the Azure Function locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="05c99-185">如果尚未提供 SPA，请打开第二个 CLI 窗口，将当前目录更改为 **./TestClient** 目录。</span><span class="sxs-lookup"><span data-stu-id="05c99-185">If not already serving the SPA, open a second CLI window and change the current directory to the **./TestClient** directory.</span></span> <span data-ttu-id="05c99-186">运行以下命令以运行测试应用程序。</span><span class="sxs-lookup"><span data-stu-id="05c99-186">Run the following command to run the test application.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="05c99-187">打开浏览器，并导航到 `http://localhost:8080`。</span><span class="sxs-lookup"><span data-stu-id="05c99-187">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="05c99-188">登录并选择"最新 **邮件"** 导航项。</span><span class="sxs-lookup"><span data-stu-id="05c99-188">Sign in and select the **Latest Message** navigation item.</span></span> <span data-ttu-id="05c99-189">应用程序显示有关用户收件箱中最新邮件的信息。</span><span class="sxs-lookup"><span data-stu-id="05c99-189">The app displays information about the newest message in the user's inbox.</span></span>
