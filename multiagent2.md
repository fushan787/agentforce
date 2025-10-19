
```mermaid
flowchart TD

%% エージェント定義
subgraph UserSide["👤 ユーザー"]
    UQ[ユーザーから問い合わせ]
end

subgraph ASA["🤖 ASA（Service Agent）"]
    ASA1[受信: 問い合わせ内容解析]
    ASA2[Flow: ケース作成 + 状態管理レコード作成]
end

subgraph Primary["🧭 オーケストレーター（Primary Agent）"]
    ORC1[Flow: Case情報取得]
    ORC2[LLM判定: 自動対応 or 人対応]
    ORC3[Flow: 分岐処理<br>→ 自動処理 or 人オペレータへ]
end

subgraph Processing["⚙️ 処理エージェント（Secondary Agent）"]
    SEC1[受信: オーケストレーター指示]
    SEC2[Flow/Apex: ケース処理実行]
    SEC3[結果報告: 状態更新・完了通知]
end

subgraph Human["👨‍💼 人間オペレーター"]
    HUM1[Service Console: ケース確認]
    HUM2[処理実施・完了報告（ボタン押下）]
end

subgraph Notify["📩 通知エージェント（Secondary Agent）"]
    NOT1[Flow: 顧客通知メール作成]
    NOT2[送信完了 → 状態更新]
end

%% データレイヤ
subgraph Data["🗃️ Salesforce Data"]
    CASE[Case オブジェクト]
    ORCH[CaseOrch__c<br>（状態・判断・結果管理）]
end

%% フロー関係
UQ --> ASA1 --> ASA2 --> ORC1
ORC1 --> ORC2 --> ORC3

ORC3 -->|自動処理可| SEC1
SEC1 --> SEC2 --> SEC3 --> NOT1
NOT1 --> NOT2 --> ORCH

ORC3 -->|人対応必要| HUM1 --> HUM2 --> NOT1

%% データ更新
ASA2 --> CASE
ASA2 --> ORCH
ORC3 --> ORCH
SEC3 --> ORCH
HUM2 --> ORCH
NOT2 --> ORCH

%% 終了
NOT2 -->|通知完了| END[(✅ Case Closed)]
```

```mermaid
sequenceDiagram
    autonumber

    participant User as 👤 ユーザー
    participant ASA as 🤖 ASA<br>(Service Agent)
    participant Case as 📄 Case / CaseOrch__c
    participant ORC as 🧭 オーケストレーター<br>(Primary Agent)
    participant SEC as ⚙️ 処理エージェント<br>(Secondary Agent)
    participant Human as 👨‍💼 人間オペレーター
    participant NOT as 📩 通知エージェント<br>(Secondary Agent)

    %% 1. ユーザー問い合わせ
    User->>ASA: 問い合わせ送信（チャット／メールなど）
    ASA->>ASA: 入力解析（LLM or NLU）
    ASA->>Case: Flow実行 → Case作成 + CaseOrch__c初期化
    ASA->>ORC: 「新規ケース」イベントをトリガー

    %% 2. オーケストレーター判断
    ORC->>Case: ケース詳細を取得
    ORC->>ORC: LLM推論 / ルール判定（自動対応 or 人対応）
    alt 自動対応可能
        ORC->>SEC: 「処理依頼」アクション送信（Agent-to-Agent / Flow起動）
    else 人対応が必要
        ORC->>Human: Service Console タスク作成 / Slack通知
    end
    ORC->>Case: 状態更新（CurrentStage = "Processing" または "Human"）

    %% 3. 処理エージェントの実行
    par 自動処理フロー
        SEC->>Case: Flow / Apex 実行（ケース更新・外部API呼出など）
        SEC->>Case: 結果を CaseOrch__c に記録
        SEC->>ORC: 完了通知イベント送信
    and 人対応フロー
        Human->>Case: 対応実施 → 結果更新
        Human->>ORC: 完了報告（Flow経由）
    end

    %% 4. 通知フロー
    ORC->>NOT: 通知指示（完了通知アクション起動）
    NOT->>Case: 顧客通知内容生成（Emailテンプレート）
    NOT->>User: メール送信（完了報告）
    NOT->>Case: CaseOrch__c に通知結果記録

    %% 5. 最終更新
    ORC->>Case: 状態更新（CurrentStage = "Closed"）
    ORC->>ASA: ケースクローズ通知（内部イベント）
    ASA-->>User: 「処理完了」メッセージ送信
```
```mermaid
%%==================================================
%% A2Aを使わない構成：Salesforce内ネイティブ連携
%%==================================================
sequenceDiagram
    autonumber
    box Internal Salesforce Org
    participant User as 👤 ユーザー
    participant ASA as 🤖 ASA<br>(Service Agent)
    participant ORC as 🧭 オーケストレーター<br>(Primary Agent)
    participant SEC as ⚙️ 処理エージェント<br>(Secondary Agent)
    participant NOT as 📩 通知エージェント
    participant Case as 📄 CaseOrch__c
    end

    Note over ASA,NOT: 🚀 A2Aなし構成（Flow / Agent Action で同期連携）

    User->>ASA: 問い合わせ送信（チャット/メール）
    ASA->>Case: Case + CaseOrch__c 作成
    ASA->>ORC: Flow（Invoke Agent）で起動
    ORC->>Case: 状況判断（自動 or 人間）
    alt 自動処理
        ORC->>SEC: Flow（Invoke Agent）で指示
        SEC->>Case: 更新 / API呼出
        SEC->>ORC: 完了通知（Flow戻り値）
    else 人間対応
        ORC->>Human: Task作成 / Slack通知
    end
    ORC->>NOT: Flow経由で通知指示
    NOT->>User: 完了メール送信
    ORC->>Case: 状態 = Closed 更新
    ASA-->>User: 「完了しました」

    Note over ORC,SEC: 🔒 Salesforce内で完結。A2A不要、低遅延、単純。
```

```mermaid
%%==================================================
%% A2Aを使う構成：疎結合／分散エージェント構成
%%==================================================
sequenceDiagram
    autonumber
    box Salesforce Org A
    participant ASA2 as 🤖 ASA<br>(Service Agent)
    participant ORC2 as 🧭 オーケストレーター<br>(Primary Agent)
    end

    box Salesforce Org B / 外部LLM
    participant SEC2 as ⚙️ 処理エージェント<br>(Secondary Agent via A2A)
    participant NOT2 as 📩 通知エージェント
    end

    participant A2A as 🌐 A2Aセッション<br>(JSON/REST/Context Bridge)
    participant User2 as 👤 ユーザー

    Note over ASA2,NOT2: 🌍 A2Aあり構成（疎結合・外部エージェント連携）

    User2->>ASA2: 問い合わせ送信
    ASA2->>ORC2: Flowでケース生成
    ORC2->>A2A: A2Aセッション作成（Agent Context登録）
    A2A->>SEC2: リクエスト転送（タスク＋コンテキスト）
    SEC2->>A2A: 処理結果（JSON）返却
    A2A->>ORC2: 結果受信（セッションID紐づけ）
    ORC2->>A2A: 通知指示を転送
    A2A->>NOT2: 顧客通知実行
    NOT2->>User2: メール送信（外部経由）
    ORC2->>ASA2: ケースクローズイベント発火

    Note over A2A: ⚙️ 外部連携・非同期・セッション管理が必要
```
