### キーワード
- ヘッドレスエージェント
- マルチエージェント「**デジタル専門家チーム**」
- パーソナライズ

---

### マルチエージェントによる協調モデル

全体の司令塔となる**Supervisor Agent**（監督エージェント）を中心に、各機能に特化した複数のエージェントを連携させる構成です。

#### 登場するエージェント
1.  **① ASA (Agentforce Service Agent)**：顧客との対話を担当するフロントエージェント。
2.  **② Supervisor Agent**：各エージェントへのタスク割り振りを行う司令塔。
3.  **③ Case Master Agent**：Salesforceのケース登録・管理を専門に行うエージェント。
4.  **④ Research Agent**：外部Webサイトからナレッジを検索・要約するエージェント。
5.  **⑤ Notifier Agent**：メール通知など、アウトバウンド通信を担当するエージェント。

---

### 具体的な処理フロー

顧客がチャットを開始してから解決通知を受け取るまでの流れを、各エージェントの役割に沿って説明します。

#### **ステップ1：チャット受付とヒアリング**
* **担当エージェント**: `① ASA (Agentforce Service Agent)`
* **処理内容**:
    * 顧客からのチャットを受け付け、会話を開始します。
    * 自然な対話を通じて、顧客の課題や要件（名前、連絡先、問題の概要など）をヒアリングします。
    * 収集した情報を構造化されたデータとして保持します。

#### **ステップ2：タスクの振り分けとケース起票**
* **担当エージェント**: `② Supervisor Agent` → `③ Case Master Agent`
* **処理内容**:
    1.  ASAがヒアリングを終えると、`② Supervisor Agent`に処理を引き渡します。
    2.  Supervisor Agentは対話内容を分析し、「これは新規ケース登録が必要なタスクだ」と判断します。
    3.  顧客情報と問題の概要を`③ Case Master Agent`に渡し、ケース作成を指示します。
    4.  Case Master Agentは受け取ったデータを元にSalesforce上でケースを作成し、ケース番号をSupervisor Agentに返却します。

#### **ステップ3：Webナレッジの検索と回答生成**
* **担当エージェント**: `② Supervisor Agent` → `④ Research Agent`
* **処理内容**:
    1.  Supervisor Agentは、ケース内容の解決には外部情報が必要だと判断した場合、`④ Research Agent`に調査を依頼します。
    2.  Research Agentは、指定されたキーワードでWebを検索し、関連性の高い情報を収集・要約します。
    3.  要約したナレッジをSupervisor Agentに返却します。
    4.  Supervisor Agentは、そのナレッジを`① ASA`に渡し、顧客への回答案として提示させます。ASAはそれを自然な文章に整えて顧客に返信します。

#### **ステップ4：ケースクローズと解決後通知**
* **担当エージェント**: `② Supervisor Agent` → `③ Case Master Agent` → `⑤ Notifier Agent`
* **処理内容**:
    1.  顧客がチャットで問題解決に同意すると、ASAはその旨を`② Supervisor Agent`に報告します。
    2.  Supervisor Agentは`③ Case Master Agent`にケースのクローズを指示します。
    3.  ケースクローズが完了したら、Supervisor Agentは`⑤ Notifier Agent`に「解決通知メール」の送信を依頼します。
    4.  Notifier Agentは、指定されたテンプレートと顧客情報を使って御礼メールを送信します。

---

graph TD
    %% 設定とスタイル定義
    classDef agent fill:#e0f7fa,stroke:#00bcd4,stroke-width:2px;
    classDef system fill:#fff3e0,stroke:#ff9800,stroke-width:2px;
    classDef final fill:#c8e6c9,stroke:#4caf50,stroke-width:2px;

    subgraph 顧客インターフェース
        User[顧客]
        A[① ASA (Agentforce Service Agent)]:::agent
    end

    subgraph 協調・制御コア
        B[② Supervisor Agent (司令塔)]:::agent
    end

    subgraph Salesforce連携レイヤ
        C[③ Case Master Agent]:::agent
        SF((Salesforce)):::system
    end

    subgraph 外部リソース連携
        D[④ Research Agent]:::agent
        E[⑤ Notifier Agent]:::agent
        Web{Webサイト / 外部ナレッジ}:::system
    end

    %% ステップ1: チャット受付とヒアリング
    User -- S1: 1.チャット開始 --> A
    A -- S1: 2.課題・要件のヒアリング --> A
    A -- S2: 3.ヒアリング完了 --> B

    %% ステップ2: タスクの振り分けとケース起票
    B -- S2: 4.新規ケース登録を指示 --> C
    C -- S2: 5.ケース作成/データ送信 --> SF
    SF -- S2: 6.ケース作成完了 (CASE-XXX) --> C
    C -- S2: 7.ケース番号を返却 --> B

    %% ステップ3: Webナレッジの検索と回答生成
    B -- S3: 8.Webナレッジ調査を依頼 --> D
    D -- S3: 9.指定キーワードで検索 --> Web
    Web -- S3: 9.検索結果取得 --> D
    D -- S3: 10.収集・要約したナレッジを返却 --> B
    B -- S3: 11.回答案を提示 --> A
    A -- S3: 12.自然な文章で回答 --> User

    %% ステップ4: ケースクローズと解決後通知
    User -- S4: 13.問題解決に同意 --> A
    A -- S4: 14.解決同意を報告 --> B
    B -- S4: 15.ケースクローズを指示 --> C
    C -- S4: 16.ケースをクローズ処理 --> SF
    B -- S4: 17.解決通知メール送信を依頼 --> E
    E -- S4: 18.御礼/解決通知メール送信 --> User
