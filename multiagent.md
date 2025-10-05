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

## 🧠 構成図

```mermaid
flowchart TD

%% =========================
%% ノード定義
%% =========================

subgraph Customer["👤 顧客"]
    A0["チャット開始"]
end

subgraph ASA["① ASA (Agentforce Service Agent)"]
    A1["課題ヒアリング<br/>顧客情報を収集"]
    A2["Supervisor Agentへ転送"]
    A3["回答提示・解決確認"]
end

subgraph Supervisor["② Supervisor Agent"]
    S1["タスク分析・判断"]
    S2["Case Masterへケース作成指示"]
    S3["Research Agentへ調査依頼"]
    S4["Notifier Agentへ通知依頼"]
end

subgraph CaseMaster["③ Case Master Agent"]
    C1["Salesforce上で<br/>ケース作成"]
    C2["ケースクローズ"]
end

subgraph Research["④ Research Agent"]
    R1["Webナレッジ検索・要約"]
end

subgraph Notifier["⑤ Notifier Agent"]
    N1["解決通知メール送信"]
end

%% =========================
%% フロー（矢印）
%% =========================

Customer --> A0 --> A1 --> A2
A2 --> S1

S1 -->|新規ケース作成| S2 --> C1 --> S1
S1 -->|情報調査が必要| S3 --> R1 --> S1
S1 -->|顧客回答指示| A3

A3 -->|問題解決報告| S1 -->|ケースクローズ| C2 --> S1
S1 -->|通知依頼| S4 --> N1 --> Customer

%% =========================
%% スタイル設定
%% =========================

style ASA fill:#f6e05e,stroke:#b7791f,stroke-width:1px
style Supervisor fill:#90cdf4,stroke:#2b6cb0,stroke-width:1px
style CaseMaster fill:#c6f6d5,stroke:#2f855a,stroke-width:1px
style Research fill:#fbb6ce,stroke:#b83280,stroke-width:1px
style Notifier fill:#fbd38d,stroke:#b7791f,stroke-width:1px
style Customer fill:#e2e8f0,stroke:#4a5568,stroke-width:1px



## 🧠 シーケンス図

```mermaid
sequenceDiagram
    autonumber
    participant Customer as 👤 顧客
    participant ASA as ① ASA<br>(Agentforce Service Agent)
    participant Supervisor as ② Supervisor Agent<br>(司令塔)
    participant CaseMaster as ③ Case Master Agent<br>(Salesforceケース管理)
    participant Research as ④ Research Agent<br>(外部ナレッジ検索)
    participant Notifier as ⑤ Notifier Agent<br>(通知・メール送信)

    %% --- ステップ1 ---
    Customer->>ASA: チャット開始・問い合わせ内容を送信
    ASA->>Customer: 挨拶とヒアリング開始
    ASA->>ASA: 顧客情報と課題を構造化データとして整理

    %% --- ステップ2 ---
    ASA->>Supervisor: ヒアリング結果を引き渡し
    Supervisor->>CaseMaster: ケース登録を指示
    CaseMaster->>Supervisor: Salesforce上でケース作成完了（ケース番号返却）

    %% --- ステップ3 ---
    Supervisor->>Research: 外部ナレッジの調査依頼
    Research->>Research: Web検索・要約生成
    Research-->>Supervisor: 要約結果を返却
    Supervisor->>ASA: 回答案を渡す
    ASA->>Customer: 回答を自然な文章で提示

    %% --- ステップ4 ---
    Customer->>ASA: 解決に同意
    ASA->>Supervisor: ケースクローズ報告
    Supervisor->>CaseMaster: ケースクローズ指示
    CaseMaster->>Supervisor: クローズ完了報告
    Supervisor->>Notifier: 解決通知メール送信を依頼
    Notifier->>Customer: 解決通知＆御礼メール送信

    Note over ASA,Notifier: 各エージェントはSupervisor Agentを中心に協調動作し、<br>顧客体験をパーソナライズして最適化。
