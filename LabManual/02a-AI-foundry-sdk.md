---
lab:
  title: 生成 AI チャット アプリを作成する
  description: Azure AI Foundry SDK を使用して、プロジェクトに接続して言語モデルとチャットするアプリを構築する方法について説明します。
---

# 生成 AI チャット アプリを作成する

この演習では、Azure AI Foundry Python SDK を使用して、プロジェクトに接続して言語モデルとチャットするシンプルなチャット アプリを作成します。

> **注**:この演習は、変更される可能性があるプレリリース SDK ソフトウェアに基づいています。 必要に応じて、特定のバージョンのパッケージを使用しました。利用可能な最新バージョンが反映されていない可能性があります。 予期しない動作、警告、またはエラーが発生する場合があります。

この演習は、Azure AI Foundry Python SDK に基づいていますが、次のような複数の言語固有の SDK を使用して AI チャット アプリケーションを開発することができます。

- [Python 向け Azure AI プロジェクト](https://pypi.org/project/azure-ai-projects)
- [Microsoft .NET 向け Azure AI プロジェクト](https://www.nuget.org/packages/Azure.AI.Projects)
- [JavaScript 向け Azure AI プロジェクト](https://www.npmjs.com/package/@azure/ai-projects)

この演習は約 **40** 分かかります。

## モデルとチャットするクライアント アプリケーションを作成する

これでモデルをデプロイしたので、Azure AI Foundry SDK と Azure OpenAI SDK を使用して、モデルとチャットするアプリケーションを開発できます。

### アプリケーション構成を準備する

1. Azure AI Foundry ポータルで、プロジェクトの **[概要]** ページを表示します。

1. **[エンドポイントとキー]** の領域で、**[Azure AI Foundry]** ライブラリが選択されていることを確認し、**[Azure AI Foundry プロジェクト エンドポイント]** を表示します。 クライアント アプリケーションで、このエンドポイントを使用してプロジェクトとモデルに接続します。

    > **注**:Azure OpenAI エンドポイントを使用することもできます。

1. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて 資格情報を使用してサインインします。

    ツアーに関する通知は閉じて、Azure portal のホーム ページを表示します。

1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成します。以下の内容を選択して作成します。

    - シェルの選択: **PowerShell**
    - ストレージアカウントの有無: **ストレージアカウントは不要です**
    - サブスクリプション: ドロップダウンリストで選択可能なサブスクリプション

    Azure portal の下部にあるペインに Cloud Shell のコマンド ライン インターフェイスが表示されます。 作業しやすくするために、このウィンドウのサイズを変更したり最大化したりすることができます。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    **<font color="red">続行する前に、クラシック バージョンの Cloud Shell に切り替えたことを確認します。</font>**

1. Cloud Shell 画面で、次のコマンドを入力して、この演習のコード ファイルを含む GitHub リポジトリを複製します (コマンドを入力するか、クリップボードにコピーしてから、コマンド ラインで右クリックし、プレーンテキストとして貼り付けます)。

   ```
   rm -r mslearn-ai-foundry -f
   git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
   ```

    > **ヒント**: Cloudshell にコマンドを入力すると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. リポジトリが複製されたら、チャット アプリケーションのコード ファイルを含んだフォルダーに移動して、それらを表示します。

   ```
   cd mslearn-ai-foundry/labfiles/chat-app/python
   ls -a -l
   ```

    フォルダーには、コード ファイルだけでなく、アプリケーション設定の設定ファイルや、プロジェクトのランタイムおよびパッケージの要件を定義するファイルが含まれています。

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

   ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
   ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

   ```
   code .env
   ```

    このファイルをコード エディターで開きます。

    > <font color="red"><b>重要</b>:</font>プロジェクトの既定のリージョンに gpt-4o モデルをデプロイした場合は、プロジェクトの <b>[概要]</b> ページで <b>[Azure AI Foundry プロジェクト]</b> または <b>[Azure OpenAI]</b> エンドポイントを使って、モデルに接続できます。 クォータが不十分で、モデルを別のリージョンにデプロイした場合は、<b>[モデル + エンドポイント]</b> ページでモデルを選び、モデルの <b>[ターゲット URI]</b> を使います。

1. コード ファイルで、**your_project_endpoint** プレースホルダーを以前のタスクで確認したエンドポイントに置き換え、**your_model_deployment** プレースホルダーを **gpt-4o** モデルに設定した名称に置き換えます。

1. プレースホルダーを置き換えたら、コード エディター内で、**Ctrl + S** コマンドを使用するか、**右クリックして保存**で変更を保存してから、**Ctrl + Q** コマンドを使用するか、**右クリックして終了**で、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

### プロジェクトに接続してモデルとチャットするためのコードを記述する

> **ヒント**: コードを追加する際は、必ず正しいインデントを維持してください。

1. 次のコマンドを入力して、提供されているコード ファイルを編集します。

    ```
   code chat-app.py
   ```

1. コード ファイルで、ファイルの先頭に追加された既存のステートメントを書き留めて、必要な SDK 名前空間をインポートします。 次に、コメント**Add references**を探して、次のコードを追加し、前にインストールしたライブラリの名前空間を参照します。

    ```python
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
   ```
   
1. **main** 関数のコメント**Get configuration settings**で、構成ファイルで定義したプロジェクト接続文字列("PROJECT_ENDPOINT")とモデル デプロイ名("MODEL_DEPLOYMENT")の値がコードで読み込まれることに注意してください。
1. コメント **Initialize the project client** を見つけて、次のコードを追加し、Azure AI Foundry プロジェクトに接続します。

    > **ヒント**: コードのインデント レベルを正しく維持するように注意してください。

    ```python
   project_client = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_endpoint,
        )
   ```
   
1. コメント**Get a chat client**を探して、次のコードを追加し、モデルとチャットするためのクライアント オブジェクトを作成します。

    ```python
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
   ```
   
1. コメント**Initialize prompt with system message**を探して、次のコードを追加し、システム プロンプトを使用してメッセージのコレクションを初期化します。

    ```python
   prompt = [
            {"role": "system", "content": "You are a helpful AI assistant that answers questions."}
   ]
   ```
   
1. コードには、ユーザーが「quit」と入力するまでプロンプトを入力できるようにするループが含まれています。 ループ セクションで、コメント **Get a chat completion**を探して、次のコードを追加し、ユーザー入力をプロンプトに追加し、モデルから完了を取得し、プロンプトに完了を追加します (今後の反復のためにチャット履歴を保持します)。

    ```python
   # Get a chat completion
   prompt.append({"role": "user", "content": input_text})
   response = openai_client.chat.completions.create(
            model=model_deployment,
            messages=prompt)
   completion = response.choices[0].message.content
   print(completion)
   prompt.append({"role": "assistant", "content": completion})
   ```

1. **CTRL + S** コマンドを使用して、変更をコード ファイルに保存します。

### Azure にサインインしてアプリを実行する

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して Azure にサインインします。

    ```
   az login
   ```

    **<font color="red">Cloud Shell セッションが既に認証されている場合でも、Azure にサインインする必要があります。</font>**

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。
   
1. メッセージが表示されたら、指示に従って新しいタブでサインイン ページを開き、指定された認証コードと Azure 資格情報を入力します。 次に、コマンド ラインでサインイン プロセスを完了し、プロンプトが表示されたら、Azure AI Foundry ハブを含むサブスクリプションを選択します。
1. サインインしたら、次のコマンドを入力してアプリケーションを実行します。

    ```
   python chat-app.py
   ```

1. メッセージが表示されたら、`What is the fastest animal on Earth?` などの質問を入力し、生成 AI モデルからの応答を確認します。
1. `Where can I see one?`や`Are they endangered?`など、フォローアップの質問をお試しください。 各反復の背景情報であるチャット履歴を使用して、会話を続行する必要があります。
1. 終了したら、`quit` を入力してプログラムを終了します。

> **ヒント**: レート制限を超えたためにアプリが使用不能になる場合。 数秒待ってから、やり直してください。 サブスクリプションで使用可能なクォータが不足している場合は、モデルが応答できない可能性があります。

## まとめ

この演習では、Azure AI Foundry SDK を使用して、Azure AI Foundry プロジェクトにデプロイした生成 AI モデル用のクライアント アプリケーションを作成しました。
