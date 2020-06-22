---
title: Azure と Azure Stack Hub を使用した分析パターンのための階層化データ
description: Azure と Azure Stack Hub を使用して、ハイブリッド クラウド全体に階層化データ ソリューションを実装する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911245"
---
# <a name="tiered-data-for-analytics-pattern"></a>分析パターンの階層化データ

このパターンでは、Azure Stack Hub と Azure を使用して、データをステージ、分析、処理、およびサニタイズし、複数のオンプレミスとクラウドの場所に格納する方法を示します。

## <a name="context-and-problem"></a>コンテキストと問題

最新のテクノロジ環境でエンタープライズ組織が直面している問題の 1 つは、データの保存、処理、分析をセキュリティで保護することです。 考慮事項は次のとおりです。

- データ コンテンツ
- location
- セキュリティとプライバシーの要件
- アクセス許可
- メンテナンス
- ストレージ ウェアハウス

Azure では、Azure Stack Hub と組み合わせることで、データに関する問題に対処し、低コストのソリューションを提供します。 このソリューションは、分散製造または物流企業を通じて最適に表現されます。

ソリューションは、次のシナリオに基づいています。

- 大規模な複数部門がある製造組織。
- グローバルな遠隔地と中央の本社の間で、高速かつ安全なデータの保存、処理、および分散が必要。
- 従業員とマシン アクティビティ、施設情報、およびビジネス レポート データを安全に保つ必要がある。 データは適切に分散され、地域のコンプライアンス ポリシーと業界の規制に準拠する必要がある。

## <a name="solution"></a>解決策

オンプレミスとパブリック クラウド両方の環境を使用して、複数の施設がある企業の需要に応えます。 Azure Stack Hub では、ローカルおよびリモート データを収集、処理、格納、分散するための高速で安全で柔軟なソリューションが提供されます。 このパターンは、場所やユーザーによってセキュリティ、機密性、会社のポリシー、規制の要件が異なる場合に特に役に立ちます。

![分析ソリューション アーキテクチャ用の階層化データ パターン](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>Components

このパターンでは、次のコンポーネントを使用します。

| レイヤー | コンポーネント | 説明 |
|----------|-----------|-------------|
| Azure | ストレージ | [Azure Storage](/azure/storage/) アカウントにより、データ消費のエンドポイントが提供されます。 Azure Storage は、最新のデータ ストレージ シナリオ用の Microsoft のクラウド ストレージ ソリューションです。 Azure Storage では、データ オブジェクトのための高度にスケーラブルなオブジェクト ストアと、クラウドのためのファイル システム サービスが提供されています。 また、信頼できるメッセージ処理のためのメッセージング ストアと、NoSQL ストアも提供されています。 |
| Azure Stack Hub | ストレージ | [Azure Stack Hub ストレージ](/azure-stack/user/azure-stack-storage-overview) アカウントは、複数のサービスに使用されます。<br><br>-  生データ ストレージ用の **BLOB ストレージ**。 BLOB ストレージには、ドキュメント、メディア ファイル、アプリ インストーラーなど、任意の種類のテキストやバイナリ データを格納できます。 すべての BLOB はコンテナーに編成されます。 コンテナーを使用すると、オブジェクトのグループにセキュリティ ポリシーの割り当てに役立つ手段が提供されます。 ストレージ アカウントの容量の上限である 500 TB (テラバイト) を超えない限り、ストレージ アカウントには任意の数のコンテナーを含めることができ、コンテナーには任意の数の BLOB を含めることができます。<br>-  データ アーカイブ用の **BLOB ストレージ**。 クール データ アーカイブ用の低コストのストレージにはベネフィットがあります。 クール データの例には、バックアップ、メディア コンテンツ、科学データ、コンプライアンス、アーカイブ データなどがあります。 一般に、頻繁にアクセスされないデータは、クール ストレージと見なされます。 アクセスの頻度や保有期間などの属性に基づいてデータが階層化されます。 顧客データは、アクセス頻度は低くくても、ホット データと同様の待ち時間とパフォーマンスが要求されます。<br>-  処理されたデータ ストレージ用の **Queue storage**。 Queue storage では、アプリ コンポーネント間のクラウド メッセージングが提供されます。 拡張性を重視してアプリを設計する場合、通常、アプリ コンポーネントを個別に拡張できるように分離します。 Queue storage は、アプリ コンポーネントがクラウド、デスクトップ、オンプレミスのサーバー、モバイル デバイスのいずれで実行されている場合でも、それらの間の通信に非同期メッセージングを提供します。 Queue Storage ではまた、非同期タスクの管理とプロセス ワークフローの構築もサポートします。 |
| | Azure Functions | [Azure Functions](/azure/azure-functions/) サービスは、[Azure App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) リソース プロバイダーによって提供されます。 Azure Functions を使用すると、さまざまなイベントに応答して、単純なサーバーレス環境でコードを実行できます。 Azure Functions では、VM を作成したり、Web アプリを発行したりしなくても、任意のプログラミング言語を使用して、需要に合わせてスケールできます。 関数は、次の場合にソリューションによって使用されます。<br><br>- **データの取り込み**<br>- **データの最適化。** 手動でトリガーされた関数では、スケジュールされたデータ処理、クリーンアップ、およびアーカイブを実行できます。 例として、夜間の顧客リストのスクラブや月次レポート処理などがあります。|

## <a name="issues-and-considerations"></a>問題と注意事項

このソリューションの実装方法を決めるときには、以下の点を考慮してください。

### <a name="scalability"></a>スケーラビリティ

Azure Functions とストレージ ソリューションは、データ ボリュームおよび処理の需要に応じてスケーリングします。 Azure のスケーラビリティ情報とターゲットについては、[Azure Storage のスケーラビリティに関するドキュメント](/azure/storage/common/storage-scalability-targets)を参照してください。

### <a name="availability"></a>可用性

ストレージは、このパターンの可用性に関する主要な考慮事項です。 大規模データ ボリュームの処理および分散には、高速リンク経由の接続が必要です。

### <a name="manageability"></a>管理の容易性

このソリューションの管理容易性は、使用している作成ツールとソース管理のエンゲージメントによって異なります。

## <a name="next-steps"></a>次のステップ

この記事で紹介したトピックの関連情報:

- [Azure Storage](/azure/storage/) と [Azure Functions](/azure/azure-functions/) のドキュメントを参照してください。 このパターンでは、Azure と Azure Stack Hub の両方で、Azure Storage アカウントと Azure Functions が頻繁に使用されます。
- ベスト プラクティスの詳細を確認し、その他の疑問への回答を得るには、[ハイブリッド アプリケーションの設計上の考慮事項](overview-app-design-considerations.md)に関する記事を参照してください。
- 製品とソリューションのポートフォリオ全体の詳細について、[Azure Stack ファミリの製品とソリューション](/azure-stack)を参照してください。

ソリューションの例をテストする準備ができたら、[分析ソリューションの階層化データのデプロイ ガイド](https://aka.ms/tiereddatadeploy)に進んでください。 デプロイ ガイドでは、コンポーネントをデプロイしてテストするための詳細な手順について説明します。