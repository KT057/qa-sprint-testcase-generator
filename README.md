# QA Sprint Testcase Generator

Claude CodeのSkills・サブエージェント・JIRA MCPを組み合わせて、QAが `/generate-testcases Sprint-5` と打つだけでテストケースを自動生成する仕組み。

## 全体構成

```
.mcp.json                                    # JIRA MCP設定
.claude/
├── skills/
│   └── generate-testcases/
│       └── SKILL.md                         # Skill（QAの入口＋テスト観点）
└── agents/
    └── qa-executor.md                       # サブエージェント（安全な実行環境）
```

### 3層構造

| コンポーネント | ファイル | 役割 |
|---|---|---|
| JIRA MCP | `.mcp.json` | JIRAチケット情報の取得基盤 |
| Skill | `.claude/skills/generate-testcases/SKILL.md` | QAの入口＋テスト観点のナレッジ |
| サブエージェント | `.claude/agents/qa-executor.md` | 安全な実行環境（コード書き換え禁止） |

## セットアップ

### 1. 環境変数の設定

`.env.example` をコピーして `.env` を作成し、JIRA認証情報を設定。

```bash
cp .env.example .env
```

```bash
JIRA_URL=https://your-domain.atlassian.net
JIRA_EMAIL=your-email@example.com
JIRA_API_TOKEN=your-api-token
```

JIRA API Tokenは [Atlassianアカウント設定](https://id.atlassian.com/manage-profile/security/api-tokens) から発行できます。

### 2. MCPサーバーの動作確認

Claude Codeを起動し、MCPツールが利用可能か確認。

```bash
claude
```

```
利用可能なMCPツールを教えて
```

`mcp__atlassian` ツールが表示されればOK。

## 使い方

### QAがやること

```
/generate-testcases MYPROJECT-Sprint-5
```

これだけ。Skill（`context: fork`）が隔離環境で以下を自動実行します。

1. JIRA MCPからスプリントのチケット一覧を取得
2. 各チケットの受け入れ条件を読み取り
3. Grep/Globで該当機能のコードを解析
4. テスト観点を適用してテストケースを生成

### テスト観点のカスタマイズ

`.claude/skills/generate-testcases/SKILL.md` の「テスト観点（プロジェクト固有）」セクションを編集してください。プロジェクト固有のビジネスルールやチェック項目を追加できます。

## 設計ポイント

- **Skill = 「何をするか」**: ワークフロー、テスト観点、出力フォーマットを定義
- **サブエージェント = 「どう安全に実行するか」**: `disallowedTools: [Write, Edit]` でコード書き換えを禁止
- **JIRA MCP = データ取得基盤**: チケット情報へのアクセスを提供

## 関連記事

- [支援先のQAチームにClaude Codeを導入してテストケース生成を自動化した話](https://zenn.dev/and_and/articles/claude-code-qa-sprint-testcase-generation)
