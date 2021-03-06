---
title: Planification d’un déploiement Azure Files | Microsoft Docs
description: Comprendre la planification d’un déploiement Azure Files. Vous pouvez soit directement monter un partage de fichiers Azure, soit mettre en cache un partage de fichiers Azure local avec Azure File Sync.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 09/15/2020
ms.author: rogarana
ms.subservice: files
ms.custom: references_regions
ms.openlocfilehash: 650ee1fc9e0e1941a7a3655bca1c75950ab878dd
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492112"
---
# <a name="planning-for-an-azure-files-deployment"></a>Planification d’un déploiement Azure Files
Le service [Azure Files](storage-files-introduction.md) peut être déployé principalement de deux façons : en montant directement les partages de fichiers Azure serverless, ou en mettant en cache les partages de fichiers Azure en local avec Azure File Sync. L'option de déploiement que vous choisissez détermine les éléments à prendre en compte lors de la planification de votre déploiement. 

- **Montage direct d'un partage de fichiers Azure** : Étant donné qu’Azure Files fournit un accès SMB (Server Message Block) ou NFS (Network File System), vous pouvez monter des partages de fichiers Azure localement ou dans le cloud à l’aide des clients SMB ou NFS standard disponibles dans votre système d’exploitation. Dans la mesure où les partages de fichiers Azure sont serverless, vous n'avez aucun serveur de fichiers ou appareil NAS à gérer lors des déploiements liés à des scénarios de production. Concrètement, cela signifie que vous n'avez aucun correctif logiciel à appliquer ni aucun disque physique à remplacer. 

- **Mise en cache d'un partage de fichiers Azure localement à l'aide d'Azure File Sync** : Azure File Sync vous permet de centraliser les partages de fichiers de votre organisation dans Azure Files, tout en conservant la flexibilité, le niveau de performance et la compatibilité d'un serveur de fichiers local. Azure File Sync transforme une instance Windows Server locale (ou cloud) en un cache rapide de votre partage de fichiers SMB Azure. 

Cet article traite principalement de considérations relatives au déploiement, afin de déployer un partage de fichiers Azure en vue de son montage directement par un client local ou un client cloud. Pour planifier un déploiement d’Azure File Sync, consultez [Planification d’un déploiement Azure File Sync](storage-sync-files-planning.md).

## <a name="available-protocols"></a>Protocoles disponibles

Azure Files offre deux protocoles pouvant être utilisés lors du montage de vos partages de fichiers, SMB et NFS (Network File System). Pour plus d’informations sur ces protocoles, consultez [Protocoles de partage de fichiers Azure](storage-files-compare-protocols.md).

> [!IMPORTANT]
> L’essentiel du contenu de cet article s’applique uniquement aux partages SMB. Tout ce qui s’applique aux partages NFS indique spécifiquement être applicable.

## <a name="management-concepts"></a>Concepts de gestion
[!INCLUDE [storage-files-file-share-management-concepts](../../../includes/storage-files-file-share-management-concepts.md)]

Lorsque vous déployez des partages de fichiers Azure dans des comptes de stockage, tenez compte des recommandations suivantes :

- Déployez uniquement des partages de fichiers Azure dans des comptes de stockage ayant d’autres partages de fichiers Azure. Bien que les comptes de stockage GPv2 vous permettent de disposer de comptes de stockage mixte, à partir du moment où les ressources de stockage (telles que les partages de fichiers Azure et les conteneurs d’objets BLOB) partagent les limites du compte de stockage, la combinaison des ressources peut compliquer la résolution des problèmes de performances par la suite. 

- Faites attention aux limites d’IOPS d’un compte de stockage lors du déploiement des partages de fichiers Azure. Dans l'idéal, une correspondance 1:1 doit être respectée entre les partages de fichiers et les comptes de stockage, mais cela n'est pas toujours possible en raison des différentes limites et restrictions imposées par votre organisation et Azure. S’il n’est pas possible d’avoir un seul partage de fichiers déployé dans un compte de stockage, tenez compte des partages qui seront très actifs et des partages qui le seront moins, afin de garantir que les partages de fichiers les plus sollicités ne soient pas mis ensemble dans le même compte de stockage.

