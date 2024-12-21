---
lab:
  title: DALL-E モデルを使用して画像を生成する
---

# DALL-E モデルを使用して画像を生成する

Azure OpenAI Service には、DALL-E という名前の画像生成モデルが含まれています。 このモデルを使用して、目的の画像を説明する自然言語プロンプトを送信できます。モデルでは、指定した説明に基づいて元の画像が生成されます。

この演習では、DALL-E バージョン 3 モデルを使って、自然言語プロンプトに基づいて画像を生成します。

この演習には、約 **25** 分かかります。

## Azure OpenAI リソースをプロビジョニングする

Azure OpenAI を使って画像を生成する前に、Azure サブスクリプションで Azure OpenAI リソースをプロビジョニングする必要があります。 リソースは、DALL-E モデルがサポートされているリージョンに存在する必要があります。

1. **Azure portal** (`https://portal.azure.com`) にサインインします。
1. 次の設定で **Azure OpenAI** リソースを作成します。
    - **[サブスクリプション]**: "DALL-E を含む Azure OpenAI Service へのアクセスが承認されている Azure サブスクリプションを選びます"**
    - **[リソース グループ]**: *リソース グループを作成または選択します*
    - **リージョン**: ***米国東部**、**オーストラリア東部**または**スウェーデン中部***\*のいずれかを選びます
    - **[名前]**: "*希望する一意の名前*"
    - **価格レベル**: Standard S0

    > \* DALL-E 3 モデルは、**米国東部**、**オーストラリア東部**および**スウェーデン中部**リージョンの Azure OpenAI Service リソースでのみ使用できます。

1. デプロイが完了するまで待ちます。 次に、Azure portal でデプロイされた Azure OpenAI リソースに移動します。

## モデルをデプロイする

次に、CLI から **dalle3** モデルのデプロイを作成します。 Azure portal で、上部のメニュー バーから **Cloud Shell** アイコンを選択し、ターミナルが **Bash** に設定されていることを確認します。 この例を参照して、次の変数を独自の値に置き換えます。

```dotnetcli
az cognitiveservices account deployment create \
   -g *your resource group* \
   -n *your Open AI resource* \
   --deployment-name dall-e-3 \
   --model-name dall-e-3 \
   --model-version 3.0  \
   --model-format OpenAI \
   --sku-name "Standard" \
   --sku-capacity 1
```

    > \* Sku-capacity is measured in thousands of tokens per minute. A rate limit of 1,000 tokens per minute is more than adequate to complete this exercise while leaving capacity for other people using the same subscription.


## REST API を使用して画像を生成する

Azure OpenAI Service には、DALL-E モデルによって生成された画像など、コンテンツ生成のプロンプトを送信するために使用できる REST API が用意されています。

### Visual Studio Code でアプリを開発する準備をする

さっそく、Azure OpenAI Service を使って画像を生成するカスタム アプリを構築する方法を見てみましょう。 Visual Studio Code を使用してアプリを開発します。 アプリのコード ファイルは、GitHub リポジトリで提供されています。

> **ヒント**: 既に **mslearn-openai** リポジトリをクローンしている場合は、Visual Studio Code で開きます。 それ以外の場合は、次の手順に従って開発環境に複製します。

1. Visual Studio Code を起動します。
2. パレットを開き (SHIFT+CTRL+P)、**Git:Clone** コマンドを実行して、`https://github.com/MicrosoftLearning/mslearn-openai` リポジトリをローカル フォルダーに複製します (どのフォルダーでも問題ありません)。
3. リポジトリを複製したら、Visual Studio Code でフォルダーを開きます。

    > **注**:Visual Studio Code に、開いているコードを信頼するかどうかを求めるポップアップ メッセージが表示された場合は、ポップアップの **[はい、作成者を信頼します]** オプションをクリックします。

4. リポジトリ内の C# コード プロジェクトをサポートするために追加のファイルがインストールされるまで待ちます。

    > **注**: ビルドとデバッグに必要なアセットを追加するように求めるプロンプトが表示された場合は、**[今はしない]** を選択します。

### アプリケーションを構成する

C# と Python の両方のアプリケーションが提供されています。 どちらのアプリにも同じ機能があります。 まず、Azure OpenAI リソースのエンドポイントとキーをアプリの構成ファイルに追加します。

1. Visual Studio Code の **[エクスプローラー]** ペインで、**Labfiles/03-image-generation** フォルダーを参照し、言語の優先順位に応じて、**CSharp** または **Python** フォルダーを展開します。 各フォルダーには、Azure OpenAI 機能を統合するアプリの言語固有のファイルが含まれています。
2. **[エクスプローラー]** ペインの **CSharp** または **Python** フォルダーで、使用する言語の構成ファイルを開きます

    - **C#**: appsettings.json
    - **Python**: .env
    
3. 作成した Azure OpenAI リソースの**エンドポイント**と**キー**を含むように構成値を更新します (Azure portal の Azure OpenAI リソースにある **[キーとエンドポイント]** ページでアクセスできます)。
4. 構成ファイルを保存します。

### アプリケーション コードを表示する

これで、REST API を呼び出して画像を生成するために使用されるコードを調べる準備ができました。

1. **[エクスプローラー]** ペインで、アプリケーションのメイン コード ファイルを選びます。

    - C#: `Program.cs`
    - Python: `generate-image.py`

2. ファイルに含まれるコードを確認し、次の主な機能に注目します。
    - このコードでは、サービスのエンドポイントに対して、ヘッダーにサービスのキーが含められた https 要求を行います。 これらの値はどちらも構成ファイルから取得されます。
    - 要求には、画像のベースとなるプロンプト、生成する画像の数、生成される画像のサイズなど、いくつかのパラメーターが含まれます。
    - 応答には、ユーザー指定のプロンプトから DALL-E モデルが推定してよりわかりやすいものに改訂されたプロンプトと、生成された画像の URL が含まれています。
    
    > **重要**: 推奨される *dalle3* 以外の名前をデプロイに付けた場合は、デプロイの名前を使用するようにコードを更新する必要があります。

### アプリを実行する

コードを確認したので、次はそれを実行し、いくつかの画像を生成します。

1. コード ファイルが含まれている **CSharp** または **Python** フォルダーを右クリックし、統合ターミナルを開きます。 次に、適切なコマンドを入力してアプリケーションを実行します。

   **C#**
   ```
   dotnet run
   ```
   
   **Python**
   ```
   pip install requests
   python generate-image.py
   ```

3. メッセージが表示されたら、画像の説明を入力します。 たとえば、''*たこ揚げをするキリン*'' です。

4. 画像が生成されるまで待ちます。ターミナル ペインにハイパーリンクが表示されます。 次に、ハイパーリンクを選択して新しいブラウザー タブを開き、生成された画像を確認します。

   > **ヒント**:アプリが応答を返さない場合は、少し待ってから再試行してください。 新しくデプロイされたリソースが使用可能になるまでに最大 5 分かかる場合があります。

5. 生成された画像を含むブラウザー タブを閉じ、アプリを再実行して、別のプロンプトで新しい画像を生成します。

## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、**Azure portal** (`https://portal.azure.com`) でリソースを忘れずに削除します。
