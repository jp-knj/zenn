---
title: "Web アクセシビリティに導くガイドライン: WCAG 2.x"
---

## 知覚可能
### ガイドライン 1.1 – テキストによる代替
#### ガイドラインの背景
Web 上には様々な情報やコンテンツが存在します。しかし、すべての人がこれらのコンテンツを同じようにアクセスできるわけではありません。例えば、視覚障害のある人は画像や動画を直接見ることが難しい場合があります。ガイドライン 1.1 は、こうした非テキストコンテンツに対して、テキストとしての代替手段を提供することを目指しています。これにより、テキストを読み上げる技術などを使用して、多くの人々が情報にアクセスできるようになります。

## レベル A
### 1.1.1 非テキストコンテンツ
Web 上には、画像や音声といった非テキストコンテンツが多く存在します。しかし、全ての人がこれらのコンテンツを正しく理解することは難しいです。視覚や聴覚に障害を持つ方は、非テキストコンテンツだけでは情報を得られません。そのため、これらの情報を文字として提供することで、様々な人々が情報を理解できるようにする必要があります。この考え方が「達成基準 1.1.1 非テキストコンテンツ」の背景となります。

#### 成功例
**画像に代替テキストを提供**
画像が Web ページ上で情報を伝える役割を果たしている場合、その画像には代替テキストを提供することが重要です。
```html
<img src="example.jpg" alt="犬が公園で遊ぶ様子">
```

**動画の説明をテキストで提供**
動画コンテンツには、その内容を説明するテキストを添えることで、聴覚に障害を持つ方も内容を理解できます。
```html
<video src="movie.mp4" controls></video>
<p>この動画は、山でのハイキングの様子を撮影したものです。</p>
```

#### 失敗例
**alt属性を省略した画像**
画像に alt 属性がない場合、画像が何を示しているのか分からなくなります。
```html
<img src="example.jpg">
```

**装飾的な画像に不要な代替テキスト**
装飾的な画像（情報を伝える目的がない）に対して、不要な代替テキストを提供するのも問題です。
```html
<img src="decorative.jpg" alt="spacer">
```

#### ガイドライン 1.2 – 時間依存メディア
時間依存メディアには代替コンテンツを提供すること。

