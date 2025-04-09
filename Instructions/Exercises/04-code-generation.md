---
lab:
  title: Azure OpenAI Service を使用してコードを生成して改善する
  status: stale
---

# Azure OpenAI Service を使用してコードを生成して改善する

Azure OpenAI Service モデルでは、自然言語のプロンプトを使用してコードを自動的に生成でき、完成したコードのバグを修正したり、コードにコメントを付けたりすることができます。 このようなモデルは、コードの実行内容とコードを改善する方法を理解するのに役立つように、既存のコードを説明し、簡略化できます。

この演習のシナリオでは、生成 AI を使ってコーディング タスクをより簡単かつ効率的にする方法を検討するソフトウェア開発者の役割を果たします。 この演習で使われる手法は、他のコード ファイル、プログラミング言語、ユース ケースに適用できます。

この演習には、約 **25** 分かかります。

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

    > \* Azure OpenAI リソースは、リージョンのクォータによって制限されます。 一覧表示されているリージョンには、この演習で使用されるモデル タイプの既定のクォータが含まれています。 リージョンをランダムに選択することで、サブスクリプションを他のユーザーと共有しているシナリオで、1 つのリージョンがクォータ制限に達するリスクが軽減されます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のリソースを作成する必要がある可能性があります。

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

## チャット プレイグラウンドでコードを生成する

アプリで使用する前に、チャット プレイグラウンドで Azure OpenAI によってコードが生成され、説明される方法を確認します。

1. **[プレイグラウンド]** セクションで、**[Chat]** ページを選びます。 **Chat** プレイグラウンド ページは、ボタンの行と 2 つのメイン パネルで構成されます (画面の解像度に応じて、右から左へ水平に、または上から下へ垂直に配置されます)。
    - **構成** - デプロイの選択、システム メッセージの定義、デプロイとやり取りするためのパラメーターの設定に使用されます。
    - **チャット セッション** - チャット メッセージを送信し、応答を表示するために使われます。
1. **デプロイ**で、モデル デプロイが選択されていることを確認します。
1. **[システム メッセージ]** 領域で、システム メッセージを `You are a programming assistant helping write code` に設定し、変更を適用します。
1. **[チャット セッション]** で、次のクエリを送信します。

    ```prompt
    Write a function in python that takes a character and a string as input, and returns how many times the character appears in the string
    ```

    モデルは関数で応答し、その関数の実行内容と呼び出し方法に関する説明が含まれます。

1. 次に、プロンプト `Do the same thing, but this time write it in C#` を送信します。

    モデルは、最初と非常によく似た応答を返しますが、今度は、C# のコードです。 任意の異なる言語、または関数で入力文字列の反転などの別のタスクを完了するように再度要求することができます。

1. 次に、AI を使ってコードを理解することを検討してみましょう。 次のプロンプトをユーザー メッセージとして送信します。

    ```prompt
    What does the following function do?  
    ---  
    def multiply(a, b):  
        result = 0  
        negative = False  
        if a < 0 and b > 0:  
            a = -a  
            negative = True  
        elif a > 0 and b < 0:  
            b = -b  
            negative = True  
        elif a < 0 and b < 0:  
            a = -a  
            b = -b  
        while b > 0:  
            result += a  
            b -= 1      
        if negative:  
            return -result  
        else:  
            return result  
    ```

    モデルは、関数が何を行うか、つまりループを使って 2 つの数値を乗算するということを説明する必要があります。

1. プロンプト `Can you simplify the function?` を送信します。

    モデルでは、関数のより単純なバージョンを記述するはずです。

1. プロンプトを送信します: `Add some comments to the function.`

    モデルはコードにコメントを追加します。

## Visual Studio Code でアプリを開発する準備をする

次に、Azure OpenAI Service を使ってコードを生成するカスタム アプリを構築する方法を見てみましょう。 Visual Studio Code を使用してアプリを開発します。 アプリのコード ファイルは、GitHub リポジトリで提供されています。

> **ヒント**: 既に **mslearn-openai** リポジトリをクローンしている場合は、Visual Studio Code で開きます。 それ以外の場合は、次の手順に従って開発環境に複製します。

