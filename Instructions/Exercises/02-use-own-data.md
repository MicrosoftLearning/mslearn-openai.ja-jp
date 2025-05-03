---
lab:
  title: Azure OpenAI Service を使用して検索拡張生成 (RAG) を実装する
  status: new
---

# Azure OpenAI Service を使用して検索拡張生成 (RAG) を実装する

Azure OpenAI Service を使用すると、基になる LLM のインテリジェンスで独自のデータを使用できます。 独自のデータのみを関連トピックに使用するようにモデルを制限したり、事前トレーニング済みモデルの結果とブレンドしたりすることができます。

この演習のシナリオで、あなたは Margie's Travel Agency で働くソフトウェア開発者の役割を演じます。 Azure AI Search を使用して独自のデータのインデックスを作成し、それを Azure OpenAI で使用してプロンプトを拡張する方法について説明します。

この演習には約 **30** 分かかります。

## Azure リソースをプロビジョニングする

この演習を完了するには、以下が必要です。

- Azure OpenAI リソース。
- Azure AI Search リソース。
- Azure ストレージ アカウント リソース。

1. **Azure portal** (`https://portal.azure.com`) にサインインします。
2. 次の設定で **Azure OpenAI** リソースを作成します。
    - **[サブスクリプション]**: "Azure OpenAI Service へのアクセスが承認されている Azure サブスクリプションを選びます"**
    - **[リソース グループ]**: *リソース グループを作成または選択します*
    - **[リージョン]**: *以下のいずれかのリージョンから**ランダム**に選択する*\*
        - 米国東部
        - 米国東部 2
        - 米国中北部
        - 米国中南部
        - スウェーデン中部
        - 米国西部
        - 米国西部 3
    - **[名前]**: "*希望する一意の名前*"
    - **価格レベル**: Standard S0

    > \* Azure OpenAI リソースは、リージョンのクォータによって制限されます。 一覧表示されているリージョンには、この演習で使用されるモデル タイプの既定のクォータが含まれています。 リージョンをランダムに選択することで、サブスクリプションを他のユーザーと共有しているシナリオで、1 つのリージョンがクォータ制限に達するリスクが軽減されます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

3. Azure OpenAI リソースのプロビジョニング中に、次の設定で **Azure AI Search** リソースを作成します。
    - **サブスクリプション**: *Azure OpenAI リソースをプロビジョニングしたサブスクリプション*
    - **[リソース グループ]**: "Azure OpenAI リソースをプロビジョニングしたリソース グループ"**
    - **サービス名**: *任意の一意の名前*
    - **位置**: Azure OpenAI リソースをプロビジョニングしたリージョン**
    - **価格レベル**: Basic
4. Azure AI 検索リソースのプロビジョニング中に、次の設定で**ストレージ アカウント** リソースを作成します。
    - **サブスクリプション**: *Azure OpenAI リソースをプロビジョニングしたサブスクリプション*
    - **[リソース グループ]**: "Azure OpenAI リソースをプロビジョニングしたリソース グループ"**
    - **ストレージ アカウント名**: *任意の一意の名前*
    - **リージョン**: *Azure OpenAI リソースをプロビジョニングしたリージョン*
    - **プライマリ サービス**: Azure Blob Storage または Azure Data Lake Storage Gen 2
    - **パフォーマンス**: 標準
    - **冗長**: ローカル冗長ストレージ (LRS)
5. 3 つのリソースがすべて Azure サブスクリプションに正常にデプロイされたら、Azure portal で確認し、次の情報を収集します (この演習の後半で必要になります)。
    - 作成した Azure OpenAI リソースの**エンドポイント**と**キー** (Azure Portal の Azure OpenAI リソースの [**キーとエンドポイント**] ページで使用できます)
    - Azure AI Search サービスのエンドポイント (Azure portal の Azure AI Search リソースの概要ページの **URL** 値)。
    - Azure AI Search リソースの**プライマリ管理者キー** (Azure portal の Azure AI Search リソースの **[キー]** ページで入手できます)。

## データをアップロードする

独自のデータを使用して、生成 AI モデルで使用するプロンプトを表示します。 この演習では、データは架空の *Margies Travel* 社の旅行パンフレットのコレクションで構成されています。

1. 新しいブラウザー タブで、`https://aka.ms/own-data-brochures` からパンフレット データのアーカイブをダウンロードします。 パンフレットを PC 上のフォルダーに展開します。
1. Azure portal で、自分のストレージ アカウントに移動し、**[ストレージ ブラウザー]** ページを表示します。
1. **[BLOB コンテナー]** を選択し、`margies-travel` という名前の新しいコンテナーを追加します。
1. **margies-travel** コンテナーを選択し、前に抽出した PDF パンフレットを BLOB コンテナーのルート フォルダーにアップロードします。

## AI モデルをデプロイする

この演習では、次の 2 つの AI モデルを使用します。

