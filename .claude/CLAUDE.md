# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

これはDify DSL（Domain Specific Language）ファイルを作成・管理するためのプロジェクトです。Dify DSLはYAMLベースの宣言的言語で、Difyプラットフォーム上でAIアプリケーション（ワークフロー/チャットフロー）を定義・エクスポート・インポートするための標準フォーマットです。

## プロジェクト構造

```
dify-sandbox/
├── .claude/
│   ├── CLAUDE.md         # このファイル（Claude Code全体ガイド）
│   ├── rules            # Dify DSL作成ルール
│   └── settings.local.json  # ローカル設定
└── workflows/           # ワークフローYAMLファイル（作成予定）
```

## Dify DSLの基本構造

すべてのDSLファイルは以下の構造を持ちます：

```yaml
app:
  description: ''
  icon: 🤖
  icon_background: '#FFEAD5'
  mode: workflow  # または chatflow
  name: ''
  use_icon_as_answer_icon: false

kind: app
version: 0.1.2

workflow:
  environment_variables: []
  conversation_variables: []
  features:
    file_upload:
      image:
        enabled: false
    opening_statement: ''
    retriever_resource:
      enabled: false
    sensitive_word_avoidance:
      enabled: false
    speech_to_text:
      enabled: false
    suggested_questions: []
    text_to_speech:
      enabled: false
      language: ''
      voice: ''
  graph:
    edges: []  # ノード間の接続
    nodes: []  # ノード定義
```

## 開発ガイドライン

### DSLファイル作成時のルール

`.claude/rules`ファイルに詳細なルールが定義されています。主なポイント：

1. **ノードID**: Unixタイムスタンプ（ミリ秒）を使用
   ```yaml
   id: '1739026589234'
   ```

2. **変数参照**: 配列形式のvariable_selectorを使用
   ```yaml
   variable_selector:
     - node_id
     - field_name
   ```

3. **テンプレート構文**: プロンプト内では`{{#node.field#}}`、Template TransformではJinja2構文

4. **YAMLフォーマット**: 2スペースインデント、タブ禁止

### ノードタイプ

主なノードタイプ（11種類）：

| タイプ | 用途 |
|--------|------|
| start | ワークフロー開始 |
| end | ワークフロー終了 |
| llm | LLM呼び出し |
| template-transform | テンプレート変換 |
| if-else | 条件分岐 |
| question-classifier | 質問分類 |
| knowledge-retrieval | ナレッジ検索 |
| code | Pythonコード実行 |
| iteration | 繰り返し処理 |
| tool | 外部ツール/API呼び出し |
| parameter-extractor | パラメータ抽出 |

### セキュリティ上の注意

- DSLファイル内にAPIキーやシークレットを含めない
- 機密情報は環境変数を通じて参照

## 参考リソース

