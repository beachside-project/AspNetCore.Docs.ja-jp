---
title: ASP.NET Core の目的階層とマルチテナント
author: rick-anderson
description: ASP.NET Core データ保護 Api に関連する、目的の文字列階層とマルチテナントについて説明します。
ms.author: riande
ms.date: 10/14/2016
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/data-protection/consumer-apis/purpose-strings-multitenancy
ms.openlocfilehash: 73edb8082d2df263bc1e6d73fee1360fa6840514
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/04/2020
ms.locfileid: "82776774"
---
# <a name="purpose-hierarchy-and-multi-tenancy-in-aspnet-core"></a>ASP.NET Core の目的階層とマルチテナント

`IDataProtector`は暗黙的に`IDataProtectionProvider`もあるため、目的を連結することができます。 この意味では`provider.CreateProtector([ "purpose1", "purpose2" ])` 、はと`provider.CreateProtector("purpose1").CreateProtector("purpose2")`同じです。

これにより、データ保護システムを通じて、いくつかの興味深い階層関係を実現できます。 前の例の[Contoso. Messaging. securemessage](xref:security/data-protection/consumer-apis/purpose-strings#data-protection-contoso-purpose)コンポーネントでは、最初に1回`provider.CreateProtector("Contoso.Messaging.SecureMessage")`呼び出し、その結果をプライベート`_myProvider`フィールドにキャッシュできます。 将来`_myProvider.CreateProtector("User: username")`のプロテクターは、の呼び出しを使用して作成できます。これらのプロテクターは、個々のメッセージをセキュリティで保護するために使用されます。

これを反転させることもできます。 複数のテナントをホストする単一の論理アプリケーションがあるとします (CMS は妥当と思われます)。各テナントは独自の認証および状態管理システムを使用して構成できます。 包括的なアプリケーションには単一のマスタープロバイダーがあり、 `provider.CreateProtector("Tenant 1")`と`provider.CreateProtector("Tenant 2")`を呼び出して、各テナントにデータ保護システムの分離されたスライスを与えます。 その後、テナントは独自のプロテクターを独自のニーズに基づいて派生させることができますが、システム内の他のテナントと競合するプロテクターを作成するのがいかに難しいかは関係ありません。 グラフィックでは、次のように表されます。

![マルチテナントの目的](purpose-strings-multitenancy/_static/purposes-multi-tenancy.png)

>[!WARNING]
> これは、包括アプリケーションが個々のテナントに使用できる Api を制御し、テナントがサーバー上で任意のコードを実行できないことを前提としています。 テナントが任意のコードを実行できる場合は、プライベートリフレクションを実行して分離の保証を解除するか、マスターキーマテリアルを直接読み取り、任意のサブキーを派生させることができます。

データ保護システムでは、既定の設定ではなく、既定の構成でマルチテナントの一種が使用されます。 既定では、マスターキー生成情報は、ワーカープロセスアカウントのユーザープロファイルフォルダー (または、IIS アプリケーションプール id の場合はレジストリ) に格納されます。 しかし、1つのアカウントを使用して複数のアプリケーションを実行することは、実際には非常に一般的です。そのため、これらすべてのアプリケーションは、マスターキー生成情報を共有することになります。 これを解決するには、データ保護システムによって、一意のアプリケーションごとの識別子が、全体的な目的のチェーンの最初の要素として自動的に挿入されます。 この暗黙的な目的は、各アプリケーションをシステム内の一意のテナントとして効果的に扱うことによって、[個々のアプリケーション](xref:security/data-protection/configuration/overview#per-application-isolation)を相互に分離するためのものであり、保護機能の作成プロセスは上記のイメージと同じです。
