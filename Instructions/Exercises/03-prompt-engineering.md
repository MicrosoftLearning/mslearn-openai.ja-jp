---
lab:
  title: アプリでプロンプト エンジニアリングを利用する
  status: stale
---

# アプリでプロンプト エンジニアリングを利用する

Azure OpenAI Service を使用する場合、開発者がプロンプトをどのように形成するかは、生成 AI モデルの応答方法に大きく影響します。 明瞭かつ簡潔に要求すれば、Azure OpenAI はコンテンツを調整し、書式設定を行うことができます。 この演習では、類似するコンテンツのさまざまなプロンプトが、要件をより十分に満たす AI モデルの応答を形成するためにどのように役立つかを学習します。

この演習のシナリオで、あなたは野生動物のマーケティング キャンペーンに取り組むソフトウェア開発者の役割を演じます。 あなたは、生成 AI を使って広告メールを改善し、チームに適用される可能性のある記事を分類する方法を検討しています。 この演習で使われるプロンプト エンジニアリング手法は、さまざまなユース ケースにも同様に適用できます。

この演習には約 **30** 分かかります。

## Azure OpenAI リソースをプロビジョニングする

まだ持っていない場合は、Azure サブスクリプションで Azure OpenAI リソースをプロビジョニングします。

1. **Azure portal** (`https://portal.azure.com`) にサインインします。
2. 次の設定で **Azure OpenAI** リソースを作成します。
    - **[サブスクリプション]**: "Azure OpenAI Service へのアクセスが承認されている Azure サブスクリプションを選びます"**
    - **[リソース グループ]**: *リソース グループを作成または選択します*
    - **[リージョン]**: *以下のいずれかのリージョンから**ランダム**に選択する*\*
        - オーストラリア東部
        - カナダ東部
        - 米国東部
        - 米国東部 2
        - フランス中部
        - 東日本
        - 米国中北部
        - スウェーデン中部
        - スイス北部
        - 英国南部
    - **[名前]**: "*希望する一意の名前*"
    - **価格レベル**: Standard S0

    > \* Azure OpenAI リソースは、リージョンのクォータによって制限されます。 一覧表示されているリージョンには、この演習で使用されるモデル タイプの既定のクォータが含まれています。 リージョンをランダムに選択することで、サブスクリプションを他のユーザーと共有しているシナリオで、1 つのリージョンがクォータ制限に達するリスクが軽減されます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

3. デプロイが完了するまで待ちます。 次に、Azure portal でデプロイされた Azure OpenAI リソースに移動します。

## モデルをデプロイする

Azure には、モデルのデプロイ、管理、調査に使用できる **Azure AI Foundry portal** という名前の Web ベース ポータルが用意されています。 Azure AI Foundry ポータルを使用してモデルをデプロイすることで、Azure OpenAI の調査を開始します。

> **注**: Azure AI Foundry ポータルを使用すると、実行するタスクを提案するメッセージ ボックスが表示される場合があります。 これらを閉じて、この演習の手順に従うことができます。

1. Azure portal にある Azure OpenAI リソースの **[概要]** ページで、**[開始する]** セクションまで下にスクロールし、ボタンを選択して **AI Foundry portal** (以前は AI Studio) に移動します。
1. Azure AI Foundry の左ペインで、**[デプロイ]** ページを選び、既存のモデル デプロイを表示します。 まだない場合は、次の設定で **gpt-4o** モデルの新しいデプロイを作成します。
    - **デプロイの名前**: *任意の一意の名前*
    - **モデル**: gpt-4o
    - **モデル バージョン**: *既定のバージョンを使用する*
    - **デプロイの種類**:Standard
    - **1 分あたりのトークンのレート制限**: 5K\*
    - **コンテンツ フィルター**: 既定
    - **動的クォータを有効にする**: 無効

    > \* この演習は、1 分あたり 5,000 トークンのレート制限内で余裕を持って完了できます。またこの制限によって、同じサブスクリプションを使用する他のユーザーのために容量を残すこともできます。

## プロンプト エンジニアリングの手法を調べる

まずは、チャット プレイグラウンドでのプロンプト エンジニアリング手法をいくつか検討してみましょう。

1. 左側ペインの **[プレイグラウンド]** セクションで、**[チャット]** ページを選択します。 **Chat** プレイグラウンド ページは、ボタンの行と 2 つのメイン パネルで構成されます (画面の解像度に応じて、右から左へ水平に、または上から下へ垂直に配置されます)。
    - **[セットアップ]**: デプロイの選択、システム メッセージの定義、デプロイとやり取りするためのパラメーターの設定に使用します。
    - **[チャットの履歴]**: チャット メッセージの送信や、応答の表示に使用されます。
1. **[デプロイ]** で、gpt-4o モデル デプロイが選択されていることを確認します。
1. 選択したデプロイのすぐ下のテキストボックスに表示された既定のシステム メッセージを確認します。それは、*"あなたは、情報の検索を支援する AI アシスタントです。"* となっているはずです。
1. **[チャットの履歴]** で、次のクエリを送信します。

    ```prompt
    What kind of article is this?
    ---
    Severe drought likely in California
    
    Millions of California residents are bracing for less water and dry lawns as drought threatens to leave a large swath of the region with a growing water shortage.
    
    In a remarkable indication of drought severity, officials in Southern California have declared a first-of-its-kind action limiting outdoor water use to one day a week for nearly 8 million residents.
    
    Much remains to be determined about how daily life will change as people adjust to a drier normal. But officials are warning the situation is dire and could lead to even more severe limits later in the year.
    ```

    応答として記事の説明が表示されます。 ただし、記事の分類のために、さらに具体的な形式が必要だとします。

1. **[設定]** セクションのシステム メッセージを `You are a news aggregator that categorizes news articles.` に変更します

1. 新しいシステム メッセージの下で、**[セクションの追加]** ボタンを選択し、**[例]** を選択します。 次に、以下の例を追加します。

    **ユーザー:**

    ```prompt
    What kind of article is this?
    ---
    New York Baseballers Wins Big Against Chicago
    
    New York Baseballers mounted a big 5-0 shutout against the Chicago Cyclones last night, solidifying their win with a 3 run homerun late in the bottom of the 7th inning.
    
    Pitcher Mario Rogers threw 96 pitches with only two hits for New York, marking his best performance this year.
    
    The Chicago Cyclones' two hits came in the 2nd and the 5th innings but were unable to get the runner home to score.
    ```

    **アシスタント:**

    ```prompt
    Sports
      ```

1. 次のテキストを含む別の例を追加します。

    **ユーザー:**

    ```prompt
    Categorize this article:
    ---
    Joyous moments at the Oscars
    
    The Oscars this past week where quite something!
    
    Though a certain scandal might have stolen the show, this year's Academy Awards were full of moments that filled us with joy and even moved us to tears.
    These actors and actresses delivered some truly emotional performances, along with some great laughs, to get us through the winter.
    
    From Robin Kline's history-making win to a full performance by none other than Casey Jensen herself, don't miss tomorrows rerun of all the festivities.
    ```

    **アシスタント:**

    ```prompt
    Entertainment
    ```

1. **[セットアップ]** セクションのシステム メッセージ テキスト ボックスの下にある **[変更の適用]** ボタンを使用して、変更を保存します。

1. **[チャットの履歴]** セクションで、次のプロンプトを再送信します。

    ```prompt
    What kind of article is this?
    ---
    Severe drought likely in California
    
    Millions of California residents are bracing for less water and dry lawns as drought threatens to leave a large swath of the region with a growing water shortage.
    
    In a remarkable indication of drought severity, officials in Southern California have declared a first-of-its-kind action limiting outdoor water use to one day a week for nearly 8 million residents.
    
    Much remains to be determined about how daily life will change as people adjust to a drier normal. But officials are warning the situation is dire and could lead to even more severe limits later in the year.
    ```

    より具体的なシステム メッセージと、予想されるクエリと応答のいくつかの例を組み合わせることで、結果の形式が一貫したものになります。

1. システム メッセージを既定のテンプレートに戻します。これは例なしの `You are an AI assistant that helps people find information.` です。 次に、変更を適用します。

1. **[チャットの履歴]** セクションで、次のプロンプトを送信します。

    ```prompt
    # 1. Create a list of animals
    # 2. Create a list of whimsical names for those animals
    # 3. Combine them randomly into a list of 25 animal and name pairs
    ```

    モデルは回答を番号付きリストに分けて応答し、それらはおそらくプロンプトを満たしています。 これは適切な応答ですが、実際に望んでいたことは、説明したタスクを実行する Python プログラムをモデルに作成させることだったとします。

1. システム メッセージを `You are a coding assistant helping write python code.` に変更し、変更を適用します。
1. 次のプロンプトをモデルに再送信します。

    ```prompt
    # 1. Create a list of animals
    # 2. Create a list of whimsical names for those animals
    # 3. Combine them randomly into a list of 25 animal and name pairs
    ```

    モデルは、コメントで要求された内容を実行する Python コードで正しく応答するはずです。

## Visual Studio Code でアプリを開発する準備をする

次に、Azure OpenAI Service SDK を使うアプリでのプロンプト エンジニアリングの使用について説明します。 Visual Studio Code を使用してアプリを開発します。 アプリのコード ファイルは、GitHub リポジトリで提供されています。

> **ヒント**: 既に **mslearn-openai** リポジトリをクローンしている場合は、Visual Studio Code で開きます。 それ以外の場合は、次の手順に従って開発環境に複製します。

1. Visual Studio Code を起動します。
2. パレットを開き (Shift + Ctrl + P キーまたは **[表示]** - **[コマンド パレット]**)、**Git: Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/mslearn-openai` リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。

    > **注**:Visual Studio Code に、開いているコードを信頼するかどうかを求めるポップアップ メッセージが表示された場合は、ポップアップの **[はい、作成者を信頼します]** オプションをクリックします。

4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、**[今はしない]** を選択します。

## アプリケーションを構成する

C# と Python の両方のアプリケーションが用意されており、どちらのアプリも同じ機能を備えています。 まず、非同期 API 呼び出しで Azure OpenAI リソースを使用できるように、アプリケーションの主要な部分をいくつか完成させます。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**Labfiles/03-prompt-engineering** フォルダーを参照し、言語の設定に応じて **CSharp** または **Python** フォルダーを展開します。 各フォルダーには、Azure OpenAI 機能を統合するアプリの言語固有のファイルが含まれています。
2. コード ファイルが含まれている **CSharp** または **Python** フォルダーを右クリックし、統合ターミナルを開きます。 次に、言語設定に応じて適切なコマンドを実行して、Azure OpenAI SDK パッケージをインストールします。

    **C#:**

    ```powershell
    dotnet add package Azure.AI.OpenAI --version 2.1.0
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
    - モデル デプロイに指定した**デプロイ名** (Azure AI Foundry ポータルの **[デプロイ]** ページで使用できます)。
5. 構成ファイルを保存します。

## Azure OpenAI サービスを使うコードを追加する

これで、Azure OpenAI SDK を使って、デプロイされたモデルを使う準備が整いました。

1. **[エクスプローラー]** ペインの **CSharp** または **Python** フォルダーで、希望の言語のコード ファイルを開き、コメント "***Add Azure OpenAI package***" を、Azure OpenAI SDK ライブラリを追加するコードに置き換えます。

    **C#** : Program.cs

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    using OpenAI.Chat;
    ```

    **Python**: prompt-engineering.py

    ```python
    # Add Azure OpenAI package
    from openai import AsyncAzureOpenAI
    ```

2. コード ファイルでコメント "***Configure the Azure OpenAI client***" を見つけて、Azure OpenAI クライアントを構成するコードを追加します。

    **C#** : Program.cs

    ```csharp
    // Configure the Azure OpenAI client
    AzureOpenAIClient azureClient = new (new Uri(oaiEndpoint), new ApiKeyCredential(oaiKey));
    ChatClient chatClient = azureClient.GetChatClient(oaiDeploymentName);
    ```

    **Python**: prompt-engineering.py

    ```python
    # Configure the Azure OpenAI client
    client = AsyncAzureOpenAI(
        azure_endpoint = azure_oai_endpoint, 
        api_key=azure_oai_key,  
        api_version="2024-02-15-preview"
        )
    ```

3. Azure OpenAI モデルを呼び出す関数のコメント "***Format and send the request to the model***" の下に、要求を書式設定してモデルに送信するコードを追加します。

    **C#** : Program.cs

    ```csharp
    // Format and send the request to the model
    var chatCompletionsOptions = new ChatCompletionOptions()
    {
        Temperature = 0.7f,
        MaxOutputTokenCount = 800
    };
    
    // Get response from Azure OpenAI
    ChatCompletion response = await chatClient.CompleteChatAsync(
        [
            new SystemChatMessage(systemMessage),
            new UserChatMessage(userMessage),
        ],
        chatCompletionsOptions);
    ```

    **Python**: prompt-engineering.py

    ```python
    # Format and send the request to the model
    messages =[
        {"role": "system", "content": system_message},
        {"role": "user", "content": user_message},
    ]
    
    print("\nSending request to Azure OpenAI model...\n")

    # Call the Azure OpenAI model
    response = await client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7,
        max_tokens=800
    )
    ```

4. コード ファイルに加えた変更を保存します。

## アプリケーションを実行する

アプリが構成されたので、それを実行してモデルに要求を送信し、応答を確認します。 異なるオプションの間で違いがあるのはプロンプトの内容のみであり、他のすべてのパラメーター (トークン数や温度など) は要求ごとに変わりがないことがわかります。

1. Visual Studio Code で、使う言語のフォルダーにある `system.txt` を開きます。 インタラクションごとに、このファイルに**システム メッセージ**を入力して保存します。 各イテレーションは、システム メッセージを変更できるように、最初に一時停止します。
1. 対話型ターミナル ペインで、フォルダー コンテキストが優先言語のフォルダーであることを確認します。 その後、次のコマンドを入力してアプリケーションを作成します。

    - **C#** : `dotnet run`
    - **Python**: `python prompt-engineering.py`

    > **ヒント**: ターミナル ツールバーの **最大化パネル サイズ** (**^**) アイコンを使用すると、コンソール テキストをさらに表示できます。

1. 最初のイテレーションでは、次のプロンプトを入力します。

    **システム メッセージ**

    ```prompt
    You are an AI assistant
    ```

    **ユーザー メッセージ:**

    ```prompt
    Write an intro for a new wildlife Rescue
    ```

1. 出力を確認します。 AI モデルは、野生動物の救護に関する適切で一般的な概要を生成します。
1. 次に、応答の形式を指定する以下のプロンプトを入力します。

    **システム メッセージ**

    ```prompt
    You are an AI assistant helping to write emails
    ```

    **ユーザー メッセージ:**

    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants 
    - Call for donations to be given at our website
    ```

    > **ヒント**: VM での自動入力が複数行プロンプトでうまく機能しない場合があります。 問題が発生した場合は、プロンプト全体をコピーし、Visual Studio Code に貼り付けます。

