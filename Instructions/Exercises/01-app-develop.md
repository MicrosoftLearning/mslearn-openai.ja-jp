---
lab:
  title: Azure OpenAI Service を使用したアプリケーションの開発
  status: new
---

# Azure OpenAI Service を使用したアプリケーションの開発

Azure OpenAI Service を使用すると、開発者はチャットボットや REST API または 言語固有の SDK を使用して、人間の自然な言語を理解することに優れたその他のアプリケーションを作成できます。 これらの言語モデルを使用する場合、開発者がプロンプトをどのように形成するかは、生成 AI モデルの応答方法に大きく影響します。 明瞭かつ簡潔に要求すれば、Azure OpenAI はコンテンツを調整し、書式設定を行うことができます。 この演習では、アプリケーションを Azure OpenAI に接続する方法と、類似するコンテンツのさまざまなプロンプトが、要件をより十分に満たす AI モデルの応答を形成するためにどのように役立つかを学習します。

この演習のシナリオで、あなたは野生動物のマーケティング キャンペーンに取り組むソフトウェア開発者の役割を演じます。 あなたは、生成 AI を使って広告メールを改善し、チームに適用される可能性のある記事を分類する方法を検討しています。 この演習で使われるプロンプト エンジニアリング手法は、さまざまなユース ケースにも同様に適用できます。

この演習には約 **30** 分かかります。

## このコースのリポジトリを複製する

まだ行っていない場合は、このコースのコード リポジトリを複製する必要があります。

1. Visual Studio Code を起動します。
2. コマンド パレットを開き (Shift + Ctrl + P キーを押すか **[表示]**、**[コマンド パレット]** の順に選択)、**Git: Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/mslearn-openai` リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。
4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、**[今はしない]** を選択します。

## Azure OpenAI リソースをプロビジョニングする

まだ持っていない場合は、Azure サブスクリプションで Azure OpenAI リソースをプロビジョニングします。

1. **Azure portal** (`https://portal.azure.com`) にサインインします。

1. 次の設定で **Azure OpenAI** リソースを作成します。
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

1. デプロイが完了するまで待ちます。 次に、Azure portal でデプロイされた Azure OpenAI リソースに移動します。

## モデルをデプロイする

次に、Cloud Shell から Azure OpenAI モデル リソースをデプロイします。

1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、***Bash*** 環境を選択し、Azure portal 内に新しい Cloud Shell を作成します。 Azure portal の下部にあるペインに、Cloud Shell のコマンド ライン インターフェイスが表示されます。

    > **注**: *PowerShell* 環境を使用するクラウド シェルを以前に作成している場合は、それを ***Bash*** に切り替えること。

1. この例を使い、次の変数を上記から独自のリソースの値に置き換えます。

    ```dotnetcli
    az cognitiveservices account deployment create \
       -g <your_resource_group> \
       -n <your_OpenAI_service> \
       --deployment-name gpt-4o \
       --model-name gpt-4o \
       --model-version 2024-05-13 \
       --model-format OpenAI \
       --sku-name "Standard" \
       --sku-capacity 5
    ```

> **注**: SKU 容量は、1 分あたりトークン数 (1,000 単位) で測定されます。 同じサブスクリプションを使用する他のユーザーのための容量を残しながらこの演習を完了するのに、1 分あたり 5,000 トークンのレート制限で十分です。

## アプリケーションを構成する

C# と Python の両方のアプリケーションが用意されており、どちらのアプリも同じ機能を備えています。 まず、非同期 API 呼び出しで Azure OpenAI リソースを使用できるように、アプリケーションの主要な部分をいくつか完成させます。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**Labfiles/01-code-generation** フォルダーを参照し、言語の優先順位に応じて、**CSharp** または **Python** フォルダーを展開します。 各フォルダーには、Azure OpenAI 機能を統合するアプリの言語固有のファイルが含まれています。
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
    - モデル デプロイに指定した**デプロイ名**。
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

    **Python**: application.py

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

    **Python**: application.py

    ```python
    # Configure the Azure OpenAI client
    client = AsyncAzureOpenAI(
        azure_endpoint = azure_oai_endpoint, 
        api_key=azure_oai_key,  
        api_version="2024-02-15-preview"
    )
    ```

