---
title: 'Démarrage rapide : Définir et récupérer un secret à partir d’Azure Key Vault'
description: Démarrage rapide montrant comment définir et récupérer un secret depuis Azure Key Vault à l’aide d’Azure CLI
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019, devx-track-azurecli
ms.date: 09/03/2019
ms.author: mbaldwin
ms.openlocfilehash: 36688586cc0b9c94a07873bacfa6210f31695d36
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94517255"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-using-azure-cli"></a>Démarrage rapide : Définir et récupérer un secret depuis Azure Key Vault à l’aide d’Azure CLI

Dans ce guide de démarrage rapide, vous créez un coffre de clés dans Azure Key Vault avec Azure CLI. Azure Key Vault est un service cloud qui fonctionne comme un magasin des secrets sécurisé. Vous pouvez stocker des clés, des mots de passe, des certificats et d’autres secrets en toute sécurité. Pour plus d’informations sur Key Vault, consultez la [présentation](../general/overview.md). Azure CLI vous permet de créer et gérer des ressources Azure à l’aide de commandes ou de scripts. Ensuite, vous y stockerez un secret.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Ce guide de démarrage rapide nécessite la version 2.0.4 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. L’exemple ci-après crée un groupe de ressources nommé *ContosoResourceGroup* à l’emplacement *eastus*.

```azurecli
az group create --name "ContosoResourceGroup" --location eastus
```

## <a name="create-a-key-vault"></a>Créer un coffre de clés

Ensuite, vous allez créer un coffre de clés dans le groupe de ressources créé à l’étape précédente. Vous devrez fournir certaines informations :

- Pour ce démarrage rapide, nous utilisons **Contoso-vault2**. Vous devez fournir un nom unique dans votre test.
- Nom du groupe de ressources **ContosoResourceGroup**.
- Emplacement **USA Est**.

```azurecli
az keyvault create --name "Contoso-Vault2" --resource-group "ContosoResourceGroup" --location eastus
```

La sortie de cette cmdlet affiche les propriétés du coffre de clés que vous venez de créer. Notez les deux propriétés ci-dessous :

- **Nom du coffre** : dans l’exemple, il s’agit de **Contoso-Vault2**. Vous allez utiliser ce nom pour les autres commandes Key Vault.
- **URI du coffre** : dans l’exemple, il s’agit de https://contoso-vault2.vault.azure.net/. Les applications qui utilisent votre coffre via son API REST doivent utiliser cet URI.

À ce stade, votre compte Azure est le seul autorisé à effectuer des opérations sur ce nouveau coffre.

## <a name="add-a-secret-to-key-vault"></a>Ajouter un secret au coffre de clés

Pour ajouter un secret au coffre, vous devez effectuer deux autres opérations. Ce mot de passe peut être utilisé par une application. Le mot de passe sera appelé **ExamplePassword** et stockera la valeur **hVFkk965BuUv**.

Tapez les commandes ci-après pour créer dans Key Vault un secret appelé **ExamplePassword**, contenant la valeur **hVFkk965BuUv** :

```azurecli
az keyvault secret set --vault-name "Contoso-Vault2" --name "ExamplePassword" --value "hVFkk965BuUv"
```

Vous pouvez maintenant référencer ce mot de passe que vous avez ajouté à Azure Key Vault à l’aide de son URI. Utilisez **« https://Contoso-Vault2.vault.azure.net/secrets/ExamplePassword  »** pour obtenir la version actuelle. 

Pour afficher sous forme de texte brut la valeur contenue dans le secret :

```azurecli
az keyvault secret show --name "ExamplePassword" --vault-name "Contoso-Vault2"
```

Vous venez de créer un coffre de clés, d’y stocker un secret et de récupérer ce dernier.

## <a name="clean-up-resources"></a>Nettoyer les ressources

D’autres guides de démarrage rapide et didacticiels de cette collection reposent sur ce guide. Si vous prévoyez d’utiliser d’autres démarrages rapides et didacticiels, il peut être utile de conserver ces ressources.
Lorsque vous n’en avez plus besoin, vous pouvez utiliser la commande [az group delete](/cli/azure/group) pour supprimer le groupe de ressources, ainsi que toutes les ressources associées. Vous pouvez supprimer les ressources comme suit :

```azurecli
az group delete --name ContosoResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un coffre de clés Key Vault et vous y avez stocké un secret. Pour en savoir plus sur Key Vault et sur la manière de l’intégrer à vos applications, consultez les articles ci-dessous.

- Lire la [vue d’ensemble Azure Key Vault](../general/overview.md)
- Consulter la référence des [commandes az keyvault Azure CLI](/cli/azure/keyvault?view=azure-cli-latest)
- Passer en revue les [bonnes pratiques relatives à Azure Key Vault](../general/best-practices.md)
