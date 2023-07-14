---
lab:
  title: Azure OpenAI をアプリに統合する
---

# Azure OpenAI をアプリに統合する

Azure OpenAI Service を使用すると、開発者はチャットボットや言語モデルをはじめとして、人間の自然な言語を理解することに優れたその他のアプリケーションを作成できます。 Azure OpenAI では、事前トレーニング済みの AI モデルにアクセスできるだけでなく、アプリケーションの特定の要件を満たすためにこれらのモデルをカスタマイズおよび微調整するための一連の API やツールが提供されます。 この演習では、モデルを Azure OpenAI にデプロイし、独自のアプリケーションでそれを使用してテキストを要約する方法について学習します。

この演習には約 **30** 分かかります。

## 開始する前に

Azure OpenAI Service へのアクセスが承認されている Azure サブスクリプションが必要になります。

- 無料の Azure サブスクリプションにサインアップするには、[https://azure.microsoft.com/free](https://azure.microsoft.com/free) にアクセスしてください。
- Azure OpenAI Service へのアクセスを要求するには、[https://aka.ms/oaiapply](https://aka.ms/oaiapply) にアクセスしてください。

## Azure OpenAI リソースをプロビジョニングする

Azure OpenAI モデルを使用する前に、Azure サブスクリプションに Azure OpenAI リソースをプロビジョニングする必要があります。

1. [Azure portal](https://portal.azure.com) にサインインします。
2. 次の設定で **Azure OpenAI** リソースを作成します。
    - **サブスクリプション**: Azure OpenAI Service のアクセスが承認されている Azure サブスクリプション。
    - **リソース グループ**: ご自分で選択した名前を持つ新しいリソース グループを作成します。
    - **リージョン**: 使用できるリージョンを選択します。
    - **名前**: 任意の一意の名前。
    - **価格レベル**: Standard S0
3. デプロイが完了するまで待ちます。 次に、Azure portal で、デプロイされた Azure OpenAI リソースに移動します。
4. **[キーとエンドポイント]** ページに移動し、後で使用するためにテキスト ファイルに保存します。

## モデルをデプロイする

Azure OpenAI API を使用するには、まず、**Azure OpenAI Studio** を介して使用するモデルをデプロイする必要があります。 デプロイが完了したら、アプリでそのモデルを参照します。

1. Azure OpenAI リソースの **[概要]** ページで、 **[探索]** ボタンを使用して、新しいブラウザー タブで Azure OpenAI Studio を開きます。
2. Azure OpenAI Studio で、次の設定で新しいデプロイを作成します。
    - **モデル**: gpt-35-turbo
    - **モデル バージョン**: "既定のバージョンを使用する"**
    - **デプロイ名**: text-turbo

> **注**: 各 Azure OpenAI モデルは、機能とパフォーマンスの異なるバランスに合わせて最適化されています。 この演習では、**GPT-3** モデル ファミリの **3.5 Turbo** モデル シリーズを使用します。これは、言語理解に非常に適しています。 この演習では 1 つのモデルのみを使いますが、デプロイする他のモデルのデプロイと使用の場合も同じように動作します。

## Cloud Shell でアプリケーションを設定する

Azure OpenAI モデルと統合する方法を示すために、Azure 上の Cloud Shell で実行される短いコマンドライン アプリケーションを使用します。 Cloud Shell を操作するには、新しいブラウザー タブを開きます。

1. [Azure portal](https://portal.azure.com?azure-portal=true) で、ページ上部の検索ボックスの右側にある **[>_]** (*Cloud Shell*) ボタンを選択します。 ポータルの下部に Cloud Shell ペインが開きます。

    ![上部の検索ボックスの右側にあるアイコンをクリックして Cloud Shell を開始している状態のスクリーンショット。](../media/cloudshell-launch-portal.png#lightbox)

2. Cloud Shell を初めて開くと、使用するシェルの種類 (*Bash* または *PowerShell*) を選択するように求められる場合があります。 **[Bash]** を選択します。 このオプションが表示されない場合は、この手順をスキップします。  

3. Cloud Shell のストレージを作成するように求めるメッセージが表示された場合は、お使いのサブスクリプションが指定されていることを確認して、**[ストレージの作成]** を選択します。 その後、ストレージが作成されるのを 1 分程度待ちます。

4. Cloud Shell ペインの左上に表示されるシェルの種類が *Bash* に切り替えられたことを確認します。 *PowerShell* の場合は、ドロップダウン メニューを使用して *Bash* に切り替えます。

5. ターミナルが起動したら、次のコマンドを入力してサンプル アプリケーションをダウンロードし、`azure-openai` という名前のフォルダーに保存します。

    ```bash
   rm -r azure-openai -f
   git clone https://github.com/MicrosoftLearning/mslearn-openai azure-openai
    ```
  
6. ファイルは、**azure-openai** という名前のフォルダーにダウンロードされます。 次のコマンドを使用して、この演習のラボ ファイルに移動します。

    ```bash
   cd azure-openai/Labfiles/02-nlp-azure-openai
    ```

C# と Python の両方のアプリケーションと、要約のテストに使用するサンプル テキスト ファイルが提供されています。 どちらのアプリにも同じ機能があります。

組み込みのコード エディターを開き、`text-files/sample-text.txt` にあるモデルで要約するテキスト ファイルを確認します。 次のコマンドを使用して、コード エディターでラボ ファイルを開きます。

```bash
code .
```

## アプリケーションの作成

この演習では、Azure OpenAI リソースの使用を有効にするために、アプリケーションのいくつかの重要な部分を完成します。

1. コード エディターで、言語の設定に応じて **CSharp** または **Python** フォルダーを展開します。

2. 言語の構成ファイルを開く

    - C#: `appsettings.json`
    - Python: `.env`
    
3. 構成値を更新して、作成した Azure OpenAI リソースの**エンドポイント**と**キー**と、デプロイしたモデル名 (`text-turbo`) を含めるようにします。 ファイルを保存します。

4. 優先する言語のフォルダーに移動し、必要なパッケージをインストールします

    **C#**

    ```bash
   cd CSharp
   dotnet add package Azure.AI.OpenAI --prerelease
    ```

    **Python**

    ```bash
   cd Python
   pip install python-dotenv
   pip install openai
    ```

5. 優先する言語フォルダーに移動し、コード ファイルを選択して、必要なライブラリを追加します。

    **C#**

    ```csharp
   // Add Azure OpenAI package
   using Azure.AI.OpenAI;
    ```

    **Python**

    ```python
   # Add OpenAI import
   import openai
    ```

5. 言語のアプリケーション コードを開き、要求をビルドするために必要なコードを追加します。これにより、`prompt` や `temperature` などのモデルのさまざまなパラメーターが指定されます。

    **C#**

    ```csharp
   // Initialize the Azure OpenAI client
   OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));

   // Build completion options object
   ChatCompletionsOptions chatCompletionsOptions = new ChatCompletionsOptions()
   {
       Messages =
       {
          new ChatMessage(ChatRole.System, "You are a helpful assistant. Summarize the following text in 60 words or less."),
          new ChatMessage(ChatRole.User, text),
       },
       MaxTokens = 120,
       Temperature = 0.7f,
   };

   // Send request to Azure OpenAI model
   ChatCompletions response = client.GetChatCompletions(
       deploymentOrModelName: oaiModelName, 
       chatCompletionsOptions);
   string completion = response.Choices[0].Message.Content;

   Console.WriteLine("Summary: " + completion + "\n");
    ```

    **Python**

    ```python
   # Set OpenAI configuration settings
   openai.api_type = "azure"
   openai.api_base = azure_oai_endpoint
   openai.api_version = "2023-03-15-preview"
   openai.api_key = azure_oai_key

   # Send request to Azure OpenAI model
   print("Sending request for summary to Azure OpenAI endpoint...\n\n")
   response = openai.ChatCompletion.create(
       engine=azure_oai_model,
       temperature=0.7,
       max_tokens=120,
       messages=[
          {"role": "system", "content": "You are a helpful assistant. Summarize the following text in 60 words or less."},
           {"role": "user", "content": text}
       ]
   )

   print("Summary: " + response.choices[0].message.content + "\n")
    ```

## アプリケーションを実行する

アプリが構成されたので、それを実行してモデルに要求を送信し、応答を確認します。

1. Cloud Shell bash ターミナルで、優先する言語のフォルダーに移動します。
1. アプリケーションを実行します。

    - **C#** : `dotnet run`
    - **Python**: `python test-openai-model.py`

1. サンプル テキスト ファイルの要約を確認します。
1. 優先する言語のコード ファイルに移動し、*temperature* の値を `1` に変更します。 ファイルを保存します。
1. アプリケーションをもう一度実行し、出力を確認します。

多くの場合、温度を上げると、ランダム性が高くなるため、同じテキストを指定した場合でも概要が変化します。 これを数回実行して、出力がどのように変化するかを確認できます。 同じ入力で温度に異なる値を使用してみてください。

## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、[Azure portal](https://portal.azure.com?azure-portal=true) 内のデプロイまたはリソース全体を必ず削除してください。