- パンフレット内のテキストを*ベクター化する*テキスト埋め込みモデル。グラウンディング プロンプトで使用するために効率的にインデックスを作成できます。
- データに基づいたプロンプトに対する応答の生成にアプリケーションで使用できる GPT モデル。

## モデルをデプロイする

次に、Cloud Shell から Azure OpenAI モデルをデプロイします。

1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、***Bash*** 環境を選択し、Azure portal 内に新しい Cloud Shell を作成します。 Azure portal の下部にあるペインに、Cloud Shell のコマンド ライン インターフェイスが表示されます。

    > **注**: *PowerShell* 環境を使用するクラウド シェルを以前に作成している場合は、それを ***Bash*** に切り替えること。

```dotnetcli
az cognitiveservices account deployment create \
   -g <your_resource_group> \
   -n <your_OpenAI_resource> \
   --deployment-name text-embedding-ada-002 \
   --model-name text-embedding-ada-002 \
   --model-version "2"  \
   --model-format OpenAI \
   --sku-name "Standard" \
   --sku-capacity 5
```

> **注**: SKU 容量は、1 分あたりトークン数 (1,000 単位) で測定されます。 同じサブスクリプションを使用する他のユーザーのための容量を残しながらこの演習を完了するのに、1 分あたり 5,000 トークンのレート制限で十分です。

テキスト埋め込みモデルがデプロイされたら、次の設定で **gpt-4o** モデルの新しいデプロイを作成します。

```dotnetcli
az cognitiveservices account deployment create \
   -g <your_resource_group> \
   -n <your_OpenAI_resource> \
   --deployment-name gpt-4o \
   --model-name gpt-4o \
   --model-version "2024-05-13" \
   --model-format OpenAI \
   --sku-name "Standard" \
   --sku-capacity 5
```

## インデックスを作成する

プロンプトで独自のデータを簡単に使用できるようにするには、Azure AI Search を使用してインデックスを作成します。 テキスト埋め込みモデルを使用して、テキスト データを*ベクトル化*します (その結果、インデックス内の各テキスト トークンが数値ベクトルで表され、生成 AI モデルがテキストを表す方法と互換性が得られます)

1. Azure portal で、Azure AI Search リソースに移動します。
1. [**概要**] ページで、[**データのインポートとベクトル化**] を選択します。
1. **[データ接続のセットアップ]** ページで、**[Azure Blob Storage]** を選択し、次の設定でデータ ソースを構成します。
    - **サブスクリプション**: ストレージ アカウントをプロビジョニングした Azure サブスクリプション。
    - **Blob Storage アカウント**: 前に作成したストレージ アカウント。
    - **BLOB コンテナー**: margies-travel
    - **BLOB フォルダー**: *空白のままにします*
    - **削除の追跡を有効にする**: 未選択
    - **マネージド ID を使用して認証する**: 未選択
1. **[テキストのベクトル化]** ページで、次の設定を選択します。
    - **サブタイプ**: Azure OpenAI
    - **サブスクリプション**: Azure OpenAI Service をプロビジョニングした Azure サブスクリプション。
    - **Azure OpenAI Service**: お使いの Azure OpenAI Service リソース
    - **モデル デプロイ**: text-embedding-ada-002
    - **認証の種類**: API キー
    - **Azure OpenAI サービスに接続すると、アカウントに追加コストが発生することを了承します**: 選択済み
1. 次のページでは、画像をベクター化したり、AI スキルを使用してデータを抽出したりするオプションは選択**しない**でください。
1. 次のページで、セマンティック ランク付けを有効にし、インデクサーを 1 回実行するようにスケジュールします。
1. 最後のページで、**オブジェクト名のプレフィックス**を `margies-index` に設定し、インデックスを作成します。

## Visual Studio Code でアプリを開発する準備をする

次に、Azure OpenAI Service SDK を使うアプリで独自のデータを使う方法を見てみましょう。 Visual Studio Code を使用してアプリを開発します。 アプリのコード ファイルは、GitHub リポジトリで提供されています。

> **ヒント**: 既に **mslearn-openai** リポジトリをクローンしている場合は、Visual Studio Code で開きます。 それ以外の場合は、次の手順に従って開発環境に複製します。

