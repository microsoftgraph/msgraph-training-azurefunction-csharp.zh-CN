---
ms.openlocfilehash: f034aca537c878d988e8c4e7db1ff28d0817761a
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445946"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5ee03-101">在此练习中，你将了解示例 Azure 函数需要哪些更改来准备发布到 Azure [Functions 应用](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)。</span><span class="sxs-lookup"><span data-stu-id="5ee03-101">In this exercise you'll learn about what changes are needed to the sample Azure Function to prepare for [publishing to an Azure Functions app](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span></span>

## <a name="update-code"></a><span data-ttu-id="5ee03-102">更新代码</span><span class="sxs-lookup"><span data-stu-id="5ee03-102">Update code</span></span>

<span data-ttu-id="5ee03-103">配置从用户密码存储中读取，仅适用于你的开发计算机。</span><span class="sxs-lookup"><span data-stu-id="5ee03-103">Configuration is read from the user secret store, which only applies to your development machine.</span></span> <span data-ttu-id="5ee03-104">在发布到 Azure 之前，你需要更改存储配置的位置，并相应地更新 **Program.cs** 中的代码。</span><span class="sxs-lookup"><span data-stu-id="5ee03-104">Before you publish to Azure, you'll need to change where you store your configuration, and update the code in **Program.cs** accordingly.</span></span>

<span data-ttu-id="5ee03-105">应用程序密钥应存储在安全存储中，如 Azure [Key Vault。](https://docs.microsoft.com/azure/key-vault/general/overview)</span><span class="sxs-lookup"><span data-stu-id="5ee03-105">Application secrets should be stored in secure storage, such as [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span></span>

## <a name="update-cors-setting-for-azure-function"></a><span data-ttu-id="5ee03-106">更新 Azure 函数的 CORS 设置</span><span class="sxs-lookup"><span data-stu-id="5ee03-106">Update CORS setting for Azure Function</span></span>

<span data-ttu-id="5ee03-107">在此示例中，我们在local.settings.js **配置** CORS 以允许测试应用程序调用 函数。</span><span class="sxs-lookup"><span data-stu-id="5ee03-107">In this sample we configured CORS in **local.settings.json** to allow the test application to call the function.</span></span> <span data-ttu-id="5ee03-108">您需要配置已发布函数以允许任何将调用函数的 SPA 应用。</span><span class="sxs-lookup"><span data-stu-id="5ee03-108">You'll need to configure your published function to allow any SPA apps that will call it.</span></span>

## <a name="update-app-registrations"></a><span data-ttu-id="5ee03-109">更新应用注册</span><span class="sxs-lookup"><span data-stu-id="5ee03-109">Update app registrations</span></span>

<span data-ttu-id="5ee03-110">Azure Function 应用注册Graph清单中的 属性将需要使用将调用 Azure Function 的任何应用的应用程序 `knownClientApplications` ID 进行更新。 </span><span class="sxs-lookup"><span data-stu-id="5ee03-110">The `knownClientApplications` property in the manifest for the **Graph Azure Function** app registration will need to be updated with the application IDs of any apps that will be calling the Azure Function.</span></span>

## <a name="recreate-existing-subscriptions"></a><span data-ttu-id="5ee03-111">重新创建现有订阅</span><span class="sxs-lookup"><span data-stu-id="5ee03-111">Recreate existing subscriptions</span></span>

<span data-ttu-id="5ee03-112">使用本地计算机或 ngrok 上的 webhook URL 创建的任何订阅都应使用 Azure Function 的生产 URL `Notify` 重新创建。</span><span class="sxs-lookup"><span data-stu-id="5ee03-112">Any subscriptions created using the webhook URL on your local machine or ngrok should be recreated using the production URL of the `Notify` Azure Function.</span></span>
