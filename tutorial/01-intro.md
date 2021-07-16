---
ms.openlocfilehash: d35318e05a5ebae2316afeb84b731da5c8342ddc
ms.sourcegitcommit: 7c550d2bcd30f505913e0cd441fbe12ae4a93b27
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/15/2021
ms.locfileid: "53445967"
---
<!-- markdownlint-disable MD002 MD041 -->

本教程指导你如何生成 Azure 函数，该函数使用 Microsoft Graph API 检索用户的日历信息。

> [!TIP]
> 如果只想下载已完成的教程，可以下载或克隆GitHub[存储库](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)。 有关使用应用 ID 和密码配置应用的说明，请参阅演示文件夹中的自述文件。

## <a name="prerequisites"></a>先决条件

在开始本教程之前，应在开发计算机上安装以下工具。

- [.NET Core SDK](https://dotnet.microsoft.com/download)
- [Azure 函数核心工具](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [ngrok](https://ngrok.com/)

还应具有 Microsoft 工作或学校帐户，具有对同一组织中全局管理员帐户的访问权限。 如果你没有 Microsoft 帐户，你可以注册开发人员计划Office 365免费[](https://developer.microsoft.com/office/dev-program)订阅Office 365订阅。

> [!NOTE]
> 本教程是使用上述工具的以下版本编写的。 本指南中的步骤可能与其他版本一起运行，但尚未经过测试。
>
> - .NET Core SDK 5.0.203
> - Azure Functions Core Tools 3.0.3442
> - Azure CLI 2.23.0
> - ngrok 2.3.40

## <a name="feedback"></a>反馈

Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).
