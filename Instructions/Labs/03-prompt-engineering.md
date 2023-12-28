---
lab:
  title: アプリでプロンプト エンジニアリングを利用する
---

# アプリでプロンプト エンジニアリングを利用する

Azure OpenAI Service を使用すると、開発者はチャットボットや言語モデルをはじめとして、人間の自然な言語を理解することに優れたその他のアプリケーションを作成できます。 Azure OpenAI では、事前トレーニング済みの AI モデルにアクセスできるだけでなく、アプリケーションの特定の要件を満たすためにこれらのモデルをカスタマイズおよび微調整するための一連の API やツールが提供されます。 この演習では、モデルを Azure OpenAI にデプロイし、独自のアプリケーションで使用してテキストを要約する方法について説明します。

Azure OpenAI Service を使用する場合、開発者がプロンプトをどのように形成するかは、生成 AI モデルの応答方法に大きく影響します。 明瞭かつ簡潔に要求すれば、Azure OpenAI はコンテンツを調整し、書式設定を行うことができます。 この演習では、類似するコンテンツのさまざまなプロンプトが、要件をより十分に満たす AI モデルの応答を形成するためにどのように役立つかを学習します。

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

Azure OpenAI API を使用するには、まず、**Azure OpenAI Studio** を介して使用するモデルをデプロイする必要があります。 デプロイが完了したら、アプリでそのモデルを参照します。

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

## チャット プレイグラウンドでプロンプト エンジニアリングを適用する

アプリを使用する前に、プロンプト エンジニアリングによってプレイグラウンドでのモデルの応答がどのように向上するかを調べます。 この最初の例では、ユーモアのある名前を持つ動物の Python アプリを作成しようとしているとします。

1. [Azure OpenAI Studio](https://oai.azure.com/?azure-portal=true) の左側のペインで、 **[チャット]** プレイグラウンドに移動します。
1. **[構成]** で、モデル デプロイが選択されていることを確認します。
1. 上部の **[アシスタントのセットアップ]** セクションで、システム メッセージとして「`You are a helpful AI assistant`」と入力します。
1. **[チャット セッション]** セクションで、次のプロンプトを入力し、*Enter* キーを押します。

    ```code
   1. Create a list of animals
   2. Create a list of whimsical names for those animals
   3. Combine them randomly into a list of 25 animal and name pairs
    ```

1. モデルは回答を番号付きリストに分けて応答し、それらはおそらくプロンプトを満たしています。 これは適切な応答ですが、求めている回答ではありません。
1. 次に、システム メッセージを更新して、`You are an AI assistant helping write python code. Complete the app based on provided comments` という指示を含めます。 **[変更内容を保存]** をクリックします。
1. 指示を Python コメントとして書式設定します。 次のプロンプトをモデルに送信します。

    ```code
   # 1. Create a list of animals
   # 2. Create a list of whimsical names for those animals
   # 3. Combine them randomly into a list of 25 animal and name pairs
    ```

1. モデルは、コメントで要求された内容を実行する完全な Python コードで正しく応答するはずです。
1. 次に、記事を分類しようとしている場合のいくつかの短いプロンプトの影響を確認します。 システム メッセージに戻り、もう一度「`You are a helpful AI assistant`」と入力し、変更を保存します。 これにより、新しいチャット セッションが作成されます。
1. 次のプロンプトをモデルに送信します。

    ```code
   Severe drought likely in California

   Millions of California residents are bracing for less water and dry lawns as drought threatens to leave a large swath of the region with a growing water shortage.
   
   In a remarkable indication of drought severity, officials in Southern California have declared a first-of-its-kind action limiting outdoor water use to one day a week for nearly 8 million residents.
   
   Much remains to be determined about how daily life will change as people adjust to a drier normal. But officials are warning the situation is dire and could lead to even more severe limits later in the year.
    ```

1. 応答では、カリフォルニアの干ばつに関するいくつかの情報が提供されます。 不適切な応答ではありませんが、求めている分類ではありません。
1. システム メッセージの近くにある **[アシスタントのセットアップ]** セクションで、 **[例の追加]** ボタンを選択します。 次の例を追加します。

    **ユーザー:**

    ```code
   New York Baseballers Wins Big Against Chicago
   
   New York Baseballers mounted a big 5-0 shutout against the Chicago Cyclones last night, solidifying their win with a 3 run homerun late in the bottom of the 7th inning.
   
   Pitcher Mario Rogers threw 96 pitches with only two hits for New York, marking his best performance this year.
   
   The Chicago Cyclones' two hits came in the 2nd and the 5th innings, but were unable to get the runner home to score.
    ```

    **アシスタント:**

    ```code
   Sports
    ```

1. 次のテキストを含む別の例を追加します。

    **ユーザー:**

    ```code
    Joyous moments at the Oscars
    
    The Oscars this past week where quite something!
    
    Though a certain scandal might have stolen the show, this year's Academy Awards were full of moments that filled us with joy and even moved us to tears.
    These actors and actresses delivered some truly emotional performances, along with some great laughs, to get us through the winter.
    
    From Robin Kline's history-making win to a full performance by none other than Casey Jensen herself, don't miss tomorrows rerun of all the festivities.
    ```

    **アシスタント:**

    ```code
    Entertainment
    ```

1. 変更した内容をアシスタントのセットアップに保存し、カリフォルニアの干ばつに関する同じプロンプトを送信して、便宜上、ここでもう一度提供します。

    ```code
   Severe drought likely in California

   Millions of California residents are bracing for less water and dry lawns as drought threatens to leave a large swath of the region with a growing water shortage.
   
   In a remarkable indication of drought severity, officials in Southern California have declared a first-of-its-kind action limiting outdoor water use to one day a week for nearly 8 million residents.
   
   Much remains to be determined about how daily life will change as people adjust to a drier normal. But officials are warning the situation is dire and could lead to even more severe limits later in the year.
    ```

1. 今度は、指示がなくても、モデルは適切な分類で応答するはずです。

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
   cd azure-openai/Labfiles/03-prompt-engineering
    ```

7. 次のコマンドを実行して、組み込みのコード エディターを開きます。

    ```bash
    code .
    ```

8. コード エディターで **prompts** フォルダーを展開し、アプリケーションでモデルに送信するプロンプトを含むテキスト ファイルを確認します。

    > **ヒント**: Azure Cloud Shell コード エディターを使用して Azure Cloud Shell 環境でファイルを操作する方法の詳細については、[Azure Cloud Shell コード エディターのドキュメント](https://learn.microsoft.com/azure/cloud-shell/using-cloud-shell-editor)を参照してください。

## アプリケーションの作成

この演習では、Azure OpenAI リソースの使用を有効にするために、アプリケーションのいくつかの重要な部分を完成します。 C# と Python の両方のアプリケーションが提供されています。 どちらのアプリにも同じ機能があります。

1. コード エディターで、言語の設定に応じて **CSharp** または **Python** フォルダーを展開します。

2. 言語の構成ファイルを開きます。

    - C#: `appsettings.json`
    - Python: `.env`
    
3. 構成値を更新して、作成した Azure OpenAI リソースの**エンドポイント**や**キー**と、デプロイしたモデル名を含めるようにします。 ファイルを保存します。

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

5. 選択した言語のフォルダーに移動し、コード ファイルを選択して、必要なライブラリを追加します。

    **C#**

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```

    **Python**

    ```python
    # Add OpenAI import
    from openai import AzureOpenAI
    ```

6. 選択した言語のアプリケーション コードを開き、クライアントを構成するために必要なコードを追加します。

    **C#**

    ```csharp
    // Initialize the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    ```

    **Python**

    ```python
    # Initialize the Azure OpenAI client
    client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2023-05-15"
            )
    ```

7. Azure OpenAI モデルを呼び出す関数で、書式を設定して要求をモデルに送信するコードを追加します。

    **C#**

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
        MaxTokens = 800,
        DeploymentName = oaiModelName
    };
    
    // Get response from Azure OpenAI
    Response<ChatCompletions> response = await client.GetChatCompletionsAsync(chatCompletionsOptions);
    
    ChatCompletions completions = response.Value;
    string completion = completions.Choices[0].Message.Content;
    ```

    **Python**

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
        max_tokens=800
    )
    ```