##### レベル A
###### 1.2.1 音声のみ及び映像のみ**
収録済の音声のみや映像のみのメディアは、特定の条件を除き、特定の要件を満たしている。
参考リソース： [1.2.1 Audio-only and Video-only (Prerecorded)](https://www.w3.org/WAI/WCAG21/Understanding/audio-only-and-video-only-prerecorded)

###### 1.2.2 キャプション**
同期したメディアの収録済の音声コンテンツには、キャプションが提供されている。
参考リソース： [1.2.2 Captions (Prerecorded)](https://www.w3.org/WAI/WCAG21/Understanding/captions-prerecorded)

###### 1.2.3 音声解説、又はメディアに対する代替**
同期したメディアの収録済の映像コンテンツには、音声解説や代替コンテンツが提供されている。
参考リソース： [1.2.3 Audio Description or Media Alternative (Prerecorded)](https://www.w3.org/WAI/WCAG21/Understanding/audio-description-or-media-alternative-prerecorded)

##### レベル AA
###### 1.2.4 キャプション**
同期したメディアのライブの音声コンテンツには、キャプションが提供されている。
参考リソース：  [1.2.4 Captions (Live)](https://www.w3.org/WAI/WCAG21/Understanding/captions-live)

###### 1.2.5 音声解説
同期したメディアの収録済の映像コンテンツには、音声解説が提供されている。
参考リソース：  [1.2.5 Audio Description (Prerecorded)](https://www.w3.org/WAI/WCAG21/Understanding/audio-description-prerecorded)

#### レベル AAA
##### 1.2.6 手話**
同期したメディアの収録済の音声コンテンツには、手話通訳が提供されている。
参考リソース： [1.2.6 Sign Language (Prerecorded)](https://www.w3.org/WAI/WCAG21/Understanding/sign-language-prerecorded)

##### 1.2.7 拡張音声解説**
同期したメディアの収録済の映像コンテンツには、拡張音声解説が提供されている。
参考リソース： [1.2.7 Extended Audio Description (Prerecorded)](https://www.w3.org/WAI/WCAG21/Understanding/extended-audio-description-prerecorded)

##### 1.2.8 メディアに対する代替**
収録済の同期したメディアや映像のみのメディアには、時間依存メディアの代替コンテンツが提供されている。
参考リソース： [1.2.8 Media Alternative (Prerecorded)](https://www.w3.org/WAI/WCAG21/Understanding/media-alternative-prerecorded)

##### 1.2.9 音声のみ
ライブの音声のみのコンテンツには、時間依存メディアの代替コンテンツが提供されている。
参考リソース： [1.2.9 Audio-only (Live)](https://www.w3.org/WAI/WCAG21/Understanding/audio-only-live)

#### ガイドライン 1.3 – 適応可能
情報や構造を損なうことなく、様々な方法（例： よりシンプルなレイアウト）で提供できるようにコンテンツを制作すること。

#### レベル A
##### 1.3.1 情報及び関係性
提示されている情報、構造、及び関係性は、プログラムによる解釈が可能、又はテキストで提供されている。

##### 1.3.2 意味のある順序**
コンテンツの提示順序が意味に影響する場合、正しい読み順序はプログラムによる解釈が可能である。
参考リソース： [1.3.2 Meaningful Sequence](https://www.w3.org/WAI/WCAG22/Understanding/meaningful-sequence)

##### 1.3.3 感覚的な特徴**
コンテンツの説明は、形、大きさ、位置、方向、音などの感覚的特徴だけに依存していない。
参考リソース：  [1.3.3 Sensory Characteristics](https://www.w3.org/WAI/WCAG22/Understanding/sensory-characteristics)

##### 1.3.4 表示の方向**
コンテンツは、縦向き、横向き表示など、1 つの表示方向に制限してはいけません
参考リソース： [1.3.4 表示の向き](https://www.w3.org/WAI/WCAG22/Understanding/orientation)

##### レベル AA
##### 1.3.5 入力目的を特定する**
ユーザーに関する情報を収集するフォーム入力の目的がプログラムによって特定できるようにする
参考リソース：  [1.3.5 Identify Input Purpose](https://www.w3.org/WAI/WCAG22/Understanding/identify-input-purpose)

##### レベル AAA
##### 1.3.6 目的を特定する (レベル AAA)**
マークアップ言語を使用して実装されたコンテンツでは、UI コンポーネント、アイコンの目的を特定できる必要がある
参考リソース：  [1.3.6 Identify Purpose](https://www.w3.org/WAI/WCAG22/Understanding/identify-purpose)

#### ガイドライン 1.4 – 判別可能
コンテンツを利用者にとって見やすく、聞きやすくすること。前景と背景の区別も含む。

##### レベル A
##### 1.4.1 色の使用**
色は情報伝達、動作指示、反応促進、視覚的要素の識別の唯一の手段として使用されていない。  
参考リソース：  [1.4.1 Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color)

##### 1.4.2 音声の制御**
ウェブページの自動再生音声が 3 秒以上続く場合、音声の一時停止、停止、または音量調整のメカニズムが提供されている。
参考リソース：  [1.4.2 Audio Control](https://www.w3.org/WAI/WCAG22/Understanding/audio-control)

##### レベル AA
##### 1.4.3 コントラスト (最低限)
テキストや文字画像の視覚的提示は、少なくとも 4.5:1 のコントラスト比を持つ。
参考リソース：  [1.4.3 Contrast (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum)

##### 1.4.4 テキストのサイズ変更
キャプションや文字画像を除き、テキストは 200%までサイズ変更可能で、コンテンツや機能が損なわれない。
参考リソース：  [1.4.4 Resize Text](https://www.w3.org/WAI/WCAG22/Understanding/resize-text)

##### 1.4.5 文字画像**
意図した視覚的提示が可能な技術を使用している場合、情報伝達には文字画像ではなくテキストが使用されている。
参考リソース：  [1.4.5 Images of Text](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text)

##### レベル AAA
##### 1.4.6 コントラスト (高度)**
テキストや文字画像の視覚的提示は、少なくとも 7:1 のコントラスト比を持つ。
参考リソース： [1.4.6 Contrast (Enhanced)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced)

##### 1.4.7 小さな背景音、又は背景音なし**
収録済の音声のみのコンテンツには、特定の要件を満たす必要がある。
参考リソース： [1.4.7 Low or No Background Audio](https://www.w3.org/WAI/WCAG22/Understanding/low-or-no-background-audio)

##### 1.4.8 視覚的提示**
テキストブロックの視覚的提示に関する特定の要件が提供されている。
参考リソース： [1.4.8 Visual Presentation](https://www.w3.org/WAI/WCAG22/Understanding/visual-presentation)

##### 1.4.9 文字画像 (例外なし)**
文字画像は、純粋な装飾やテキストの特定の表現が情報伝達に必要不可欠な場合にのみ使用されている。
参考リソース：  [1.4.9 Images of Text (No Exception)](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text-no-exception)

##### 1.4.10 リフロー**
情報や機能の損失なくて、スクロールの必要ないコンテンツが提供されている
参考リソース： [1.4.10 Reflow](https://www.w3.org/WAI/WCAG22/Understanding/reflow)

##### 1.4.11 非テキストのコントラスト**
要素の視覚的な表示は、隣接する色とのコントラスト比が少なくとも 3:1 である必要があります
参考リソース： [1.4.11 Non-text Contrast](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast)

##### 1.4.12 テキストの間隔**
文字の間隔や行間などのテキストの見た目を調整する特定のスタイル設定があります。これらの設定をフォントサイズに対して一定以上に設定しておくことです。
参考リソース：  [1.4.12 Text Spacing](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing)

##### 1.4.13 ホバー又はフォーカスで表示されるコンテンツ**
追加のコンテンツを知覚できること、ページのエクスペリエンスを妨げずにそれを非表示にできなければならない
参考リソース： [1.4.13 Content on Hover or Focus](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus)