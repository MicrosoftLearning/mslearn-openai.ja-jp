---
lab:
  title: アプリで Azure OpenAI SDK を使う
  status: stale
---

# アプリで Azure OpenAI API を使う

Azure OpenAI Service を使用すると、開発者はチャットボットや言語モデルをはじめとして、人間の自然な言語を理解することに優れたその他のアプリケーションを作成できます。 Azure OpenAI では、事前トレーニング済みの AI モデルにアクセスできるだけでなく、アプリケーションの特定の要件を満たすためにこれらのモデルをカスタマイズおよび微調整するための一連の API やツールが提供されます。 この演習では、Azure OpenAI にモデルをデプロイし、それを独自のアプリケーションで使う方法について学習します。

この演習のシナリオでは、生成 AI を使ってハイキングの推奨事項を提供できるアプリの実装を任されたソフトウェア開発者の役割を果たします。 この演習で使われる手法は、Azure OpenAI API を使う任意のアプリに適用できます。

この演習には約 **30** 分かかります。

## Azure OpenAI リソースをプロビジョニングする

まだ持っていない場合は、Azure サブスクリプションで Azure OpenAI リソースをプロビジョニングします。

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

## Visual Studio Code でアプリを開発する準備をする

Visual Studio Code を使って Azure OpenAI アプリを開発します。 アプリのコード ファイルは、GitHub リポジトリで提供されています。

> **ヒント**: 既に **mslearn-openai** リポジトリをクローンしている場合は、Visual Studio Code で開きます。 それ以外の場合は、次の手順に従って開発環境に複製します。

1. Visual Studio Code を起動します。
2. コマンド パレットを開き (Shift + Ctrl + P キーを押すか **[表示]**、**[コマンド パレット]** の順に選択)、**Git: Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/mslearn-openai` リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。

    > **注**:Visual Studio Code に、開いているコードを信頼するかどうかを求めるポップアップ メッセージが表示された場合は、ポップアップの **[はい、作成者を信頼します]** オプションをクリックします。

4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、**[今はしない]** を選択します。

## アプリケーションを構成する

C# と Python の両方のアプリケーションが提供されています。 どちらのアプリにも同じ機能があります。 まず、Azure OpenAI リソースの使用を有効にするために、アプリケーションの主要な部分をいくつか完成させます。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**Labfiles/02-azure-openai-api** フォルダーを参照し、言語の設定に応じて **CSharp** または **Python** フォルダーを展開します。 各フォルダーには、Azure OpenAI 機能を統合するアプリの言語固有のファイルが含まれています。
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
    // Add Azure OpenAI packages
    using Azure.AI.OpenAI;
    using OpenAI.Chat;
    ```

    **Python**: test-openai-model.py

    ```python
    # Add Azure OpenAI package
    from openai import AzureOpenAI
    ```

1. 使用する言語のアプリケーション コードで、コメント "***Initialize the Azure OpenAI client...***" を次のコードに置き換えて、クライアントを初期化し、システム メッセージを定義します。

    **C#** : Program.cs

    ```csharp
    // Initialize the Azure OpenAI client
    AzureOpenAIClient azureClient = new (new Uri(oaiEndpoint), new ApiKeyCredential(oaiKey));
    ChatClient chatClient = azureClient.GetChatClient(oaiDeploymentName);
    
    // System message to provide context to the model
    string systemMessage = "I am a hiking enthusiast named Forest who helps people discover hikes in their area. If no area is specified, I will default to near Rainier National Park. I will then provide three suggestions for nearby hikes that vary in length. I will also share an interesting fact about the local nature on the hikes when making a recommendation.";
    ```

    **Python**: test-openai-model.py

    ```python
    # Initialize the Azure OpenAI client
    client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2024-02-15-preview"
            )
    
    # Create a system message
    system_message = """I am a hiking enthusiast named Forest who helps people discover hikes in their area. 
        If no area is specified, I will default to near Rainier National Park. 
        I will then provide three suggestions for nearby hikes that vary in length. 
        I will also share an interesting fact about the local nature on the hikes when making a recommendation.
        """
    ```

1. コメント "***Add code to send request...***" を要求の構築に必要なコードに置き換え、`Temperature` や `MaxOutputTokenCount` など、モデルのさまざまなパラメーターを指定します。

    **C#** : Program.cs

    ```csharp
    // Add code to send request...
    // Get response from Azure OpenAI
    ChatCompletionOptions chatCompletionOptions = new ChatCompletionOptions()
    {
        Temperature = 0.7f,
        MaxOutputTokenCount = 800
    };

    ChatCompletion completion = chatClient.CompleteChat(
        [
            new SystemChatMessage(systemMessage),
            new UserChatMessage(inputText)
        ],
        chatCompletionOptions
    );

    Console.WriteLine($"{completion.Role}: {completion.Content[0].Text}");
    ```

    **Python**: test-openai-model.py

    ```python
    # Add code to send request...
    # Send request to Azure OpenAI model
    response = client.chat.completions.create(
        model=azure_oai_deployment,
        temperature=0.7,
        max_tokens=400,
        messages=[
            {"role": "system", "content": system_message},
            {"role": "user", "content": input_text}
        ]
    )
    generated_text = response.choices[0].message.content

    # Print the response
    print("Response: " + generated_text + "\n")
    ```

