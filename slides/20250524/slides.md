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
defaults:
  layout: custom-default
fonts:
  sans: "Lexend Exa, IBM Plex Sans, Noto Sans JP"
---

# Valibot Schema Driven UI<br>ノーコード Web サイトビルダーを実装してみよう！

TSKaigi 2025 - 2025/05/24 @MH4GF

---

# 宮城広隆 / @MH4GF

<div flex="~ gap-1" items-center justify-center class="h-[70%]">
<div>

- <div class="flex gap-1 absolute"><a class="inline-flex gap-1" href="https://route06.co.jp/"><img class="w-6 h-6" src="https://avatars.githubusercontent.com/u/62282963?v=4" />ROUTE06, inc.</a></div>
- 好きなcompiler optionは `erasableSyntaxOnly`
- <div class="flex gap-1 absolute"><a class="inline-flex gap-1" href="https://liambx.com/"><img class="w-6 h-6" src="https://avatars.githubusercontent.com/u/187765721?v=4" />Liam ERD</a> PM 兼テックリード</div>
- [mh4gf.dev](https://mh4gf.dev/) / [X](https://x.com/MH4GF) / [GitHub](https://github.com/MH4GF)

</div>
<img src="https://avatars.githubusercontent.com/u/31152321?v=4" class="rounded-full w-50 h-50 mx-auto">
</div>


---

# ノーコードWebサイトビルダーとは

<v-clicks>

- プログラミングの知識がなくても視覚的にWebサイトを構築できるツール
- 代表例: Bubble、Webflow、STUDIO など
- よくある機能
  - ドラッグ&ドロップでパーツを選び UI を構築
  - ビジュアルエディタでスタイル（色・サイズ・配置）を直感的に調整
  - ワンクリックでデプロイ・公開が可能

</v-clicks>

<!--
ノーコードWebサイトビルダーは、コーディングなしでアプリを作れる画期的なツールです。例えばWebflowなら、普段HTMLやCSSを書いたことがない方でも美しいWebサイトが作れます。

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
これからノーコードWebサイトビルダーの技術的な側面について考えていきましょう。

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

- ノーコードWebサイトビルダーの UI 構築部分に焦点を当てた実装パターンの紹介
  - APIとの連携やデータバインディングなどは話しません
- **Valibot** と **TypeScript** を活用した型安全なアプローチ
- あくまで一例で、全てのノーコードビルダーがこの実装というわけではありません

ゴール: **動的 UI 構築機能の型安全な実装パターンを理解し、日々の開発に活かせるようになる**

</v-clicks>

<!--
今日の発表では、ノーコードWebサイトビルダーのUI構築機能に焦点を当て、その実装パターンを紹介します。

特に、ValibotというTypeScript向けスキーマバリデーションライブラリを使った型安全な実装方法に注目します。これは単にノーコードビルダーだけでなく、動的なUIを扱うあらゆるケースで応用できる設計パターンです。

例えば、ユーザー権限によって表示内容が変わるダッシュボード、設定に応じて形が変わるフォーム、カスタマイズ可能なレポート画面など、皆さんの日常開発でも「動的に変わるUI」は頻繁に登場すると思います。

今日お話しする型安全なスキーマ駆動アプローチを学ぶことで、そうした動的UIを柔軟かつ安全に実装するヒントを持ち帰っていただけると思います。

図に示すように、ノーコードWebサイトビルダーには様々な機能がありますが、今回は赤色で示したUI構築部分を深掘りしていきます。
-->

---

# アジェンダ


1. **デモ**
2. **Valibot Schema Driven UI の基本概念**
3. **スキーマ定義とアーキテクチャ**
4. **基本機能の実装**
5. **拡張パターン**
6. **まとめとおまけ**


<!--
これから進めていく内容の全体像をご説明します。

まずはデモを通じて、実際に何を作るのかを具体的にイメージしていただきます。ノーコードビルダーの基本的な動作や機能を確認しましょう。

次に、このアプローチの基本となる「Valibot Schema Driven UI」の概念について説明します。TypeScriptとの親和性が高いValibotの特徴や、スキーマ駆動開発のメリットについてお話しします。

続いて、スキーマ定義とアーキテクチャについて掘り下げます。どのようにコンポーネント構造を設計し、型安全性を担保するかを見ていきます。

実装パターンでは、実際のコードレベルで、スキーマからUIコンポーネントへの変換方法や、エディタの実装手法を紹介します。

さらに発展的な内容として、スタイリングの実装や生成AIを活用したText-to-UI機能など、拡張パターンもご紹介します。

最後に、今回学んだ内容を日常の開発にどう活かせるかをまとめます。

基礎から応用へと順を追って進めていきますので、ぜひ最後までお付き合いください。
-->

---

# デモ

<!-- <iframe src="http://localhost:3000" class="scaled-iframe"></iframe> -->
<img src="/demo-vsdui.gif" class="h-[89%]" />


<!--
今回実装するノーコードWebサイトビルダーのデモをお見せします。

左ペインでブロックを追加・編集・削除し、結果のプレビューがリアルタイムで右ペインに表示されるようなビジュアルエディタを実装しています。

ヘッダーにはプレビュー画面への導線があり、プレビュー画面ではユーザーが作成したアプリケーションだけを閲覧することができます。

この機能を実現するためには、UIコンポーネントをスキーマで表現し、そのスキーマを編集できるインターフェースと、スキーマからUIを生成する仕組みが必要になります。

これからの発表では、この機能を型安全に実装する方法について、詳しく説明していきます。実際のコードを見ながら、スキーマ駆動UIの考え方を理解していきましょう。

なお、これから紹介するコード例では、エディタのスタイリングに関するコードは除外し、本質的なロジックに焦点を当てて説明します。実際のソースコードはGitHubで公開していますので、よろしければ参照してください。
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
  - バンドルサイズが小さく、最小700バイトから組み込み可能
- **豊富な機能**
  - ビルトインのバリデーション（`v.minLength`, `v.email`など）
  - ビルトインの変換関数(`v.transform`, `v.toLowerCase`など)
  - パイプラインAPIによる柔軟な合成(`v.pipe`)
- **充実したエコシステム**
  - Nest.js, Drizzle ORM, AI SDK, conformなど


</v-clicks>

<img src="/valibot-icon.png" class="absolute top-5 right-12 w-50">

<!--
まず前提知識として、Valibotについて説明します。  
Valibotは、TypeScript/JavaScript向けのスキーマバリデーションライブラリで、型安全性と軽量さを両立させています。

スキーマからTypeScriptの型を自動的に推論でき、実行時のバリデーションも行うことで、型と実データの整合性を保証します。また、モジュラー設計により必要な機能だけをバンドルでき、最小700バイトから組み込み可能です。

機能面では、minLengthやemailなどの豊富なビルトインアクションを提供し、パイプラインAPIによって柔軟なバリデーションの合成が可能です。さらに、非同期バリデーションや国際化（i18n）にも対応しています。

エコシステムも充実しており、Nitro/h3ルート向けのバリデーション、Reactフォームとの連携、移行支援ツールなどが提供されています。
-->

---

# Valibot の基本的な使い方


```ts{all|3-7|9-11|12-15|all}
import * as v from "valibot"

// スキーマ定義
const imageSchema = v.object({
  src: v.string(),
  alt: v.optional(v.string()),
})

// 型推論
type Image = v.InferOutput<typeof imageSchema>
// => { src: string alt?: string }

// 実行時バリデーション
const result = v.parse(imageSchema, { src: "https://example.com/image.png" })
// => { src: "https://example.com/image.png", alt: undefined }
```

---

# Server-Driven UI

- サーバー側が「**どの UI コンポーネントを、どのように配置・表示するか**」をクライアントに指示し、クライアントは受け取った指示に従ってレンダリングするアーキテクチャパターン
- Airbnb が2021年に提唱した概念 [^1] で、**Ghost**と呼ばれる自社フレームワークを紹介した
- 従来のClient-Driven UIでは、クライアント（Web／iOS／Android）が表示ロジックを実装し、サーバーはデータのみを返していた。しかしこの方式だと、各プラットフォーム間で UI 表示ロジックの重複や乖離が発生しやすく、アプリのアップデートが頻発する問題があった

[^1]:[A Deep Dive into Airbnb’s Server-Driven UI System - The Airbnb Tech Blog](https://medium.com/airbnb-engineering/a-deep-dive-into-airbnbs-server-driven-ui-system-842244c5f5)

<!-- 

動的なUIレンダリングを実現するためのアプローチとして、Server-Driven UIという概念があります。これは、サーバー側がUIコンポーネントの配置や表示方法を指示し、クライアントがその指示に従ってレンダリングする仕組みです。

-->

---
layout: image
image: /sdui.png
backgroundSize: contain
---

---

# 今回のノーコードWebサイトビルダーでの動的UI構築

<v-clicks>

- 今回はUIの構成要素を Block と呼び、 Block を組み合わせてページを構成する
- ユーザーはエディタ上でブロックを追加・削除・編集でき、リアルタイムでプレビューできる
- ブロックは各自プロパティを持ち、スタイルも指定できる

→ これらをValibotでスキーマ化し、型安全に実装する

</v-clicks>

---

# Valibot Schema Driven UI

<img src="/vsdui.png" class="object-contain">

<!-- このスキーマで定義したブロックは、対応する React コンポーネントに変換され UI に反映されます。以下のフローでデータを保存・ UI レンダリングを行います：

1. ユーザーが ブロック（UI コンポーネント）を追加
2. Valibot スキーマに基づいてデータをバリデーション
3. 各スキーマに対応する React コンポーネントが動的に呼び出され、UI に表示

これにより、ユーザーはノーコードで UI を自由に構築できる環境が実現します。 
-->

---
layout: center
---

# 基本アーキテクチャ設計

---

# 前提

- 今回はReactとNext.jsを使用
- ソースコード全体は紹介せず、重要な部分のみ抜粋して説明します。スタイリングも省略します
- ソースコード: https://github.com/MH4GF/valibot-schema-driven-ui
- 本発表はテックブログ記事をベースにしています
  - [Valibot Schema Driven UI - ユーザーがノーコードで自由に UI を組み立てられるエディタを Next.js と Valibot で構築する - ROUTE06 Tech Blog](https://tech.route06.co.jp/entry/2024/09/26/122250#fn:2)

---

# Valibotスキーマ設計

```ts{all|1-2|4-17|19-20|all}
// ブロックの共通情報を持つスキーマ。全てのブロックはIDを持つ
const baseBlockSchema = v.object({ id: v.string() })

// 今回はボタン・パラグラフ・画像のブロックを定義
// それぞれのブロックが持つプロパティを定義
const buttonSchema = v.intersect([
  baseBlockSchema,
  v.object({ type: v.literal("button"), text: string() }),
])
const paragraphSchema = v.intersect([
  baseBlockSchema,
  v.object({ type: v.literal("paragraph"), text: v.string() }),
])
const imageSchema = v.intersect([
  baseBlockSchema,
  v.object({ type: v.literal("image"), src: v.string(), alt: v.optional(v.string()) }),
])

// 全ブロックタイプのユニオン型
const blockSchema = v.union([buttonSchema, paragraphSchema, imageSchema])
```

---

# Valibotスキーマ設計

```ts
// ページは複数のブロックを持つ
export const pageSchema = v.object({
  name: v.string(),
  blocks: v.record(v.string(), blockSchema), // IDによる全ブロックのマップ
})

// ValibotスキーマからTypeScript型を定義
export type Page = v.InferOutput<typeof pageSchema>
export type Block = v.InferOutput<typeof blockSchema>
export type Button = v.InferOutput<typeof buttonSchema>
export type Paragraph = v.InferOutput<typeof paragraphSchema>
export type Image = v.InferOutput<typeof imageSchema>
```

---

# Intersection型とUnion型

- **Intersection型**: `v.intersect()` / `type A = B & C`
  - 複数のスキーマを組み合わせて新しいスキーマを作成
  - 例: `baseBlockSchema`と`v.object`を組み合わせて、ボタンブロックのスキーマを定義
  - 共通プロパティと固有プロパティを分離して定義
- **Union型**: `v.union()` / `type A = B | C`
  - 複数のスキーマのいずれかにマッチするスキーマを定義
  - 例: `buttonSchema`、`paragraphSchema`、`imageSchema`を組み合わせて、全ブロックタイプのスキーマを定義
  - 異なるブロックタイプを一つのスキーマで扱い、絞り込み(Type Narrowing)を行う

---
layout: center
---

# 基本機能の実装

---

# エディタの実装

<img src="/layout-1.png" class="object-contain">

---

# エディタの実装

<img src="/layout-2.png" class="object-contain">

---

# エディタの実装 - Editorコンポーネント

<img src="/layout-2.png" class="object-contain absolute top-6 right-6 w-50 z-10">

```tsx{all|2-3|5-11|13-22|all}
const Editor: FC = () => {
  // ページの状態管理
  const [page, setPage] = useState<Page>({ name: "My Page", blocks: {} })
  const [selectedBlockId, setSelectedBlockId] = useState<string | null>(null)
  // ブロックの追加と編集
  const addBlock = (block: Block) => {
    setPage((prev) => ({ ...prev, blocks: { ...prev.blocks, [block.id]: block } }))
  }
  const updateBlock = (block: Block) => {
    setPage((prev) => ({ ...prev, blocks: { ...prev.blocks, [block.id]: block } }))
  }

  return (
    <div>
      <div>
        <AddBlockPanel addBlock={addBlock} />
        <BlockTree page={page} selectedBlockId={selectedBlockId} setSelectedBlockId={setSelectedBlockId} />
      </div>
      <PageRenderer page={page} />
      <BlockFormPanel block={page.blocks[selectedBlockId]} updateBlock={updateBlock} />
    </div>
  )
}
```

---

# エディタの実装 - ブロックのレンダリング

<img src="/layout-3.png" class="object-contain absolute top-6 right-6 w-50 z-10">

- ブロックのオブジェクトを受け取り、プロパティをレンダリングする
- 今回のブロックはかなりプリミティブなUIコンポーネント

```tsx
const ButtonBlock: FC<{ block: Button }> = ({ block }) => (
  <button type="button">{block.text}</button>
)
const ParagraphBlock: FC<{ block: Paragraph }> = ({ block }) => (
  <p>{block.text}</p>
)
const ImageBlock: FC<{ block: Image }> = ({ block }) => (
  <img src={block.src} alt={block.alt || ""} />
)
```

---

# エディタの実装 - PageRenderer

<img src="/layout-3.png" class="object-contain absolute top-6 right-6 w-50 z-10">

- ブロックのtypeで絞り込み、各ブロックのコンポーネントを呼び出す

```tsx
const BlockRenderer: FC<{ block: Block }> = ({ block }) => {
  switch (block.type) {
    case "button":
      return <ButtonBlock key={block.id} block={block} />
    case "paragraph":
      return <ParagraphBlock key={block.id} block={block} />
    case "image":
      return <ImageBlock key={block.id} block={block} />
  }
}

const PageRenderer: FC<{ page: Page }> = ({ page }) => {
  return (
    <div>
      <h1>{page.name}</h1>
      {Object.values(page.blocks).map((block) => (
        <BlockRenderer key={block.id} block={block} />
      ))}
    </div>
  )
}
```

<!-- かなりプリミティブなUIを定義した 
switchで分岐し、型の絞り込みを行っている。これにより各ブロックのコンポーネントでは特定のプロパティにアクセスできるようになる
-->



---

# エディタの実装 - AddBlockPanel

<img src="/layout-4.png" class="object-contain absolute top-6 right-6 w-50 z-10">

- ボタンを押すとブロックを追加できる

```tsx
const AddBlockPanel: FC<{ addBlock: (block: Block) => void }> = ({ addBlock }) => {
  const addButton = () => {
    addBlock({ id: crypto.randomUUID(), type: "button", text: "Click me" })
  }
  const addParagraph = () => {...} // 省略
  const addImage = () => {...} // 省略

  return (
    <aside>
      <button onClick={() => addButton()}>Add Button</button>
      <button onClick={() => addParagraph()}>Add Paragraph</button>
      <button onClick={() => addImage()}>Add Image</button>
    </aside>
  )
}
```

---

# エディタの実装 - BlockFormPanel

<img src="/layout-5.png" class="object-contain absolute top-6 right-6 w-50 z-10">

- ブロックのtypeに応じて、フォームの内容を変える

```tsx{all|3-4|all}
const BlockFormPanel: FC<{ block: Block, updateBlock: (block: Block) => void }> = ({ block, updateBlock }) => {
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target
    const newBlock = v.parse(blockSchema, { ...block, [name]: value })
    updateBlock(newBlock)
  }

  return (
    <div>
      {block.type === "button" && (
        <div>
          <label htmlFor="text">Button text</label>
          <input type="text" name="text" value={block.text} onChange={handleChange} />
        </div>
      )}
      {block.type === "paragraph" && (...省略...)}
      {block.type === "image" && (...省略...)}
    </div>
  )
}
```

---

# データの永続化

<img src="/layout-2.png" class="object-contain absolute top-6 right-6 w-50 z-10">

- 今回はページのデータをURLのクエリパラメータに保存

```tsx{2-13}
const Editor: FC = () => {
  // クエリパラメータからページのデータを読み込む
  const [page, setPage] = useState<Page>(() => {
    const savedData = searchParams.data
    if (savedData) return parse(pageSchema, JSON.parse(decodeURIComponent(savedData)))
      
    return { name: "New Page", blocks: {} }
  })
  // ページのデータをクエリパラメータに保存
  useEffect(() => {
    const queryData = encodeURIComponent(JSON.stringify(page))
    window.history.pushState({}, "", `?data=${queryData}`);
  }, [page])

  const [selectedBlockId, setSelectedBlockId] = useState<string | null>(null)
  const addBlock = (block: Block) => {
    setPage((prev) => ({ ...prev, blocks: { ...prev.blocks, [block.id]: block } }))
  }
  const updateBlock = (block: Block) => {
    setPage((prev) => ({ ...prev, blocks: { ...prev.blocks, [block.id]: block } }))
  }

  return (
    <div>
      <AddBlockPanel addBlock={addBlock} />
      <PageRenderer page={page} />
      <BlockFormPanel block={page.blocks[selectedBlockId]} updateBlock={updateBlock} />
    </div>
  )
}
```

<!-- 今回はクエリパラメータに保存 -->

---
layout: center
---

# 拡張パターン

---

# 拡張パターン

- 親子関係・UIのネストの表現
- スタイリング
- JavaScriptの実行・ローコード機能の追加
- 生成AIを利用したText-to-UI

---
layout: two-cols-header
---

# 親子関係・UIのネストの表現

- 現在のblocksはフラットなデータ構造で、ブロック間の階層構造が表現できない
- `<div><p>Paragraph</p></div>` のようなネストをどのように表現するか？

::left::

<img src="/flat.png" />

::right::

```js
{
  blocks: {
    "yG4_zM": {
      id: "yG4_zM",
      type: "button",
      text: "Click me",
    },
    "10Eylt": {
      id: "10Eylt",
      type: "paragraph",
      text: "Paragraph text",
    },
    "DT2ptl": {
      id: "DT2ptl",
      type: "paragraph",
      text: "Paragraph text",
    },
  }
}
```

---

# 参考: React(JSX)のデータ構造

- JSX をトランスパイルし実行すると、戻り値として右のようなプレーンオブジェクトが返る
- childrenは `ReactNode` 型が入り、0 個ならundefined、1 個なら単体、複数なら配列

<div class="grid grid-cols-2 gap-4">

```tsx
function Greeting({ name }) {
  return (
    <div className="card">
      <h1>Hello, {name}</h1>
    </div>
  )
}

const result = Greeting("World")
console.dir(result, { depth: null })
```

```ts
{
  '$$typeof': Symbol(react.transitional.element),
  type: 'div',
  key: null,
  props: {
    className: 'card',
    children: {
      '$$typeof': Symbol(react.transitional.element),
      type: 'h1',
      key: null,
      props: { children: [ 'Hello, ', 'World' ] },
      _owner: null,
      _store: {}
    }
  },
  _owner: null,
  _store: {}
}
```

</div>

---

# 親子関係・UIのネストの表現

- blockのプロパティとして `children: Block[]` を追加すると良い？

```tsx
const divSchema = v.object({
  type: v.literal("div"),
  children: v.array(blockSchema),
})
```

- 👌 Reactコンポーネントでレンダリングする際の構造に沿う
- ❌ ブロックデータの検索に再帰処理が必要になり、更新・追加・削除が煩雑になる
- ❌ 将来的に並び替え機能を追加する際に、構造を保ちながら位置を更新する際の変更量が多くなる

---

# 親子関係・UIのネストの表現

- baseBlockSchema に `parentId` プロパティを追加し、ブロックが親ブロックを参照できるようにし、どのブロックに所属するかを明示できるようにする 

```ts{all|3|6-10}
const baseBlockSchema = v.object({
  id: v.string(),
  parentId: v.optional(v.string()),
})

// divブロックもこのタイミングで追加
const divBlockSchema = v.intersect([
  baseBlockSchema,
  v.object({ type: v.literal("div") }),
])
```

---

# 親子関係・UIのネストの表現

- レンダリングの前にだけブロックをReactの階層構造に変換する

```tsx
type BlockWithChildren = Block & { children: BlockWithChildren[] }

const buildHierarchy = (blocks: Block[]): BlockWithChildren[] => {
  const map = new Map<string, BlockWithChildren>()

  // 各ブロックをchildrenプロパティ付きで初期化
  blocks.forEach((block) => {
    map.set(block.id, { ...block, children: [] })
  })

  const result: BlockWithChildren[] = []

  // 親子構造を構築
  blocks.forEach((block) => {
    const parentId = block.parentId
    if (parentId && map.has(parentId)) {
      map.get(parentId)!.children.push(map.get(block.id)!)
    } else {
      result.push(map.get(block.id)!)
    }
  })

  return result
}
```

---

# 親子関係・UIのネストの表現


```tsx{all|1-4|12-16}
const DivBlock: FC<{ block: DivBlock, children?: ReactNode }> = ({ block, children }) => (
  <div>{children}</div>
)

export const BlockRenderer: FC<Props> = ({ block }) => {
  switch (block.type) {
    
    // ~~ 他のブロックのレンダリング処理 ~~

    case "div":
      return (
        <DivBlock key={block.id} block={block}>
          {block.children.map((child) => (
            <BlockRenderer key={child.id} block={child} />
          ))}
        </DivBlock>
      )
  }
}
```

---
layout: two-cols-header
---

# 親子関係・UIのネストの表現

::left::

- ブロックの追加時に、選択中のブロックに所属させるようにする

```tsx
const addBlock = (block: Block) => {
  // 選択中だったら設定
  const parentIdObj = selectedBlockId ? { parentId: selectedBlockId } : {} 
  setPage((prev) => ({ 
    ...prev, 
    blocks: { 
      ...prev.blocks, 
      [block.id]: { ...block, ...parentIdObj }
    }
  }))
}
```

- フラットな構造のままなので、更新処理は変更が不要

::right::

<img src="/nesting.gif" />

---

# 親子関係・UIのネストの表現


- 👌 `page.blocks[blockId]` でブロックの検索が可能になり、フラットな構造のままのため更新・追加・削除が容易
- 👌 並び替え機能を追加するとしても、親ブロックの `order` プロパティの値を更新するだけで良い

---
layout: center
---

# スタイリングの実装

---

# スタイリングの実装

<img src="/styling.gif" class="h-[89%]" />

---

# スタイリングの実装

- スタイルをスキーマで定義し、それをブロックのスキーマに追加する

```ts{all|1-5|12}
const stylesSchema = v.object({
  color: v.optional(v.string()),
  backgroundColor: v.optional(v.string()),
  fontSize: v.optional(v.string()),
})

// ベースブロックのスキーマに styles プロパティを追加
const baseBlockSchema = v.object({
  id: v.string(),
  parentId: v.optional(v.string()),
  name: v.string(),
  styles: stylesSchema, // スタイルプロパティの追加
})
```

---

# スタイリングの実装

- 各ブロックのUIコンポーネントでスタイルを適用する

```tsx{all|2}
const ButtonBlock: FC<BlockProps<Button>> = ({ block }) => (
  <button type="button" style={block.styles}>
    {block.text}
  </button>
)
```

---

# スタイリングの実装

- ユーザーが自由にスタイルを設定できる柔軟な仕組みを提供できる
- ただし、セキュリティ面（XSSなど）には注意が必要
- 実際のシステムでは、ユーザー入力のスタイル値をサニタイズ・検証することが重要
  - color や backgroundColor に無効な値や危険なコードが入っていないか検証
  - HTML/CSSの仕様に沿った安全な値のみ許可

<!-- この実装はユーザーが自由にスタイルを設定できる柔軟な仕組みを提供しますが、セキュリティ面に関しては考慮が不足していることに注意してください。実際のシステムに導入する場合は、ユーザーが入力したスタイルデータに対して適切なサニタイズ処理を施し、不正な入力や潜在的な XSS 攻撃を防ぐ対策が必要です。

たとえば、color や backgroundColor に無効な値や危険なコードが入っていないかを検証し、危険な文字列が含まれている場合はフィルタリングすることが重要です。また、HTML や CSS の仕様に沿った安全な値を使用するようにする必要があります。 -->


---
layout: center
---

# JavaScriptの実行(ローコード機能の追加)

---

# JavaScriptの実行(ローコード機能の追加)

<img src="/lowcode.gif" class="h-[89%]" />

---

# JavaScriptの実行(ローコード機能の追加)

- ボタンの onClick イベントに任意の JavaScript アクションを設定できる機能を実装する
- ユーザーがエディタ上で JavaScript コードを記述し、ボタン押下時に実行
- 直接的な JavaScript 実行はセキュリティリスクがあるため、サンドボックス環境で実行する必要がある
- 今回は iframe の sandbox 属性を活用し、外部とのやりとりを制限してリスクを最小限に抑える

<!-- この機能では、ユーザーが簡単にボタンの onClick イベントにアクションを設定できるようにします。具体的には、エディタ上で任意の JavaScript コードを記述し、ボタンがクリックされたときにそのコードが実行される仕組みを実装します。

重要な点は、直接的な JavaScript の実行にはセキュリティリスクが伴うため、サンドボックス環境でコードを実行する必要があることです。今回は iframe を使用し、JavaScript コードを安全に分離された環境で実行する方法を採用します。 iframe 要素の sandbox 属性を活用して外部とのやりとりを制限し、リスクを最小限に抑えます。 -->

---

# JavaScriptの実行(ローコード機能の追加)

```ts{all|6}
const buttonSchema = v.intersect([
  baseBlockSchema,
  v.object({
    type: v.literal("button"),
    text: v.string(),
    onClick: v.string(), // クリック時に実行される JavaScript コード
  }),
])
```

---
layout: two-cols-header
---

# iframe によるサンドボックス化された JavaScript 実行

<style>
  code {
    font-size: 10px!important;
  }
</style>

::left::
```tsx{all|4-13|15-21|all}
export const ButtonBlock: FC<{ block: Button }> = ({ block }) => {
  const iframeRef = useRef<HTMLIFrameElement>(null)

  useEffect(() => {
    const iframeWindow = iframeRef.current?.contentWindow

    if (iframeWindow) {
      iframeWindow.postMessage(
        { type: "LOAD_SCRIPT", script: block.onClick },
        "*"
      )
    }
  }, [iframeRef.current, block.onClick])

  const handleClick = useCallback(() => {
    const iframeWindow = iframeRef.current?.contentWindow
    if (iframeWindow) {
      iframeWindow.postMessage({ type: "EXECUTE_SCRIPT" }, "*")
    }
  }, [])
```

::right::

```tsx{all|9-10|all}
  const srcDoc = `
    <script>
      let script = ''
      window.addEventListener('message', (event) => {
        if (event.data.type === 'LOAD_SCRIPT') {
          script = event.data.script
        } else if (event.data.type === 'EXECUTE_SCRIPT') {
          try {
            const func = new Function(script)
            func()
          } catch (error) {
            console.error('Script execution error:', error)
          }
        }
      })
    </script>
  `
  return (
    ...ButtonBlockの既存実装、省略...
    <iframe ref={iframeRef} sandbox="allow-scripts allow-modals" srcDoc={srcDoc} />
  )
}
```

---

# iframe によるサンドボックス化された JavaScript 実行

- allow-scripts: JavaScript のみを許可し、他の危険な操作（フォーム送信やポップアップの生成など）は一切禁止
- allow-modals: モーダルウィンドウの生成を許可し、ユーザーによる操作を受け付けることができます。具体的には `alert('hello')` のようなコードが実行可能
- 実際のシステムでは JavaScript コードのサニタイズ処理や、CSPの設定も必要

---
layout: center
---

# 生成AIを利用したText-to-UI機能

---

# 生成AIを利用したText-to-UI機能

<img src="/text-to-ui.gif" class="h-[89%]" />

---

# 生成AIを利用したText-to-UI機能

- OpenAIなどのAIモデルでは、JSON Schemaに準拠したレスポンスを生成できるStructured Outputという機能がある
- ValibotはスキーマをJSON Schemaに変換できるため、AIにValibotスキーマに準拠したページを生成させる
- 今回はVercel AI SDK, OpenAI APIを利用

---

# 生成AIを利用したText-to-UI機能

```ts{all|14|15|all}
// api/chat.ts
import { openai } from "@ai-sdk/openai"
import { streamObject } from "ai"
import { pageSchema } from "../../schema"
import { valibotSchema } from "@ai-sdk/valibot"

export const maxDuration = 30

export async function POST(req: Request) {
  const { prompt } = await req.json()

  const result = streamObject({
    model: openai("gpt-4o"),
    schema: valibotSchema(pageSchema),
    system: "You are a UI block generator. Create a complete page layout based on the user's request.",
    prompt,
  })

  return result.toTextStreamResponse()
}
```

---

# 生成AIを利用したText-to-UI機能

```tsx{all|2-5|6-13|all}
export const ChatPopup: FC<{ setPage: (page: Page) => void }> = ({ setPage }) => {
  const { object, error, submit, isLoading } = useObject({
    api: "/api/chat",
    schema: valibotSchema(pageSchema),
  })

  useEffect(() => {
    const result = v.safeParse(pageSchema, object)

    if (result.success) {
      setPage(result.output)
    }
  }, [object, setPage])

  const onSubmit = (e: FormEvent<HTMLFormElement>) => submit({ prompt: e.target.value })

  return (
    <form onSubmit={onSubmit}>
      <input disabled={isLoading} />
      <button type="submit" disabled={isLoading}>Submit</button>
    </form>
  )
}
```

---
layout: center
---

# まとめとおまけ

---

# 本発表のまとめ

- **Valibot Schema Driven UI とは**：UI ブロックを Valibot スキーマで定義し、型安全とバリデーションを活用しながら、スキーマを中心にデータのレンダリングと更新を実装する設計手法
- **基礎**：フォーム入力 → スキーマ変換 → React でコンポーネントをレンダリングする流れを解説し、編集を即時プレビューできるエディタを実装
- **発展**：`parentId` による親子構造の定義、スタイリング・ローコード機能・生成 AI を使った Text-to-UI といった拡張例を紹介

---

# 今回のノーコードWebサイトビルダーの考察

<style>
  ul {
    font-size: 17px;
  }
</style>

- ブロックの単位 === ノーコードの制約の表現
  - プリミティブなブロック(p, button, img): カスタマイズ性👌 学習コスト❌ 複雑度❌
  - 組み合わせ済みブロック(Hero, ProfileCard, List): カスタマイズ性❌ 学習コスト👌 複雑度👌
  - 突き詰めるとDOMの再実装となる
- 生成AIによるText-to-UI
  - v0などのText-to-Codeと違い生成対象はJSON Schema形式のスキーマであり、制約の中で高速にアプリ構築できるのは強み
  - スキーマが複雑化した際に実用か？
  - コーディングエージェントのようなプロンプトチューニング・ツール実装・HITL(Human in the Loop)などが必要
- 拡張アイデア
  - より多くのブロックのサポート
  - テンプレートエンジン・各種パラメータのサポート
  - バックエンド API とのデータ連携
  - ドラッグ&ドロップでの順序変更や、バウンディングボックスでのサイズ変更などのエディタの改善

---
layout: center
---

# Thank you for listening!

Slides on [mh4gf.dev](https://talks.mh4gf.dev/20250524)