- [Dify公式ドキュメント](https://docs.dify.ai/)
- [Awesome Dify Workflow](https://github.com/svcvit/Awesome-Dify-Workflow)
- [Dify DSL Examples](https://github.com/Winson-030/dify-DSL)

---

以下はDify DSLの詳細リファレンスです。

---

# Dify DSL ドキュメント

## セクション1: Dify DSL入門

### DSLとは何か

Dify DSL（Domain Specific Language）は、YAMLベースの宣言的言語で、Difyプラットフォーム上でAIアプリケーションを定義・エクスポート・インポートするための標準フォーマットです。バージョン0.6以降で導入され、ワークフローとチャットフローをテキストベースで管理できるようにします。

### 主要な利点

- **バージョン管理**: Gitなどのバージョン管理システムでワークフローを管理可能
- **コラボレーション**: チームメンバー間での共有とレビューが容易
- **再現性**: 同じワークフローを異なる環境で再現可能
- **自動化**: CI/CDパイプラインでのワークフローの自動生成・デプロイ

### バージョン仕様（v0.6+）

DSLは以下のバージョンフィールドで識別されます：

```yaml
kind: app
version: 0.1.2
```

---

## セクション2: DSL構造概要

### トップレベル要素

Dify DSLファイルは以下のトップレベル要素で構成されます：

```yaml
app:                 # アプリケーションメタデータ
  description: ''    # アプリの説明
  icon: 🤖          # アイコン（絵文字）
  icon_background: '#FFEAD5'  # アイコン背景色
  mode: workflow     # モード: workflow または chatflow
  name: App Name     # アプリ名
  use_icon_as_answer_icon: false

kind: app            # リソース種類
version: 0.1.2       # DSLバージョン

workflow:            # ワークフロー定義
  environment_variables: []        # 環境変数
  conversation_variables: []        # 会話変数
  features:                        # 機能設定
    file_upload: {...}
    opening_statement: ''
    retriever_resource: {...}
    sensitive_word_avoidance: {...}
    speech_to_text: {...}
    suggested_questions: []
    text_to_speech: {...}

  graph:              # グラフ構造
    edges: [...]      # ノード間の接続
    nodes: [...]      # ノード定義
```

### workflow vs chatflowの違い

| 特徴 | workflow | chatflow |
|------|----------|----------|
| 目的 | タスク自動化、バッチ処理 | 対話型AIアシスタント |
| 実行 | 一方向のデータフロー | 状態を保持する対話 |
| 開始ノード | 入力変数を定義 | オープニングメッセージ |
| 終了ノード | 出力値を返す | 回答を生成 |

---

## セクション3: ノードタイプリファレンス

### 1. Startノード

ワークフローの開始点を定義します。

```yaml
- data:
    type: start
    variables:
      - variable: query
        type: text-input
        label: クエリ
        required: true
        max_length: 1000
        default: ''
  id: '1739026589234'
  position: {x: 80, y: 200}
  type: custom
```

**フィールド:**
- `type`: "start"（固定）
- `variables`: 入力変数の配列
  - `variable`: 変数名
  - `type`: text-input, paragraph, select, number, file など
  - `label`: 表示ラベル
  - `required`: 必須フラグ
  - `default`: デフォルト値

### 2. Endノード

ワークフローの終了点を定義します。

```yaml
- data:
    type: end
    variables:
      - variable: answer
        label: 回答
  id: '1739026589235'
  position: {x: 800, y: 200}
  type: custom
```

**フィールド:**
- `type`: "end"（固定）
- `variables`: 出力変数の配列
  - `variable`: 参照する変数セレクタ
  - `label`: 表示ラベル

### 3. LLMノード

大規模言語モデルを呼び出します。

```yaml
- data:
    type: llm
    model:
      provider: openai
      name: gpt-4
      mode: chat
      completion_params:
        temperature: 0.7
        max_tokens: 2000
    prompt_template:
      - role: system
        text: |
          あなたは親切なAIアシスタントです。
          ユーザーの質問に日本語で答えてください。
      - role: user
        text: |
          {{#start.query#}}
    context:
      enabled: false
      variable_selector: []
  id: '1739026589236'
  position: {x: 300, y: 200}
  type: custom
```

**フィールド:**
- `model`: モデル設定
  - `provider`: プロバイダー名（openai, anthropic, etc.）
  - `name`: モデル名
  - `completion_params`: 生成パラメータ
- `prompt_template`: プロンプトテンプレート
  - `role`: system, user, assistant
  - `text`: プロンプトテキスト（Jinja2構文）
- `context`: コンテキスト設定（オプション）

### 4. Template Transformノード

Jinja2テンプレートを使用してテキストを変換します。

```yaml
- data:
    type: template-transform
    template: |
      ## タイトル: {{#start.title#}}

      要約:
      {{#llm.summary#}}

      カテゴリ: {{#classifier.category#}}
    output_type: string
    variables: []
  id: '1739026589237'
  position: {x: 500, y: 200}
  type: custom
```

**フィールド:**
- `type`: "template-transform"（固定）
- `template`: Jinja2テンプレート
- `output_type`: 出力型（string, array_string, number, array_number, object）
- `variables`: テンプレートで使用する変数（自動検出）

### 5. If-Elseノード

条件分岐を実装します。

```yaml
- data:
    type: if-else
    conditions:
      - id: 'condition_1'
        variable_selector:
          - start
          - score
        comparison_operator: '>='
        value: '60'
    logical_operator: and
  id: '1739026589238'
  position: {x: 400, y: 200}
  type: custom
```

**フィールド:**
- `type`: "if-else"（固定）
- `conditions`: 条件の配列
  - `id`: 条件ID
  - `variable_selector`: 比較対象の変数
  - `comparison_operator`: 比較演算子（contains, not contains, =, !=, >, <, >=, <=, is empty, is not empty）
  - `value`: 比較値
- `logical_operator`: 論理演算子（and, or）

### 6. Question Classifierノード

質問をカテゴリに分類します。

```yaml
- data:
    type: question-classifier
    classes:
      - id: 'class_1'
        name: 技術的な質問
        description: プログラミングや技術に関する質問
      - id: 'class_2'
        name: 一般的な質問
        description: 日常的な質問
    model:
      provider: openai
      name: gpt-3.5-turbo
    query_variable_selector:
      - start
      - query
    instruction: ''
  id: '1739026589239'
  position: {x: 200, y: 200}
  type: custom
```

**フィールド:**
- `type`: "question-classifier"（固定）
- `classes`: 分類クラスの配列
  - `id`: クラスID
  - `name`: クラス名
  - `description`: クラスの説明
- `model`: 使用するモデル
- `query_variable_selector`: 分類対象の変数
- `instruction`: 追加の指示

### 7. Knowledge Retrievalノード

ナレッジベースから情報を検索します。

```yaml
- data:
    type: knowledge-retrieval
    dataset_ids:
      - 'dataset_uuid_here'
    retrieval_mode: single
    multiple_retrieval_config:
      top_k: 3
      score_threshold: 0.5
    query_variable_selector:
      - start
      - query
  id: '1739026589240'
  position: {x: 300, y: 300}
  type: custom
```

**フィールド:**
- `type`: "knowledge-retrieval"（固定）
- `dataset_ids`: データセットIDの配列
- `retrieval_mode`: 検索モード（single, multiple）
- `multiple_retrieval_config`: 複数検索設定
  - `top_k`: 取得する結果数
  - `score_threshold`: 類似度閾値
- `query_variable_selector`: 検索クエリの変数

### 8. Codeノード

Pythonコードを実行します。

```yaml
- data:
    type: code
    code: |
      def main(input_text: str) -> dict:
          result = input_text.upper()
          return {
              "output": result,
              "length": len(result)
          }
    outputs:
      - variable: output
        type: string
      - variable: length
        type: number
    outputs_mapping: {}
  id: '1739026589241'
  position: {x: 400, y: 300}
  type: custom
```

**フィールド:**
- `type`: "code"（固定）
- `code`: Pythonコード（main関数を定義）
- `outputs`: 出力定義
  - `variable`: 変数名
  - `type`: 変数型
- `outputs_mapping`: 出力マッピング

### 9. Iterationノード

リスト内のアイテムを繰り返し処理します。

```yaml
- data:
    type: iteration
    iterator_selector:
      - code
      - items
    output_selector:
      - item
      - title
    output_type: array[string]
  id: '1739026589242'
  position: {x: 500, y: 300}
  type: custom
```

**フィールド:**
- `type`: "iteration"（固定）
- `iterator_selector`: 繰り返し処理するリスト
- `output_selector`: 出力として抽出するフィールド
- `output_type`: 出力型

### 10. Toolノード

外部ツールやAPIを呼び出します。

```yaml
- data:
    type: tool
    provider_id: 'weather'
    tool_name: 'get_current_weather'
    tool_configurations: {}
    tool_parameters:
      location:
        variable: '{{#start.city#}}'
        value: ''
  id: '1739026589243'
  position: {x: 600, y: 300}
  type: custom
```

**フィールド:**
- `type`: "tool"（固定）
- `provider_id`: ツールプロバイダーID
- `tool_name`: ツール名
- `tool_configurations`: ツール設定
- `tool_parameters`: ツールパラメータ
  - `variable`: パラメータ値（変数参照または固定値）

### 11. Parameter Extractorノード

テキストから構造化されたパラメータを抽出します。

```yaml
- data:
    type: parameter-extractor
    model:
      provider: openai
      name: gpt-3.5-turbo
    parameters:
      - name: email
        type: string
        description: ユーザーのメールアドレス
        required: true
    query_variable_selector:
      - start
      - text
    instruction: 'ユーザー情報を抽出してください'
  id: '1739026589244'
  position: {x: 700, y: 300}
  type: custom
```

**フィールド:**
- `type`: "parameter-extractor"（固定）
- `model`: 使用するモデル
- `parameters`: 抽出するパラメータ
  - `name`: パラメータ名
  - `type`: データ型
  - `description`: 説明
  - `required`: 必須フラグ
- `query_variable_selector`: 抽出対象のテキスト
- `instruction`: 追加の指示

---

## セクション4: 変数参照

### variable_selector構文

変数セレクタは配列形式でノードの出力を参照します：

```yaml
variable_selector:
  - node_id      # ソースノードID
  - field_name   # フィールド名
  - nested_field # ネストされたフィールド（オプション）
```

**例:**
```yaml
# Startノードのquery変数を参照
variable_selector:
  - start
  - query

# LLMノードのtext出力を参照
variable_selector:
  - llm_node
  - text

# Codeノードのネストされた出力を参照
variable_selector:
  - code_node
  - result
  - items
  - 0
  - title
```

### テンプレート変数構文（Jinja2）

プロンプトテンプレート内ではJinja2構文を使用します：

```yaml
# Dify構文（プロンプトテンプレート内）
{{#node_id.field_name#}}

# 配列アイテムへのアクセス
{{#code.items[0].title#}}

# Jinja2構文（Template Transformノード内）
{{ variable_name }}
{{ items[0].title }}
{% if condition %}...{% endif %}
```

### システム変数

| 変数 | 説明 |
|------|------|
| `sys.user_id` | ユーザーID |
| `sys.conversation_id` | 会話ID |
| `sys.workflow_run_id` | ワークフロー実行ID |

---

## セクション5: エッジと接続

### エッジ構造

ノード間の接続はedges配列で定義します：

```yaml
edges:
  - source: 'source_node_id'
    sourceHandle: source
    target: 'target_node_id'
    targetHandle: target
    type: custom
    data:
      sourceType: llm
      targetType: end
      isInIteration: false
```

**フィールド:**
- `source`: ソースノードID
- `sourceHandle`: ソースハンドル名（通常は"source"）
- `target`: ターゲットノードID
- `targetHandle`: ターゲットハンドル名（条件分岐では条件ID）
- `type`: 接続タイプ（通常は"custom"）
- `data`: 接続メタデータ
  - `sourceType`: ソースノードタイプ
  - `targetType`: ターゲットノードタイプ
  - `isInIteration`: 繰り返し内かどうか

### 接続ルール

1. **DAG構造**: ノードは有向非巡回グラフ（DAG）を形成する必要があります
2. **ハンドル命名**:
   - 通常のノード: `source` → `target`
   - If-Else: `source` → 条件ID（例: `true`, `false`）
   - Question Classifier: `source` → クラスID（例: `class_1`）

3. **If-Else分岐の例:**
```yaml
edges:
  - source: 'if_else_node'
    sourceHandle: 'true'
    target: 'node_a'
    targetHandle: target
  - source: 'if_else_node'
    sourceHandle: 'false'
    target: 'node_b'
    targetHandle: target
```

4. **Question Classifier分岐の例:**
```yaml
edges:
  - source: 'classifier_node'
    sourceHandle: 'class_1'
    target: 'technical_handler'
    targetHandle: target
  - source: 'classifier_node'
    sourceHandle: 'class_2'
    target: 'general_handler'
    targetHandle: target
```

---

## セクション6: ベストプラクティス

### YAMLフォーマット

- **インデント**: 2スペースを使用（タブ禁止）
- **引用符**: 特殊文字を含む文字列はシングルクォートで囲む
- **行長**: 80-100文字以内で折り返す

### ノード編成

- **ID命名**: タイムスタンプベースの一意IDを使用
  ```yaml
  id: '1739026589234'  # Unixタイムスタンプ（ミリ秒）
  ```
- **位置配置**: 視覚的なフローを考慮して配置
  ```yaml
  position: {x: 100, y: 200}  # 左から右へ、上から下へ
  ```

### 変数管理

- **命名規則**: スネークケースを使用（例: `user_query`, `api_response`）
- **スコープ**: 変数は定義されたノード以降で使用可能
- **型の一貫性**: 変数の型をドキュメント化

### エラー防止

- **参照検証**: すべてのvariable_selectorで参照先を確認
- **エッジ検証**: すべてのエッジが有効なノードIDを指しているか確認
- **サイクル検出**: 循環参照がないか確認

---

## セクション7: 一般的なエラーと解決策

### YAML検証エラー

| エラー | 原因 | 解決策 |
|--------|------|--------|
| `mapping values are not allowed here` | インデントが不正 | 2スペースインデントを使用 |
| `could not find expected ':'` | コロンがないか位置が不正 | キーと値の間にコロンスペースを入れる |
| `bad indentation of a mapping entry` | インデントが不統一 | すべてのインデントを2スペースに統一 |

### ワークフロー実行エラー

| エラー | 原因 | 解決策 |
|--------|------|--------|
| `Variable not found` | variable_selectorが間違っている | ノードIDとフィールド名を確認 |
| `Node not found` | エッジ参照が間違っている | すべてのノードIDが存在するか確認 |
| `Invalid template syntax` | Jinja2構文が間違っている | テンプレート構文を検証 |
| `Type mismatch` | 変数の型が一致しない | 型変換を追加 |

---

## セクション8: 参考リソース

### 公式ドキュメント

- [Dify Key Concepts](https://docs.dify.ai/en/use-dify/getting-started/key-concepts)
- [Dify Quick Start](https://docs.dify.ai/en/use-dify/getting-started/quick-start)
- [Template Node](https://docs.dify.ai/en/use-dify/nodes/template)
- [If-Else Node](https://docs.dify.ai/en/use-dify/nodes/ifelse)
- [Iteration Node](https://docs.dify.ai/en/use-dify/nodes/iteration)
- [Knowledge Retrieval](https://docs.dify.ai/en/use-dify/nodes/knowledge-retrieval)
- [Code Execution](https://docs.dify.ai/en/use-dify/nodes/code)

### コミュニティリソース

- [Awesome Dify Workflow](https://github.com/svcvit/Awesome-Dify-Workflow) - ワークフローの例とベストプラクティス
- [Dify DSL Examples](https://github.com/Winson-030/dify-DSL) - DSLファイルのサンプルコレクション
- [Dify Community Discord](https://discord.gg/dify) - コミュニティサポート

---

## 付録: 完全なワークフロー例

```yaml
app:
  description: サンプルワークフロー
  icon: 🤖
  icon_background: '#FFEAD5'
  mode: workflow
  name: Sample Workflow
  use_icon_as_answer_icon: false

kind: app
version: 0.1.2

workflow:
  environment_variables: []
  conversation_variables: []
  features:
    file_upload:
      image:
        enabled: false
    opening_statement: ''
    retriever_resource:
      enabled: false
    sensitive_word_avoidance:
      enabled: false
    speech_to_text:
      enabled: false
    suggested_questions: []
    text_to_speech:
      enabled: false
      language: ''
      voice: ''

  graph:
    edges:
      - source: '1739026589234'
        sourceHandle: source
        target: '1739026589236'
        targetHandle: target
        type: custom
        data:
          sourceType: start
          targetType: llm
          isInIteration: false
      - source: '1739026589236'
        sourceHandle: source
        target: '1739026589235'
        targetHandle: target
        type: custom
        data:
          sourceType: llm
          targetType: end
          isInIteration: false

    nodes:
      - data:
          type: start
          variables:
            - variable: query
              type: text-input
              label: クエリ
              required: true
              max_length: 1000
              default: ''
        id: '1739026589234'
        position: {x: 80, y: 200}
        type: custom

      - data:
          type: llm
          title: LLM
          model:
            provider: openai
            name: gpt-4
            mode: chat
            completion_params:
              temperature: 0.7
          prompt_template:
            - role: system
              text: |
                あなたは親切なAIアシスタントです。
            - role: user
              text: |
                {{#start.query#}}
          context:
            enabled: false
            variable_selector: []
          vision:
            enabled: false
          voice: null
        id: '1739026589236'
        position: {x: 380, y: 200}
        type: custom

      - data:
          type: end
          variables:
            - variable: '1739026589236.text'
              label: 回答
        id: '1739026589235'
        position: {x: 680, y: 200}
        type: custom
```