1. Visual Studio Code を起動します。
2. コマンド パレットを開き (Shift + Ctrl + P キーを押すか **[表示]**、**[コマンド パレット]** の順に選択)、**Git: Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/mslearn-openai` リポジトリをローカル フォルダーにクローンします (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。

    > **注**:Visual Studio Code に、開いているコードを信頼するかどうかを求めるポップアップ メッセージが表示された場合は、ポップアップの **[はい、作成者を信頼します]** オプションをクリックします。

4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、**[今はしない]** を選択します。

## アプリケーションを構成する

C# と Python の両方のアプリケーションと、要約のテストに使用するサンプル テキスト ファイルが提供されています。 どちらのアプリにも同じ機能があります。 まず、Azure OpenAI リソースの使用を有効にするために、アプリケーションの主要な部分をいくつか完成させます。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**Labfiles/04-code-generation** フォルダーを参照し、言語の設定に応じて、**CSharp** または **Python** フォルダーを展開します。 各フォルダーには、Azure OpenAI 機能を統合するアプリの言語固有のファイルが含まれています。
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

## Azure OpenAI Service モデルを使うコードを追加する

これで、Azure OpenAI SDK を使って、デプロイされたモデルを使う準備が整いました。

1. **[エクスプローラー]** ペインの **CSharp** または **Python** フォルダーで、使用する言語のコード ファイルを開きます。 Azure OpenAI モデルを呼び出す関数のコメント "***Format and send the request to the model***" の下に、要求を書式設定してモデルに送信するコードを追加します。

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
            new SystemChatMessage(systemPrompt),
            new UserChatMessage(userPrompt),
        ],
        chatCompletionsOptions);
    ```

    **Python**: code-generation.py

    ```python
    # Format and send the request to the model
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

1. コード ファイルに加えた変更を保存します。

## アプリケーションの実行

アプリが構成されたので、それを実行して、各ユース ケースのコードを生成してみましょう。 ユース ケースは、アプリ内で番号付けされ、任意の順序で実行できます。

> **注**: ユーザーによるモデルの呼び出し頻度が高すぎると、レート制限が発生する場合があります。 トークン レート制限に関するエラーが発生した場合は、少しの間待ってからもう一度試してください。

1. **[エクスプローラー]** ペインで、**Labfiles/04-code-generation/sample-code** フォルダーを展開し、使用する言語の関数と *go-fish* アプリを確認します。 これらのファイルは、アプリ内のタスクに使用されます。
2. 対話型ターミナル ペインで、フォルダー コンテキストが優先言語のフォルダーであることを確認します。 その後、次のコマンドを入力してアプリケーションを作成します。

    - **C#** : `dotnet run`
    - **Python**: `python code-generation.py`

    > **ヒント**: ターミナル ツールバーの **最大化パネル サイズ** (**^**) アイコンを使用すると、コンソール テキストをさらに表示できます。

3. オプション **1** を選んでコードにコメントを追加し、次のプロンプトを入力します。 これらの各タスクの応答には数秒かかる場合があることに注意してください。

    ```prompt
    Add comments to the following function. Return only the commented code.\n---\n
    ```

    結果は **result/app.txt** に保存されます。 そのファイルを開いて、**sample-code** の関数ファイルと比較します。

4. 次に、オプション **2** を選んで同じ関数の単体テストを作成し、次のプロンプトを入力します。

    ```prompt
    Write four unit tests for the following function.\n---\n
    ```

    結果は、**result/app.txt** の内容を置き換えるもので、その関数の 4 つの単体テストの詳細を示します。

5. 次に、オプション **3** を選択して、Go Fish を再生するためのアプリのバグを修正します。 次のプロンプトを入力します。

    ```prompt
    Fix the code below for an app to play Go Fish with the user. Return only the corrected code.\n---\n
    ```

    結果は、**result/app.txt** の内容を置き換えるもので、いくつかの点が修正されたものの非常によく似たコードになります。

    - **C#** : 修正は、行 30 と 59 で行われます
    - **Python**: 修正は、行 18 と 31 で行われます

    バグのある行を Azure OpenAI からの応答に置き換えると、**sample-code** 内の Go Fish アプリを実行できます。 修正しないで実行すると、ただしく動作しません。

    > **注**:この Go Fish アプリのコードは一部の構文が修正されてはいますが、ゲームを厳密には正しく表現したものではないことに注意することが重要です。 よく見ると、カードを引くときに山札が空かどうかを確認しない、プレーヤーがペアを手に入れたときにペアを手札から削除しないなどの問題があり、その他にもカード ゲームを理解する必要があるバグがいくつかあります。 これは、コード生成を支援するのに生成 AI モデルがいかに役立つかを示す好例ですが、正しいと信頼することはできず、開発者による検証が必要です。

    Azure OpenAI からの完全な応答を確認したい場合は、**printFullResponse** 変数を `True` に設定し、アプリを再実行します。

## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、**Azure portal** (`https://portal.azure.com`) でデプロイまたはリソース全体を忘れずに削除します。
