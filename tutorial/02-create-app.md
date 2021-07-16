---
ms.openlocfilehash: 914957809f268ad29f8cfc44c21dd9fb63699322
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445960"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="091ef-101">在本教程中，你将创建一个简单的 Azure 函数，该函数实现调用 Microsoft Graph 的 HTTP 触发器Graph。</span><span class="sxs-lookup"><span data-stu-id="091ef-101">In this tutorial, you will create a simple Azure Function that implements HTTP trigger functions that call Microsoft Graph.</span></span> <span data-ttu-id="091ef-102">这些函数将涵盖以下方案：</span><span class="sxs-lookup"><span data-stu-id="091ef-102">These functions will cover the following scenarios:</span></span>

- <span data-ttu-id="091ef-103">实现 API 以使用代表流身份验证访问 [用户的](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) 收件箱。</span><span class="sxs-lookup"><span data-stu-id="091ef-103">Implements an API to access a user's inbox using [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) authentication.</span></span>
- <span data-ttu-id="091ef-104">使用客户端凭据授予流身份验证，实现用于订阅和取消订阅用户收件箱通知的 API。 [](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)</span><span class="sxs-lookup"><span data-stu-id="091ef-104">Implements an API to subscribe and unsubscribe for notifications on a user's inbox, using using [client credentials grant flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) authentication.</span></span>
- <span data-ttu-id="091ef-105">实现 Webhook[以接收来自](https://docs.microsoft.com/graph/webhooks)Microsoft Graph更改通知，以及使用客户端凭据授予流访问数据。</span><span class="sxs-lookup"><span data-stu-id="091ef-105">Implements a webhook to receive [change notifications](https://docs.microsoft.com/graph/webhooks) from Microsoft Graph and access data using client credentials grant flow.</span></span>

<span data-ttu-id="091ef-106">此外，你还将在 SPA (创建简单的 JavaScript 单页) ，以调用在 Azure 函数中实现的 API。</span><span class="sxs-lookup"><span data-stu-id="091ef-106">You will also create a simple JavaScript single-page application (SPA) to call the APIs implemented in the Azure Function.</span></span>

## <a name="create-azure-functions-project"></a><span data-ttu-id="091ef-107">创建 Azure Functions 项目</span><span class="sxs-lookup"><span data-stu-id="091ef-107">Create Azure Functions project</span></span>

1. <span data-ttu-id="091ef-108">在要创建项目的 (CLI) 打开命令行接口。</span><span class="sxs-lookup"><span data-stu-id="091ef-108">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="091ef-109">运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="091ef-109">Run the following command.</span></span>

    ```Shell
    func init GraphTutorial --worker-runtime dotnetisolated
    ```

1. <span data-ttu-id="091ef-110">将 CLI 中的当前目录更改为 **GraphTu一l** 目录，并运行以下命令在项目中创建三个函数。</span><span class="sxs-lookup"><span data-stu-id="091ef-110">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands to create three functions in the project.</span></span>

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger"
    func new --name SetSubscription --template "HTTP trigger"
    func new --name Notify --template "HTTP trigger"
    ```

1. <span data-ttu-id="091ef-111">打开 **local.settings.js，** 将以下内容添加到 文件以允许来自 的测试应用程序的 URL 中的 `http://localhost:8080` CORS。</span><span class="sxs-lookup"><span data-stu-id="091ef-111">Open **local.settings.json** and add the following to the file to allow CORS from `http://localhost:8080`, the URL for the test application.</span></span>

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. <span data-ttu-id="091ef-112">运行以下命令以在本地运行项目。</span><span class="sxs-lookup"><span data-stu-id="091ef-112">Run the following command to run the project locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="091ef-113">如果一切正常，你将看到以下输出：</span><span class="sxs-lookup"><span data-stu-id="091ef-113">If everything is working, you will see the following output:</span></span>

    ```Shell
    Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. <span data-ttu-id="091ef-114">打开浏览器并浏览到输出中显示的函数 URL，验证函数是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="091ef-114">Verify that the functions are working correctly by opening your browser and browsing to the function URLs shown in the output.</span></span> <span data-ttu-id="091ef-115">您应该在浏览器中看到以下消息 `Welcome to Azure Functions!` ：。</span><span class="sxs-lookup"><span data-stu-id="091ef-115">You should see the following message in your browser: `Welcome to Azure Functions!`.</span></span>

## <a name="create-single-page-application"></a><span data-ttu-id="091ef-116">创建单页应用程序</span><span class="sxs-lookup"><span data-stu-id="091ef-116">Create single-page application</span></span>

1. <span data-ttu-id="091ef-117">在要创建项目的目录中打开 CLI。</span><span class="sxs-lookup"><span data-stu-id="091ef-117">Open your CLI in a directory where you want to create the project.</span></span> <span data-ttu-id="091ef-118">创建一个名为 **TestClient 的** 目录以保存 HTML 和 JavaScript 文件。</span><span class="sxs-lookup"><span data-stu-id="091ef-118">Create a directory named **TestClient** to hold your HTML and JavaScript files.</span></span>

1. <span data-ttu-id="091ef-119">在 **TestClient** **index.htm创建** 一个名为index.html 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="091ef-119">Create a new file named **index.html** in the **TestClient** directory and add the following code.</span></span>

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    <span data-ttu-id="091ef-120">这将定义应用的基本布局，包括导航栏。</span><span class="sxs-lookup"><span data-stu-id="091ef-120">This defines the basic layout of the app, including a navigation bar.</span></span> <span data-ttu-id="091ef-121">它还添加了以下内容：</span><span class="sxs-lookup"><span data-stu-id="091ef-121">It also adds the following:</span></span>

    - <span data-ttu-id="091ef-122">[Bootstrap](https://getbootstrap.com/) 及其支持的 JavaScript</span><span class="sxs-lookup"><span data-stu-id="091ef-122">[Bootstrap](https://getbootstrap.com/) and its supporting JavaScript</span></span>
    - [<span data-ttu-id="091ef-123">Font提供</span><span class="sxs-lookup"><span data-stu-id="091ef-123">FontAwesome</span></span>](https://fontawesome.com/)
    - [<span data-ttu-id="091ef-124">适用于 JavaScript 2.0 (MSAL.js) Microsoft 身份验证库</span><span class="sxs-lookup"><span data-stu-id="091ef-124">Microsoft Authentication Library for JavaScript (MSAL.js) 2.0</span></span>](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > <span data-ttu-id="091ef-125">此页面包括一个 favicon、 (`<link rel="shortcut icon" href="g-raph.png">`) 。</span><span class="sxs-lookup"><span data-stu-id="091ef-125">The page includes a favicon, (`<link rel="shortcut icon" href="g-raph.png">`).</span></span> <span data-ttu-id="091ef-126">可以删除此行，也可以从g-raph.png **下载** GitHub。 [](https://github.com/microsoftgraph/g-raph)</span><span class="sxs-lookup"><span data-stu-id="091ef-126">You can remove this line, or you can download the **g-raph.png** file from [GitHub](https://github.com/microsoftgraph/g-raph).</span></span>

1. <span data-ttu-id="091ef-127">在 **TestClient** 目录中创建一个名为 **style.css** 的新文件并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="091ef-127">Create a new file named **style.css** in the **TestClient** directory and add the following code.</span></span>

    :::code language="css" source="../demo/TestClient/style.css":::

1. <span data-ttu-id="091ef-128">在 **TestClient** **ui.js** 一个名为ui.js文件并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="091ef-128">Create a new file named **ui.js** in the **TestClient** directory and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    <span data-ttu-id="091ef-129">此代码使用 JavaScript 根据所选视图呈现当前页面。</span><span class="sxs-lookup"><span data-stu-id="091ef-129">This code uses JavaScript to render the current page based on the selected view.</span></span>

### <a name="test-the-single-page-application"></a><span data-ttu-id="091ef-130">测试单页应用程序</span><span class="sxs-lookup"><span data-stu-id="091ef-130">Test the single-page application</span></span>

> [!NOTE]
> <span data-ttu-id="091ef-131">本节包含有关使用 [dotnet-serve 在](https://github.com/natemcmaster/dotnet-serve) 开发计算机上运行简单测试 HTTP 服务器的说明。</span><span class="sxs-lookup"><span data-stu-id="091ef-131">This section includes instructions for using [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) to run a simple testing HTTP server on your development machine.</span></span> <span data-ttu-id="091ef-132">不需要使用此特定工具。</span><span class="sxs-lookup"><span data-stu-id="091ef-132">Using this specific tool is not required.</span></span> <span data-ttu-id="091ef-133">可以使用您喜欢的任何测试服务器来为 **TestClient** 目录提供服务。</span><span class="sxs-lookup"><span data-stu-id="091ef-133">You can use any testing server you prefer to serve the **TestClient** directory.</span></span>

1. <span data-ttu-id="091ef-134">在 CLI 中运行以下命令以安装 **dotnet-serve**。</span><span class="sxs-lookup"><span data-stu-id="091ef-134">Run the following command in your CLI to install **dotnet-serve**.</span></span>

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. <span data-ttu-id="091ef-135">将 CLI 中的当前目录更改为 **TestClient** 目录并运行以下命令以启动 HTTP 服务器。</span><span class="sxs-lookup"><span data-stu-id="091ef-135">Change the current directory in your CLI to the **TestClient** directory and run the following command to start an HTTP server.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. <span data-ttu-id="091ef-136">打开浏览器，并导航到 `http://localhost:8080`。</span><span class="sxs-lookup"><span data-stu-id="091ef-136">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="091ef-137">页面应呈现，但当前没有任何按钮可正常工作。</span><span class="sxs-lookup"><span data-stu-id="091ef-137">The page should render, but none of the buttons currently work.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="091ef-138">添加 Nuget 程序包</span><span class="sxs-lookup"><span data-stu-id="091ef-138">Add NuGet packages</span></span>

<span data-ttu-id="091ef-139">在继续之前，请安装一些NuGet包，你稍后会使用它。</span><span class="sxs-lookup"><span data-stu-id="091ef-139">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="091ef-140">[Microsoft.Azure.Functions.Extensions，](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) 用于启用 Azure Functions 项目中的依赖关系注入。</span><span class="sxs-lookup"><span data-stu-id="091ef-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) to enable dependency injection in the Azure Functions project.</span></span>
- <span data-ttu-id="091ef-141">[Microsoft.Extensions.Configuration。UserSecrets，](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) 用于读取 .NET 开发密码存储 [中的应用程序配置](https://docs.microsoft.com/aspnet/core/security/app-secrets)。</span><span class="sxs-lookup"><span data-stu-id="091ef-141">[Microsoft.Extensions.Configuration.UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) to read application configuration from the [.NET development secret store](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span></span>
- <span data-ttu-id="091ef-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) 用来呼叫 Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="091ef-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="091ef-143">[用于验证和管理令牌的 Microsoft.Identity.Client。](https://www.nuget.org/packages/Microsoft.Identity.Client/)</span><span class="sxs-lookup"><span data-stu-id="091ef-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) for authenticating and managing tokens.</span></span>
- <span data-ttu-id="091ef-144">用于检索 OpenID 配置以用于令牌验证的[Microsoft.IdentityModel.Protocols.OpenIdConnect。](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect)</span><span class="sxs-lookup"><span data-stu-id="091ef-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) for retrieving OpenID configuration for token validation.</span></span>
- <span data-ttu-id="091ef-145">[System.IdentityModel.Tokens.Jwt，](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) 用于验证发送到 Web API 的令牌。</span><span class="sxs-lookup"><span data-stu-id="091ef-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) for validating tokens sent to the web API.</span></span>

1. <span data-ttu-id="091ef-146">将 CLI 中的当前目录更改为 **GraphTu一l** 目录并运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="091ef-146">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands.</span></span>

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.1.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 5.0.0
    dotnet add package Microsoft.Graph --version 4.0.0
    dotnet add package Microsoft.Identity.Client --version 4.34.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.11.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.11.1
    ```
