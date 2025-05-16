---
class: text-center
highlighter: shiki
mdc: true
lineNumbers: false
drawings:
  persist: false
transition: fade-out
title: AI生成コードの「70％問題」を突破するTypeScript Lintルール設計
colorSchema: dark
---

# AI 生成コードの「70％問題」を<br>突破する TypeScript Lint ルール設計

スタートアップの TypeScript 実践 Tips #StractDeepDive - 2025/05/19

<!--
今日は「AI生成コードの70%問題を突破するTypeScript Lintルール設計」というテーマでお話しします。よろしくお願いします。
-->

---

# 宮城広隆 / @MH4GF

<div flex="~ gap-1" items-center justify-center class="h-[70%]">
<div>

- ROUTE06, inc.
- GraphQL, 静的解析, GitHub Actions が好き
- Liam ERD（OSS ER 図自動生成ツール）PM 兼テックリード

</div>
<img src="https://avatars.githubusercontent.com/u/31152321?v=4" class="rounded-full w-50 h-50 mx-auto">
</div>

<!--
宮城広隆と申します。ROUTE06という会社でエンジニアをしています。

GraphQLや静的解析、GitHub Actionsが好きで、最近はLiam ERDというOSSのER図自動生成ツールをTypeScriptで開発しており、そのプロダクトマネージャー兼テックリードを務めています。

今年の初めあたりからAIコーディングツールの活用が一気に加速したと思うんですけれども、私たちのプロダクトでも多分に漏れずAIコーディングツールの活用を模索する中で、今日のテーマに興味を持ちました。
-->

---

# "[The 70% problem: Hard truths about AI-assisted coding](https://addyo.substack.com/p/the-70-problem-hard-truths-about)" by Addy Osmani

<div>

<v-clicks>

- AI コーディングは 70%まで驚くほど素早く進むが、Production Ready にするための残りの 30%で苦戦する
- AI は「熱心なジュニア開発者」のように振る舞う。迅速にコードを書くが、常に監督と修正が必要
- **残りの 30%が真の課題**
  - 二歩後退パターン（修正 → 別の箇所が壊れる → さらなる修正 → さらに別の箇所が壊れる）
  - エラー処理や例外ケースの見落とし
  - 型安全性の確保
  - アーキテクチャの一貫性維持
  - パフォーマンス・セキュリティの考慮不足

</v-clicks>

</div>

<div v-click="4">

> "Junior engineers often miss these crucial steps. They accept the AI's output more readily, leading to what I call "house of cards code" – it looks complete but collapses under real-world pressure."
>
> — Addy Osmani

</div>

<!--
さっそくですが、Googleのシニアエンジニアリングディレクター、Addy Osmaniの記事で紹介されている「70%問題」について紹介します。これはAIによるコーディング支援の本質的な課題を示しています。

AIツールを使うと、コーディング作業の約70%を驚くほど素早く進められます。しかし、残りの30%を完成させる過程で多くのエンジニアが苦戦しています。Osmaniは「AIは熱心なジュニア開発者のようなもの」と表現しています。コードを迅速に書くものの、常に監督と修正が必要なのです。

残りの30%の何が難しいのでしょうか？例えば連鎖的に別の箇所が壊れ続ける二歩後退パターン、エラー処理の不足、型安全性の不足、アーキテクチャの一貫性の無視、そしてパフォーマンスやセキュリティの考慮といった、経験豊富なエンジニアが持つ深い知識が必要な部分が残っています。

Osmaniは生成AIコードを「トランプの家」に例えています。表面上は完璧に見えるものの、本質的に脆弱であり、実際のプロダクション環境の圧力にさらされると崩壊してしまうコードなのです。
-->

---

# AI コーディングの現実的スタンス

<div class="grid grid-cols-[60%,40%] gap-4">
<div>

<v-clicks>

- **70%問題は確かに存在する**
  - しかし圧倒的な速度で実装可能な AI の特性は活用したい
  - 定型的なコーディング作業や日常的なコーディング作業に費やす時間が大幅に短縮され、興味深い問題に集中できるようになる