1. 変更内容をコード ファイルに保存します。

## アプリケーションのテスト

アプリが構成されたので、それを実行してモデルに要求を送信し、応答を確認します。

1. 対話型ターミナル ペインで、フォルダー コンテキストが優先言語のフォルダーであることを確認します。 その後、次のコマンドを入力してアプリケーションを作成します。

    - **C#** : `dotnet run`
    - **Python**: `python test-openai-model.py`

    > **ヒント**: ターミナル ツールバーの **最大化パネル サイズ** (**^**) アイコンを使用すると、コンソール テキストをさらに表示できます。

1. プロンプトが表示されたら、テキスト `What hike should I do near Rainier?` を入力します。
1. 応答が、*messages* 配列に追加したシステム メッセージに示されているガイドラインに従ってしていることに注意して、出力を確認します。
1. プロンプト `Where should I hike near Boise? I'm looking for something of easy difficulty, between 2 to 3 miles, with moderate elevation gain.` を入力し、出力を確認します。
1. 使用する言語のコード ファイルで、要求の *temperature* パラメーター値を **1.0** に変更し、ファイルを保存します。
1. 上記のプロンプトを使ってアプリケーションを再度実行し、出力を確認します。

温度を上げると、ランダム性が高くなるため、同じテキストが指定された場合でも、多くの場合、応答が変化します。 これを数回実行して、出力がどのように変化するかを確認できます。 同じ入力で温度に異なる値を使用してみてください。

## 会話履歴を維持する

ほとんどの実際のアプリケーションでは、会話の前の部分を参照できることにより、AI エージェントとのより現実的な対話が可能になります。 Azure OpenAI API は設計上ステートレスですが、プロンプトに会話の履歴を提供することで、AI モデルが過去のメッセージを参照できるようになります。

1. アプリを再度実行し、プロンプト `Where is a good hike near Boise?` を指定します。
1. 出力を確認し、それからプロンプト `How difficult is the second hike you suggested?` を指定します。
1. モデルからの応答は、言及しているハイキングを理解できないことを示している可能性があります。 これを修正するには、モデルが参照用に過去の会話メッセージを保持できるようにします。
1. アプリケーションでは、以前のプロンプトと応答を、今後送信するプロンプトに追加する必要があります。 **システム メッセージ**の定義の下に、次のコードを追加します。

    **C#** : Program.cs

    ```csharp
    // Initialize messages list
    var messagesList = new List<ChatMessage>()
    {
        new SystemChatMessage(systemMessage),
    };
    ```

    **Python**: test-openai-model.py

    ```python
    # Initialize messages array
    messages_array = [{"role": "system", "content": system_message}]
    ```

1. コメント "***Add code to send request...***"の下で、コメントから **while** ループの最後までのすべてのコードを次のコードに置き換えて、ファイルを保存します。 コードはほとんど同じですが、メッセージ配列を使って会話履歴を格納するようになりました。

    **C#** : Program.cs

    ```csharp
    // Add code to send request...
    // Build completion options object
    messagesList.Add(new UserChatMessage(inputText));

    ChatCompletionOptions chatCompletionOptions = new ChatCompletionOptions()
    {
        Temperature = 0.7f,
        MaxOutputTokenCount = 800
    };

    ChatCompletion completion = chatClient.CompleteChat(
        messagesList,
        chatCompletionOptions
    );

    // Return the response
    string response = completion.Content[0].Text;

    // Add generated text to messages list
    messagesList.Add(new AssistantChatMessage(response));

    Console.WriteLine("Response: " + response + "\n");
    ```

    **Python**: test-openai-model.py

    ```python
    # Add code to send request...
    # Send request to Azure OpenAI model
    messages_array.append({"role": "user", "content": input_text})

    response = client.chat.completions.create(
        model=azure_oai_deployment,
        temperature=0.7,
        max_tokens=1200,
        messages=messages_array
    )
    generated_text = response.choices[0].message.content
    # Add generated text to messages array
    messages_array.append({"role": "assistant", "content": generated_text})

    # Print generated text
    print("Summary: " + generated_text + "\n")
    ```

1. ファイルを保存します。 追加したコードでは、以前の入力と応答がプロンプト配列に追加され、モデルが会話の履歴を理解できるようになっていることに注目してください。
1. ターミナル ペインで次のコマンドを入力してアプリケーションを実行します。

    - **C#** : `dotnet run`
    - **Python**: `python test-openai-model.py`

1. アプリを再度実行し、プロンプト `Where is a good hike near Boise?` を指定します。
1. 出力を確認し、それからプロンプト `How difficult is the second hike you suggested?` を指定します。
1. モデルが提案した 2 番目のハイキングについての応答が得られる可能性が高く、より現実的な会話が得られます。 以前の回答を参照して追加のフォロー アップの質問をすることができ、そのたびに履歴によってモデルが回答するためのコンテキストが提供されます。

    > **ヒント**: 出力トークン数は 800 のみに設定されているため、会話が長くなりすぎると、アプリケーションで使用できるトークンがなくなり、プロンプトが不完全になります。 運用環境での使用では、履歴の長さを最新の入力と応答に制限すると、必要なトークンの数を制御するのに役立ちます。

## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、**Azure portal** (`https://portal.azure.com`) でデプロイまたはリソース全体を忘れずに削除します。