1. Visual Studio Code を起動します。
2. パレットを開き (Shift + Ctrl + P キーまたは **[表示]** - **[コマンド パレット]**)、**Git: Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/mslearn-openai` リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。

    > **注**:Visual Studio Code に、開いているコードを信頼するかどうかを求めるポップアップ メッセージが表示された場合は、ポップアップの **[はい、作成者を信頼します]** オプションをクリックします。

4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、**[今はしない]** を選択します。

## アプリケーションを構成する

C# と Python の両方のアプリケーションが用意されており、どちらのアプリも同じ機能を備えています。 まず、Azure OpenAI リソースの使用を有効にするために、アプリケーションの主要な部分をいくつか完成させます。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**Labfiles/02-use-own-data** フォルダーを参照し、言語の優先順位に応じて、**CSharp** または **Python** フォルダーを展開します。 各フォルダーには、Azure OpenAI 機能を統合するアプリの言語固有のファイルが含まれています。
2. コード ファイルが含まれている **CSharp** または **Python** フォルダーを右クリックし、統合ターミナルを開きます。 次に、言語設定に応じて適切なコマンドを実行して、Azure OpenAI SDK パッケージをインストールします。

    **C#:**

    ```powershell
    dotnet add package Azure.AI.OpenAI --version 2.1.0
    dotnet add package Azure.Search.Documents --version 11.6.0
    ```

    **Python**:

    ```powershell
    pip install openai==1.65.2
    ```

3. **[エクスプローラー]** ペインの **CSharp** または **Python** フォルダーで、使用する言語の構成ファイルを開きます

    - **C#**: appsettings.json
    - **Python**: .env

4. 次を含めて構成値を更新します。
    - 作成した Azure OpenAI リソースの**エンドポイント**と**キー** (Azure Portal の Azure OpenAI リソースの [**キーとエンドポイント**] ページで使用できます)
    - gpt-4o モデル デプロイに指定した**デプロイ名** (`gpt-4o` であるはずです)。
    - 検索サービスのエンドポイント (Azure Portal の検索リソースの概要ページの **URL** 値)。
    - 検索リソースの**キー** (Azure Portal の検索リソースの [**キー**] ページで使用できます。管理者キーのいずれかを使用できます)。
    - 検索インデックスの名前 (`margies-index` になります)。
5. 構成ファイルを保存します。

### Azure OpenAI サービスを使うコードを追加する

これで、Azure OpenAI SDK を使って、デプロイされたモデルを使う準備が整いました。

1. **[エクスプローラー]** ペインの **CSharp** または **Python** フォルダーで、使用する言語のコード ファイルを開き、コメント "***Configure your data source***" を、チャット入力候補のデータ ソースとして、インデックスに対するコードに置き換えます。

    **C#**: ownData.cs

    ```csharp
    // Configure your data source
    // Extension methods to use data sources with options are subject to SDK surface changes. Suppress the warning to acknowledge this and use the subject-to-change AddDataSource method.
    #pragma warning disable AOAI001
    
    ChatCompletionOptions chatCompletionsOptions = new ChatCompletionOptions()
    {
        MaxOutputTokenCount = 600,
        Temperature = 0.9f,
    };
    
    chatCompletionsOptions.AddDataSource(new AzureSearchChatDataSource()
    {
        Endpoint = new Uri(azureSearchEndpoint),
        IndexName = azureSearchIndex,
        Authentication = DataSourceAuthentication.FromApiKey(azureSearchKey),
    });
    ```

    **Python**: ownData.py

    ```python
    # Configure your data source
    text = input('\nEnter a question:\n')
    
    completion = client.chat.completions.create(
        model=deployment,
        messages=[
            {
                "role": "user",
                "content": text,
            },
        ],
        extra_body={
            "data_sources":[
                {
                    "type": "azure_search",
                    "parameters": {
                        "endpoint": os.environ["AZURE_SEARCH_ENDPOINT"],
                        "index_name": os.environ["AZURE_SEARCH_INDEX"],
                        "authentication": {
                            "type": "api_key",
                            "key": os.environ["AZURE_SEARCH_KEY"],
                        }
                    }
                }
            ],
        }
    )
    ```

1. コード ファイルに加えた変更を保存します。

## アプリケーションを実行する

アプリが構成されたので、それを実行してモデルに要求を送信し、応答を確認します。 異なるオプションの間で違いがあるのはプロンプトの内容のみであり、他のすべてのパラメーター (トークン数や温度など) は要求ごとに変わりがないことがわかります。

1. 対話型ターミナル ペインで、フォルダー コンテキストが優先言語のフォルダーであることを確認します。 その後、次のコマンドを入力してアプリケーションを作成します。

    - **C#** : `dotnet run`
    - **Python**: `python ownData.py`

    > **ヒント**: ターミナル ツールバーの **最大化パネル サイズ** (**^**) アイコンを使用すると、コンソール テキストをさらに表示できます。

2. プロンプト `Tell me about London` に対する応答を確認します。これには、回答だけでなく、検索サービスから取得したプロンプトのグラウンディングに使われるデータの詳細が含まれているはずです。

    > **ヒント**: 検索インデックスからの引用を表示する場合は、コード ファイルの先頭近くにある変数 ***show citations*** を **true** に設定します。

## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、**Azure portal** (`https://portal.azure.com`) でリソースを忘れずに削除します。 これには、ストレージ アカウントと検索リソースも含まれます。これらによって比較的多額のコストが発生する可能性があるため、必ず削除してください。