- **妥協の受け入れ**
  - 期待する水準ではないコードが生成されることは前提で、何を守るべきか？を考える
- **分担の明確化**
  - エンジニアの価値：実装の詳細でなく、全体設計と品質保証に

</v-clicks>

</div>
<div>

</div>
</div>

<!--
70%問題は確かに現実なのですが、AIの圧倒的な実装速度は無視できないメリットです。定型的なコーディング作業や日常的なコーディング作業に費やす時間が大幅に短縮され、エンジニアはより興味深い問題に集中できるようになる可能性があります。

我々はどう活用すべきなのでしょうか？

今の私のスタンスは、AIが生成するコードが必ずしも期待通りの品質ではないことを前提とし、その上で何を守るべきか考えるのが良いと考えています。実装の詳細はAIに任せ、全体設計や品質保証がエンジニアの価値になっていくと思います。

-->

---

# Reconciliation Loop

<div class="grid grid-cols-5 gap-6">

<div class="col-span-2">
  <Tweet id="1922110843270885789" />
</div>

<div class="col-span-3">
<v-clicks>

- 理想の状態(評価関数)を定義し、AI エージェントにその状態に到達するまで自動で修正させる
- エンジニアは「理想状態の定義」と「評価関数の設計」を行う
  - lint やテストを AI エージェントが実行可能な形で整備する

</v-clicks>
</div>
</div>

<!--
もう一つツイートを引用し、つい先週のTakuto Wadaさんのコメントを紹介します。

Wadaさんは「Agentic CodingとはReconciliation Loopである」と述べています。これは「理想の状態を評価関数として宣言的に定義し、エージェントにその状態に収束するよう自律的に修正させる」というモデルです。

エンジニアは理想状態の定義と評価関数の設計を行います。具体的には、LintやテストをAIエージェントが実行可能な形で整備することが求められます。

-->

---

# 評価関数の三本柱

<div class="grid grid-cols-3 gap-4">
<div class="box">

<v-click>

## 型システム

- データ構造の整合性を自動検証
- インターフェース契約の明示
- コンパイル時のバグ発見と予防

</v-click>

</div>
<div class="box">
<v-click>

## テスト自動化

- 期待する振る舞いを自動確認
- アウトプットの正確性を検証
- エッジケースのカバレッジ確保
- リグレッション防止

</v-click>
</div>
<div class="box">

<v-click>

## Lint ルール

- 構文・構造レベルの品質を自動担保
- 一貫したコードスタイルと構造
- コーディング規約の機械可読化
- **今回のフォーカス**

</v-click>

</div>
</div>

<!--
AIと協働する際の評価関数には、三つの重要な柱があります。

一つ目は「Lintルール」です。これは構文や構造レベルの品質を自動的に担保するもので、一貫したコードスタイルと構造を強制し、コーディング規約を機械可読な形で実装します。今回のプレゼンテーションでは、特にこのLintルールに焦点を当てています。

二つ目の柱は「型システム」です。これはデータ構造の整合性を自動的に検証するもので、インターフェース契約を明示的に定義し強制することで、コンパイル時にバグを発見し予防します。

三つ目の柱は「テスト自動化」です。これは期待する振る舞いを自動的に確認するもので、アウトプットの正確性を検証し、エッジケースのカバレッジを確保します。

この三本柱を活用することで、理想状態を明確に定義し、AIに自己修正を促すサイクルを確立できます。
-->

---

# AI の妥協パターンと Lint 対策

<div class="grid grid-cols-[60%,40%] gap-4">
<div>

<v-clicks>

- 巨大な関数で手続き的に解決する
- 「型安全にします！」と言いながら as キャストをする
- 「コードを別ファイルに移動して」と伝えると、新規作成し古いコードを残す
- テストコードを書かせるも失敗し続け、妥協に妥協を重ね最終的に関数を完全にモックに置き換えてテストが通ったことにする

</v-clicks>

</div>
<div>

</div>
</div>

<!--
ここからは、AIコーディングでよく見られる妥協パターンと、それに対する具体的なLintルールによる対策を紹介します。これはかなり主観的な観察に基づいていますが、皆さんにも心当たりがあるのではないでしょうか？