1. 出力を確認します。 今度は、特定の動物と寄付の呼びかけを含む応答がメール形式で表示されます。
1. 次に、コンテンツを追加で指定する以下のプロンプトを入力します。

    **システム メッセージ**

    ```prompt
    You are an AI assistant helping to write emails
    ```

    **ユーザー メッセージ:**

    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants, as well as zebras and giraffes 
    - Call for donations to be given at our website 
    Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```

1. 出力を確認します。また、明確な指示に基づいてメールがどのように変わったかを確認します。
1. 次に、以下のプロンプトを入力して、トーンに関する詳細をシステム メッセージに追加します。

    **システム メッセージ**

    ```prompt
    You are an AI assistant that helps write promotional emails to generate interest in a new business. Your tone is light, chit-chat oriented and you always include at least two jokes.
    ```

    **ユーザー メッセージ:**

    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants, as well as zebras and giraffes 
    - Call for donations to be given at our website 
    Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```

1. 出力を確認します。 メールは同様の形式で表示されますが、今度は、よりくだけたトーンになります。 ジョークが含まれている可能性もあります。
1. 最後のイテレーションでは、メールの生成から離れて、"グラウンディング コンテキスト" を検討します。** ここでは、単純なシステム メッセージを指定し、ユーザー プロンプトの始まりとしてグラウンディング コンテキストを提供するようにアプリを変更します。 その後、アプリはユーザー入力を追加し、グラウンディング コンテキストから情報を抽出してユーザー プロンプトに回答するようになります。
1. ファイル `grounding.txt` を開き、挿入するグラウンディング コンテキストをざっと読みます。
1. アプリのコメント "***Format and send the request to the model***" の直後、既存のコードの前に、`grounding.txt` からテキストを読み取る次のコード スニペットを追加して、ユーザー プロンプトにグラウンディング コンテキストを追加します。

    **C#** : Program.cs

    ```csharp
    // Format and send the request to the model
    Console.WriteLine("\nAdding grounding context from grounding.txt");
    string groundingText = System.IO.File.ReadAllText("grounding.txt");
    userMessage = groundingText + userMessage;
    ```

    **Python**: prompt-engineering.py

    ```python
    # Format and send the request to the model
    print("\nAdding grounding context from grounding.txt")
    grounding_text = open(file="grounding.txt", encoding="utf8").read().strip()
    user_message = grounding_text + user_message
    ```

1. ファイルを保存し、アプリを再実行します。
1. 次のプロンプトを入力します (**システム メッセージ**は入力されたままで、`system.txt` に保存されています)。

    **システム メッセージ**

    ```prompt
    You're an AI assistant who helps people find information. You'll provide answers from the text provided in the prompt, and respond concisely.
    ```

    **ユーザー メッセージ:**

    ```prompt
    What animal is the favorite of children at Contoso?
    ```

> **ヒント**: Azure OpenAI からの完全な応答を確認したい場合は、**printFullResponse** 変数を `True` に設定し、アプリを再実行します。

## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、**Azure portal** (`https://portal.azure.com`) でデプロイまたはリソース全体を忘れずに削除します。
