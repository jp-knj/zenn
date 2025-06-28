---
title: "Hydration Strategies"
---
## 0. 痛みの共有 ⚡Quick Fact
> LCP が *+1.3 s* / JS DL が *+46 %* 悪化した社内計測例 (グラフ)

---

## 1. Hydrationディレクティブ速習 ⚡
| 戦略 | いつ | 主要API | 代表ユースケース |
|------|------|---------|-----------------|
| load | ページロード直後 | `load` evt | ナビゲーション |
| idle | Idle 時 | `requestIdleCallback` | 関連記事 |
| visible | ビューポート内 | `IntersectionObserver` | コメント欄 |
| media | MQ一致時 | `matchMedia` | PC専用UI |

*→ 詳細実装は 2 章へ*

## 2. Deep Dive 🔎 — ランタイム実装
```js
// media strategy (最新 API)
const mql = matchMedia(this.hydrate.value)
const handler = (e)=> e.matches && hydrate()
mql.matches ? hydrate() : mql.addEventListener('change', handler)
````

## 3. パフォーマンス実測 💬Column

* `client:idle` 化でLCP **-480 ms** / TBT **-28 %** (WebPageTest)

## 4. フレームワーク別 Hydration 呼び出し

## 5. ハンズオン🧪 — 自サイトで計測

1. `npm run astro build --watch`
2. `<MyWidget client:idle />` に変更
3. Lighthouse before/after比較
4. PRに結果を添えて共有

### 最終チェックリスト

- [ ] 冒頭に数字or失敗談で“痛み”を示した  
- [ ] 5分で流れが掴める速習表を追加した  
- [ ] `matchMedia` APIを最新構文に更新した  
- [ ] パフォーマンス節に実測グラフを1個置いた  
- [ ] 章末ハンズオンで読者の行動を促した  

この修正で、**ストーリー性・多層読み・数字スパイス** の3つが入り、読みやすさと説得力がさらに高まります！