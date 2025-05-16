# AI 生成コードの「70％問題」を突破する TypeScript Lint ルール設計

## アウトライン（10 分登壇）

### 1. オープニング：70%問題の現実 (1 分)

- **Addy Osmani「AI 生成コードの 70%問題」引用**
  - AI は 70%まで動くコードは生成できるが、残り 30%が本当に難しい
  - この 30%が「production-ready, maintainable, robust」な品質

### 2. AI コーディングの現実的スタンス (1.5 分)

- **妥協の受け入れ**
  - 期待する水準ではないコードが上がってくることは前提
  - ただ、圧倒的な速度で実装が可能な AI の特性は活用していきたい
  - モデルの進化で解決する部分と、エンジニアが対処すべき部分を区別

### 3. なぜ Lint ルールに着目するか：Reconciliation Loop (1.5 分)

- > エージェント型 AI を定額使い放題になって感じたのは「Agentic Coding とは Reconciliation Loop である」ということ。望ましい状態を宣言的に定義し、評価関数を与える。エージェントはその状態に収束するよう自律的に働く。理想状態の定義と評価関数の設計が重要。

  - Takuto Wada さんの投稿: https://x.com/tatti40211/status/1922440450675216437

- **Reconciliation Loop**
  - 「理想の状態(評価関数)を定義して、そこに現実を近づけるために繰り返し調整する」
  - AI に出力させたコードと理想状態の差分を検出し、自動で修正させる
- **評価関数の三本柱**
  - 💡 **Lint ルール**：構文・構造レベルの品質を自動担保 ← 今回はこちらにフォーカス
  - 💡 **型システム**：データ構造の整合性を自動検証
  - 💡 **テスト自動化**：期待する振る舞いを自動確認

### 5. AI の妥協パターンと Lint 対策 (3 分)

#### 導入 (30 秒)

「具体的にどのような Lint ルールを活用しているか、現場でよくある AI 妥協パターンと対策を紹介します。かなり主観的ですが、心当たりはありませんか？」

- 巨大な関数で手続き的に解決する
- 「型安全にします！」と言いながら as キャストをする
- 「コードを別ファイルに移動して」と伝えると、新規作成し古いコードを残す
- テストコードを書いている途中に失敗し続け、妥協に妥協を重ね最終的に関数を完全にモックに置き換えてテストが通ったことにする

#### パターン 1：巨大関数問題 (40 秒)

- **あるある**：「ユーザー情報を取得して、権限をチェックして、データを変換して...」→500 行の関数誕生
- **AI の思考**：「とにかくユーザーの要求を満たす動くコードを書けば良い」
- **問題点**：デバッグ困難、テスト困難、再利用不可
- **対策**： biome の noExcessiveCognitiveComplexity ルールを活用
- **結果**：スコア 15 を超えるとエラーになり、AI が自動的に関数分割。(感想)想像以上に可読性が向上する

```typescript
type Cell = "X" | "O" | "";
type Board = Cell[][]; // 3x3

export function checkWinner(b: Board): Cell {
  for (let r = 0; r < 3; r++) {
    if (b[r][0] && b[r][0] === b[r][1] && b[r][1] === b[r][2]) return b[r][0];
  }
  for (let c = 0; c < 3; c++) {
    if (b[0][c] && b[0][c] === b[1][c] && b[1][c] === b[2][c]) return b[0][c];
  }
  if (b[0][0] && b[0][0] === b[1][1] && b[1][1] === b[2][2]) return b[0][0];
  if (b[0][2] && b[0][2] === b[1][1] && b[1][1] === b[2][0]) return b[0][2];

  for (let r = 0; r < 3; r++) {
    for (let c = 0; c < 3; c++) {
      if (!b[r][c]) return "";
    }
  }
  return "T";
}
```

Claude code に頼んだ例

```sh
! pnpm lint:biome
  ⎿
     > sample-project@0.2.2 lint:biome /Users/mh4gf/sample-project
     > biome check . --log-level=warn

     Checked 44 files in 20ms. No fixes applied.
     Found 1 error.
      ELIFECYCLE  Command failed with exit code 1.

  ⎿  ./src/check-winner.ts:4:17 lint/complexity/noExcessiveCognitiveComplexity ━━━━━━━━━━━━━━━━━━━━━━

       × Excessive complexity of 18 detected (max: 15).

         2 │ type Board = Cell[][] // 3x3

     … +10 lines (ctrl+r to expand)

       × Some errors were emitted while running checks.

> エラーを修正してください

⏺ I'll fix the excessive cognitive complexity in the checkWinner function.

⏺ Read(scripts/check-winner.ts)…
  ⎿  Read 21 lines (ctrl+r to expand)

⏺ Let me refactor this function to reduce its cognitive complexity:
```

