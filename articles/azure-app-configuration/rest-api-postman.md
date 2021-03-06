---
title: API REST Azure Active Directory - Test avec Postman
description: Utiliser Postman pour tester l’API REST Azure App Configuration
author: lisaguthrie
ms.author: lcozzens
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 9690678fc7b794c694e588a7993cb131d8264a72
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93423685"
---
# <a name="test-the-azure-app-configuration-rest-api-using-postman"></a>Tester l’API REST Azure App Configuration à l’aide de Postman

Pour tester l’API REST à l’aide de [Postman](https://www.getpostman.com/), vous devez inclure les en-têtes HTTP requis pour l’[authentification](./rest-api-authentication-hmac.md) dans vos requêtes. Voici la procédure à suivre pour configurer Postman de manière à tester l’API REST en générant automatiquement les en-têtes d’authentification :

1. Créer une [requête](https://learning.getpostman.com/docs/postman/sending_api_requests/requests/)
1. Ajoutez la fonction `signRequest` à partir de l’[exemple d’authentification JavaScript](./rest-api-authentication-hmac.md#javascript) au [script de pré-requête](https://learning.getpostman.com/docs/postman/scripts/pre_request_scripts/) pour la requête
1. Ajoutez le code suivant à la fin du script de pré-requête. Mettre à jour la clé d’accès comme indiqué par le commentaire TODO

    ```js
    // TODO: Replace the following placeholders with your access key
    var credential = "<Credential>"; // Id
    var secret = "<Secret>"; // Value

    var isBodyEmpty = pm.request.body === null || pm.request.body === undefined || pm.request.body.isEmpty();

    var headers = signRequest(
        pm.request.url.getHost(),
        pm.request.method,
        pm.request.url.getPathWithQuery(),
        isBodyEmpty ? undefined : pm.request.body.toString(),
        credential,
        secret);

    // Add headers to the request
    headers.forEach(header => {
        pm.request.headers.upsert({key: header.name, value: header.value});
    })
    ```

1. Envoyer la demande
