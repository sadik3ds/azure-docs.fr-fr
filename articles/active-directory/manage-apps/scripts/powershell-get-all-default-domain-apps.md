---
title: Exemple PowerShell - Applications de proxy d’application utilisant le domaine par défaut
description: Exemple PowerShell listant toutes les applications de proxy d’application Azure Active Directory (Azure AD) qui utilisent des domaines par défaut (.msappproxy.net).
services: active-directory
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: sample
ms.date: 12/05/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: e586a85abed4907de780f73e8464200d0b6582fe
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94653944"
---
# <a name="get-all-application-proxy-apps-using-default-domains-msappproxynet"></a>Récupérer toutes les applications de proxy d’application utilisant des domaines par défaut (.msappproxy.net)

Cet exemple de script PowerShell liste toutes les applications de proxy d’application Azure Active Directory (Azure AD) qui utilisent des domaines par défaut (.msappproxy.net).

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Cet exemple requiert le [module AzureAD v2 PowerShell pour Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0) (AzureAD) ou la [préversion du Module AzureAD v2 PowerShell pour Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview) (AzureADPreview).

## <a name="sample-script"></a>Exemple de script

[!code-azurepowershell[main](~/powershell_scripts/application-proxy/get-all-default-domain-apps.ps1 "Get all Application Proxy apps using default domains (.msappproxy.net)")]

## <a name="script-explanation"></a>Explication du script

| Commande | Notes |
|---|---|
|[Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal?view=azureadps-2.0) | Permet d’obtenir un principal de service. |
|[Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication?view=azureadps-2.0) | Permet d’obtenir une application Azure AD. |
|[Get-AzureADApplicationProxyApplication](/powershell/module/azuread/get-azureadapplicationproxyapplication?view=azureadps-2.0) | Permet de récupérer une application configurée pour le proxy d’application dans Azure AD. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le Module Azure AD PowerShell, consultez [Présentation du Module Azure AD PowerShell](/powershell/azure/active-directory/overview?view=azureadps-2.0).

Pour d’autres exemples PowerShell pour le proxy d’application, consultez [Exemples Azure AD PowerShell pour le Proxy d’application Azure Active Directory](../application-proxy-powershell-samples.md).