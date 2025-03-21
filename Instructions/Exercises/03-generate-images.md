---
lab:
  title: AI を使用して画像を生成する
  description: DALL-E OpenAI モデルを使用して画像を生成する方法を学習します。
  status: new
---

# AI を使用して画像を生成する

この演習では、OpenAI DALL-E 生成 AI モデルを使用して画像を生成します。 Azure AI Foundry と Azure OpenAI Service を使用してアプリを開発します。

この演習は約 **30** 分かかります。

## Azure AI Foundry プロジェクトを作成する

まず、Azure AI Foundry プロジェクトを作成します。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります。

    ![Azure AI Foundry ポータルのスクリーンショット。](../media/ai-foundry-home.png)

1. ホーム ページで、**[+ 作成]** を選択します。
1. **プロジェクトの作成** ウィザードで、適切なプロジェクト名 (たとえば、`my-ai-project`) を入力してから、プロジェクトをサポートするために自動的に作成される Azure リソースを確認します。
1. **[カスタマイズ]** を選択し、ハブに次の設定を指定します。
    - **[ハブ名]**: *一意の名前 - たとえば `my-ai-hub`*
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **[リソース グループ]**: *一意の名前 (たとえば、`my-ai-resources`) で新しいリソース グループを作成するか、既存のものを選びます*
    - **場所**: **[選択に関するヘルプ]** を選択してから、[場所ヘルパー] ウィンドウで **DALL-E** を選択し、推奨されるリージョンを選択します\*
    - **Azure AI サービスまたは Azure OpenAI の接続**: *適切な名前 (たとえば、`my-ai-services`) を使用して新しい AI サービス リソースを作成するか、既存のものを使用します*
    - **Azure AI 検索への接続**:接続をスキップする

    > \* Azure OpenAI リソースは、リージョンのクォータによってテナント レベルで制限されます。 演習の後半でクォータ制限に達した場合は、別のリージョンに別のリソースを作成する必要が生じる可能性があります。

1. **[次へ]** を選択し、構成を確認します。 **[作成]** を選択し、プロセスが完了するまで待ちます。
1. プロジェクトが作成されたら、表示されているヒントをすべて閉じて、Azure AI Foundry ポータルのプロジェクト ページを確認します。これは次の画像のようになっているはずです。

    ![Azure AI Foundry ポータルの Azure AI プロジェクトの詳細のスクリーンショット。](../media/ai-foundry-project.png)

## DALL-E モデルをデプロイする

これで、画像生成をサポートする DALL-E モデルをデプロイする準備ができました。

1. Azure AI Foundry プロジェクト ページの右上にあるツール バーで、**プレビュー機能**アイコンを使用して、**[Azure AI モデル推論サービスにモデルをデプロイする]** 機能を有効にします。
1. プロジェクトの左側のウィンドウの **[マイ アセット]** セクションで、**[モデル + エンドポイント]** ページを選択します。
1. **[モデル + エンドポイント]** ページの **[モデル デプロイ]** タブの **[+ モデルのデプロイ]** メニューで、**[基本モデルのデプロイ]** を選択します。
1. 一覧で **DALL-E-3** モデルを検索してから、それを選択して確認します。
1. メッセージに応じて使用許諾契約書に同意したあと、デプロイの詳細で **[カスタマイズ]** を選択して、以下の設定でモデルをデプロイします。
    - **デプロイ名**: *モデル デプロイの一意の名前 - たとえば、`dall-e-3` (割り当てた名前を覚えておいてください。後で必要になります*)
    - **デプロイの種類**:Standard
    - **デプロイの詳細**: *既定の設定を使用します*
1. デプロイのプロビジョニングの状態が**完了**になるまで待ちます。

## プレイグラウンドでモデルをテストする

クライアント アプリケーションを作成する前に、プレイグラウンドで DALL-E モデルをテストしましょう。

1. デプロイした DALL-E モデルのページで、**[プレイグラウンドで開く]** を選択します (または **[プレイグラウンド]** ページで、**[画像プレイグラウンド]** を開きます)。
1. DALL-E モデルのデプロイが選択されていることを確認します。 それから、**プロンプト** ボックスに`Create an image of an robot eating spaghetti`などのプロンプトを入力します。
1. プレイグラウンドで結果の画像を確認します。

    ![生成された画像を含む画像プレイグラウンドのスクリーンショット。](../media/images-playground.png)

1. `Show the robot in a restaurant`などのフォローアップ プロンプトを入力し、結果の画像を確認します。
1. 新しいプロンプトでテストを続けて、満足のいく結果が得られるまで画像の条件を絞ります。

## クライアント アプリケーションを作成する

モデルはプレイグラウンドで動作するようです。 これで、Azure OpenAI SDK を使用してクライアント アプリケーションで使用できるようになりました。

> **ヒント**: Python または Microsoft C# を使用してソリューションを開発することを選択できます。 選択した言語の適切なセクションの指示に従います。

### アプリケーション構成を準備する

1. Azure AI Foundry ポータルで、プロジェクトの **[概要]** ページを表示します。
1. **[プロジェクトの詳細]** エリアで、**[プロジェクト接続文字列]** の内容を書き留めます。 この接続文字列を使用して、クライアント アプリケーションでプロジェクトに接続します。
1. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて Azure 資格情報を使用してサインインします。
1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成します。***PowerShell*** 環境を選択します。 Azure portal の下部にあるペインに、Cloud Shell のコマンド ライン インターフェイスが表示されます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    > **ヒント**: Cloudshell にコマンドを貼り付けると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. PowerShell ペインで、次のコマンドを入力して、この演習用の GitHub リポジトリを複製します。

    ```
    rm -r mslearn-openai -f
    git clone https://github.com/microsoftlearning/mslearn-openai mslearn-openai
    ```

