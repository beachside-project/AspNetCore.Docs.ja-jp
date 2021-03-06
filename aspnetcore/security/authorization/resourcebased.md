---
title: ASP.NET Core でのリソースベースの承認
author: scottaddie
description: 承認属性では不十分な場合に、ASP.NET Core アプリにリソースベースの承認を実装する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 11/15/2018
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/authorization/resourcebased
ms.openlocfilehash: 5af4dd6a33e43191dbb5e7a8431fd8468a5fa11b
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/04/2020
ms.locfileid: "82774315"
---
# <a name="resource-based-authorization-in-aspnet-core"></a>ASP.NET Core でのリソースベースの承認

承認方法は、アクセスされるリソースによって異なります。 Author プロパティを持つドキュメントについて考えてみましょう。 作成者だけがドキュメントを更新できます。 そのため、承認評価を行う前に、データストアからドキュメントを取得する必要があります。

属性の評価は、データバインディングの前、およびドキュメントを読み込むページハンドラーまたはアクションの実行前に行われます。 このような理由から、 `[Authorize]`属性を使用した宣言型の承認は十分ではありません。 代わりに、カスタム承認メソッド&mdash;を、命令型*認証*と呼ばれるスタイルとして呼び出すことができます。

::: moniker range=">= aspnetcore-3.0"
[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/resourcebased/samples/3_0)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。
::: moniker-end

 ::: moniker range=">= aspnetcore-2.0 < aspnetcore-3.0"
[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/resourcebased/samples/2_2)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。
::: moniker-end

::: moniker range="<= aspnetcore-1.1"
[サンプル コードを表示またはダウンロード](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/resourcebased/samples/1_1)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。
::: moniker-end

[承認によって保護されたユーザーデータを含む ASP.NET Core アプリを作成](xref:security/authorization/secure-data)するには、リソースベースの承認を使用するサンプルアプリが含まれています。

## <a name="use-imperative-authorization"></a>強制認証を使用する

承認は[IAuthorizationService](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationservice)サービスとして実装され、 `Startup`クラス内のサービスコレクションに登録されます。 サービスは、ページハンドラーまたはアクションへの[依存関係の挿入](xref:fundamentals/dependency-injection)によって利用可能になります。

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Controllers/DocumentController.cs?name=snippet_IAuthServiceDI&highlight=6)]

`IAuthorizationService`2つ`AuthorizeAsync`のメソッドオーバーロードがあります。1つはリソースとポリシー名を受け取り、もう1つはリソースを受け入れ、もう1つは評価する要件のリストです。

::: moniker range=">= aspnetcore-2.0"

```csharp
Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          IEnumerable<IAuthorizationRequirement> requirements);
Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          string policyName);
```

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

```csharp
Task<bool> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          IEnumerable<IAuthorizationRequirement> requirements);
Task<bool> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          string policyName);
```

::: moniker-end

<a name="security-authorization-resource-based-imperative"></a>

次の例では、セキュリティで保護されるリソースがカスタム`Document`オブジェクトに読み込まれます。 現在`AuthorizeAsync`のユーザーが指定されたドキュメントの編集を許可されているかどうかを判断するために、オーバーロードが呼び出されます。 カスタムの "EditPolicy" 承認ポリシーが決定に考慮されます。 承認ポリシーの作成の詳細については[、「カスタムポリシーベースの承認](xref:security/authorization/policies)」を参照してください。

> [!NOTE]
> 次のコードサンプルでは、認証が実行さ`User`れていることを前提として、プロパティを設定します。

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Pages/Document/Edit.cshtml.cs?name=snippet_DocumentEditHandler)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Controllers/DocumentController.cs?name=snippet_DocumentEditAction)]

::: moniker-end

## <a name="write-a-resource-based-handler"></a>リソースベースのハンドラーを記述する

