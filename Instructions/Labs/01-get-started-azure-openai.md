---
lab:
  title: Azure OpenAI で作業を開始する
---

# Azure OpenAI Service で作業を開始する

Azure OpenAI Service は OpenAI によって開発された生成 AI モデルを Azure プラットフォームに導入します。これにより、Azure クラウド プラットフォームによって提供されるサービスのセキュリティ、スケーラビリティ、統合の恩恵を受ける強力な AI ソリューションを開発できるようになります。 この演習では、サービスを Azure リソースとしてプロビジョニングし、Azure OpenAI Studio を使用して OpenAI モデルをデプロイおよび探索することで、Azure OpenAI の使用を開始する方法について学習します。

この演習は約 **30** 分かかります。

## 開始する前に

Azure OpenAI サービスへのアクセスが承認されている Azure サブスクリプションが必要になります。

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
3. デプロイが完了するまで待ちます。 次に、Azure portal でデプロイされた Azure OpenAI リソースに移動します。

## モデルをデプロイする

Azure OpenAI には、モデルのデプロイ、管理、探索に使用できる **Azure OpenAI Studio** という名前の Web ベースのポータルが用意されています。 Azure OpenAI Studio を使用してモデルをデプロイすることで、Azure OpenAI の探索を開始します。

1. Azure OpenAI リソースの **[概要]** ページで、 **[Azure OpenAI Studio に移動する]** ボタンを使用して、新しいブラウザー タブで Azure OpenAI Studio を開きます。
2. Azure OpenAI Studio の [**デプロイ**] ページで、既存のモデルのデプロイを表示します。 まだデプロイがない場合は、次の設定で **gpt-35-turbo-16k** モデルの新しいデプロイを作成します。
    - **モデル**: gpt-35-turbo-16k
    - **モデル バージョン**: 既定値に自動更新
    - **デプロイの名前**: *任意の一意の名前*
    - **詳細オプション**
        - **コンテンツ フィルター**: 既定
        - **1 分あたりのトークンのレート制限**: 5K\*
        - **動的クォータを有効にする**: 有効

    > \* この演習は、1 分あたり 5,000 トークンのレート制限内で余裕を持って完了できます。またこの制限によって、同じサブスクリプションを使用する他のユーザーのために容量を残すこともできます。

> **注**: 一部のリージョンでは、新しいモデル デプロイ インターフェイスに [**モデル バージョン**] オプションが表示されません。 この場合は、オプションを設定せずにそのまま続行してください。

## チャット プレイグラウンドを使用する

"チャット" プレイグラウンドには、GPT 3.5 以降のモデル用のチャットボット インターフェイスが用意されています。** 以前の *Completions* API ではなく *ChatCompletions* API が使用されます。

1. [**プレイグラウンド**] セクションで [**チャット**] ページを選択し、右側の構成ウィンドウでモデルが選択されていることを確認します。
2. **[アシスタントのセットアップ]** セクションの **[システム メッセージ]** ボックスで、現在のテキストを `The system is an AI teacher that helps people learn about AI` のステートメントに置き換えます。

3. **[システム メッセージ]** ボックスの下にある **[Add few-shot examples] (Few-shot の例を追加する)** をクリックし、指定されたボックスに次のメッセージと応答を入力します。

    - **ユーザー**: `What are different types of artificial intelligence?`
    - **アシスタント**: `There are three main types of artificial intelligence: Narrow or Weak AI (such as virtual assistants like Siri or Alexa, image recognition software, and spam filters), General or Strong AI (AI designed to be as intelligent as a human being. This type of AI does not currently exist and is purely theoretical), and Artificial Superintelligence (AI that is more intelligent than any human being and can perform tasks that are beyond human comprehension. This type of AI is also purely theoretical and has not yet been developed).`

    > **注**: Few-shot の例は、予測される応答の種類の例をモデルに提供するために使用されます。 モデルは、例のトーンとスタイルを自分の応答に反映させようとします。

4. 変更を保存して新しいセッションを開始し、チャット システムの動作コンテキストを設定します。
5. ページの下部にあるクエリ ボックスに、テキスト `What is artificial intelligence?` を入力します
6. **[送信]** ボタンを使用してメッセージを送信し、応答を表示します。

    > **注**: API デプロイの準備がまだできていないという応答を受け取る場合があります。 その場合は、数分待ってからもう一度やり直してください。

7. 応答を確認し、`How is it related to machine learning?` のメッセージを送信して会話を続けます。
8. 応答を確認し、前の対話式操作のコンテキストが保持されていることを確認します ("それ" が人工知能を指していることをモデルが認識しています)。
9. **[コードの表示]** ボタンを使用して、対話式操作のコードを表示します。 プロンプトは、"システム" メッセージ、"ユーザー" と "アシスタント" のメッセージの Few-shot の例、そしてチャット セッション内の "ユーザー" と "アシスタント" のメッセージのシーケンスで構成されています。** ** ** ** **

## プロンプトとパラメーターを探索する

プロンプトとパラメーターを使用して、必要な応答が生成される可能性を最大限に高めることができます。

1. **[パラメーター]** ペインでは、次のパラメーター値を設定します。
    - **[温度]** : 0
    - **最大応答**: 500

2. 次のメッセージを送信します

    ```
    Write three multiple choice questions based on the following text.

    Most computer vision solutions are based on machine learning models that can be applied to visual input from cameras, videos, or images.*

    - Image classification involves training a machine learning model to classify images based on their contents. For example, in a traffic monitoring solution you might use an image classification model to classify images based on the type of vehicle they contain, such as taxis, buses, cyclists, and so on.*

    - Object detection machine learning models are trained to classify individual objects within an image, and identify their location with a bounding box. For example, a traffic monitoring solution might use object detection to identify the location of different classes of vehicle.*

    - Semantic segmentation is an advanced machine learning technique in which individual pixels in the image are classified according to the object to which they belong. For example, a traffic monitoring solution might overlay traffic images with "mask" layers to highlight different vehicles using specific colors.
    ```

3. 結果を確認します。教師がコンピューター ビジョンのトピックについてプロンプトで学生をテストするために使用できる複数選択の質問で構成されている必要があります。 応答の合計は、パラメーターとして指定した最大長よりも短くする必要があります。

    使用したプロンプトとパラメーターで、次の点を確認してください。

    - プロンプトでは、目的の出力が 3 つの複数選択の質問であることが具体的に示されます。
    - パラメーターに、応答生成にどの程度の確率的要素を含むかを制御する "温度" が含まれている。** 送信で **0** の値を使用すると、ランダム性が最小限に抑えられ、結果として安定した予測可能な応答が得られます。

## コード生成について探索する

自然言語による応答を生成するだけでなく、GPT モデルを使用してコードを生成することもできます。

1. **[アシスタントのセットアップ]** ペインで、 **[空の例]** テンプレートを選択してシステム メッセージをリセットします。
2. システム メッセージ `You are a Python developer.` を入力し、変更を保存します。
3. **[チャット セッション]** ペインで、 **[チャットのクリア]** を選択してチャット履歴をクリアし、新しいセッションを開始します。
4. 次のユーザー メッセージを送信します。

    ```
    Write a Python function named Multiply that multiplies two numeric parameters.
    ```

5. 応答を確認します。プロンプトの要件を満たすサンプルの Python コードが含まれている必要があります。

## クリーンアップ

Azure OpenAI リソースでの作業が完了したら、[Azure portal](https://portal.azure.com) 内のデプロイまたはリソース全体を必ず削除してください。