## アプリケーションを実行する

アプリが構成されたので、それを実行してモデルに要求を送信し、応答を確認します。 異なるオプションの間で違いがあるのはプロンプトの内容のみであり、他のすべてのパラメーター (トークン数や温度など) は要求ごとに変わりがないことがわかります。

各プロンプトは送信時にコンソールに表示され、生成される応答が、プロンプトの違いによってどのように異なるかを確認できます。

1. Cloud Shell bash ターミナルで、選択した言語のフォルダーに移動します。
1. アプリケーションを実行します。ターミナルを拡張してブラウザー ウィンドウのほぼ全体に表示されるようにします。

    - **C#** : `dotnet run`
    - **Python**: `python prompt-engineering.py`

1. 最も基本的なプロンプトでは、オプション **1** を選択します。
1. プロンプト入力と生成された出力を確認します。 AI モデルは、野生動物の救護に関する適切で一般的な概要を生成します。
1. 次に、オプション **2** を選択して、野生動物の救護に関するいくつかの詳細と共に、概要メールを求めるプロンプトを提示します。
1. プロンプト入力と生成された出力を確認します。 今度は、特定の動物と寄付の呼びかけを含む応答がメール形式で表示されます。
1. 次に、オプション **3** を選択して、上記と同様のメールを要求します。ただし、追加の動物を含むテーブルを書式設定します。
1. プロンプト入力と生成された出力を確認します。 今度は、特定の方法で書式設定された (この場合は、末尾付近にテーブルを含む) テキストを含む同様のメールが表示され、要求されたときに生成 AI モデルが出力をどのように書式設定できるかが示されます。
1. 次に、オプション **4** を選択して、同様のメールを要求します。ただし、今度は、トーンの異なるシステム メッセージを指定します。
1. プロンプト入力と生成された出力を確認します。 メールは同様の形式で表示されますが、今度は、よりくだけたトーンになります。 ジョークが含まれている可能性もあります。

多くの場合、温度を上げると、ランダム性が高くなるため、同じプロンプトを指定した場合でも応答が変化します。 異なる温度または top_p 値で複数回実行すると、同じプロンプトに対する応答にどのような影響があるかを確認できます。

Azure OpenAI からの完全な応答を確認したい場合は、`printFullResponse` 変数を `True` に設定し、アプリを再実行できます。

## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、[Azure portal](https://portal.azure.com/?azure-portal=true) 内のデプロイまたはリソース全体を必ず削除してください。