3. Azure OpenAI モデルを呼び出す関数で、***Azure OpenAI サービスから応答を取得します***のコメントの下で、書式を設定して要求をモデルに送信するコードを追加します。

    **C#** : Program.cs

    ```csharp
    // Get response from Azure OpenAI
    ChatCompletionOptions chatCompletionOptions = new ChatCompletionOptions()
    {
        Temperature = 0.7f,
        MaxOutputTokenCount = 800
    };

    ChatCompletion completion = chatClient.CompleteChat(
        [
            new SystemChatMessage(systemMessage),
            new UserChatMessage(userMessage)
        ],
        chatCompletionOptions
    );

    Console.WriteLine($"{completion.Role}: {completion.Content[0].Text}");
    ```

    **Python**: application.py

    ```python
    # Get response from Azure OpenAI
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
    - **Python**: `python application.py`

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

## 根拠付けるコンテキストを使用しチャット履歴をメンテナンスする

1. 最後のイテレーションでは、メールの生成から離れて、*根拠付けるコンテキスト*を検討し、チャット履歴をメンテナンスします。 ここでは、単純なシステム メッセージを指定し、根拠付けるコンテキストをチャット履歴の始まりとして提供するようにアプリを変更します。 その後、アプリはユーザー入力を追加し、グラウンディング コンテキストから情報を抽出してユーザー プロンプトに回答するようになります。
1. ファイル `grounding.txt` を開き、挿入するグラウンディング コンテキストをざっと読みます。
1. アプリの ***Initialize messages list*** (メッセージ リストを初期化する) というコメントの直後で既存コードの前に、`grounding.txt` からテキストを読み取り、根拠付けるコンテキストでチャット履歴を初期化する次のコード スニペットを追加します。

    **C#** : Program.cs

    ```csharp
    // Initialize messages list
    Console.WriteLine("\nAdding grounding context from grounding.txt");
    string groundingText = System.IO.File.ReadAllText("grounding.txt");
    var messagesList = new List<ChatMessage>()
    {
        new UserChatMessage(groundingText),
    };
    ```

    **Python**: application.py

    ```python
    # Initialize messages array
    print("\nAdding grounding context from grounding.txt")
    grounding_text = open(file="grounding.txt", encoding="utf8").read().strip()
    messages_array = [{"role": "user", "content": grounding_text}]
    ```

1. ***Format and send the request to the model*** (要求を書式設定しモデルに送信する) というコメントの下で、コメントから **while** ループの終わりまでのコードを次のコードに置き換えます。 コードはほとんど同じですが、メッセージ配列を使用して要求をモデルに送信するようになりました。

    **C#** : Program.cs
   
    ```csharp
    // Format and send the request to the model
    messagesList.Add(new SystemChatMessage(systemMessage));
    messagesList.Add(new UserChatMessage(userMessage));
    GetResponseFromOpenAI(messagesList);
    ```

    **Python**: application.py

    ```python
    # Format and send the request to the model
    messages_array.append({"role": "system", "content": system_text})
    messages_array.append({"role": "user", "content": user_text})
    await call_openai_model(messages=messages_array, 
        model=azure_oai_deployment, 
        client=client
    )
    ```

1. ***Define the function that will get the response from Azure OpenAI endpoint*** (Azure OpenAI エンドポイントから応答を取得する関数を定義する) というコメントの下で、関数宣言を次のコードに置き換えて、関数 `GetResponseFromOpenAI` (C# の場合) または `call_openai_model` (Python の場合) を呼び出すときにチャット履歴リストを使用するようにします。

    **C#** : Program.cs
   
    ```csharp
    // Define the function that gets the response from Azure OpenAI endpoint
    private static void GetResponseFromOpenAI(List<ChatMessage> messagesList)
    ```

    **Python**: application.py

    ```python
    # Define the function that will get the response from Azure OpenAI endpoint
    async def call_openai_model(messages, model, client):
    ```
    
1. 最後に、***Get response from Azure OpenAI*** (Azure OpenAI の応答を取得する) というコメントの下のすべてのコードを置き換えます。 コードはほとんど同じですが、メッセージ配列を使って会話履歴を格納するようになりました。

    **C#** : Program.cs
   
    ```csharp
    // Get response from Azure OpenAI
    ChatCompletionOptions chatCompletionOptions = new ChatCompletionOptions()
    {
        Temperature = 0.7f,
        MaxOutputTokenCount = 800
    };

    ChatCompletion completion = chatClient.CompleteChat(
        messagesList,
        chatCompletionOptions
    );

    Console.WriteLine($"{completion.Role}: {completion.Content[0].Text}");
    messagesList.Add(new AssistantChatMessage(completion.Content[0].Text));
    ```

    **Python**: application.py

    ```python
    # Get response from Azure OpenAI
    print("\nSending request to Azure OpenAI model...\n")

    # Call the Azure OpenAI model
    response = await client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7,
        max_tokens=800
    )   

    print("Response:\n" + response.choices[0].message.content + "\n")
    messages.append({"role": "assistant", "content": response.choices[0].message.content})
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

   モデルでは、根拠付けるテキスト情報を使用して質問に回答することに注意してください。

1. システム メッセージを変更せずに、ユーザー メッセージに対して次のプロンプトを入力します。

    **ユーザー メッセージ:**

    ```prompt
    How can they interact with it at Contoso?
    ```

    このモデルでは、チャット履歴にある以前の質問にアクセスできるようになったので、"they" が子供として認識され、"it" が子供が好きな動物として認識されていることに注目してください。
   
## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、**Azure portal** (`https://portal.azure.com`) でデプロイまたはリソース全体を忘れずに削除します。
