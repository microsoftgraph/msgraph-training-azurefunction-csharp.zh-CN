---
ms.openlocfilehash: f034aca537c878d988e8c4e7db1ff28d0817761a
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445946"
---
<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将了解示例 Azure 函数需要哪些更改来准备发布到 Azure [Functions 应用](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)。

## <a name="update-code"></a>更新代码

配置从用户密码存储中读取，仅适用于你的开发计算机。 在发布到 Azure 之前，你需要更改存储配置的位置，并相应地更新 **Program.cs** 中的代码。

应用程序密钥应存储在安全存储中，如 Azure [Key Vault。](https://docs.microsoft.com/azure/key-vault/general/overview)

## <a name="update-cors-setting-for-azure-function"></a>更新 Azure 函数的 CORS 设置

在此示例中，我们在local.settings.js **配置** CORS 以允许测试应用程序调用 函数。 您需要配置已发布函数以允许任何将调用函数的 SPA 应用。

## <a name="update-app-registrations"></a>更新应用注册

Azure Function 应用注册Graph清单中的 属性将需要使用将调用 Azure Function 的任何应用的应用程序 `knownClientApplications` ID 进行更新。 

## <a name="recreate-existing-subscriptions"></a>重新创建现有订阅

使用本地计算机或 ngrok 上的 webhook URL 创建的任何订阅都应使用 Azure Function 的生产 URL `Notify` 重新创建。