例えば、「ユーザー情報を取得して、権限をチェックして、データを変換して…」という複数の機能を一つの巨大な関数にまとめてしまうパターン。または、「型安全に実装しました！」と言いながら実際のコードにはasキャストが頻出しているケース。さらに、コードを別ファイルに移動してほしいと伝えると、新しいファイルを作成するだけで古いコードをそのまま残してしまうパターン。そして、テストコードを書いているときに失敗が続くと、最終的には関数を完全にモックに置き換えてテストを無理やり通すといった妥協も見られます。

これらのパターンは、AIが「とりあえず動くコード」を生成するのに対し、我々が求める「保守可能な品質のコード」とのギャップを表しています。
-->

---

# パターン 1：巨大関数問題

<div class="grid grid-cols-[60%,40%] gap-4">
<div>

<v-clicks>

- 「ユーザー情報を取得して、権限をチェックして、データを変換して...」→ 結果、500 行の関数が誕生してしまう
- 対策: biome の noExcessiveCognitiveComplexity ルールを活用
  - 認知的複雑度のスコアが 15 を超えるとエラーにし、関数の分割を促す

</v-clicks>

</div>
</div>

<!--
最初のパターンは「巨大関数問題」です。AIに「ユーザー情報を取得して、権限をチェックして、データを変換して…」といった複数の処理を実装するよう依頼すると、しばしば500行を超えるような巨大な関数を生成します。

LLMは「とにかくユーザーの要求を満たす動くコードを書けば良い」という思考を持っている様に思います。個別の機能を分割する意識が弱いのです。

このような巨大関数には、デバッグが困難になる、テストが困難になる、コードの再利用が不可能になるといった問題点があります。

対策として、biomeの「noExcessiveCognitiveComplexity」ルールが非常に効果的です。これは関数の認知的複雑性を計測し、設定したスコア（例えば15）を超えるとエラーとして検出するルールです。

このルールを設定すると、AIは自動的に関数を適切なサイズに分割し始めます。結果として、想像以上に可読性が向上するコードが生成されるようになります。
-->

---

# パターン 1：問題のあるコード例

```typescript {all}
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

<!--
ここに問題のあるコード例を示します。これは三目並べのゲームで勝者を判定する関数です。

競技プログラミングをやる方は見慣れているかもしれませんが、AIがこのコードを出力してきた時に人間が正しさを判断することは難しいと思います。

このコードをbiomeのlinterにかけると、認知的複雑性が高すぎるというエラーが発生します。biomeは関数の条件分岐やループのネストなどを分析し、認知的な負荷を数値化します。この関数は設定した閾値（15）を超えています。
-->

---

# パターン 1：Claude Code に修正させる

```bash {all}
! npx biome check . --log-level=warn
  ⎿
     Checked 44 files in 20ms. No fixes applied.
     Found 1 error.

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

<!--
これをClaude Codeに修正させてみましょう。Claude Codeは `!` をつけるとシェルコマンドをそのまま実行できます。
Biomeを実行すると、このように「Excessive complexity of 18 detected (max: 15)」というエラーが検出されます。

このエラーを修正するように指示すると、Claude Codeは「checkWinner関数の認知的複雑性を下げるためにリファクタリングします」と言って、コードを読み込み、修正をし始めます。
-->

---

# パターン 1：AI による修正結果