出力結果

```typescript
export function checkWinner(b: Board): Cell {
  // Check for winner in rows, columns, and diagonals
  const rowWinner = checkRows(b);
  if (rowWinner) return rowWinner;

  const columnWinner = checkColumns(b);
  if (columnWinner) return columnWinner;

  const diagonalWinner = checkDiagonals(b);
  if (diagonalWinner) return diagonalWinner;

  // If no winner and board still has empty cells, game is in progress
  if (isBoardInPlay(b)) return "";

  // Board is full with no winner
  return "T"; // Tie
}
```

#### パターン 2：型安全性の虚偽申告 (40 秒)

- **あるある**：「型安全に実装しました！」→ コード内に as キャストだらけ
- **AI の思考**：「とりあえずコンパイル通すために型を合わせよう」
- **問題点**：ランタイムエラー、型の恩恵なし
- **対策**： typescript-eslint の recommended-type-checked を活用
- **補足**：2025 年の Biome では型情報活用ルールが追加予定

#### パターン 3：デッドコード放置問題 (40 秒)

- 「この機能を別ファイルに移動して」→ 新規作成し、古いファイルを残したまま完了にしてしまう...
- **問題点**：レビュー負荷の向上、デッドコードのチェックの手間
- **対策**：

  ```bash
  npx knip

  Unused files (1)
  scripts/check-winner.ts
   ELIFECYCLE  Command failed with exit code 1.
  ```

#### パターン 4：テストの妥協（vi.mock 濫用） (40 秒)

- **あるある**：テストコードを書いている途中に失敗し続け、妥協に妥協を重ね最終的に関数を完全にモックに置き換えてテストが通ったことにする
  ```typescript
  vi.mock("../services/userService", () => ({
    getUserData: vi.fn().mockResolvedValue({}),
    updateUser: vi.fn().mockResolvedValue({}),
    // 12個の関数全てmock...
  }));
  ```
- **AI の思考**：「とにかくテストが通れば OK、テストの価値は二の次」
  - 本当にモックが必要な場合もあるものの、安易に使う傾向があるように思う
- **対策**： eslint-plugin-vitest の no-restricted-vi-methods ルールを活用し、 `vi.mock` / `vi.mocked` / `vi.spyOn` の使用を禁止
  - 選択肢から除外させ、強制的に実装を見直させる
  - テストコード生成は人間のチェックが不可欠

### 6. 実運用のコツ (0.5 分)

- **AI 主導のルール整備**
  - Lint ルール整備こそAIに任せる
  - ルールの調査：ChatGPT o3 に検索させる「関数の型定義を明示するルールを調べて。できればBiome, なければESLintで」
  - 設定の有効化：Devinに実装→PRを出させる「useExplicitType ルールを有効にしてください。まずはルールの導入だけしたいため、コードに問題があれば biome-ignore して良いです」
- **warn を避けて error に**
  - AI はLintエラーがwarnだと「今は対応不要」と勝手に判断して無視するので、できるだけ error にする
- **失敗回数制限の設定**
  - 「3 回解決失敗したらユーザー相談」ルールの追加
  - Lint ルール無効化への逃げ道を防ぐ

### 7. まとめ：継続的品質改善のサイクル (1 分)

- **Lint ルール = 機械可読な理想状態定義**
  - エンジニアの暗黙知を形式知化
  - AI 自身がルールを理解し、遵守するサイクル構築
- **呼びかけ**
  - 「曖昧な指示を頑張る」から「理想状態を機械可読化」への発想転換
  - 意図的に Lint ルールを増やし、品質を押し上げる文化へ

---

**キーメッセージ**：AI との協働において、人間の役割は「曖昧な自然言語での修正指示」ではなく、「機械が理解できるルールとしての理想状態定義」である。

## 参考資料

- [Addy Osmani「The 70% problem: Hard truths about AI-assisted coding」](https://addyo.substack.com/p/the-70-problem-hard-truths-about)
- [Biome linter rules](https://biomejs.dev/linter/)
- [TypeScript ESLint](https://typescript-eslint.io/)
- [eslint-plugin-vitest](https://github.com/veritem/eslint-plugin-vitest)
- [knip](https://knip.dev/)