- Déployez uniquement des comptes GPv2 et FileStorage, et mettez à niveau les comptes de stockage GPv1 et Classic lorsque vous les trouvez dans votre environnement. 

## <a name="identity"></a>Identité
Pour accéder à un partage de fichiers Azure, l’utilisateur du partage de fichiers doit être authentifié et disposer de l’autorisation d’accéder au partage. Cette opération s’effectue en fonction de l’identité de l’utilisateur accédant au partage de fichiers. Azure Files intègre trois fournisseurs d’identité principaux :
- **Active Directory Domain Services en local (AD DS ou AD DS en local)**  : Les comptes de stockage Azure peuvent être joints à un domaine sur une instance Active Directory Domain Services appartenant à un client, exactement comme un serveur de fichiers Windows Server ou un appareil NAS. Votre contrôleur de domaine peut être déployé localement, dans une machine virtuelle Azure, ou même en tant que machine virtuelle dans un autre fournisseur de cloud. Azure Files ne dépend pas de l’emplacement où votre contrôleur de domaine est hébergé. Dès lors qu’un compte de stockage est joint à un domaine, l’utilisateur final peut monter un partage de fichiers avec le compte d’utilisateur dont il s’est servi pour se connecter à son PC. L’authentification basée sur Active Directory utilise le protocole d’authentification Kerberos.
- **Azure Active Directory Domain Services (Azure AD DS)** . Azure AD DS fournit un contrôleur de domaine géré par Microsoft et pouvant être utilisé pour les ressources Azure. Joindre le domaine de votre compte de stockage à Azure AD DS offre des avantages similaires à le joindre à une instance Active Directory détenue par le client. Cette option de déploiement est particulièrement utile pour les scénarios lift-and-shift d’application qui nécessitent des autorisations basées sur AD. Étant donné qu’Azure AD DS fournit une authentification basée sur Active Directory, cette option utilise également le protocole d’authentification Kerberos.
- **Clé du compte de Stockage Azure** : Les partages de fichiers Azure peuvent également être montés à l’aide d’une clé de compte de stockage Azure. Pour monter un partage de fichiers de cette façon, le nom du compte de stockage est utilisé comme nom d’utilisateur, et la clé du compte de stockage comme mot de passe. L’utilisation de la clé de compte de stockage pour monter le partage de fichiers Azure est en fait une opération d’administrateur, car le partage de fichiers monté disposera d’autorisations complètes sur tous les fichiers et dossiers du partage, même s’ils ont des listes de contrôle d’accès. Lors de l’utilisation de la clé de compte de stockage pour le montage sur SMB, le protocole d’authentification NTLMv2 est utilisé.

Pour les clients qui migrent à partir de serveurs de fichiers locaux, ou qui créent dans Azure Files de nouveaux partages de fichiers destinés à se comporter comme des serveurs de fichiers Windows ou des appliances NAS, le fait de joindre le domaine de votre compte de stockage à l’instance **Active Directory appartenant au client** représente l’option recommandée. Pour en savoir plus sur la jonction du domaine de votre compte de stockage à une instance Active Directory détenue par le client, consultez [Vue d’ensemble Azure Files Active Directory](storage-files-active-directory-overview.md).