```typescript {all}
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

<!--
AIに修正を依頼した結果、このようなコードが生成されます。元の関数は責務ごとに分割され、それぞれの処理が適切に名前付けされた補助関数に移動されています。

checkRows、checkColumns、checkDiagonals、isBoardInPlayという明確な名前の関数が新たに作成され、メイン関数はこれらを呼び出すだけのシンプルな構造になりました。

これにより、コードの可読性が大幅に向上し、各部分が何をしているのかが一目でわかるようになりました。また、テストも容易になり、バグの発見も簡単になります。認知的複雑性も大幅に下がり、Lintエラーが解消されます。

このようにLintルールを活用することで、AIに対して「もっと良いコードを書いて」といった曖昧な指示ではなく、具体的な評価基準を与えることができ、AIの出力品質を向上させることができるのです。
-->

---

# パターン 2：型安全性の問題

<div class="grid grid-cols-[100%] gap-4">
<div>

<v-clicks>

- 「型安全に実装しました！」と言いながらコードに `as` キャストが多用されている...
- AI は「とりあえずコンパイルを通すために型を合わせる」傾向があるように見える
- **問題点**：ランタイムエラーのリスク、型システムの恩恵を失う
- **対策**： typescript-eslint の recommended-type-checked 設定を活用
  - コンパイラの型情報を使用した高度な静的解析
  - any 型の使用制限、安全でないキャストの検出
  - 型の恩恵を最大化する厳格なルールセット
- **補足**：2025 年の Biome のロードマップでは型情報活用ルールが追加予定

</v-clicks>

</div>
</div>

<!--
次のパターンは型安全性に関する問題です。AIが「型安全に実装しました」と言いながら、実際には「as」キャストを多用しているケースが見られます。

AIには「とりあえずコンパイルを通すために型を合わせる」傾向があるように見えます。型システムの本来の目的よりも、エラーを表示させないことを優先しているように思われます。

この問題の結果として、ランタイムでエラーが発生するリスクが高まり、TypeScriptの型システムがもたらす恩恵を十分に受けられなくなります。

対策として、typescript-eslintのrecommended-type-checked共有設定を活用することが効果的です。この設定はTypeScriptコンパイラの型情報を使用した高度な静的解析を可能にし、any型の使用制限や安全でないキャストの検出など、型の恩恵を最大化する厳格なルールセットを提供します。

2025年にはBiomeでも型情報を活用したルールが追加される予定ですが、現時点ではtypescript-eslintを活用するのが最も効果的です。
-->

---

# パターン 3：デッドコード放置問題

<div class="grid grid-cols-[100%] gap-4">
<div>

<v-clicks>

- 「この機能を別ファイルに移動して」→ 新規作成し、古いファイルを残したまま完了にしてしまう...
- **問題点**：
  - コードベースの肥大化
  - レビュー負荷の増加(デッドコードがないことを確認する必要がある)
  - AI がそのコードを参考にしてしまう
- **対策**： knip を活用したデッドコード検出
  - 未使用のファイル、エクスポート、依存関係を自動的に検出
  - CI/CD パイプラインに組み込んで継続的に監視

</v-clicks>

</div>
</div>

<!--
三つ目のパターンは「デッドコード放置問題」です。AIに「この機能を別ファイルに移動して」と指示すると、新しいファイルにコードをコピーするだけで、元のコードを削除せずに残してしまうことがよくあります。

AIはコードの削除に対して慎重な傾向があります。移動=コピーと考え、削除は危険な操作と見なす傾向があるように見受けられます。

この問題により、コードベースが不必要に肥大化し、レビュー負荷が増加します。また、残されたデッドコードが新しい開発者を混乱させたり、未使用コードが将来のメンテナンスコストを増加させたりする原因になります。

対策として、knipというツールが非常に効果的です。knipは未使用のファイル、エクスポート、依存関係を自動的に検出し、プロジェクトから削除すべきデッドコードを特定します。

CI/CDパイプラインにknipを組み込むことで、デッドコードの放置を継続的に監視し、コードベースのクリーンな状態を維持することができます。
-->

---

# パターン 3：knip による未使用コード検出

```bash {all}
$ npx knip

Unused files (1)
scripts/check-winner.ts   # 元のファイルが使われていない

Unused exports (3)
scripts/utils.ts:checkWinner      # 新しいファイルに移動された関数

