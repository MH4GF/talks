近年 [Bubble](https://bubble.io/) や [Webflow](https://webflow.com/)、日本だと [STUDIO](https://studio.design/ja) などのノーコードプラットフォームが注目を集めています。これらのツールは、プログラミングの知識がなくてもユーザーが思い思いの UI を構築できる機能を提供し、アプリケーション開発の民主化に貢献しています。

このようなノーコードでユーザーが自由に UI を組み立てる仕組みはどのように構築されているのでしょうか。一見複雑に見えますが、基本的な考え方を理解すれば独自のエディタを構築することも不可能ではありません。アーキテクチャの中核となるのは、動的な UI コンポーネントを表現するスキーマ定義と、それを実際の UI 要素に変換する仕組みです。この過程では、データのバリデーション、データ型の絞り込みと UI コンポーネントの動的生成、そしてそれらの適切な配置が重要な役割を果たします。

本記事では、ユーザーがブラウザ上で UI を自由に構築できるビジュアルエディタの基礎的な実装方法について解説します。スキーマ定義に Valibot を、UI レンダリングに React と Next.js を使用して、ステップバイステップで実践的なビジュアルエディタを構築していきます。

現代のソフトウェア開発では、上述のプラットフォームのようなノーコードに特化したサービスだけでなく、SaaS などに部分的にノーコードによるカスタマイズ機能を組み込むケースも増えています。この記事で紹介する実装方法はそのような場面でも応用可能かと思います。

[:contents]

## Valibot によるスキーマ定義

[Valibot](https://valibot.dev/) は、TypeScript で書かれた軽量かつ高速なスキーマ検証ライブラリです。  
**型安全なスキーマ定義**と**実行時のバリデーション**が簡単に実現でき、入力データが期待通りの形式であることを保証できます。さらにスキーマ定義から自動で TypeScript の型を推論できるため、スキーマと型定義の重複を防ぎ、保守性を向上させます。

ノーコードビジュアルエディタでは、ユーザーが画面上で追加する UI コンポーネントをデータとして定義し、そのデータを元に UI を動的にレンダリングすることが鍵となります。Valibot を利用することで、実行時にバリデーションを行いながら、型安全に UI コンポーネントをスキーマとして表現できます。

### ブロックのスキーマ定義

Valibot で UI コンポーネントをスキーマとして定義する例を見てみましょう。  
本記事では UI の構成要素を「ブロック」と呼ぶことにし、まず Button・Paragraph・Image などの基本的なブロックをスキーマとして定義します。  
それぞれのスキーマは React コンポーネントに対応し、UI レンダリングに利用されます。

```typescript
import {
  type InferOutput,
  object,
  string,
  optional,
  literal,
  union,
  intersect,
  record,
} from "valibot";

// ブロックの共通情報を持つスキーマ
const baseBlockSchema = object({
  id: string(),
});

// ボタンブロックが持つプロパティを定義
const buttonBlockSchema = intersect([
  baseBlockSchema,
  object({
    type: literal("button"),
    text: string(),
  }),
]);

const paragraphBlockSchema = intersect([
  baseBlockSchema,
  object({
    type: literal("paragraph"),
    text: string(),
  }),
]);

const imageBlockSchema = intersect([
  baseBlockSchema,
  object({
    type: literal("image"),
    src: string(),
    alt: optional(string()),
  }),
]);

// 全ブロックタイプのユニオン
const blockSchema = union([
  buttonBlockSchema,
  paragraphBlockSchema,
  imageBlockSchema,
]);

// ページレイアウトのスキーマ
export const pageSchema = object({
  name: string(),
  blocks: record(string(), blockSchema), // IDによる全ブロックのマップ
});

// Valibotスキーマから型を推論
export type Page = InferOutput<typeof pageSchema>;
export type Block = InferOutput<typeof blockSchema>;
```

`union()` を利用し、TypeScript のタグ付きユニオン型[^1]として定義していることに注目してください。これにより `type` フィールドを用いた型安全なブロックの識別が可能となります。

### スキーマを使った動的な UI レンダリング

このスキーマで定義したブロックは、対応する React コンポーネントに変換され UI に反映されます。以下のフローでデータを保存・ UI レンダリングを行います：

1. ユーザーが ブロック（UI コンポーネント）を追加
2. Valibot スキーマに基づいてデータをバリデーション
3. 各スキーマに対応する React コンポーネントが動的に呼び出され、UI に表示

これにより、ユーザーはノーコードで UI を自由に構築できる環境が実現します。

[f:id:route06:20240926124604p:plain]

この方法を、筆者は「**Valibot Schema Driven UI**」と呼んでいます。これは、Server-Driven UI の概念からインスパイアされたものです。
Server-Driven UI は Airbnb が提唱した概念[^2]で、サーバー側で UI 構造とデータを提供することにより、モバイルや Web など全プラットフォームで一貫性を保ちながら開発効率を向上させることを目的としています。Valibot Schema Driven UI は Server-Driven UI の実装の一つともいえますが、サーバーに限らず様々な環境で動作し、かつデータの保存時にも同じスキーマを用いた検証が可能となる点がメリットです。

データの入出力はスキーマに準拠していれば良いため、永続化層としてバックエンド API から受け取ったデータを用いたり、クエリパラメータや IndexedDB に保存したりなど、どのようなデータソースであっても変換層を用意すれば連携可能となります。
出力に関しても、今回の例では React コンポーネントを生成しますが、適切な変換処理さえ実装すれば Server Driven UI 同様にあらゆるプラットフォームで利用可能となります。

次のセクションからは、上述した Valibot スキーマを利用したノーコードビジュアルエディタの実装を紹介していきます。

## デモ

今回の記事で構築するノーコードビジュアルエディタのデモがこちらです。

[f:id:MH4GF:20240925121133g:plain]

[https://codesandbox.io/p/devbox/hopeful-platform-8427x2]



左ペインでブロックを追加・編集・削除し、結果のプレビューがリアルタイムで右ペインに表示されるようなビジュアルエディタを実装します。
ヘッダーにはプレビュー画面への導線があり、プレビュー画面ではユーザーが作成したアプリケーションだけを閲覧することができます。

## プロジェクトのセットアップ

今回の記事では、Next.js、Valibot、TypeScript、Tailwind CSS、nanoid、 lucide-react を使用します。まず、Next.js のプロジェクトをセットアップしましょう。

```sh
npx create-next-app@latest no-code-ui-buider --typescript --tailwind
cd no-code-ui-buider
npm install valibot nanoid lucide-react
```

今回は簡易的なノーコード UI 構築アプリケーションとして、以下の 2 つのページを作成します：

- `/`: リアルタイムでレンダリングの変更を確認できるエディタページ
- `/preview`: 最終的なレンダリングのみを確認できるページ

また、本記事ではロジックに焦点を当てるためサンプルコードからスタイリングは取り除いていますが、スクリーンショットとしてはスタイリングを施したものを掲載しています。実際のソースコードについては上述した codesandbox からご覧ください。

## スキーマ定義の実装

まず `src/app/schema.ts` にブロックとページの Valibot スキーマを定義します。上述したものと近いですが、個別のブロック型の定義や、ブロックのオブジェクトを作成する関数の追加など、少しだけ拡張を加えています。

```typescript
import { type InferOutput, object, string, optional, literal, union, intersect, record } from 'valibot';

// ブロックの共通情報を持つスキーマ
const baseBlockSchema = object({
  id: string(),
});

const buttonBlockSchema = intersect([
  baseBlockSchema,
  object({
    type: literal('button'),
    text: string(),
  }),
]);

const paragraphBlockSchema = intersect([
  baseBlockSchema,
  object({
    type: literal('paragraph'),
    text: string(),
  }),
]);

const imageBlockSchema = intersect([
  baseBlockSchema,
  object({
    type: literal('image'),
    src: string(),
    alt: optional(string()),
  }),
]);

// 全ブロックタイプのユニオン
export const blockSchema = union([buttonBlockSchema, paragraphBlockSchema, imageBlockSchema]);

// ページレイアウトのスキーマ
export const pageSchema = object({
  name: string(),
  blocks: record(string(), blockSchema), // IDによる全ブロックのマップ
});

// Valibotスキーマから型を推論
export type Page = InferOutput<typeof pageSchema>;
export type Block = InferOutput<typeof blockSchema>;
export type BlockType = Block['type'];
export type Button = InferOutput<typeof buttonBlockSchema>;
export type Paragraph = InferOutput<typeof paragraphBlockSchema>;
export type Image = InferOutput<typeof imageBlockSchema>;

const newButton = (id: string): Button => ({
  id,
  type: 'button',
  text: 'Click me',
})

const newParagraph = (id: string): Paragraph => ({
  id,
  type: 'paragraph',
  text: 'Paragraph text',
})

const newImage = (id: string): Image => ({
  id,
  type: 'image',
  src: 'https://via.placeholder.com/150',
  alt: '',
})

export const newBlock = (type: BlockType, id: string): Block => {
  switch (type) {
    case 'button':
      return newButton(id, parentId);
    case 'paragraph':
      return newParagraph(id, parentId);
    case 'image':
      return newImage(id, parentId);
  }
}
```

## エディタページの実装

まず、 エディタページとしてリアルタイムでレンダリングの変更を確認できるページを実装していきます。以下の 3 ステップで実装していきます。

- スキーマから UI をレンダリングする `BlockRenderer` コンポーネント
- ブロックの情報を編集する `BlockForm` コンポーネント
- それらを組み合わせる `Editor` コンポーネント

### Step 1: BlockRenderer コンポーネントの実装

`src/app/_components/BlockRenderer.tsx` ファイルを作成し、以下のコードを実装します：

```typescript
import type { FC, PropsWithChildren } from "react";
import type { Button, Paragraph, Image } from "../schema";

interface BlockProps<T extends Block> {
  block: T;
}

const ButtonBlock: FC<BlockProps<Button>> = ({ block }) => (
  <button type="button">{block.text}</button>
);

const ParagraphBlock: FC<BlockProps<Paragraph>> = ({ block }) => (
  <p>{block.text}</p>
);

const ImageBlock: FC<BlockProps<Image>> = ({ block }) => (
  <img src={block.src} alt={block.alt || ""} />
);

interface Props {
  block: Block;
}

export const BlockRenderer: FC<Props> = ({ block }) => {
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
```

この `BlockRenderer` コンポーネントは、各ブロックタイプに応じて適切な HTML 要素をレンダリングします。
Valibot によるスキーマをベースとしているため、Paragraph ブロックでは text プロパティ、Image ブロックでは src や alt プロパティなど、各ブロック固有のプロパティは型付けされています。

### Step 2: BlockForm コンポーネントの実装

次に、`src/app/_components/BlockForm.tsx` ファイルを作成し、以下のコードを実装します：

```typescript
import type { ChangeEvent, FC } from "react";
import type { Block } from "../schema";

interface Props {
  block: Block;
  onUpdate: (updates: Partial<Block>) => void;
  onDelete: () => void;
}

export const BlockForm: FC<Props> = ({ block, onUpdate, onDelete }) => {
  const handleChange = (
    e: ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    onUpdate({ [e.target.name]: e.target.value });
  };

  return (
    <div>
      <h3>Edit {block.type}</h3>
      {block.type === "button" && (
        <div>
          <label htmlFor="text">Button text</label>
          <input
            type="text"
            name="text"
            value={block.text}
            onChange={handleChange}
            placeholder="Button text"
          />
        </div>
      )}
      {block.type === "paragraph" && (
        <div>
          <label htmlFor="text">Paragraph text</label>
          <textarea
            name="text"
            value={block.text}
            onChange={handleChange}
            placeholder="Paragraph text"
          />
        </div>
      )}
      {block.type === "image" && (
        <>
          <div>
            <label htmlFor="src">Image URL</label>
            <input
              type="text"
              name="src"
              value={block.src}
              onChange={handleChange}
              placeholder="Image URL"
            />
          </div>
          <div>
            <label htmlFor="alt">Alt text</label>
            <input
              type="text"
              name="alt"
              value={block.alt || ""}
              onChange={handleChange}
              placeholder="Alt text"
            />
          </div>
        </>
      )}
      <button type="button" onClick={onDelete}>
        Delete Block
      </button>
    </div>
  );
};
```

この `BlockForm` コンポーネントは、選択されたブロックのプロパティを編集するためのフォームを提供します。ブロックタイプに応じて適切な入力フィールドを表示し、変更があった場合に `onUpdate` コールバックを呼び出します。

### Step 3: Editor の実装

最後に、`src/app/page.tsx` ファイルを作成し、以下のコードを実装します：

```typescript
"use client";

import { useState, useEffect, useMemo } from "react";
import { nanoid } from "nanoid";
import { parse } from "valibot";
import {
  type Page,
  type Block,
  type BlockType,
  pageSchema,
  blockSchema,
  newBlock,
} from "./schema";
import { BlockForm } from "./_components/BlockForm";
import { BlockRenderer } from "./_components/BlockRenderer";

export default function Editor({
  searchParams,
}: {
  searchParams: Record<string, string>;
}) {
  // 選択中のブロックIDを管理
  const [selectedBlockId, setSelectedBlockId] = useState<string | undefined>(undefined);

  // ページの状態を管理
  const [page, setPage] = useState<Page>(() => {
    const savedData = searchParams.data;
    if (savedData) {
      return parse(pageSchema, JSON.parse(decodeURIComponent(savedData)));
    }
    return { name: "New Page", blocks: {} };
  });

  // クエリパラメータに保存するデータ
  const queryData = useMemo(
    () => encodeURIComponent(JSON.stringify(page)),
    [page]
  );

  // ページデータをクエリパラメータに保存
  useEffect(() => {
    window.history.replaceState({}, "", `?data=${queryData}`);
  }, [queryData]);

  // ブロックを追加し選択状態にする
  const addBlock = (type: BlockType, parentId?: string) => {
    const id = nanoid();
    setPage((prevPage) => ({
      ...prevPage,
      blocks: {
        ...prevPage.blocks,
        [id]: newBlock(type, id),
      },
    }));
    setSelectedBlockId(id);
  };

  // ブロックを更新
  const updateBlock = (id: string, updates: Partial<Block>) => {
    setPage((prevPage) => ({
      ...prevPage,
      blocks: {
        ...prevPage.blocks,
        [id]: parse(blockSchema, { ...prevPage.blocks[id], ...updates }),
      },
    }));
  };

  // ブロックを削除
  const deleteBlock = (id: string) => {
    setPage((prevPage) => {
      const { [id]: _, ...remainingBlocks } = prevPage.blocks;
      return { ...prevPage, blocks: remainingBlocks };
    });
    setSelectedBlockId(undefined);
  };

  // ブロックツリーをレンダリング
  const renderBlockTree = (blocks: Block[], depth = 0) => {
    return (
      <ul className={depth > 0 ? "ml-4" : ""}>
        {blocks.map((block) => (
          <li key={block.id} className="my-1">
            <button
              type="button"
              onClick={() => setSelectedBlockId(block.id)}
              className={
                selectedBlockId === block.id
                  ? "bg-gray-600 text-white"
                  : "hover:bg-gray-700"
              }
            >
              {block.type} - {block.id.slice(0, 6)}
            </button>
          </li>
        ))}
      </ul>
    );
  };

  return (
    <div>
      <div>
        <aside>
          <nav>
            <h2>Add Blocks</h2>
            <ul>
              {(["button", "paragraph", "image"] as const).map((type) => (
                <li key={type}>
                  <button
                    type="button"
                    onClick={() => addBlock(type, selectedBlockId)}
                  >
                    Add {type}
                  </button>
                </li>
              ))}
            </ul>
            <h2>Block Tree</h2>
            {renderBlockTree(Object.values(page.blocks))}
          </nav>
        </aside>

        <main>
          <article>
            {Object.values(page.blocks).map((block) => (
              <div key={block.id}>
                <BlockRenderer block={block} />
              </div>
            ))}
          </article>
        </main>

        <aside>
          {selectedBlockId && (
            <BlockForm
              block={page.blocks[selectedBlockId]}
              onUpdate={(updates) => updateBlock(selectedBlockId, updates)}
              onDelete={() => deleteBlock(selectedBlockId)}
            />
          )}
        </aside>
      </div>
    </div>
  );
}

```

この `Editor` コンポーネントは、現在のページの状態を useState で持ち、 `BlockRenderer` でブロックのレンダリングを、 `BlockForm` で選択中のブロックの編集・削除を行います。

ページの状態は URL のクエリパラメータに JSON 文字列としても保存し、Next.js のルーティングから props 経由でクエリパラメータを取得します。これにより、ページの再読み込み時にも状態を復元できます。

以上がエディタページの実装の詳細です。これらの実装により、ユーザーはノーコードで動的に UI を構築し、リアルタイムでプレビューを確認することができるようになりました！

[f:id:MH4GF:20240924190954g:plain]

## `/preview` ページの実装

`src/app/preview/page.tsx` に、プレビューページの実装を行います。

```typescript
import { parse } from "valibot";
import { pageSchema } from "../schema";
import { BlockRenderer } from "../_components/BlockRenderer";

export default function Preview({
  searchParams,
}: {
  searchParams: Record<string, string>;
}) {
  const data = searchParams.data;

  if (!data) {
    return <div>No data provided</div>;
  }

  const page = parse(pageSchema, JSON.parse(decodeURIComponent(data)));
  return (
    <main>
      {Object.values(page.blocks).map((block) => (
        <BlockRenderer key={block.id} block={block} />
      ))}
    </main>
  );
}
```

このコードでは、クエリパラメータからページ情報を取り出し、BlockRenderer コンポーネントを使用して UI をレンダリングしています。

これで、ユーザーが構築した UI だけをレンダリングするページも実装できました！

ここまでで、ノーコードでの動的な UI 構築機能の最も基礎的な部分が実装できました。ここからは実践的な例をいくつか紹介します。

## 実践例 1: 親子構造の表現

現在の実装では、各ブロックはフラットな構造で管理されています。つまり、ブロック同士の階層関係を持たせることができず、すべてがトップレベルに配置されています。React では通常 `children` プロパティを使用して親子構造を表現しますが、現状の実装ではそれができない状態です。

親子構造を表現するために、新たに `DivisionBlock` というブロックタイプを導入します。これは HTML での `div` 要素に相当するコンテナブロックで、他のブロックを内部に含む役割を果たします。ただし、`children` プロパティを直接持たせるのではなく、各ブロックが `parentId` を持つ形で親子関係を表現します。この理由については後ほど説明します。

### スキーマ定義の変更

`baseBlockSchema` に `parentId` プロパティを追加し、ブロックが親ブロックを参照できるようにします。これにより、各ブロックがどのブロックに属しているかを明確に定義できます。

```typescript
const baseBlockSchema = object({
  id: string(),
  parentId: optional(string()), // 親ブロックのIDを参照
});

// ~~ その他のブロックのスキーマ定義 ~~

// DivisionBlockを追加
const divisionBlockSchema = intersect([
  baseBlockSchema,
  object({
    type: literal("division"),
  }),
]);

// BlockのユニオンにDivisionBlockを追加
export const blockSchema = union([
  buttonBlockSchema,
  paragraphBlockSchema,
  imageBlockSchema,
  divisionBlockSchema,
]);

// 親になれるブロックは DivisionBlock のみ
export const parentableBlockSchema = union([divisionBlockSchema]);
```

### データ構造の変換とレンダリング

`BlockWithChildren` 型を新たに定義し、親子関係を持つブロックを適切に管理します。`parentId` をもとに、フラットなブロック構造を再帰的に変換して親子関係を表現できるようにします。

```typescript
type BlockWithChildren = Block & {
  children: BlockWithChildren[];
};

function buildHierarchy(blocks: Block[]): BlockWithChildren[] {
  const map = new Map<string, BlockWithChildren>();

  // 各ブロックをchildrenプロパティ付きで初期化
  blocks.forEach((block) => {
    map.set(block.id, { ...block, children: [] });
  });

  const result: BlockWithChildren[] = [];

  // 親子構造を構築
  blocks.forEach((block) => {
    const parentId = block.parentId;
    if (parentId && map.has(parentId)) {
      map.get(parentId)!.children.push(map.get(block.id)!);
    } else {
      result.push(map.get(block.id)!);
    }
  });

  return result;
}
```

### レンダリングの変更

`BlockRenderer` コンポーネントでは `children` を再帰的にレンダリングするように変更します。これにより、親ブロック内に子ブロックを含む構造を自然に表現できます。

```typescript
const DivisionBlock: FC<BlockProps<DivisionBlock>> = ({ block, children }) => (
  <div>{children}</div>
);

export const BlockRenderer: FC<Props> = ({ block }) => {
  switch (block.type) {
    
    // ~~ 他のブロックのレンダリング処理 ~~

    case "division":
      return (
        <DivisionBlock key={block.id} block={block}>
          {block.children.map((child) => (
            <BlockRenderer key={child.id} block={child} />
          ))}
        </DivisionBlock>
      );
  }
};
```

### ブロックの追加処理の変更

選択中のブロックがある場合、その子階層にブロックを追加するようにします。parentId を追加する形で調整します。  
子ブロックを持てるのは `parentableBlockSchema` でDivisionBlockのみとしているため、親ブロックが DivisionBlock であるかどうかを判定し、適切な parentId を設定します。

```typescript
const addBlock = (type: BlockType, parentId?: string) => {
  let validParentId = undefined;
  if (parentId) {
    const parentBlock = page.blocks[parentId];
    const isParentable = safeParse(
      parentableBlockSchema,
      parentBlock
    ).success;
    if (isParentable) {
      validParentId = parentId;
    }
  }

  const id = nanoid();
  setPage((prevPage) => ({
    ...prevPage,
    blocks: {
      ...prevPage.blocks,
      [id]: newBlock(type, id, validParentId),
    },
  }));
  setSelectedBlockId(id);
};
```

親子構造の概念を追加しても、ブロックの更新処理を変更する必要はありません。これは全てのブロックがフラットな key/value で管理されており、 `page.blocks[blockId]`で取り出すことができるためです。以下のまま変更する必要はありません。  

```typescript
const updateBlock = (id: string, updates: Partial<Block>) => {
  setPage((prevPage) => ({
    ...prevPage,
    blocks: {
      ...prevPage.blocks,
      [id]: parse(blockSchema, { ...prevPage.blocks[id], ...updates }),
    },
  }));
};
```

別の実装方法として、例えば React の children と似た構造で block に children プロパティを持たせていた場合、ブロックの検索のためには再帰処理が多くの箇所で必要となってしまいます。  
今回は`parentId` をブロックに持たせレンダリングの時に children を計算するようにしたため、ブロックの検索・編集処理では実行コストを減らしシンプルに実装ができるようになりました。

これで、親子構造を持つブロックを追加できるようになりました。これにより、より複雑な UI を構築する際にも柔軟に対応できるようになります。

[f:id:MH4GF:20240924185009g:plain]

## 実践例 2: スタイリングの実現

次に、動的な UI 構築において重要な要素である「スタイリング」の実装に進みます。  
ユーザーがノーコードで自由に UI を組み立てる際、テキストの色や背景色、フォントサイズなど、見た目のカスタマイズが求められるのは言うまでもありません。Valibot を用いたスキーマ定義にスタイリング要素を加えることで、簡単にカスタマイズ可能な UI エディタを実現できます。

### スタイリング用プロパティの追加

まず、ブロックにスタイル情報を保持する `styles` プロパティを追加しましょう。このプロパティは、CSS の基本的なスタイル設定を格納するオブジェクトとして定義します。

```typescript
const stylesSchema = object({
  color: optional(string()),
  backgroundColor: optional(string()),
  fontSize: optional(string()),
});

// ベースブロックのスキーマに styles プロパティを追加
const baseBlockSchema = object({
  id: string(),
  parentId: optional(string()),
  name: string(),
  styles: stylesSchema, // スタイルプロパティの追加
});
```

baseBlockSchema に `styles` プロパティを追加しました。今回の例ではシンプルな 3 つのプロパティ（`color`、`backgroundColor`、`fontSize`）を扱います。  
今回は共通のスタイリングとして追加しましたが、それぞれのブロックの `styles` にブロック固有のプロパティを追加することも可能です。

### スタイリングの反映

次に、スタイル情報を UI に反映させるため、`BlockRenderer` コンポーネントにて各ブロックの `styles` プロパティを適用するように変更します。

```typescript
const ButtonBlock: FC<BlockProps<Button>> = ({ block }) => (
  <button type="button" style={block.styles}>
    {block.text}
  </button>
);

const ParagraphBlock: FC<BlockProps<Paragraph>> = ({ block }) => (
  <p style={block.styles}>{block.text}</p>
);

const ImageBlock: FC<BlockProps<Image>> = ({ block }) => (
  <img src={block.src} alt={block.alt || ""} style={block.styles} />
);
```

ここでは、`block.styles` を `style` 属性に直接渡すことで、ユーザーが設定したスタイリングがそのまま UI に反映されるようにします。

### スタイリング編集フォームの追加

エディタ上でスタイルを編集できるように、`BlockForm` コンポーネントにスタイリングの入力フィールドを追加します。ユーザーは色やサイズを自由に設定でき、リアルタイムで UI に反映される仕組みを実装します。

```typescript
const BlockForm: FC<Props> = ({ block, onUpdate, onDelete }) => {
  const handleChange = (
    e: ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const { name, value } = e.target;

    // styles 内のプロパティを更新する場合
    if (name.startsWith("styles.")) {
      const styleProp = name.split(".")[1];
      onUpdate({ styles: { ...block.styles, [styleProp]: value } });
    } else {
      // その他のプロパティを更新する場合
      onUpdate({ [name]: value });
    }
    // 本来はstyles以外のネストされたプロパティも更新できるように汎用的な処理を実装するべきだが、今回は省略
  };

  return (
    <div>
      <h3>Edit {block.type}</h3>

      {/* ~~ ブロックタイプごとのフィールド ~~ */}

      {/* スタイリング用フィールド */}
      <h4>Styles</h4>
      <div>
        <label htmlFor="color">Text Color</label>
        <input
          type="text"
          name="styles.color"
          value={block.styles?.color || ""}
          onChange={handleChange}
          placeholder="e.g., #000000"
        />
      </div>
      <div>
        <label htmlFor="backgroundColor">Background Color</label>
        <input
          type="text"
          name="styles.backgroundColor"
          value={block.styles?.backgroundColor || ""}
          onChange={handleChange}
          placeholder="e.g., #ffffff"
        />
      </div>
      <div>
        <label htmlFor="fontSize">Font Size</label>
        <input
          type="text"
          name="styles.fontSize"
          value={block.styles?.fontSize || ""}
          onChange={handleChange}
          placeholder="e.g., 16px"
        />
      </div>
    </div>
  );
};
```

このフォームでは、スタイルに関するフィールド（`color`、`backgroundColor`、`fontSize`）を追加し、それぞれの値を編集できるようにしました。入力された値はリアルタイムで `onUpdate` コールバックを通じて反映され、`BlockRenderer` が即座に更新されます。

[f:id:MH4GF:20240924185555g:plain]

### セキュリティの考慮

この実装はユーザーが自由にスタイルを設定できる柔軟な仕組みを提供しますが、セキュリティ面に関しては考慮が不足していることに注意してください。実際のシステムに導入する場合は、ユーザーが入力したスタイルデータに対して適切な**サニタイズ処理**を施し、不正な入力や潜在的な XSS 攻撃を防ぐ対策が必要です。

たとえば、`color` や `backgroundColor` に無効な値や危険なコードが入っていないかを検証し、危険な文字列が含まれている場合はフィルタリングすることが重要です。また、HTML や CSS の仕様に沿った安全な値を使用するようにする必要があります。

## 実践例 3: Button の onClick でのローコード機能の実現

最後に、より発展的な機能として、ボタンをクリックした際に任意の JavaScript コードを実行できるローコード機能を実装します。いくつかの実装方法はありますが、ここでは `iframe` を使ったサンドボックス環境で JavaScript コードを安全に実行する方法を採用します。

### JavaScript 実行の基本的な考え方

この機能では、ユーザーが簡単にボタンの `onClick` イベントにアクションを設定できるようにします。具体的には、エディタ上で任意の JavaScript コードを記述し、ボタンがクリックされたときにそのコードが実行される仕組みを実装します。

重要な点は、直接的な JavaScript の実行にはセキュリティリスクが伴うため、**サンドボックス環境でコードを実行**する必要があることです。今回は `iframe` を使用し、JavaScript コードを安全に分離された環境で実行する方法を採用します。 `iframe` 要素の `sandbox` 属性を活用して外部とのやりとりを制限し、リスクを最小限に抑えます。

### スキーマに `onClick` プロパティを追加

まず、ボタンブロックのスキーマに `onClick` プロパティを追加します。このプロパティにユーザーが記述した JavaScript コードを格納し、サンドボックス内で実行します。

```typescript
const buttonBlockSchema = intersect([
  baseBlockSchema,
  object({
    type: literal("button"),
    text: string(),
    onClick: string(), // クリック時に実行される JavaScript コード
  }),
]);
```

この `onClick` プロパティに格納された JavaScript コードが、ボタンのクリック時に `iframe` 内で実行される形となります。

### `iframe` によるサンドボックス化された JavaScript 実行

`iframe` を利用してユーザーの JavaScript コードをサンドボックス内で実行します。以下のコードでは、`iframe` 内にユーザーのスクリプトを埋め込み、ボタンがクリックされたときに実行される仕組みを構築します。

```typescript
import { useRef } from "react";

export const ButtonBlock: FC<BlockProps<Button>> = ({ block }) => {
  const iframeRef = useRef<HTMLIFrameElement>(null);

  useEffect(() => {
    const iframeWindow = iframeRef.current?.contentWindow;

    if (iframeWindow) {
      iframeWindow.postMessage(
        { type: "LOAD_SCRIPT", script: block.onClick },
        "*"
      );
    }
  }, [iframeRef.current, block.onClick]);

  const handleClick = useCallback(() => {
    const iframeWindow = iframeRef.current?.contentWindow;
    if (iframeWindow) {
      iframeWindow.postMessage({ type: "EXECUTE_SCRIPT" }, "*");
    }
  }, []);

  return (
    <div>
      <button type="button" style={block.styles} onClick={handleClick}>
        {block.text}
      </button>
      {/* サンドボックス用のiframe */}
      <iframe
        ref={iframeRef}
        sandbox="allow-scripts allow-modals"
        style={{ display: "none" }}
        title="sandbox"
        srcDoc={`
          <script>
            let script = '';
            window.addEventListener('message', (event) => {
              if (event.data.type === 'LOAD_SCRIPT') {
                script = event.data.script;
              } else if (event.data.type === 'EXECUTE_SCRIPT') {
                try {
                  const func = new Function(script)
                  func();
                } catch (error) {
                  console.error('Script execution error:', error);
                }
              }
            });
          </script>
        `}
      ></iframe>
    </div>
  );
};
```

この `iframe` は、JavaScript の実行だけを許可するために `sandbox="allow-scripts allow-modals"` 属性を持っています。この方法により、ボタンがクリックされた際に `onClick` プロパティに保存された JavaScript コードがサンドボックス環境で安全に実行されます。

### エディタでの `onClick` 設定フォームの追加

次に、ユーザーが `onClick` イベントに任意の JavaScript コードを設定できるように、`BlockForm` コンポーネントに入力フォームを追加します。これにより、ユーザーはエディタ上で JavaScript コードを記述し、それを `iframe` 内で実行できるようになります。

```typescript
const BlockForm: FC<Props> = ({ block, onUpdate, onDelete }) => {
  return (
    <div>
      <h3>Edit {block.type}</h3>
      {block.type === "button" && (
        <div>
          <label htmlFor="text">Button text</label>
          <input
            type="text"
            name="text"
            value={block.text}
            onChange={handleChange}
            placeholder="Button text"
          />
          <label htmlFor="onClick">onClick JavaScript</label>
          <textarea
            name="onClick"
            value={block.onClick || ""}
            onChange={handleChange}
            placeholder="JavaScript code"
          />
        </div>
      )}
      {/* その他のフォームフィールド */}
    </div>
  );
};
```

このフォームでは、ユーザーが `onClick` に JavaScript コードを記述できるようにしています。入力されたコードはリアルタイムで更新され、ボタンがクリックされた際に `iframe` 内で実行されます。

[f:id:MH4GF:20240924185126g:plain]

### セキュリティの考慮

今回の実装では、ユーザーの入力に基づくコードが `iframe` 内で実行されるため、**外部からの不正な操作を防ぐためのサンドボックス**機能を利用しています。`iframe` の `sandbox` 属性を使い、以下のような制限を加えることでセキュリティを確保しています：

- `allow-scripts`: JavaScript のみを許可し、他の危険な操作（フォーム送信やポップアップの生成など）は一切禁止します。
- `allow-modals`: モーダルウィンドウの生成を許可し、ユーザーによる操作を受け付けることができます。具体的には `alert('hello')` のようなコードが実行できます。
- `iframe` 内のコードは外部のドキュメントや API にアクセスできないため、意図しない操作や情報漏洩のリスクが低減されます。

しかしこれだけでは不十分であり、実際のシステムでは JavaScript コードのサニタイズ処理も考慮する必要があります。例えば JavaScript コードに不正なインジェクションが含まれていないかを事前にチェックし、安全な範囲内でのみ実行を許可する仕組みを導入することが重要です。

## まとめ

本記事では、**Valibot Schema Driven UI** のコンセプトに基づき、ユーザーがノーコードで自由に UI を構築できるエディタを Next.js と Valibot を使って実装する方法を紹介しました。ノーコードで動的な UI 構築を行うための基本的なスキーマ定義や、リアルタイムでプレビュー可能なエディタページを構築する手順を説明しました。

### 今後の発展

本記事で構築したノーコードエディタにはさらなる機能拡張の余地があります。さらに先に進みたい場合は、以下のような機能を追加してみると良いでしょう。

- より多くのブロックのサポート
- 複数ページのサポートや、アンカーリンクのサポート
- バックエンド API とのデータ連携
- ドラッグ&ドロップでのブロックの順序変更や、バウンディングボックスでのサイズ変更などのエディタの改善
- Figma や Notion のような、複数人でリアルタイムに共同編集できる機能のサポート

こちらも合わせてご覧下さい：



[https://tech.route06.co.jp/entry/2024/07/03/154219:embed:cite]



### 参考になるノーコードプラットフォームの OSS

ノーコードエディタの基礎はこの記事で説明しましたが、実際に実装する上では多くの考慮事項が存在することも事実です。そんな中で、オープンソースのノーコードプラットフォームも多数存在しており、実装の参考になるはずです。以下に参考になる OSS プロジェクトを紹介します。

- **Tooljet**  
  [Tooljet](https://github.com/ToolJet/ToolJet) は、データベースや API と連携してインタラクティブなアプリケーションを構築できるオープンソースのローコードツールです。直感的な UI と多様なデータソースのサポートを特徴とし、ビジネスアプリケーションの作成に優れています。

- **Budibase**  
  [Budibase](https://github.com/Budibase/budibase) は、ノーコードとローコードの両方をサポートするオープンソースプラットフォームで、データモデルの構築、フォーム作成、カスタムアクションなどが簡単に行えます。フルスタックのアプリケーションを短時間で開発可能な点が特徴です。

- **NocoDB**  
  [NocoDB](https://github.com/nocodb/nocodb) は、既存の MySQL、PostgreSQL などのリレーショナルデータベースをスプレッドシートのように管理できるオープンソースのノーコードツールです。データベースの管理とアプリケーション開発が統合された環境を提供します。

---

ROUTE06 ではローコード開発プラットフォームの開発を行なっており、今回紹介したようなユーザーが自由に UI を構築できるエディタも提供しています。チャレンジングな課題だらけの開発に興味があるエンジニアを募集しています。詳しくは以下をご覧ください。

[https://jobs.route06.co.jp/](https://jobs.route06.co.jp/)

[^1]: https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html#discriminated-unions
[^2]: https://medium.com/airbnb-engineering/a-deep-dive-into-airbnbs-server-driven-ui-system-842244c5f5