リソースベースの承認のハンドラーを記述することは、[プレーンな要件ハンドラーを記述](xref:security/authorization/policies#security-authorization-policies-based-authorization-handler)する方法とは大きく異なります。 カスタム要件クラスを作成し、要件ハンドラークラスを実装します。 要件クラスの作成の詳細については、「[要件](xref:security/authorization/policies#requirements)」を参照してください。

ハンドラークラスは、要件とリソースの種類の両方を指定します。 たとえば、 `SameAuthorRequirement`と`Document`リソースを使用するハンドラーは次のようになります。

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Services/DocumentAuthorizationHandler.cs?name=snippet_HandlerAndRequirement)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Services/DocumentAuthorizationHandler.cs?name=snippet_HandlerAndRequirement)]

::: moniker-end

前の例では、が`SameAuthorRequirement`より汎用`SpecificAuthorRequirement`的なクラスの特殊なケースであると仮定します。 `SpecificAuthorRequirement`クラス (表示されません) `Name`には、作成者の名前を表すプロパティが含まれています。 `Name`プロパティは、現在のユーザーに設定できます。

で`Startup.ConfigureServices`要件とハンドラーを登録します。

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Startup.cs?name=snippet_ConfigureServicesSample&highlight=4-8,10)]
::: moniker-end

 ::: moniker range=">= aspnetcore-2.0 < aspnetcore-3.0"
[!code-csharp[](resourcebased/samples/2_2/ResourceBasedAuthApp2/Startup.cs?name=snippet_ConfigureServicesSample&highlight=3-7,9)]
::: moniker-end

::: moniker range="<= aspnetcore-1.1"
[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Startup.cs?name=snippet_ConfigureServicesSample&highlight=3-7,9)]
::: moniker-end

### <a name="operational-requirements"></a>運用上の要件

CRUD (作成、読み取り、更新、削除) 操作の結果に基づいて意思決定を行う場合は、 [Operationauthorizationrequirement](/dotnet/api/microsoft.aspnetcore.authorization.infrastructure.operationauthorizationrequirement)ヘルパークラスを使用します。 このクラスを使用すると、操作の種類ごとに個別のクラスではなく、1つのハンドラーを記述できます。 これを使用するには、いくつかの操作名を指定します。

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_OperationsClass)]

ハンドラーは、 `OperationAuthorizationRequirement`要件と`Document`リソースを使用して次のように実装されます。

 ::: moniker range=">= aspnetcore-2.0"
[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_Handler)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_Handler)]

::: moniker-end

上のハンドラーは、リソース、ユーザーの id、および要件の`Name`プロパティを使用して操作を検証します。

## <a name="challenge-and-forbid-with-an-operational-resource-handler"></a>運用リソースハンドラーを使用したチャレンジと禁止

このセクションでは、チャレンジと禁止アクションの結果がどのように処理されるか、およびチャレンジと禁止の違いについて説明します。

操作リソースハンドラーを呼び出すには、ページハンドラーまたは`AuthorizeAsync`アクションでを呼び出すときに操作を指定します。 次の例では、認証されたユーザーが、指定されたドキュメントを表示できるかどうかを判断します。

> [!NOTE]
> 次のコードサンプルでは、認証が実行さ`User`れていることを前提として、プロパティを設定します。

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Pages/Document/View.cshtml.cs?name=snippet_DocumentViewHandler&highlight=10-11)]

承認が成功した場合は、ドキュメントを表示するためのページが返されます。 認証に失敗しても、ユーザーが認証`ForbidResult`されると、認証が失敗したことを認証ミドルウェアに通知します。 認証`ChallengeResult`を実行する必要がある場合は、が返されます。 対話型ブラウザークライアントの場合は、ユーザーをログインページにリダイレクトすることが適切な場合があります。

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Controllers/DocumentController.cs?name=snippet_DocumentViewAction&highlight=11-12)]

承認が成功すると、ドキュメントのビューが返されます。 承認が失敗した`ChallengeResult`場合、を返すと、認証が失敗したことが認証ミドルウェアに通知され、ミドルウェアは適切な応答を受け取ることができます。 適切な応答は、401または403状態コードを返す可能性があります。 対話型ブラウザークライアントの場合、ユーザーをログインページにリダイレクトすることがあります。

::: moniker-end