> **注**: 選択したプログラミング言語の手順に従います。

1. リポジトリが複製されたら、アプリケーション コード ファイルを含んだフォルダーに移動します。  

    **Python**

    ```
   cd mslearn-openai/Labfiles/03-image-generation/Python
    ```

    **C#**

    ```
   cd mslearn-openai/Labfiles/03-image-generation/CSharp
    ```

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、これから使用するライブラリをインストールします。

    **Python**

    ```
   pip install python-dotenv azure-identity azure-ai-projects openai requests
    ```

    *pip のバージョンとローカル パスに関するエラーは無視できます*

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Azure.AI.OpenAI
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、**your_project_endpoint** プレースホルダーをプロジェクトの接続文字列 (Azure AI Foundry ポータルでプロジェクトの **[概要]** ページからコピーしたもの) に置き換え、**your_model_deployment** プレースホルダーを DALL-E-3 モデル デプロイに割り当てた名前に置き換えます。
1. プレースホルダーを置き換えたら、**Ctrl + S** キー コマンドを使用して変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### プロジェクトに接続してモデルとチャットするためのコードを記述する

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。

1. 次のコマンドを入力して、提供されているコード ファイルを編集します。

    **Python**

    ```
   code dalle-client.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. コード ファイルで、ファイルの先頭に追加された既存のステートメントを書き留めて、必要な SDK 名前空間をインポートします。 次に、コメント**参照の追加**の下に、次のコードを追加して、前にインストールしたライブラリの名前空間を参照します。

    **Python**

    ```
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
   import requests
    ```

    **C#**

    ```
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.OpenAI;
   using OpenAI.Images;
    ```

1. **main** 関数のコメント**構成設定の取得**で、構成ファイルで定義したプロジェクト接続文字列とモデル デプロイ名の値がコードで読み込まれることに注意してください。
1. コメント**プロジェクト クライアントの初期化**で、次のコードを追加して、現在のサインインに使用した Azure 資格情報で Azure AI Foundry プロジェクトに接続します。

    **Python**

    ```
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. コメント**OpenAI クライアントの取得**で、次のコードを追加して、モデルとチャットするためのクライアント オブジェクトを作成します。

    **Python**

    ```
   openai_client = project_client.inference.get_azure_openai_client(api_version="2024-06-01")

    ```

    **C#**

    ```
   ConnectionResponse connection = projectClient.GetConnectionsClient().GetDefaultConnection(ConnectionType.AzureOpenAI, withCredential: true);

   var connectionProperties = connection.Properties as ConnectionPropertiesApiKeyAuth;

   AzureOpenAIClient openAIClient = new(
        new Uri(connectionProperties.Target),
        new AzureKeyCredential(connectionProperties.Credentials.Key));

   ImageClient openAIimageClient = openAIClient.GetImageClient(model_deployment);

    ```

1. コードには、ユーザーが「quit」と入力するまでプロンプトを入力できるようにするループが含まれていることに注意してください。 次に、ループ セクションのコメント**画像の生成**で、次のコードを追加して、プロンプトを送信し、モデルから生成された画像の URL を取得します。

    **Python**

    ```python
   result = openai_client.images.generate(
        model=model_deployment,
        prompt=input_text,
        n=1
    )

    json_response = json.loads(result.model_dump_json())
    image_url = json_response["data"][0]["url"] 
    ```

    **C#**

    ```
   var imageGeneration = await openAIimageClient.GenerateImageAsync(
            input_text,
            new ImageGenerationOptions()
            {
                Size = GeneratedImageSize.W1024xH1024
            }
   );
   imageUrl= imageGeneration.Value.ImageUri;
    ```

1. **main** 関数の残りの部分のコードは、画像の URL とファイル名を指定された関数に渡します。これにより、生成された画像がダウンロードされ、.png ファイルとして保存されます。

1. **Ctrl + S** キー コマンドを使用してコード ファイルに変更を保存してから、**Ctrl + Q** キー コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### クライアント アプリケーションを実行する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力してアプリを実行します。

    **Python**

    ```
   python dalle-client.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. メッセージが表示されたら、`Create an image of a robot eating pizza`などの画像のリクエストを入力します。 しばらくすると、アプリは画像が保存されたことを確認する必要があります。
1. いくつかのプロンプトを試してみましょう。 終了したら、`quit` を入力してプログラムを終了します。

    > **注**: このシンプルなアプリには、会話履歴を保持するためのロジックが含まれていないので、モデルは、各プロンプトを前のプロンプトのコンテキストを持たない新しいリクエストとして処理します。

1. アプリによって生成された画像をダウンロードして表示するには、Cloud Shell 画面のツール バーで、**[ファイルのアップロード/ダウンロード]** ボタンを使用して、ファイルをダウンロードしてから開きます。 ファイルをダウンロードするには、ダウンロード インターフェイスでそのファイル パスを完了します。例えば:

    **Python**

    /home/*user*`/mslearn-openai/Labfiles/03-image-generation/Python/images/image_1.png`

    **C#**

    /home/*user*`/mslearn-openai/Labfiles/03-image-generation/CSharp/images/image_1.png`

## まとめ

この演習では、Azure AI Foundry と Azure OpenAI SDK を使用し、DALL-E モデルを使用して画像を生成するクライアント アプリケーションを作成しました。

## クリーンアップ

DALL-E の探索が終わったら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. Azure portal が表示されているブラウザー タブに戻り (または、新しいブラウザー タブで `https://portal.azure.com` の [Azure portal](https://portal.azure.com) をもう一度開き)、この演習で使用したリソースがデプロイされているリソース グループの内容を表示します。
1. ツール バーの **[リソース グループの削除]** を選びます。
1. リソース グループ名を入力し、削除することを確認します。
