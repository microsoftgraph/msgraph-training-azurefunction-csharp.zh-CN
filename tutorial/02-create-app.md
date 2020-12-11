---
ms.openlocfilehash: 17c93f353c84ea2db28cd2e0203d30c5f320e36e
ms.sourcegitcommit: 141fe5c30dea84029ef61cf82558c35f2a744b65
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/11/2020
ms.locfileid: "49655231"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="763e7-101">在本教程中，你将创建一个简单的 Azure 函数，该函数实现调用 Microsoft Graph 的 HTTP 触发器函数。</span><span class="sxs-lookup"><span data-stu-id="763e7-101">In this tutorial, you will create a simple Azure Function that implements HTTP trigger functions that call Microsoft Graph.</span></span> <span data-ttu-id="763e7-102">这些函数将涵盖以下方案：</span><span class="sxs-lookup"><span data-stu-id="763e7-102">These functions will cover the following scenarios:</span></span>

- <span data-ttu-id="763e7-103">实现 API 以使用代表流身份验证访问 [用户的](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) 收件箱。</span><span class="sxs-lookup"><span data-stu-id="763e7-103">Implements an API to access a user's inbox using [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) authentication.</span></span>
- <span data-ttu-id="763e7-104">使用客户端凭据授予流身份验证，实现 API 来订阅和取消订阅用户收件箱中的[通知。](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)</span><span class="sxs-lookup"><span data-stu-id="763e7-104">Implements an API to subscribe and unsubscribe for notifications on a user's inbox, using using [client credentials grant flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) authentication.</span></span>
- <span data-ttu-id="763e7-105">实现 Webhook 以从 Microsoft Graph [接收](https://docs.microsoft.com/graph/webhooks) 更改通知，以及使用客户端凭据授予流访问数据。</span><span class="sxs-lookup"><span data-stu-id="763e7-105">Implements a webhook to receive [change notifications](https://docs.microsoft.com/graph/webhooks) from Microsoft Graph and access data using client credentials grant flow.</span></span>

<span data-ttu-id="763e7-106">你还将在 SPA (创建简单的 JavaScript 单页) 以调用在 Azure 函数中实现的 API。</span><span class="sxs-lookup"><span data-stu-id="763e7-106">You will also create a simple JavaScript single-page application (SPA) to call the APIs implemented in the Azure Function.</span></span>

## <a name="create-azure-functions-project"></a><span data-ttu-id="763e7-107">创建 Azure 函数项目</span><span class="sxs-lookup"><span data-stu-id="763e7-107">Create Azure Functions project</span></span>

1. <span data-ttu-id="763e7-108">在要创建项目的目录中 (CLI) 打开命令行界面。</span><span class="sxs-lookup"><span data-stu-id="763e7-108">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="763e7-109">运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="763e7-109">Run the following command.</span></span>

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. <span data-ttu-id="763e7-110">将 CLI 中的当前目录更改为 **GraphTu一l** 目录，并运行以下命令在项目中创建三个函数。</span><span class="sxs-lookup"><span data-stu-id="763e7-110">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands to create three functions in the project.</span></span>

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. <span data-ttu-id="763e7-111">打开 **local.settings.js，** 然后向文件添加以下内容以允许来自测试应用程序的 URL 的 `http://localhost:8080` CORS。</span><span class="sxs-lookup"><span data-stu-id="763e7-111">Open **local.settings.json** and add the following to the file to allow CORS from `http://localhost:8080`, the URL for the test application.</span></span>

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. <span data-ttu-id="763e7-112">运行以下命令以在本地运行项目。</span><span class="sxs-lookup"><span data-stu-id="763e7-112">Run the following command to run the project locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="763e7-113">如果一切正常，你将看到以下输出：</span><span class="sxs-lookup"><span data-stu-id="763e7-113">If everything is working, you will see the following output:</span></span>

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. <span data-ttu-id="763e7-114">打开浏览器并浏览到输出中显示的函数 URL，验证这些函数是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="763e7-114">Verify that the functions are working correctly by opening your browser and browsing to the function URLs shown in the output.</span></span> <span data-ttu-id="763e7-115">你应该在浏览器中看到以下 `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` 消息：</span><span class="sxs-lookup"><span data-stu-id="763e7-115">You should see the following message in your browser: `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.`.</span></span>

## <a name="create-single-page-application"></a><span data-ttu-id="763e7-116">创建单页应用程序</span><span class="sxs-lookup"><span data-stu-id="763e7-116">Create single-page application</span></span>

1. <span data-ttu-id="763e7-117">在要创建项目的目录中打开 CLI。</span><span class="sxs-lookup"><span data-stu-id="763e7-117">Open your CLI in a directory where you want to create the project.</span></span> <span data-ttu-id="763e7-118">创建一个名为 **TestClient 的** 目录来保存 HTML 和 JavaScript 文件。</span><span class="sxs-lookup"><span data-stu-id="763e7-118">Create a directory named **TestClient** to hold your HTML and JavaScript files.</span></span>

1. <span data-ttu-id="763e7-119">在 **TestClient** index.htm创建一个名为index.htm **l** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="763e7-119">Create a new file named **index.html** in the **TestClient** directory and add the following code.</span></span>

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    <span data-ttu-id="763e7-120">这将定义应用的基本布局，包括导航栏。</span><span class="sxs-lookup"><span data-stu-id="763e7-120">This defines the basic layout of the app, including a navigation bar.</span></span> <span data-ttu-id="763e7-121">它还添加以下内容：</span><span class="sxs-lookup"><span data-stu-id="763e7-121">It also adds the following:</span></span>

    - <span data-ttu-id="763e7-122">[Bootstrap](https://getbootstrap.com/) 及其支持 JavaScript</span><span class="sxs-lookup"><span data-stu-id="763e7-122">[Bootstrap](https://getbootstrap.com/) and its supporting JavaScript</span></span>
    - [<span data-ttu-id="763e7-123">FontAwesome</span><span class="sxs-lookup"><span data-stu-id="763e7-123">FontAwesome</span></span>](https://fontawesome.com/)
    - [<span data-ttu-id="763e7-124">适用于 JavaScript (MSAL.js) 2.0 的 Microsoft 身份验证库</span><span class="sxs-lookup"><span data-stu-id="763e7-124">Microsoft Authentication Library for JavaScript (MSAL.js) 2.0</span></span>](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > <span data-ttu-id="763e7-125">该页面包含一个 `<link rel="shortcut icon" href="g-raph.png">` () 。</span><span class="sxs-lookup"><span data-stu-id="763e7-125">The page includes a favicon, (`<link rel="shortcut icon" href="g-raph.png">`).</span></span> <span data-ttu-id="763e7-126">你可以删除此行，也可以从 [GitHub](https://github.com/microsoftgraph/g-raph) **g-raph.png** 文件。</span><span class="sxs-lookup"><span data-stu-id="763e7-126">You can remove this line, or you can download the **g-raph.png** file from [GitHub](https://github.com/microsoftgraph/g-raph).</span></span>

1. <span data-ttu-id="763e7-127">在 **TestClient** 目录中创建一个名为 **style.css** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="763e7-127">Create a new file named **style.css** in the **TestClient** directory and add the following code.</span></span>

    :::code language="css" source="../demo/TestClient/style.css":::

1. <span data-ttu-id="763e7-128">在 **TestClient** **ui.js** 创建一个名为ui.js的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="763e7-128">Create a new file named **ui.js** in the **TestClient** directory and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    <span data-ttu-id="763e7-129">此代码使用 JavaScript 根据所选视图呈现当前页面。</span><span class="sxs-lookup"><span data-stu-id="763e7-129">This code uses JavaScript to render the current page based on the selected view.</span></span>

### <a name="test-the-single-page-application"></a><span data-ttu-id="763e7-130">测试单页应用程序</span><span class="sxs-lookup"><span data-stu-id="763e7-130">Test the single-page application</span></span>

> [!NOTE]
> <span data-ttu-id="763e7-131">本节包含有关使用 [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) 在开发计算机上运行简单测试 HTTP 服务器的说明。</span><span class="sxs-lookup"><span data-stu-id="763e7-131">This section includes instructions for using [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) to run a simple testing HTTP server on your development machine.</span></span> <span data-ttu-id="763e7-132">不需要使用此特定工具。</span><span class="sxs-lookup"><span data-stu-id="763e7-132">Using this specific tool is not required.</span></span> <span data-ttu-id="763e7-133">可以使用您喜欢的任何测试服务器来为 **TestClient** 目录提供服务。</span><span class="sxs-lookup"><span data-stu-id="763e7-133">You can use any testing server you prefer to serve the **TestClient** directory.</span></span>

1. <span data-ttu-id="763e7-134">在 CLI 中运行以下命令以安装 **dotnet-serve。**</span><span class="sxs-lookup"><span data-stu-id="763e7-134">Run the following command in your CLI to install **dotnet-serve**.</span></span>

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. <span data-ttu-id="763e7-135">将 CLI 中的当前目录更改为 **TestClient** 目录，并运行以下命令以启动 HTTP 服务器。</span><span class="sxs-lookup"><span data-stu-id="763e7-135">Change the current directory in your CLI to the **TestClient** directory and run the following command to start an HTTP server.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate" -p 8080
    ```

1. <span data-ttu-id="763e7-136">打开浏览器，并导航到 `http://localhost:8080`。</span><span class="sxs-lookup"><span data-stu-id="763e7-136">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="763e7-137">页面应呈现，但当前没有任何按钮可以正常工作。</span><span class="sxs-lookup"><span data-stu-id="763e7-137">The page should render, but none of the buttons currently work.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="763e7-138">添加 NuGet 程序包</span><span class="sxs-lookup"><span data-stu-id="763e7-138">Add NuGet packages</span></span>

<span data-ttu-id="763e7-139">在继续之前，请安装一些稍后将使用的其他 NuGet 程序包。</span><span class="sxs-lookup"><span data-stu-id="763e7-139">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="763e7-140">在 Azure Functions 项目中启用依赖项注入的[Microsoft.Azure.Functions.Extensions。](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions)</span><span class="sxs-lookup"><span data-stu-id="763e7-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) to enable dependency injection in the Azure Functions project.</span></span>
- <span data-ttu-id="763e7-141">[Microsoft.Extensions.Config审核。用于从](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) [.NET](https://docs.microsoft.com/aspnet/core/security/app-secrets)开发密码存储读取应用程序配置的 UserSecret。</span><span class="sxs-lookup"><span data-stu-id="763e7-141">[Microsoft.Extensions.Configuration.UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) to read application configuration from the [.NET development secret store](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span></span>
- <span data-ttu-id="763e7-142">[用于调用 Microsoft Graph 的 Microsoft.Graph。](https://www.nuget.org/packages/Microsoft.Graph/)</span><span class="sxs-lookup"><span data-stu-id="763e7-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="763e7-143">用于验证和管理令牌的[Microsoft.Identity.Client。](https://www.nuget.org/packages/Microsoft.Identity.Client/)</span><span class="sxs-lookup"><span data-stu-id="763e7-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) for authenticating and managing tokens.</span></span>
- <span data-ttu-id="763e7-144">用于检索 OpenID 配置以用于令牌验证的[Microsoft.IdentityModel.Protocols.OpenIdConnect。](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect)</span><span class="sxs-lookup"><span data-stu-id="763e7-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) for retrieving OpenID configuration for token validation.</span></span>
- <span data-ttu-id="763e7-145">[System.IdentityModel.Tokens.Jwt，](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) 用于验证发送到 Web API 的令牌。</span><span class="sxs-lookup"><span data-stu-id="763e7-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) for validating tokens sent to the web API.</span></span>

1. <span data-ttu-id="763e7-146">将 CLI 中的当前目录更改为 **GraphTu一l** 目录并运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="763e7-146">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands.</span></span>

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
