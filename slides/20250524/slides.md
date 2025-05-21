---
layout: cover
class: text-center
highlighter: shiki
mdc: true
lineNumbers: false
drawings:
  persist: false
transition: fade-out
title: Valibot Schema Driven UI - ノーコードWebサイトビルダーを実装してみよう！
colorSchema: dark
theme: default
fonts:
  sans: "Lexend Exa, Noto Sans JP"
---

# Valibot Schema Driven UI<br>ノーコード Web サイトビルダーを実装してみよう！

TSKaigi 2025 - 2025/05/24

<div class="abs-br m-6 flex gap-2">
  <button class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    @MH4GF <carbon:logo-github />
  </button>
</div>

---

# 宮城広隆 / @MH4GF

<div flex="~ gap-1" items-center justify-center class="h-[70%]">
<div>

- <div class="flex gap-1 absolute"><a class="inline-flex gap-1" href="https://route06.co.jp/"><img class="w-6 h-6" src="https://avatars.githubusercontent.com/u/62282963?v=4" />ROUTE06, inc.</a></div>
- GraphQL, 静的解析, GitHub Actions が好き
- <div class="flex gap-1 absolute"><a class="inline-flex gap-1" href="https://liambx.com/"><img class="w-6 h-6" src="https://avatars.githubusercontent.com/u/187765721?v=4" />Liam ERD</a> PM 兼テックリード</div>
- [mh4gf.dev](https://mh4gf.dev/) / [X](https://x.com/MH4GF) / [GitHub](https://github.com/MH4GF)

</div>
<img src="https://avatars.githubusercontent.com/u/31152321?v=4" class="rounded-full w-50 h-50 mx-auto">
</div>

---

# ノーコードプラットフォームとは

<v-clicks>

- プログラミングの知識がなくてもアプリケーションを構築できるツール
- 代表例: Bubble、Webflow、STUDIO など
- よくある機能
  - ドラッグ&ドロップでパーツを選び、視覚的に UI を構築
  - ビジュアルエディタでスタイル（色・サイズ・配置）を直感的に調整
  - ワンクリックでデプロイ・公開が可能

</v-clicks>

<!--
ノーコードプラットフォームは、コーディングなしでアプリを作れる画期的なツールです。例えばWebflowなら、普段HTMLやCSSを書いたことがない方でも美しいWebサイトが作れます。

よくある機能を見ていきましょう。まず基本となるのがドラッグ&ドロップのUIです。ボタンやテキスト入力欄などの部品をマウス操作だけで配置できるため、HTMLやCSSを書く必要がありません。

次に、配置した要素のスタイル調整も視覚的に行えます。色やサイズ、配置などをスライダーやカラーピッカーで直感的に設定でき、CSSプロパティの知識がなくても美しいデザインが可能です。

そして完成したら、複雑なデプロイ設定なしにワンクリックで公開できるのも大きな特徴です。
-->

---

# どうやって実現されている？

<v-clicks>

- UI パーツのドラッグ&ドロップが、どのように HTML や CSS に変換されるのか？
- ビジュアルエディタでの変更を、どうやってリアルタイムでプレビューに反映しているのか？
- 多様なスタイリングやレイアウトの組み合わせを、どう効率的に管理しているのか？

</v-clicks>

<!--
これからノーコードプラットフォームの技術的な側面について考えていきましょう。

まず、ドラッグ&ドロップ操作をコードに変換する仕組みはどうなっているでしょうか？
視覚的な操作がどのようにHTMLやCSSに変わるのか、考えたことはありますか？

次に、エディタで色やサイズを変更したとき、その変更がすぐに反映される仕組みは？
リアルタイムでの更新を実現するためには、何が必要でしょうか？

そして、多様なデザインの組み合わせをどう管理しているのでしょう？
無限とも言える組み合わせを効率的に扱うための工夫とは？

こうした疑問に対する解決策が、実はWeb開発全般でも応用できるヒントになります。
-->

---

# 本発表の概要とゴール

<v-clicks>

## 概要

- ノーコードビルダーの UI 構築部分に焦点を当てた実装パターンの紹介
- **Valibot** と **TypeScript** を活用した型安全なアプローチ
- スキーマ駆動による拡張性の高い UI コンポーネント設計

## ゴール

**動的 UI 構築機能の型安全な実装パターンを理解し、日々の開発に活かせるようになる**

</v-clicks>

<!--
今日の発表では、ノーコードプラットフォームのUI構築機能に焦点を当て、その実装パターンを紹介します。

特に、ValibotというTypeScript向けスキーマバリデーションライブラリを使った型安全な実装方法に注目します。これは単にノーコードビルダーだけでなく、動的なUIを扱うあらゆるケースで応用できる設計パターンです。

例えば、ユーザー権限によって表示内容が変わるダッシュボード、設定に応じて形が変わるフォーム、カスタマイズ可能なレポート画面など、皆さんの日常開発でも「動的に変わるUI」は頻繁に登場すると思います。

今日お話しする型安全なスキーマ駆動アプローチを学ぶことで、そうした動的UIを柔軟かつ安全に実装するヒントを持ち帰っていただけると思います。

図に示すように、ノーコードプラットフォームには様々な機能がありますが、今回は赤色で示したUI構築部分を深掘りしていきます。
-->

---

# アジェンダ

<v-clicks>

1. **Valibot Schema Driven UI の基本概念**
2. **デモ**
3. **スキーマ定義とアーキテクチャ**
4. **実装パターン**
5. **拡張パターン**
6. **まとめ**

</v-clicks>

<!--
これから進めていく内容の全体像をご説明します。

まずはデモを通じて、実際に何を作るのかを具体的にイメージしていただきます。ノーコードビルダーの基本的な動作や機能を確認しましょう。

次に、このアプローチの基本となる「Valibot Schema Driven UI」の概念について説明します。TypeScriptとの親和性が高いValibotの特徴や、スキーマ駆動開発のメリットについてお話しします。

続いて、スキーマ定義とアーキテクチャについて掘り下げます。どのようにコンポーネント構造を設計し、型安全性を担保するかを見ていきます。

実装パターンでは、実際のコードレベルで、スキーマからUIコンポーネントへの変換方法や、エディタの実装手法を紹介します。

さらに発展的な内容として、スタイリングの実装やLLMを活用したText-to-UI機能など、拡張パターンもご紹介します。

最後に、今回学んだ内容を日常の開発にどう活かせるかをまとめます。

基礎から応用へと順を追って進めていきますので、ぜひ最後までお付き合いください。
-->


---
layout: center
---

# Valibot Schema Driven UI の基本概念

---

# Valibot とは

<v-clicks>

- TypeScript/JavaScript向けのスキーマバリデーションライブラリ
- **型安全性 × 軽量**
  - スキーマからTypeScript型を自動推論
  - 最小700バイトから組み込み可能
- **豊富な機能**
  - ビルトインアクション（`minLength`, `email`など）
  - パイプラインAPIによる柔軟な合成
  - 非同期バリデーションとi18n対応
- **充実したエコシステム**
  - `h3-valibot`: Nitro/h3ルート向け
  - `conform-to-valibot`: Reactフォーム連携
  - `valibot-codemod`: 移行支援

</v-clicks>

<!--
まず前提知識として、Valibotについて説明します。  
Valibotは、TypeScript/JavaScript向けのスキーマバリデーションライブラリで、型安全性と軽量さを両立させています。

スキーマからTypeScriptの型を自動的に推論でき、実行時のバリデーションも行うことで、型と実データの整合性を保証します。また、モジュラー設計により必要な機能だけをバンドルでき、最小700バイトから組み込み可能です。

機能面では、minLengthやemailなどの豊富なビルトインアクションを提供し、パイプラインAPIによって柔軟なバリデーションの合成が可能です。さらに、非同期バリデーションや国際化（i18n）にも対応しています。

エコシステムも充実しており、Nitro/h3ルート向けのバリデーション、Reactフォームとの連携、移行支援ツールなどが提供されています。
-->

  - **基本的な使い方**  
    ```ts
    import * as v from "valibot";
  
    // スキーマ定義
    const imageSchema = v.object({
      src: v.string(),
      alt: v.optional(v.string()),
    });
  
    // 型推論
    type Image = v.InferOutput<typeof imageSchema>;
    // { src: string; alt?: string; }
  
    // 実行時バリデーション
    const result = v.parse(imageSchema, { src: "https://example.com/image.png" });
    // { src: "https://example.com/image.png", alt: undefined }
    ```
  - **ポイント**  
    - 型安全性とランタイムの安全性を同時に担保  
    - スキーマをそのまま「UI定義データ」としても再利用可能  

7-2. Server-Driven UI

- サーバー側が「どの UI コンポーネントを、どのように配置・表示するか」をクライアントに指示し、クライアントは受け取った指示に従ってレンダリングするアーキテクチャパターン
- Airbnb が2021年に提唱した概念で、Ghostと呼ばれる自社フレームワークを紹介した
- 従来のClient-Driven UIでは、クライアント（Web／iOS／Android）が表示ロジックを実装し、サーバーはデータのみを返していた。しかしこの方式だと、各プラットフォーム間で UI 表示ロジックの重複や乖離が発生しやすく、アプリのアップデートが頻発する問題があった

<!-- 

動的なUIレンダリングを実現するためのアプローチとして、Server-Driven UIという概念があります。これは、サーバー側がUIコンポーネントの配置や表示方法を指示し、クライアントがその指示に従ってレンダリングする仕組みです。

-->

7-3. Server-Driven UIの概念図

![Server-Driven UIの概念図](/2.svg)

7-4. 今回のノーコードプラットフォームでの動的UI構築

- 今回はUIの構成要素を Block と呼び、 Block を組み合わせてページを構成する
- ユーザーはエディタ上でブロックを追加・削除・編集でき、リアルタイムでプレビューできる
- ブロックは各自プロパティを持ち、スタイルも指定できる

→ これらをValibotでスキーマ化し、型安全に実装する

1. **スキーマ＝コンポーネント定義**: ブロックごとのプロパティの型・必須チェックを一元管理  
2. **TypeScript型推論**: UIレンダラーやエディタ側で補完が効く  
3. **実行時バリデーション**: 型だけでなく実行時の検証もできる

7-5. Valibot Schema Driven UIの概念図

WIP

<!-- このスキーマで定義したブロックは、対応する React コンポーネントに変換され UI に反映されます。以下のフローでデータを保存・ UI レンダリングを行います：

1. ユーザーが ブロック（UI コンポーネント）を追加
2. Valibot スキーマに基づいてデータをバリデーション
3. 各スキーマに対応する React コンポーネントが動的に呼び出され、UI に表示

これにより、ユーザーはノーコードで UI を自由に構築できる環境が実現します。 
-->

---

# デモ

<div class="grid grid-cols-2 gap-4">
<div>

![ノーコードビルダーデモ](/1.gif)

</div>
</div>

<!--
今回実装するノーコードWebサイトビルダーのデモをお見せします。

左ペインでブロックを追加・編集・削除し、結果のプレビューがリアルタイムで右ペインに表示されるようなビジュアルエディタを実装しています。

ヘッダーにはプレビュー画面への導線があり、プレビュー画面ではユーザーが作成したアプリケーションだけを閲覧することができます。

この機能を実現するためには、UIコンポーネントをスキーマで表現し、そのスキーマを編集できるインターフェースと、スキーマからUIを生成する仕組みが必要になります。

これからの発表では、この機能を型安全に実装する方法について、詳しく説明していきます。実際のコードを見ながら、スキーマ駆動UIの考え方を理解していきましょう。

なお、これから紹介するコード例では、エディタのスタイリングに関するコードは除外し、本質的なロジックに焦点を当てて説明します。実際のソースコードはGitHubで公開していますので、よろしければ参照してください。
-->


8. **基本アーキテクチャ設計**

8-1. 前提

- 今回はReactを使用
- ソースコード全体は紹介せず、重要な部分を抜粋して説明します。スタイリングも省略します
- ソースコード: https://github.com/MH4GF/valibot-schema-driven-ui

8-2. スキーマ設計

```ts
import * as v from "valibot";

// ブロックの共通情報を持つスキーマ。全てのブロックはIDを持つ
const baseBlockSchema = v.object({ id: v.string() })

// ボタンブロックが持つプロパティを定義
const buttonSchema = v.intersect([
  baseBlockSchema,
  v.object({ type: v.literal("button"), text: string() }),
])

const paragraphSchema = intersect([
  baseBlockSchema,
  object({ type: literal("paragraph"), text: string() }),
]);

const imageSchema = intersect([
  baseBlockSchema,
  object({ type: literal("image"), src: string(), alt: optional(string()) }),
]);

// 全ブロックタイプのユニオン型
const blockSchema = union([buttonSchema, paragraphSchema, imageSchema]);

// ページレイアウトのスキーマ
export const pageSchema = object({
  name: string(),
  blocks: record(string(), blockSchema), // IDによる全ブロックのマップ
});

// Valibotスキーマから型を定義
export type Page = InferOutput<typeof pageSchema>;
export type Block = InferOutput<typeof blockSchema>;
export type Button = InferOutput<typeof buttonSchema>;
export type Paragraph = InferOutput<typeof paragraphSchema>;
export type Image = InferOutput<typeof imageSchema>;
```

8-3. Intersection型とUnion型

- **Intersection型**: 複数のスキーマを組み合わせて新しいスキーマを作成
  - 例: `baseBlockSchema`と`buttonSchema`を組み合わせて、ボタンブロックのスキーマを定義
  - 共通プロパティと固有プロパティを分離して定義
- **Union型**: 複数のスキーマのいずれかにマッチするスキーマを定義
  - 例: `buttonSchema`、`paragraphSchema`、`imageSchema`を組み合わせて、全ブロックタイプのスキーマを定義
  - 異なるブロックタイプを一つのスキーマで扱い、絞り込み(Type Narrowing)を行う

9. **実装パターン**

9-1. **ブロックスキーマからコンポーネントへの変換**

```tsx
import type { FC } from "react";
import type { Button, Paragraph, Image, Block } from "../schema";

const ButtonBlock: FC<{ block: Button }> = ({ block }) => (
  <button type="button">{block.text}</button>
);

const ParagraphBlock: FC<{ block: Paragraph }> = ({ block }) => (
  <p>{block.text}</p>
);

const ImageBlock: FC<{ block: Image }> = ({ block }) => (
  <img src={block.src} alt={block.alt || ""} />
);

const BlockRenderer: FC<{ block: Block }> = ({ block }) => {
  switch (block.type) {
    case "button":
      return <ButtonBlock key={block.id} block={block} />;
    case "paragraph":
      return <ParagraphBlock key={block.id} block={block} />;
    case "image":
      return <ImageBlock key={block.id} block={block} />;
    default:
      return null;
  }
};

const PageRenderer: FC<{ page: Page }> = ({ page }) => {
  return (
    <div>
      <h1>{page.name}</h1>
      {Object.values(page.blocks).map((block) => (
        <BlockRenderer key={block.id} block={block} />
      ))}
    </div>
  );
};
```

<!-- かなりプリミティブなUIを定義した 
switchで分岐し、型の絞り込みを行っている。これにより各ブロックのコンポーネントでは特定のプロパティにアクセスできるようになる
-->

9-2. **エディタの実装**

```tsx
const Editor = () => {
  const [page, setPage] = useState<Page>({
    name: "My Page",
    blocks: {},
  });

  const addBlock = (type: Block["type"]) => {
    switch (type) {
      case "button":
        const id = crypto.randomUUID();
        setPage((prev) => ({
          ...prev,
          blocks: {
            ...prev.blocks,
            [id]: { id: id, type, text: "" },
          },
        }));
        return;

      ... 省略 ...

    }
    }
  };

  return (
    <div>
      <aside>
        <button onClick={() => addBlock("button")}>Add Button</button>
        <button onClick={() => addBlock("paragraph")}>Add Paragraph</button>
        <button onClick={() => addBlock("image")}>Add Image</button>
      </aside>

      <PageRenderer page={page} />
    </div>
  );
};
```

  - 状態管理
  - 双方向バインディング
  - バリデーション処理
- **データの永続化**
  - スキーマと JSON の変換
  - クエリパラメータによる状態保存

10. **拡張パターン**
    - **親子関係の表現**
      - ネストされた UI の扱い方
      - フラットデータでの階層管理
    - **スタイリングの実装**
      - スタイルのスキーマ化
      - 型安全なスタイル管理
    - **JavaScriptの実行(ローコード機能の追加)**
      - スキーマでのイベントハンドラ定義
      - 安全なJavaScript実行環境
    - **LLMを利用したText-to-UI機能**
      - Vercel AI SDKとValibotスキーマを連携
      - チャットUIとStructured Outputによる出力

### 結論

11. **実践と応用**
    - **まとめ: Schema Driven UI の利点**
      - 型安全性
      - 拡張性
      - 開発効率
    - **参考資料と次のステップ**

## プレゼンテーション全体の流れ

1. **導入**: ノーコードプラットフォームの概念と背景
2. **問題提起**: どうやって実現されているのか？という技術的疑問
3. **提案**: Valibot Schema Driven UIによる型安全なアプローチ
4. **実装**: 基本から応用までの具体的な実装パターン
5. **発展**: 拡張機能や応用例の紹介
6. **結論**: 利点のまとめと次のステップ

## 主要なポイント

- **型安全性**: TypeScriptとValibotによる型安全なUI構築
- **拡張性**: スキーマ駆動によるコンポーネント設計の柔軟性
- **実用性**: 実際の開発で応用できる実装パターン
- **最新技術**: LLMとの連携による可能性の広がり


---
layout: center
class: text-center
---

# Thank you for listening!

Slides on [mh4gf.dev](https://talks.mh4gf.dev/20250524)
