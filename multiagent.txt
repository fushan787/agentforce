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
    subgraph "顧客"
        Customer((顧客))
    end

    subgraph "デジタル専門家チーム"
        ASA["① ASA<br/>(対話担当)"]
        Supervisor{"② Supervisor Agent<br/>(司令塔)"}
        CaseMaster["③ Case Master Agent<br/>(Salesforce担当)"]
        Research["④ Research Agent<br/>(Web検索担当)"]
        Notifier["⑤ Notifier Agent<br/>(通知担当)"]
    end

    subgraph "外部システム"
        SalesforceDB[(Salesforce)]
        WebSource[(Web)]
    end

    %% --- 処理フロー ---

    %% ステップ1：チャット受付とヒアリング
    Customer -- 1. チャット問合せ --> ASA;
    ASA -- 2. ヒアリング内容を連携 --> Supervisor;

    %% ステップ2：タスクの振り分けとケース起票
    Supervisor -- 3. ケース作成を指示 --> CaseMaster;
    CaseMaster -- 4. Salesforceにケース登録 --> SalesforceDB;
    SalesforceDB -- 5. ケース番号を返却 --> CaseMaster;
    CaseMaster -- 6. ケース番号を報告 --> Supervisor;

    %% ステップ3：Webナレッジの検索と回答生成
    Supervisor -- 7. 調査を依頼 --> Research;
    Research -- 8. Webを検索 --> WebSource;
    WebSource -- 9. 関連情報を収集 --> Research;
    Research -- 10. 要約ナレッジを報告 --> Supervisor;
    Supervisor -- 11. 回答案を連携 --> ASA;
    ASA -- 12. 顧客へ回答 --> Customer;

    %% ステップ4：ケースクローズと解決後通知
    Customer -- 13. 問題解決に同意 --> ASA;
    ASA -- 14. 解決を報告 --> Supervisor;
    Supervisor -- 15. ケースクローズを指示 --> CaseMaster;
    CaseMaster -- 16. Salesforceのケースを更新 --> SalesforceDB;
    Supervisor -- 17. 解決通知を依頼 --> Notifier;
    Notifier -- 18. 御礼メールを送信 --> Customer;

    %% --- スタイル定義 ---
    style Supervisor fill:#ffc8dd,stroke:#333,stroke-width:2px
    style ASA fill:#c8e6ff,stroke:#333,stroke-width:2px
    style CaseMaster fill:#b9e7e7,stroke:#333,stroke-width:2px
    style Research fill:#ffedc8,stroke:#333,stroke-width:2px
    style Notifier fill:#d8f8d8,stroke:#333,stroke-width:2px
    style Customer fill:#ffffd0,stroke:#333,stroke-width:2px
