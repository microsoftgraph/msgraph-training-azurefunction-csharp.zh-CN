---
ms.openlocfilehash: d35318e05a5ebae2316afeb84b731da5c8342ddc
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445967"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="10591-101">本教程指导你如何生成 Azure 函数，该函数使用 Microsoft Graph API 检索用户的日历信息。</span><span class="sxs-lookup"><span data-stu-id="10591-101">This tutorial teaches you how to build an Azure Function that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="10591-102">如果只想下载已完成的教程，可以下载或克隆GitHub[存储库](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)。</span><span class="sxs-lookup"><span data-stu-id="10591-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span> <span data-ttu-id="10591-103">有关使用应用 ID 和密码配置应用的说明，请参阅演示文件夹中的自述文件。</span><span class="sxs-lookup"><span data-stu-id="10591-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="10591-104">先决条件</span><span class="sxs-lookup"><span data-stu-id="10591-104">Prerequisites</span></span>

<span data-ttu-id="10591-105">在开始本教程之前，应在开发计算机上安装以下工具。</span><span class="sxs-lookup"><span data-stu-id="10591-105">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="10591-106">.NET Core SDK</span><span class="sxs-lookup"><span data-stu-id="10591-106">.NET Core SDK</span></span>](https://dotnet.microsoft.com/download)
- [<span data-ttu-id="10591-107">Azure 函数核心工具</span><span class="sxs-lookup"><span data-stu-id="10591-107">Azure Functions Core Tools</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [<span data-ttu-id="10591-108">Azure CLI</span><span class="sxs-lookup"><span data-stu-id="10591-108">Azure CLI</span></span>](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [<span data-ttu-id="10591-109">ngrok</span><span class="sxs-lookup"><span data-stu-id="10591-109">ngrok</span></span>](https://ngrok.com/)

<span data-ttu-id="10591-110">还应具有 Microsoft 工作或学校帐户，具有对同一组织中全局管理员帐户的访问权限。</span><span class="sxs-lookup"><span data-stu-id="10591-110">You should also have a Microsoft work or school account, with access to a global administrator account in the same organization.</span></span> <span data-ttu-id="10591-111">如果你没有 Microsoft 帐户，你可以注册开发人员计划Office 365免费[](https://developer.microsoft.com/office/dev-program)订阅Office 365订阅。</span><span class="sxs-lookup"><span data-stu-id="10591-111">If you don't have a Microsoft account, you can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="10591-112">本教程是使用上述工具的以下版本编写的。</span><span class="sxs-lookup"><span data-stu-id="10591-112">This tutorial was written with the following versions of the above tools.</span></span> <span data-ttu-id="10591-113">本指南中的步骤可能与其他版本一起运行，但尚未经过测试。</span><span class="sxs-lookup"><span data-stu-id="10591-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="10591-114">.NET Core SDK 5.0.203</span><span class="sxs-lookup"><span data-stu-id="10591-114">.NET Core SDK 5.0.203</span></span>
> - <span data-ttu-id="10591-115">Azure Functions Core Tools 3.0.3442</span><span class="sxs-lookup"><span data-stu-id="10591-115">Azure Functions Core Tools 3.0.3442</span></span>
> - <span data-ttu-id="10591-116">Azure CLI 2.23.0</span><span class="sxs-lookup"><span data-stu-id="10591-116">Azure CLI 2.23.0</span></span>
> - <span data-ttu-id="10591-117">ngrok 2.3.40</span><span class="sxs-lookup"><span data-stu-id="10591-117">ngrok 2.3.40</span></span>

## <a name="feedback"></a><span data-ttu-id="10591-118">反馈</span><span class="sxs-lookup"><span data-stu-id="10591-118">Feedback</span></span>

<span data-ttu-id="10591-119">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span><span class="sxs-lookup"><span data-stu-id="10591-119">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span>
