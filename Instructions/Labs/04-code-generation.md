---
lab:
  title: Azure OpenAI Service を使用してコードを生成して改善する
---

# Azure OpenAI Service を使用してコードを生成して改善する

Azure OpenAI Service モデルでは、自然言語のプロンプトを使用してコードを自動的に生成でき、完成したコードのバグを修正したり、コードにコメントを付けたりすることができます。 このようなモデルは、コードの実行内容とコードを改善する方法を理解するのに役立つように、既存のコードを説明し、簡略化できます。

この演習には、約 **25** 分かかります。

## 開始する前に

Azure OpenAI Service へのアクセスが承認されている Azure サブスクリプションが必要になります。

- 無料の Azure サブスクリプションにサインアップするには、[https://azure.microsoft.com/free](https://azure.microsoft.com/free) にアクセスしてください。
- Azure OpenAI Service へのアクセスを要求するには、[https://aka.ms/oaiapply](https://aka.ms/oaiapply) にアクセスしてください。

## Azure OpenAI リソースをプロビジョニングする

Azure OpenAI モデルを使用する前に、Azure サブスクリプションに Azure OpenAI リソースをプロビジョニングする必要があります。

1. [Azure portal](https://portal.azure.com) にサインインします。
2. 次の設定で **Azure OpenAI** リソースを作成します。
    - **サブスクリプション**: Azure OpenAI Service のアクセスが承認されている Azure サブスクリプション。
    - **リソース グループ**: 既存のリソース グループを選択するか、任意の名前を使用して新規に作成します。
    - **リージョン**: 使用できるリージョンを選択します。
    - **名前**: 任意の一意の名前。
    - **価格レベル**: Standard S0
3. デプロイが完了するまで待ちます。 次に、Azure portal で、デプロイされた Azure OpenAI リソースに移動します。
4. **[キーとエンドポイント]** ページに移動し、後で使用するためにテキスト ファイルに保存します。

## モデルをデプロイする

Azure OpenAI API をコードの生成に使用するには、まず、**Azure OpenAI Studio** を介して使用するモデルをデプロイする必要があります。 デプロイが完了したら、モデルをプレイグラウンドで使用し、そのモデルをアプリで参照します。

1. Azure OpenAI リソースの **[概要]** ページで、 **[探索]** ボタンを使用して、新しいブラウザー タブで Azure OpenAI Studio を開きます。または、[Azure OpenAI Studio](https://oai.azure.com/?azure-portal=true) に直接移動します。
2. Azure OpenAI Studio の [**デプロイ**] ページで、既存のモデルのデプロイを表示します。 まだデプロイがない場合は、次の設定で **gpt-35-turbo-16k** モデルの新しいデプロイを作成します。
    - **モデル**: gpt-35-turbo-16k
    - **モデル バージョン**: 既定値に自動更新
    - **デプロイの名前**: *任意の一意の名前*
    - **詳細オプション**
        - **コンテンツ フィルター**: 既定
        - **1 分あたりのトークンのレート制限**: 5K\*
        - **動的クォータを有効にする**: 有効

    > \* この演習は、1 分あたり 5,000 トークンのレート制限内で余裕を持って完了できます。またこの制限によって、同じサブスクリプションを使用する他のユーザーのために容量を残すこともできます。

> **注**: 一部のリージョンでは、新しいモデル デプロイ インターフェイスに [**モデル バージョン**] オプションが表示されません。 この場合は、オプションを設定せずにそのまま続行してください

## チャット プレイグラウンドでコードを生成する

アプリで使用する前に、チャット プレイグラウンドで Azure OpenAI によってコードが生成され、説明される方法を確認します。

1. [Azure OpenAI Studio](https://oai.azure.com/?azure-portal=true) の左側のペインで、 **[チャット]** プレイグラウンドに移動します。
1. 1. [**構成**] で、モデルのデプロイが選択されていることを確認します。
1. 上部の **[アシスタントのセットアップ]** セクションで、 **[既定]** のシステム メッセージ テンプレートを選択します。
1. **[チャット セッション]** セクションで、次のプロンプトを入力し、*Enter* キーを押します。

    ```code
   Write a function in python that takes a character and string as input, and returns how many times that character appears in the string
    ```

1. モデルは関数で応答し、その関数の実行内容と呼び出し方法に関する説明が含まれます。
1. 次に、プロンプト `Do the same thing, but this time write it in C#` を送信します。
1. 出力を確認します。 モデルは、最初と非常によく似た応答を返しますが、今度は、C# のコードです。 任意の異なる言語、または関数で入力文字列の反転などの別のタスクを完了するように再度要求することができます。
1. 次に、Ruby で記述されたランダム関数の次の例を使用して、AI を使用してコードを理解する方法を調べることにしましょう。 次のプロンプトをユーザー メッセージとして送信します。

    ```code
    What does the following function do?  
    ---  
    def random_func(n)
      start = [0, 1]
      (n - 2).times do
        start << start[-1] + start[-2]
      end
      start.shuffle.each do |num|
        puts num
      end
    end
    ```

1. 出力を確認します。これは、関数の実行内容を自然言語で説明しています。 これを、自分が使い慣れた言語で書き直すようにモデルに要求します。

## Cloud Shell でアプリケーションを設定する

Azure OpenAI モデルと統合する方法を示すために、Azure 上の Cloud Shell で実行される短いコマンドライン アプリケーションを使用します。 Cloud Shell を操作するには、新しいブラウザー タブを開きます。

1. [Azure portal](https://portal.azure.com?azure-portal=true) で、ページ上部の検索ボックスの右側にある **[>_]** (*Cloud Shell*) ボタンを選択します。 ポータルの下部に Cloud Shell ペインが開きます。

    ![上部の検索ボックスの右側にあるアイコンをクリックして Cloud Shell を開始している状態のスクリーンショット。](../media/cloudshell-launch-portal.png#lightbox)

2. Cloud Shell を初めて開くと、使用するシェルの種類 (*Bash* または *PowerShell*) を選択するように求められる場合があります。 **[Bash]** を選択します。 このオプションが表示されない場合は、この手順をスキップします。  

3. Cloud Shell 用のストレージを作成するように求められたら、 **[詳細設定の表示]** を選び、次の設定を選びます。
    - **[サブスクリプション]**: 自分のサブスクリプション
    - **Cloud Shell リージョン**: 使用できるリージョンを選びます
    - **Show VNET isolation setings (VNET 分離の設定を表示する)** : オフ
    - **リソース グループ**: Azure OpenAI リソースをプロビジョニングした既存のリソース グループを使います
    - **ストレージ アカウント**: 一意の名前で新しいストレージ アカウントを作成します
    - **ファイル共有**: 一意の名前で新しいファイル共有を作成します

    その後、ストレージが作成されるのを 1 分程度待ちます。

    > **注**: Azure サブスクリプションに既に Cloud Shell を設定している場合は、⚙️ メニューの **[ユーザー設定のリセット]** オプションを使用して、最新バージョンの Python と .NET Framework がインストールされていることを確かめる必要がある場合があります。

4. Cloud Shell ペインの左上に表示されるシェルの種類が *Bash* であることを確認します。 *PowerShell* の場合は、ドロップダウン メニューを使用して *Bash* に切り替えます。

5. ターミナルが起動したら、次のコマンドを入力してサンプル アプリケーションをダウンロードし、`azure-openai` という名前のフォルダーに保存します。

    ```bash
    rm -r azure-openai -f
    git clone https://github.com/MicrosoftLearning/mslearn-openai azure-openai
    ```

6. ファイルは、**azure-openai** という名前のフォルダーにダウンロードされます。 次のコマンドを使用して、この演習のラボ ファイルに移動します。

    ```bash
    cd azure-openai/Labfiles/04-code-generation
    ```

7. 次のコマンドを実行して、組み込みのコード エディターを開きます。

    ```bash
    code .
    ```

8. コード エディターで、**サンプル コード** フォルダーを展開し、アプリケーションでモデルを使用して改善するコード ファイルを確認します。

    > **ヒント**: Azure Cloud Shell コード エディターを使用して Azure Cloud Shell 環境でファイルを操作する方法の詳細については、[Azure Cloud Shell コード エディターのドキュメント](https://learn.microsoft.com/azure/cloud-shell/using-cloud-shell-editor)を参照してください。

## アプリケーションの作成

この演習では、Azure OpenAI リソースの使用を有効にするために、アプリケーションのいくつかの重要な部分を完成します。 C# と Python の両方のアプリケーションが提供されています。

1. コード エディターで、選択した言語の言語フォルダーを展開します。

2. 言語の構成ファイルを開きます。

    - **C#** : `appsettings.json`
    - **Python**: `.env`

3. 構成値を更新して、作成した Azure OpenAI リソースの**エンドポイント**や**キー**と、デプロイの名前を含めるようにします。 ファイルを保存します。

4. コンソール ウィンドウで次のコマンドを入力して、優先言語のフォルダーに移動し、必要なパッケージをインストールします。

    **C#**

    ```bash
    cd CSharp
    dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.9
    ```

    **Python**

    ```bash
    cd Python
    pip install python-dotenv
    pip install openai==1.2.0
    ```

5. 選択した言語のこのフォルダーでコード ファイルを選択し、必要なライブラリを追加します。

    **C#** : Program.cs

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```

    **Python**: code-generation.py

    ```python
    # Add OpenAI import
    from openai import AzureOpenAI
    ```

6. クライアントを構成するために必要なコードを追加します。

    **C#** : Program.cs

    ```csharp
    // Initialize the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    ```

    **Python**: code-generation.py

    ```python
    # Set OpenAI configuration settings
    client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2023-05-15"
            )
    ```

7. Azure OpenAI モデルを呼び出す関数で、書式を設定して要求をモデルに送信するコードを追加します。

    **C#** : Program.cs

    ```csharp
    // Create chat completion options
    var chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatMessage(ChatRole.System, systemPrompt),
            new ChatMessage(ChatRole.User, userPrompt)
        },
        Temperature = 0.7f,
        MaxTokens = 1000,
        DeploymentName = oaiModelName
    };

    // Get response from Azure OpenAI
    Response<ChatCompletions> response = await client.GetChatCompletionsAsync(chatCompletionsOptions);

    ChatCompletions completions = response.Value;
    string completion = completions.Choices[0].Message.Content;
    ```

    **Python**: code-generation.py

    ```python
    # Build the messages array
    messages =[
        {"role": "system", "content": system_message},
        {"role": "user", "content": user_message},
    ]
    
    # Call the Azure OpenAI model
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7,
        max_tokens=1000
    )
    ```

## アプリケーションを実行する

アプリが構成されたので、それを実行して、各ユース ケースのコードを生成してみましょう。 ユース ケースは、アプリ内で番号付けされ、任意の順序で実行できます。

> **注**: ユーザーによるモデルの呼び出し頻度が高すぎると、レート制限が発生する場合があります。 トークン レート制限に関するエラーが発生した場合は、少しの間待ってからもう一度試してください。

1. コード エディターで、`sample-code` フォルダーを展開し、関数と選択した言語のアプリを少しの間観察します。 これらのファイルは、アプリ内のタスクに使用されます。
1. Cloud Shell bash ターミナルで、選択した言語のフォルダーに移動します。
1. アプリケーションを実行します。

    - **C#** : `dotnet run`
    - **Python**: `python code-generation.py`

1. オプション **1** を選択して、コードにコメントを追加します。 これらの各タスクの応答には数秒かかる場合があることに注意してください。
1. 結果は、`result/app.txt` に格納されます。 そのファイルを開き、`sample-code` 内の関数ファイルと比較します。
1. 次に、オプション **2** を選択して、その同じ関数の単体テストを作成します。
1. 結果は `result/app.txt` の内容に置き換わり、その関数の 4 つの単体テストの詳細を示します。
1. 次に、オプション **3** を選択して、Go Fish を再生するためのアプリのバグを修正します。
1. 結果は `result/app.txt` の内容に置き換わり、いくつかの点が修正された非常によく似たコードが含まれます。

    - **C#** : 修正は、行 30 と 59 で行われます
    - **Python**: 修正は、行 18 と 31 で行われます

バグのある行を Azure OpenAI からの応答に置き換えると、`sample-code` 内の Go Fish のアプリを実行できます。 修正しないで実行すると、ただしく動作しません。

この Go Fish アプリのコードは一部の構文が修正されてはいますが、ゲームを厳密には正しく表現したものではないことに注意することが重要です。 よく見ると、カードを引くときに山札が空かどうかを確認しない、プレーヤーがペアを手に入れたときにペアを手札から削除しないなどの問題があり、その他にもカード ゲームを理解する必要があるバグがいくつかあります。 これは、コード生成を支援するのに生成 AI モデルがいかに役立つかを示す好例ですが、正しいと信頼することはできず、開発者による検証が必要です。

Azure OpenAI からの完全な応答を確認したい場合は、`printFullResponse` 変数を `True` に設定し、アプリを再実行できます。

## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、[Azure portal](https://portal.azure.com/?azure-portal=true) 内のデプロイまたはリソース全体を必ず削除してください。
