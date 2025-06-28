# Content Collections System: 型安全なCMS機能の内部実装

## 概要

AstroのContent Collections機能は、MarkdownやMDXファイルを型安全に管理するCMS機能です。Zodスキーマとの統合により、コンパイル時の型チェックを実現している内部実装を解析します。

## Content Collectionsの基本構造

### スキーマ定義
```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const blogCollection = defineCollection({
  type: 'content', // または 'data'
  schema: z.object({
    title: z.string(),
    description: z.string(),
    publishDate: z.date(),
    tags: z.array(z.string()),
    draft: z.boolean().default(false),
    heroImage: z.string().optional(),
  }),
});

export const collections = {
  'blog': blogCollection,
  'docs': defineCollection({
    type: 'content',
    schema: z.object({
      title: z.string(),
      order: z.number(),
      category: z.enum(['tutorial', 'reference', 'guide']),
    }),
  }),
};
```

## 内部実装の詳細

### 1. コンパイル時の型生成

Astroは`.astro/types.d.ts`ファイルを自動生成し、型安全性を提供：

```typescript
// .astro/types.d.ts (自動生成)
declare module 'astro:content' {
  interface ContentCollectionMap {
    'blog': {
      id: string;
      slug: string;
      body: string;
      collection: 'blog';
      data: {
        title: string;
        description: string;
        publishDate: Date;
        tags: string[];
        draft: boolean;
        heroImage?: string;
      }
    };
  }
}
```

### 2. ファイルシステムの監視と解析

```javascript
// 内部実装の簡略版
class ContentCollectionWatcher {
  constructor(collections) {
    this.collections = collections;
    this.entries = new Map();
    this.setupWatcher();
  }

  async scanCollections() {
    for (const [name, config] of Object.entries(this.collections)) {
      const collectionDir = path.join('src/content', name);
      const files = await glob(`${collectionDir}/**/*.{md,mdx}`);
      
      for (const file of files) {
        await this.processFile(name, file, config);
      }
    }
  }

  async processFile(collectionName, filePath, config) {
    const content = await fs.readFile(filePath, 'utf-8');
    const { data: frontmatter, content: body } = matter(content);
    
    // Zodスキーマによる検証
    try {
      const validatedData = config.schema.parse(frontmatter);
      const entry = {
        id: this.generateId(filePath),
        slug: this.generateSlug(filePath),
        collection: collectionName,
        data: validatedData,
        body,
        filePath,
      };
      
      this.entries.set(entry.id, entry);
    } catch (error) {
      // スキーマ検証エラーをコンパイル時に報告
      this.reportSchemaError(filePath, error);
    }
  }
}
```

### 3. ランタイムAPI

```typescript
// astro:contentモジュールの実装
import type { ContentCollectionMap } from './types';

export async function getCollection<C extends keyof ContentCollectionMap>(
  collection: C,
  filter?: (entry: ContentCollectionMap[C]) => boolean
): Promise<ContentCollectionMap[C][]> {
  const entries = await import(`../content-cache/${collection}.mjs`);
  
  if (filter) {
    return entries.default.filter(filter);
  }
  
  return entries.default;
}

export async function getEntry<C extends keyof ContentCollectionMap>(
  collection: C,
  slug: string
): Promise<ContentCollectionMap[C] | undefined> {
  const entries = await getCollection(collection);
  return entries.find(entry => entry.slug === slug);
}

export async function render(entry: any) {
  const { default: Component, getHeadings } = await import(
    `../content-cache/${entry.collection}/${entry.id}.mjs`
  );
  
  return {
    Content: Component,
    headings: await getHeadings(),
    remarkPluginFrontmatter: entry.data,
  };
}
```

## データ型の種類

### Content型 (Markdown/MDX)
```typescript
const posts = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    publishDate: z.date(),
  }),
});

// 使用例
const allPosts = await getCollection('posts');
const { Content } = await render(allPosts[0]);
```

### Data型 (JSON/YAML)
```typescript
const authors = defineCollection({
  type: 'data',
  schema: z.object({
    name: z.string(),
    bio: z.string(),
    avatar: z.string(),
    social: z.object({
      twitter: z.string().optional(),
      github: z.string().optional(),
    }),
  }),
});
```

## ビルド時の最適化

### 1. 静的解析とプリコンパイル
- Markdownコンテンツの事前レンダリング
- 画像の最適化とWebP変換
- メタデータの抽出と索引化

### 2. キャッシュ戦略
```javascript
// コンテンツキャッシュの生成
await generateContentCache({
  'blog': blogEntries,
  'docs': docEntries,
});

// 出力: .astro/content-cache/
// ├── blog.mjs          # エントリー一覧
// ├── blog/
// │   ├── entry-1.mjs   # 個別エントリー
// │   └── entry-2.mjs
// └── docs.mjs
```

## 他のCMSとの比較

### 従来のヘッドレスCMS
```javascript
// Contentful/Strapiの場合
const posts = await fetch('https://api.contentful.com/spaces/xxx/entries')
  .then(res => res.json());
// 型安全性なし、ランタイムでのAPI呼び出し
```

### Astro Content Collections
```typescript
// コンパイル時に型チェック、ビルド時に静的生成
const posts = await getCollection('blog');
// 完全な型安全性、ゼロランタイムコスト
```

## 実践例：ブログシステム

```astro
---
// src/pages/blog/[...slug].astro
import { getCollection, render } from 'astro:content';
import Layout from '../../layouts/Layout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog', ({ data }) => {
    return data.draft !== true;
  });
  
  return posts.map((post) => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content, headings } = await render(post);
---

<Layout title={post.data.title}>
  <article>
    <h1>{post.data.title}</h1>
    <time>{post.data.publishDate.toLocaleDateString()}</time>
    <Content />
  </article>
</Layout>
```

## まとめ

Content Collections システムは、従来のCMSとMarkdownベースのサイト生成の良いとこ取りを実現しています。型安全性、パフォーマンス、開発体験の全てを高次元でバランスさせた、Astroの技術的な優位性を示す機能の一つです。