Si vous envisagez d’utiliser la clé de compte de stockage pour accéder à vos partages de fichiers Azure, nous vous recommandons d’utiliser des points de terminaison de service, comme décrit à la section [Mise en réseau](#networking).

## <a name="networking"></a>Mise en réseau
Les partages de fichiers Azure sont accessibles de n’importe quel endroit via le point de terminaison public du compte de stockage. Ainsi, les demandes authentifiées, comme celles qui sont autorisées par l’identité d’ouverture de session d’un utilisateur, peuvent provenir de manière sécurisée, de l’intérieur comme de l’extérieur d’Azure. Dans de nombreux environnements de clients, un montage initial du partage de fichiers Azure sur votre station de travail locale échouera, alors même que les montages à partir de machines virtuelles Azure réussissent. L’explication tient au fait que de nombreuses organisations et autres fournisseurs de services Internet bloquent le port dont se sert SMB pour communiquer, à savoir le port 445. Pour afficher le récapitulatif des FAI qui autorisent ou interdisent l’accès depuis le port 445, consultez [TechNet](https://social.technet.microsoft.com/wiki/contents/articles/32346.azure-summary-of-isps-that-allow-disallow-access-from-port-445.aspx).

Pour débloquer l’accès à votre partage de fichiers Azure, vous avez principalement deux options à votre disposition :

- Débloquer le port 445 pour le réseau local de votre organisation. Les partages de fichiers Azure sont uniquement accessibles en externe, via le point de terminaison public, par le biais de protocoles sûrs, tels que SMB 3.0 et l’API FileREST. Il s’agit du moyen le plus simple pour accéder à votre partage de fichiers Azure localement, car il ne nécessite pas de configuration réseau avancée au-delà de la modification des règles du port de sortie de votre organisation. Toutefois, nous vous recommandons de supprimer les versions héritées et dépréciées du protocole SMB, à savoir SMB 1.0. Pour savoir comment faire, consultez [Sécurisation de Windows/Windows Server](storage-how-to-use-files-windows.md#securing-windowswindows-server) et [Sécurisation de Linux](storage-how-to-use-files-linux.md#securing-linux).

- Accéder aux partages de fichiers Azure via une connexion ExpressRoute ou VPN. Lorsque vous accédez à votre partage de fichiers Azure via un tunnel réseau, vous pouvez monter votre partage de fichiers Azure comme un partage de fichiers en local, car le trafic SMB ne traverse pas les limites de votre organisation.   

Bien que d’un point de vue technique il soit beaucoup plus facile de monter vos partages de fichiers Azure via le point de terminaison public, nous pensons que la plupart des clients choisiront de monter leurs partages de fichiers Azure sur une connexion ExpressRoute ou VPN. Le montage avec ces options est possible avec les partages SMB et NFS. Pour ce faire, vous devez configurer les éléments suivants pour votre environnement :  

- **Tunneling réseau à l’aide d’un VPN ExpressRoute, de site à site ou de point à site**. Le tunneling dans un réseau virtuel permet d’accéder aux partages de fichiers Azure depuis l’environnement local, même si le port 445 est bloqué.
- **Points de terminaison privés**. Les points de terminaison privés attribuent à votre compte de stockage une adresse IP dédiée depuis l’espace d’adressage du réseau virtuel. Le tunneling réseau est ainsi possible sans avoir à ouvrir de réseaux locaux sur la totalité des plages d’adresses IP détenues par les clusters de stockage Azure. 
- **Transfert DNS**. Configurez votre DNS local afin de résoudre le nom de votre compte de stockage (c’est-à-dire `storageaccount.file.core.windows.net` pour les régions du cloud public) sur l’adresse IP de vos points de terminaison privés.

Pour planifier la mise en réseau associée au déploiement d’un partage de fichiers Azure, consultez [Considérations relatives à la mise en réseau Azure Files](storage-files-networking-overview.md).

## <a name="encryption"></a>Chiffrement
Azure Files prend en charge deux types de chiffrement : le chiffrement en transit, qui se rapporte au chiffrement utilisé lors du montage/de l’accès au partage de fichiers Azure, et le chiffrement au repos, qui a trait à la façon dont les données sont chiffrées lorsqu’elles sont stockées sur le disque. 

### <a name="encryption-in-transit"></a>Chiffrement en transit

> [!IMPORTANT]
> Cette section traite en détail du chiffrement en transit pour les partages SMB. Pour plus d’informations sur le chiffrement en transit avec les partages NFS, consultez [Sécurité](storage-files-compare-protocols.md#security).

Par défaut, le chiffrement en transit est activé pour tous les comptes de stockage Azure. Cela signifie que, lorsque vous montez un partage de fichiers sur SMB ou y accédez en utilisant le protocole FileREST (par exemple, via le portail Azure, PowerShell/CLI ou des kits de développement logiciel (SDK) Azure), Azure Files n’autorise la connexion que si elle est établie à l’aide des protocoles SMB 3.0+ avec chiffrement ou HTTPS. Les clients qui ne prennent pas en charge le protocole SMB 3.0, ou qui prennent en charge le protocole SMB 3.0 mais pas le chiffrement SMB, ne peuvent pas monter le partage de fichiers Azure si le chiffrement en transit est activé. Pour plus d’informations sur les systèmes d’exploitation prenant en charge SMB 3.0 avec chiffrement, consultez notre documentation détaillée pour [Windows](storage-how-to-use-files-windows.md), [macOS](storage-how-to-use-files-mac.md) et [Linux](storage-how-to-use-files-linux.md). Toutes les versions actuelles de PowerShell, de CLI et des SDK prennent en charge le protocole HTTPS.  

Vous pouvez désactiver le chiffrement en transit pour un compte de stockage Azure. Lorsque le chiffrement est désactivé, Azure Files autorise également les protocoles SMB 2.1 et SMB 3.0 sans chiffrement, ainsi que les appels d’API FileREST non chiffrés via le protocole HTTP. La principale raison justifiant de désactiver le chiffrement en transit est la nécessité de prendre en charge une application héritée devant être exécutée sur un système d’exploitation plus ancien, tel que Windows Server 2008 R2 ou une distribution Linux non récente. Azure Files n’autorise que les connexions SMB 2.1 au sein de la même région Azure que le partage de fichiers Azure. Ainsi, un client SMB 2.1 situé en dehors de la région Azure dans laquelle se trouve le partage de fichiers Azure, par exemple, localement ou dans une autre région Azure, ne peut pas accéder au partage de fichiers.

Nous recommandons fortement de vérifier que le chiffrement des données en transit est activé.

Pour plus d’informations sur le chiffrement en transit, voir [ Exiger un transfert sécurisé dans Stockage Azure](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

### <a name="encryption-at-rest"></a>Chiffrement au repos
[!INCLUDE [storage-files-encryption-at-rest](../../../includes/storage-files-encryption-at-rest.md)]

## <a name="data-protection"></a>Protection de données
Azure Files adopte une approche multicouche pour garantir que vos données sont sauvegardées, récupérables et protégées contre les menaces de sécurité.

### <a name="soft-delete"></a>Suppression réversible
La suppression réversible pour les partages de fichiers (préversion) est un paramètre de niveau de compte de stockage qui vous permet de récupérer votre partage de fichiers en cas de suppression accidentelle. Lors de la suppression d’un partage de fichiers, celui-ci passe par un état transitoire de suppression réversible au lieu d’être supprimé définitivement. Vous pouvez configurer la durée pendant laquelle les données supprimées de manière réversible sont récupérables avant leur suppression définitive, et annuler la suppression du partage à tout moment lors de la période de rétention. 

Nous vous recommandons d’activer la suppression réversible pour la plupart des partages de fichiers. Si vous avez un flux de travail dans lequel la suppression de partage est courante et attendue, vous pouvez décider de configurer une période de rétention très brève ou de ne pas activer la suppression réversible.

Pour plus d’informations sur la suppression réversible, consultez [Prévenir les suppressions de données accidentelles](./storage-files-prevent-file-share-deletion.md).

### <a name="backup"></a>Sauvegarde
Vous pouvez sauvegarder votre partage de fichiers Azure via des [captures instantanées de partage](./storage-snapshots-files.md), qui sont des copies en lecture seule de votre partage à un instant dans le passé. Les captures instantanées sont incrémentielles, ce qui signifie qu’elles ne contiennent que les données modifiés depuis la capture instantanée précédente. Vous pouvez avoir jusqu’à 200 captures instantanées par partage de fichiers et les conserver pendant jusqu’à 10 ans. Vous pouvez soit prendre manuellement ces captures instantanées dans le portail Azure, via PowerShell ou l’interface de ligne de commande (CLI), soit d’utiliser [Sauvegarde Azure](../../backup/azure-file-share-backup-overview.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json). Les captures instantanées sont stockées dans votre partage de fichiers, ce qui signifie que, si vous supprimez votre partage de fichiers, vos captures instantanées sont également supprimées. Pour protéger vos sauvegardes de captures instantanées contre des suppressions accidentelles, assurez-vous que la suppression réversible est activée pour votre partage.

La [Sauvegarde Azure pour les partages de fichiers Azure](../../backup/azure-file-share-backup-overview.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) gère la planification et la rétention des captures instantanées. Ses fonctionnalités grand-père-père-fils vous permettent de prendre des captures instantanées quotidiennes, hebdomadaires, mensuelles et annuelles, dont les périodes de rétention diffèrent. Le service Sauvegarde Azure orchestre également l’activation de la suppression réversible, et pose un verrou de suppression sur un compte de stockage dès qu’un partage de fichiers dans celui-ci est configuré pour la sauvegarde. Enfin, le service Sauvegarde Azure offre certaines fonctionnalités de surveillance et d’alerte clés, qui permettent aux clients d’avoir une vue consolidée de leur parc de sauvegarde.

Sauvegarde Azure vous permet d’effectuer des restaurations au niveau élément et au niveau partage dans le portail Azure. Il vous suffit de choisir le point de restauration (une capture instantanée particulière), le fichier ou répertoire particulier le cas échéant, puis l’emplacement (d’origine ou de remplacement) sur lequel vous souhaitez effectuer la restauration. Le service de sauvegarde gère la copie des données de captures instantanées et affiche la progression de la restauration dans le portail.

Pour plus d’informations sur la sauvegarde, consultez [À propos de la sauvegarde des partages de fichiers Azure](../../backup/azure-file-share-backup-overview.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

### <a name="advanced-threat-protection-for-azure-files-preview"></a>Advanced Threat Protection pour Azure Files (préversion)
Advanced Threat Protection (ATP) pour Stockage Azure fournit une couche supplémentaire de renseignement sur la sécurité, qui génère des alertes quand elle détecte une activité anormale sur votre compte de stockage, par exemple, des tentatives inhabituelles d’accès au compte de stockage. ATP exécute également une analyse de réputation du hachage de programme malveillant et déclenche une alerte concernant les programmes malveillants connus. Vous pouvez configurer ATP au niveau d’un abonnement ou d’un compte de stockage via Azure Security Center. 

Pour plus d’informations, consultez [Advanced Threat Protection pour Stockage Azure](../common/azure-defender-storage-configure.md).

## <a name="storage-tiers"></a>Niveaux de stockage
[!INCLUDE [storage-files-tiers-overview](../../../includes/storage-files-tiers-overview.md)]

En général, les fonctionnalités d’Azure Files et l’interopérabilité avec d’autres services sont les mêmes entre les partages de fichiers Premium et les partages de fichiers Standard (incluant les partages de fichiers optimisés pour les transactions, chauds et froids). Il existe, cependant, quelques différences importantes :
- **Modèle de facturation**
    - Les partages de fichiers Premium sont facturés selon un modèle de facturation provisionné, ce qui signifie que vous payez le volume de stockage que vous provisionnez au prix fixe plutôt que la quantité de stockage que vous utilisez. Il n’y a aucun coût supplémentaire pour les transactions et les métadonnées au repos.
    - Les partages de fichiers Standard sont facturés selon un modèle de paiement à l’utilisation : il comprend un coût de base du stockage pour la quantité de stockage que vous consommez, et un coût de transaction supplémentaire en fonction de la façon dont vous utilisez le partage. Avec les partages de fichiers Standard, votre facture augmente si vous utilisez davantage (en lecture/écriture/montage) le partage de fichiers Azure.
- **Options de redondance**
    - Les partages de fichiers Premium sont uniquement disponibles pour le stockage localement redondant (LRS) et le stockage redondant interzone (ZRS).
    - Les partages de fichiers Standard sont disponibles pour le stockage localement redondant, redondant interzone, géoredondant (GRS) et géoredondant interzone (GZRS).
- **Taille maximale d’un partage de fichiers**
    - Les partages de fichiers Premium peuvent être provisionnés jusqu’à 100 Tio sans aucun travail supplémentaire.
    - Par défaut, les partages de fichiers Standard ne peuvent atteindre que 5 Tio, mais la limite de partage peut être augmentée jusqu’à 100 Tio en optant pour l’indicateur de fonctionnalité de compte de stockage *Partage de fichiers volumineux*. Les partages de fichiers Standard peuvent couvrir jusqu’à 100 Tio uniquement pour les comptes de stockage redondants en local ou interzones. Pour plus d’informations sur l’augmentation de la taille des partages de fichiers, consultez [Activer et créer des partages de fichiers volumineux](./storage-files-how-to-create-large-file-share.md).
- **Disponibilité régionale**
    - Les partages de fichiers Premium sont disponibles dans la plupart des régions Azure, à l’exception de quelques-unes. La prise en charge de la redondance interzone est disponible dans un sous-ensemble de régions. Pour savoir si les partages de fichiers Premium sont actuellement disponibles dans votre région, consultez la page des [produits disponibles par région](https://azure.microsoft.com/global-infrastructure/services/?products=storage) pour Azure. Pour connaître les régions prenant en charge ZRS, consultez [Stockage redondant interzone](../common/storage-redundancy.md#zone-redundant-storage). Pour nous aider à hiérarchiser les nouvelles régions et les fonctionnalités du niveau Premium, répondez à ce [sondage](https://aka.ms/pfsfeedback).
    - Le partage de fichiers Standard est disponible dans toutes les régions Azure.
- Azure Kubernetes Service (AKS) prend en charge le partage de fichiers Premium dans la version 1.13 et ultérieure.

Une fois qu’un partage de fichiers est créé, qu’il soit Premium ou Standard, vous ne pouvez pas le convertir automatiquement à l’autre niveau. Si vous souhaitez basculer vers l’autre niveau, vous devez créer un nouveau partage de fichiers dans le niveau voulu, puis copier manuellement les données depuis votre partage d’origine sur le partage que vous avez créé. Nous vous recommandons d’utiliser `robocopy` pour Windows, `rsync` pour macOS et Linux afin d’effectuer cette copie.

### <a name="understanding-provisioning-for-premium-file-shares"></a>Comprendre le provisionnement des partages de fichiers Premium
Les partages de fichiers Premium sont approvisionnés selon un ratio Gio/IOPS/débit fixe. Toutes les tailles de partages sont proposées à la ligne de base/au débit minimum avec possibilité de rafale. Pour chaque Gio approvisionné, le partage a des IOPS/un débit minimaux et un débit d’IOPS de 0,1 Mio/s, dans les limites maximales autorisées par partage. L’approvisionnement minimal autorisé est de 100 Gio avec un minimum d’IOPS par seconde/débit. 

Tous les partages Premium sont proposés avec rafale gratuite dans la mesure du possible. Toutes les tailles de partages peuvent atteindre des rafales allant jusqu’à 4 000 IOPS ou jusqu’à trois E/S par Gio approvisionné, ce qui offre un nombre d’IOPS en rafale plus important au partage. Tous les partages prennent en charge la rafale pour une durée maximale de 60 minutes avec une limite maximale pour la rafale. Les nouveaux partages démarrent avec le crédit de rafale complète basé sur la capacité approvisionnée.

Les partages doivent être provisionnés par incréments de 1 Gio. La taille minimale est de 100 Gio, la taille suivante de 101 Gio, et ainsi de suite.

> [!TIP]
> IOPS de base = 400 + 1 * par Gio provisionné. (Jusqu’à 100 000 IOPS maximum).
>
> Limite de rafale = MAX (4 000, 3 * IOPS de base). (en fonction de la limite supérieure, jusqu’à un maximum de 100 000 IOPS).
>
> Débit de sortie = 60 Mio/s + 0,06 * Gio provisionnés
>
> Débit d’entrée = 40 Mio/s + 0,04 * Gio provisionnés

La taille de partage approvisionnée est spécifiée par le quota de partage. Le quota du partage peut être augmenté à tout moment, mais il ne peut être réduit qu’au bout des 24 heures suivant la dernière augmentation. À l’issue des 24 heures sans augmentation de quota, vous pouvez diminuer le quota du partage autant de fois que vous le souhaitez, jusqu’à ce que vous l’augmentiez à nouveau. Les modifications de mise à l’échelle IOPS/débit prennent effet quelques minutes après le changement de taille.

Il est possible de diminuer la taille de votre partage provisionné en dessous de votre Gio utilisé. Si vous faites cela, vous ne perdez pas les données, mais vous êtes toujours facturé pour la taille utilisée et recevez les performances (IOPS de base, débit et IOPS en rafale) du partage provisionné, pas de la taille utilisée.

Le tableau suivant illustre quelques exemples de ces formules pour les tailles de partage provisionné :

|Capacité (Gio) | IOPS de base | IOPS en rafale | Sortie (Mio/s) | Entrée (Mio/s) |
|---------|---------|---------|---------|---------|
|100         | 500     | Jusqu’à 4 000     | 66   | 44   |
|500         | 900     | Jusqu’à 4 000  | 90   | 60   |
|1 024       | 1 424   | Jusqu’à 4 000   | 122   | 81   |
|5 120       | 5 520   | Jusqu’à 15 360  | 368   | 245   |
|10 240      | 10 640  | Jusqu’à 30 720  | 675   | 450   |
|33 792      | 34 192  | Jusqu’à 100 000 | 2 088 | 1 392   |
|51 200      | 51 600  | Jusqu’à 100 000 | 3 132 | 2 088   |
|102 400     | 100 000 | Jusqu’à 100 000 | 6 204 | 4 136   |

Il est essentiel de noter que les partages de fichiers efficaces sont soumis aux limites du réseau des machines, à la bande passante réseau disponible, aux tailles d’e/s, au parallélisme, entre autres nombreux facteurs. Par exemple, sur la base d’un test interne avec des tailles d’e/s en lecture/écriture de 8 Kio, une seule machine virtuelle Windows sans SMB Multichannel activé, *F16s_v2 standard*, connectée au partage de fichiers Premium sur SMB pourrait atteindre 20 000 e/s par seconde en écriture et 15 000 e/s par seconde. Avec les tailles d’e/s en lecture/écriture de 512 Mio, la même machine virtuelle peut atteindre 1,1 Gio/s en sortie et 370 Mio/s de débit d’entrée. Le même client peut atteindre des \~performances trois fois supérieures si SMB Multichannel est activé sur les partages Premium. Pour obtenir une mise à l’échelle des performances maximales, [activez SMB Multichannel](storage-files-enable-smb-multichannel.md) et répartissez la charge entre plusieurs machines virtuelles. Reportez-vous à [Performances de SMB Multichannel](storage-files-smb-multichannel-performance.md) et au [Guide de dépannage](storage-troubleshooting-files-performance.md) pour certains problèmes de performances courants et leurs solutions de contournement.

#### <a name="bursting"></a>Mode en rafales
Si votre charge de travail a besoin de performances supplémentaires pour répondre aux pics de demande, votre partage peut utiliser des crédits de rafale pour atteindre la limite d’IOPS de la ligne de base du partage pour offrir les performances de partage dont il a besoin pour répondre à la demande. Les partages de fichiers Premium peuvent prévoir des rafales de leurs IOPS jusqu’à 4 000 ou jusqu’à multiplier leur nombre par trois, selon la valeur la plus élevée. Ce mode en rafales est automatisé et fonctionne selon un système de crédits. Il fonctionne dans la mesure des possibilités et la limite de rafale n’est pas une garantie : les partages de fichiers peuvent croître par rafales *jusqu’à* cette limite, pour une durée maximale de 60 minutes.

Des crédits s’accumulent dans un compartiment à rafales chaque fois que le trafic de votre partage de fichiers se trouve en dessous des IOPS de base. Par exemple, un partage de 100 Gio dispose de 500 IOPS de base. Si le trafic réel sur le partage est de 100 IOPS pour un intervalle spécifique de 1 seconde, les 400 IOPS inutilisées sont créditées dans un compartiment à rafales. De même, un partage inactif de 1 Tio accumule du crédit de rafale à 1 424 IOPS. Ces crédits sont ensuite utilisés lorsque des opérations dépassent les IOPS de base.

Chaque fois qu’un partage dépasse les IOPS de base et qu’il dispose de crédits dans un compartiment à rafales, il est augmenté par rafales pour atteindre le taux de rafales maximal autorisé. Les partages peuvent continuer de fonctionner en rafale tant qu’il reste des crédits, jusqu’à une durée maximale de 60 minutes, mais cela se base sur le nombre de crédits en rafale accumulés. Chaque e/s située au-delà des IOPS de base consomme un crédit ; une fois que tous les crédits sont consommés, le partage retourne aux IOPS de base.

Les crédits de partage présentent trois états :

- En hausse, lorsque le partage de fichiers utilise un nombre inférieur à celui des IOPS de base.
- En baisse, lorsque le partage de fichiers utilise plus que les IOPS de la ligne de base et en mode de rafale.
- Constant, lorsque le partage de fichiers utilise exactement les IOPS de la ligne de base, il n’y a aucun crédit accumulé ou utilisé.

Au départ, les nouveaux partages de fichiers se voient attribuer un nombre total de crédits dans leur compartiment à rafales. Les crédits de rafale ne seront pas augmentés si les IOPS du partage chutent en dessous des IOPS de base, en raison de la limitation par le serveur.

### <a name="enable-standard-file-shares-to-span-up-to-100-tib"></a>Activer les partages de fichiers Standard pour couvrir jusqu'à 100 Tio
[!INCLUDE [storage-files-tiers-enable-large-shares](../../../includes/storage-files-tiers-enable-large-shares.md)]

#### <a name="limitations"></a>Limites
[!INCLUDE [storage-files-tiers-large-file-share-availability](../../../includes/storage-files-tiers-large-file-share-availability.md)]

## <a name="redundancy"></a>Redondance
[!INCLUDE [storage-files-redundancy-overview](../../../includes/storage-files-redundancy-overview.md)]

## <a name="migration"></a>Migration
Dans de nombreux cas, vous n’établirez pas de nouveau partage de fichiers net pour votre organisation, mais migrerez plutôt un partage de fichiers existant, depuis un serveur de fichiers local ou un appareil NAS, vers Azure Files. Le choix de la stratégie et de l’outil de migration appropriés pour votre scénario est important pour la réussite de votre migration. 

L’[article vue d’ensemble de la migration](storage-files-migration-overview.md) aborde brièvement les bases et contient un tableau qui vous amène à des guides de migration susceptibles de couvrir votre scénario.

## <a name="next-steps"></a>Étapes suivantes
* [Planification d’un déploiement Azure File Sync](storage-sync-files-planning.md)
* [Déploiement d’Azure Files](storage-files-deployment-guide.md)
* [Déploiement d’Azure File Sync](storage-sync-files-deployment-guide.md)
* [Consultez l’article de vue d’ensemble de la migration pour trouver le guide de migration de votre scénario](storage-files-migration-overview.md)