Unused dependencies (2)
package.json:lodash               # インストールしたが使わなかったライブラリ
package.json:dayjs                # 別のライブラリに置き換えたが削除していない
```

<!--
これはknipを実行した際の出力例です。knipはプロジェクト内の未使用コードを分類して表示してくれます。

例えば、「Unused files」セクションでは、scripts/check-winner.tsという別のファイルに移動されたコードが元のファイルとして残っていることが検出されています。

「Unused exports」セクションでは、utilsファイルの中のcheckWinner関数や、userファイルのcalculatePoints関数、apiファイルのfetchData関数など、もはや使われていない関数やメソッドが一覧表示されます。

さらに「Unused dependencies」セクションでは、インストールされたがコード内で使用されていないライブラリも検出します。

このようにknipを活用することで、AIが残したデッドコードを効率的に特定し、削除することができるのです。
-->

---

# パターン 4：テストの妥協（vi.mock 濫用）

<div class="grid grid-cols-[100%] gap-4">
<div>

<v-clicks>

- テストコードを書かせるも失敗し続け、妥協に妥協を重ね最終的に関数を完全にモックに置き換えてテストが通ったことにする

```typescript
// 本来テストしたい関数（日付フォーマットのユーティリティ）
export function formatDate(date: Date, format: string): string {
  const result = ... // 日付を特定のフォーマットに変換する複雑なロジック
  return result
}

// AIが作成したテスト（モック濫用）
describe('formatDate', () => {
  vi.mock('../utils/dateUtils', () => ({
    complexDateFormattingLogic: vi.fn().mockReturnValue('2025-05-19'),
  }))

  it('formats a date correctly', () => {
    const result = formatDate(new Date(), 'YYYY-MM-DD');
    expect(result).toBe('2025-05-19');
  })
});
```

- **対策**：eslint-plugin-vitest で `vi.mock` / `vi.mocked` / `vi.spyOn` の使用を制限
  - モックを選択肢から除外させ、強制的に実装を見直させる
  - テストコード生成は特に人間のチェックが不可欠

</v-clicks>

</div>
</div>

<!--
四つ目のパターンは「テストの妥協（vi.mock濫用）」問題です。AIにテストコードを書かせると、テストが失敗し続けると安易にモックに頼る傾向があります。

この問題の本質は、テストが実装と乖離してしまうことです。モック化されたテストは、実装が変更されてもモックが固定値を返すため常に成功し、「偽陽性」のテストになり、リファクタリング時の保証機能を果たしません。

上の例では、日付フォーマットのユーティリティ関数をテストするために、内部で使用している関数をモックに置き換えています。これにより、formatDate関数の実際のロジックはテストされず、単にモックが返す値が正しいことだけを確認するテストになっています。

AIには「テストを通すこと」を優先し、テストの本来の価値を軽視する傾向があります。対策として、eslint-plugin-vitestを使用してvi.mockの使用を制限し、より適切なテスト設計を促すことが効果的です。

テストコード生成は現状AIが最も苦手とする領域の一つであり、人間による細やかなレビューが特に重要です。
-->

---

# 実運用のコツ

<div class="grid grid-cols-[100%] gap-4">
<div>

<v-clicks>

- **AI 主導のルール整備**: Lint ルール整備こそ AI に任せる
  - ルールの調査：ChatGPT o3 に検索させる「関数の型定義を明示するルールを調べて。できれば Biome, なければ ESLint で」
  - 設定の有効化：Devin に実装 →PR を出させる「useExplicitType ルールを有効にしてください。まずはルールの導入だけしたいため、コードに問題があれば biome-ignore して良いです」
- **warn を避けて error に**
  - AI は Lint エラーが warn だと「今は対応不要」と勝手に判断して無視するので、できるだけ error にする
- **失敗回数制限の設定**
  - 制限を強くしていくと、どう修正しても解決できず混乱することがある。また設定ファイルから無効化して解決しようとすることもある
  - 「3 回解決に失敗したらユーザーに相談して」とカスタムインストラクションに追加すると良い

</v-clicks>

</div>
</div>

---

# まとめ：継続的品質改善のサイクル

<div class="grid grid-cols-[100%] gap-4">
<div>

<v-clicks>

- **Lint ルール = 機械可読な理想状態定義**
  - エンジニアの暗黙知を形式知化
  - AI 自身がルールを理解し、遵守するサイクル構築
- **品質向上文化の醸成**
  - 意図的に Lint ルールを増やし、コード品質を押し上げる
  - AI との協働において、人間の役割は「詳細実装」から「理想の定義」へ

</v-clicks>

</div>
</div>
