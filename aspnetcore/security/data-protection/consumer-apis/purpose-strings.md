---
title: ASP.NET Core の目的の文字列
author: rick-anderson
description: ASP.NET Core データ保護 Api で目的の文字列を使用する方法について説明します。
ms.author: riande
ms.date: 10/14/2016
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/data-protection/consumer-apis/purpose-strings
ms.openlocfilehash: 9cdc762a6dd3cbf6e248d0b72d915d09dee25eb0
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/04/2020
ms.locfileid: "82774172"
---
# <a name="purpose-strings-in-aspnet-core"></a>ASP.NET Core の目的の文字列

<a name="data-protection-consumer-apis-purposes"></a>

を使用`IDataProtectionProvider`するコンポーネントは、 `CreateProtector`一意の*目的*のパラメーターをメソッドに渡す必要があります。 目的の*パラメーター*は、暗号化コンシューマー間の分離を提供するため、データ保護システムのセキュリティに固有のものです。これは、ルート暗号化キーが同じ場合でも同様です。

コンシューマーが目的を指定する場合は、目的の文字列をルート暗号化キーと共に使用して、そのコンシューマーに固有の暗号化サブキーを派生させます。 これにより、アプリケーション内の他のすべての暗号化コンシューマーからコンシューマーが分離されます。その他のコンポーネントはペイロードを読み取ることができず、他のコンポーネントのペイロードを読み取ることはできません。 また、この分離により、コンポーネントに対する攻撃のすべての実行可能なカテゴリがレンダリングされます。

![目的の図の例](purpose-strings/_static/purposes.png)

上の図では`IDataProtector` 、インスタンス A と B は互いのペイロードを読み取る**ことができません**。

目的の文字列は、シークレットである必要はありません。 これは、他の適切に動作するコンポーネントが同じ目的の文字列を提供することがないという意味で、単に一意である必要があります。

>[!TIP]
> データ保護 Api を使用するコンポーネントの名前空間と型名を使用することは、経験則として適しています。実際には、この情報は競合しません。
>
>ベアラートークンを処理する Contoso で作成されたコンポーネントは、目的の文字列として BearerToken を使用する場合があります。 または、さらに優れた方法として、BearerToken を目的の文字列として使用することもできます。 バージョン番号を追加することで、将来のバージョンで BearerToken を目的として使用できるようになります。また、ペイロードの場合に限り、異なるバージョンが相互に完全に分離されます。

の目的のパラメーター `CreateProtector`は文字列配列であるため、上記のはと`[ "Contoso.Security.BearerToken", "v1" ]`して指定されている可能性があります。 これにより、目的の階層を確立し、データ保護システムでマルチテナントシナリオの可能性を開くことができます。

<a name="data-protection-contoso-purpose"></a>

>[!WARNING]
> コンポーネントは、信頼されていないユーザー入力を目的のチェーンの入力の唯一のソースにすることを許可しないでください。
>
>たとえば、セキュリティで保護されたメッセージを格納するコンポーネントの Contoso. Messaging. SecureMessage を考えてみます。 セキュリティで保護されたメッセージングコンポーネント`CreateProtector([ username ])`がを呼び出す場合、悪意のあるユーザーが、コンポーネントを呼び出す`CreateProtector([ "Contoso.Security.BearerToken" ])`ために、ユーザー名 "BearerToken" を持つアカウントを作成する可能性があります。これにより、セキュリティで保護されたメッセージングシステムは、認証トークンとして認識される可能性のあるペイロードを mint します。
>
>メッセージングコンポーネントに適した目的のチェーンは、 `CreateProtector([ "Contoso.Messaging.SecureMessage", "User: username" ])`適切な分離を提供するになります。

によって提供される分離`IDataProtectionProvider`と`IDataProtector`、、、およびの動作は、次のとおりです。

* 特定`IDataProtectionProvider`のオブジェクトに対して`CreateProtector` 、メソッドは、 `IDataProtector`それを作成した`IDataProtectionProvider`オブジェクトと、メソッドに渡された目的のパラメーターの両方に一意に関連付けられたオブジェクトを作成します。

* 目的のパラメーターを null にすることはできません。 (目的が配列として指定されている場合は、配列の長さが0ではなく、配列のすべての要素が null 以外でなければならないことを意味します)。空の文字列の目的は技術的には許可されますが、推奨されません。

* 同じ順序で (序数の比較子を使用して) 同じ文字列が含まれている場合にのみ、2つの目的の引数が等価になります。 単一の目的の引数は、対応する単一要素の目的の配列に相当します。

* 2 `IDataProtector`つのオブジェクトは、同等の目的のパラメーターを持つ`IDataProtectionProvider`同等のオブジェクトから作成された場合にのみ、等しいと見なされます。

* 特定`IDataProtector`のオブジェクトに対して、を`Unprotect(protectedData)`呼び出すと、 `unprotectedData`同等`IDataProtector`のオブジェクトの`protectedData := Protect(unprotectedData)`場合にのみ、元のが返されます。

> [!NOTE]
> コンポーネントによっては、他のコンポーネントと競合していることがわかっている目的の文字列が意図的に選択されている場合は考慮されません。 このようなコンポーネントは、本質的に悪意のあるものと見なされます。このシステムは、悪意のあるコードがワーカープロセス内で既に実行されている場合にセキュリティ保証を提供するためのものではありません。
