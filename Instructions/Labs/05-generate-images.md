---
lab:
  title: DALL-E モデルを使用して画像を生成する
---

# DALL-E モデルを使用して画像を生成する

Azure OpenAI Service には、DALL-E という名前の画像生成モデルが含まれています。 このモデルを使用して、目的の画像を説明する自然言語プロンプトを送信できます。モデルでは、指定した説明に基づいて元の画像が生成されます。

この演習には約 **25** 分かかります。

## 開始する前に

DALL-E を含む、Azure OpenAI Service へのアクセスが承認された Azure サブスクリプションが必要です。 以前に Azure OpenAI Service へのアクセスを求めたことがある場合は、DALL-E にアクセスするために別のアプリケーションを送信する必要がある場合があります。

- 無料の Azure サブスクリプションにサインアップするには、[https://azure.microsoft.com/free](https://azure.microsoft.com/free) にアクセスしてください。
- Azure OpenAI Service へのアクセスを要求するには、[https://aka.ms/oaiapply](https://aka.ms/oaiapply) にアクセスしてください。

## Azure OpenAI リソースをプロビジョニングする

Azure OpenAI モデルを使用する前に、Azure サブスクリプションに Azure OpenAI リソースをプロビジョニングする必要があります。

1. [Azure portal](https://portal.azure.com) にサインインします。
2. 次の設定で **Azure OpenAI** リソースを作成します。
    - **サブスクリプション**: Azure OpenAI Service のアクセスが承認されている Azure サブスクリプション。
    - **リソース グループ**: 既存のリソース グループを選択するか、任意の名前を使用して新規に作成します。
    - **リージョン**: リージョンとして **EastUS** を選択する
    - **名前**: 任意の一意の名前。
    - **価格レベル**: Standard S0
3. デプロイが完了するまで待ちます。 次に、Azure portal でデプロイされた Azure OpenAI リソースに移動します。
4. **[キーとエンドポイント]** ページに移動します。 サービスの一意のエンドポイントと認証キーは、ここから取得できます。これらは後で必要になります。

## DALL-E プレイグラウンドで画像生成について調べる

**Azure OpenAI Studio** の DALL-E プレイグラウンドを使用して、画像生成を試すことができます。

1. Azure portal の Azure OpenAI リソースの **[概要]** ページで、 **[探索]** ボタンを使用して、新しいブラウザー タブで Azure OpenAI Studio を開きます。または、[Azure OpenAI Studio](https://oai.azure.com/?azure-portal=true) に直接移動します。
2. **[DALL-E プレイグラウンド]** を選択します。
3. **[プロンプト]** ボックスに、生成する画像の説明を入力します。 たとえば、''*スケートボード上の象*'' です。 次に、 **[生成]** を選択し、生成された画像を表示します。

    ![生成された画像を含む Azure OpenAI Studio の DALL-E プレイグラウンド。](../media/dall-e-playground.png)

4. より具体的な説明を入力するようにプロンプトを変更します。 たとえば、''*ピカソ風のスケートボード上の象*'' です。 次に、新しい画像を生成し、結果を確認します。

    ![2 つの生成された画像を含む Azure OpenAI Studio の DALL-E プレイグラウンド。](../media/dall-e-playground-new-image.png)

## REST API を使用して画像を生成する

Azure OpenAI Service には、DALL-E モデルによって生成された画像など、コンテンツ生成のプロンプトを送信するために使用できる REST API が用意されています。

### アプリ環境を準備する

この演習では、シンプルな Python または Microsoft C# アプリを使用し、REST API を呼び出して画像を生成します。 Azure portal の Cloud Shell コンソール インターフェイスでコードを実行します。

1. [Azure portal](https://portal.azure.com?azure-portal=true) で、ページ上部の検索ボックスの右側にある **[>_]** (*Cloud Shell*) ボタンを選びます。 ポータルの下部に Cloud Shell ペインが開きます。 

    ![上部の検索ボックスの右側にあるアイコンをクリックして Cloud Shell を開始している状態のスクリーンショット。](../media/cloudshell-launch-portal.png#lightbox)

2. Cloud Shell を初めて開くと、使用するシェルの種類 (*Bash* または *PowerShell*) を選択するように求められる場合があります。 **[Bash]** を選択します。 このオプションが表示されない場合は、この手順をスキップします。  

3. Cloud Shell 用のストレージを作成するように求められたら、 **[詳細設定の表示]** を選び、次の設定を選びます。
    - **[サブスクリプション]**: 自分のサブスクリプション
    - **Cloud Shell リージョン**: 使用できるリージョンを選びます
    - **Show VNET isolation settings (VNET 分離の設定を表示する)**: オフ
    - **リソース グループ**: Azure OpenAI リソースをプロビジョニングした既存のリソース グループを使います
    - **ストレージ アカウント**: 一意の名前で新しいストレージ アカウントを作成します
    - **ファイル共有**: 一意の名前で新しいファイル共有を作成します

    その後、ストレージが作成されるのを 1 分程度待ちます。

    > **注**: Azure サブスクリプションに既に Cloud Shell を設定している場合は、⚙️ メニューの **[ユーザー設定のリセット]** オプションを使用して、最新バージョンの Python と .NET Framework がインストールされていることを確かめる必要がある場合があります。

4. Cloud Shell ペインの左上に表示されるシェルの種類が *Bash* であることを確認します。 *PowerShell* の場合は、ドロップダウン メニューを使用して *Bash* に切り替えます。

5. ターミナルが起動したら、次のコマンドを入力して、使用するアプリケーション コードをダウンロードします。

    ```bash
    rm -r azure-openai -f
    git clone https://github.com/MicrosoftLearning/mslearn-openai azure-openai
    ```

    ファイルは、**azure-openai** という名前のフォルダーにダウンロードされます。 C# と Python の両方のアプリケーションが提供されています。 どちらのアプリにも同じ機能があります。

6. 適切なコマンドを実行して、使用する言語のフォルダーに移動します。

    **Python**

    ```bash
    cd azure-openai/Labfiles/05-image-generation/Python
    ```

    **C#**

    ```bash
    cd azure-openai/Labfiles/05-image-generation/CSharp
    ```

7. 次のコマンドを使用して、組み込みのコード エディターを開き、使用するコード ファイルを確認します。

    ```bash
    code .
    ```

    > **ヒント**: Azure Cloud Shell コード エディターを使用して Azure Cloud Shell 環境でファイルを操作する方法の詳細については、[Azure Cloud Shell コード エディターのドキュメント](https://learn.microsoft.com/azure/cloud-shell/using-cloud-shell-editor)を参照してください。

### アプリケーションの作成

アプリケーションでは、構成ファイルを使用して、Azure OpenAI Service アカウントへの接続に必要な詳細が格納されます。

1. コード エディターで、言語の設定に応じて、アプリの構成ファイルを選択します。

    - C#: `appsettings.json`
    - Python: `.env`
    
2. Azure OpenAI Service の **エンドポイント**と **Key1** を含むように構成値を更新してから、ファイルを保存します。

    > **ヒント**: Cloud Shell ペインの上部にある分割を調整して Azure portal を確認し、Azure OpenAI Service の **[キーとエンドポイント]** ページからエンドポイントとキーの値を取得できます。

3. **Python** を使用している場合は、構成ファイルの読み取りに使用される **python-dotenv** パッケージもインストールする必要があります。 コンソール プロンプト ペインで、現在のフォルダーが **~/azure-openai/Labfiles/05-image-generation/Python** であることを確かめます。 その後、このコマンドを入力します。

    ```bash
    pip install python-dotenv
    ```

### アプリケーション コードを表示する

これで、REST API を呼び出して画像を生成するために使用されるコードを調べる準備ができました。

1. コード エディター ペインで、アプリケーションのメイン コード ファイルを選択します。

    - C#: `Program.cs`
    - Python: `generate-image.py`

2. ファイルに含まれるコードを確認し、次の主な機能に注目します。
    - このコードでは、ヘッダー内のサービスのキーを含め、サービスのエンドポイントに対して https 要求を行います。 これらの値はどちらも構成ファイルから取得されます。
    - このプロセスは <u>2 つ</u>の REST 要求で構成されます。1 つは画像生成要求を開始し、もう 1 つは結果を取得するためのものです。
    最初の要求には、次のデータが含まれます。
        - 生成する画像を説明するユーザー指定のプロンプト
        - 生成される画像の数 (この場合は 1)
        - 生成される画像の解像度 (サイズ)。
    - 最初の要求からの応答ヘッダーには、後続のコールバックで結果を取得するために使用される **operation-location** の値が含まれています。
    - このコードでは、画像生成タスクの状態が ''*成功*'' となるまでコールバック URL をポーリングし、生成された画像の URL を抽出して表示します。

### アプリを実行する

コードを確認したので、次はそれを実行し、いくつかの画像を生成します。

1. コンソール プロンプト ペインで、アプリケーションを実行するための適切なコマンドを入力します。

    **Python**

    ```bash
    python generate-image.py
    ```

    **C#**

    ```bash
    dotnet run
    ```

2. メッセージが表示されたら、画像の説明を入力します。 たとえば、''*たこ揚げをするキリン*'' です。

3. 画像が生成されるまで待ちます。コンソール ペインにハイパーリンクが表示されます。 次に、ハイパーリンクを選択して新しいブラウザー タブを開き、生成された画像を確認します。

4. 生成された画像を含むタブを閉じ、アプリを再実行して、別のプロンプトで新しい画像を生成します。

## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、[Azure portal](https://portal.azure.com/?azure-portal=true) 内のリソースを必ず削除してください。
