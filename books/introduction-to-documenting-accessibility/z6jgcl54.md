---
title: "ライティング: 認知負荷を軽減する"
---
## 概要
このページでは、障害者にとってアクセシブルな Web コンテンツを書き始めるための基本的な考慮事項を紹介します。
これらのヒントは、Web コンテンツ・アクセシビリティ・ガイドライン（WCAG）の要件を満たすためのグッドプラクティスです。
関連する WCAG 要件、「理解する」ドキュメントでの詳細な背景、チュートリアルからのガイダンス、ユーザーストーリーなどへのリンクを参照してください。

### 情報提供とユニークなページタイトルの提供
各 Web ページには、ページの内容を説明し、他のページと区別する短いタイトルを提供します。
ページのタイトルは、多くの場合、ページの主要な見出しと同じです。
ユニークな関連性の高い情報を最初に配置します。
例えば、組織の名前の前にページの名前を入れます。マルチステッププロセスの一部としてのページでは、ページタイトルに現在のステップを含めます。

Example

https://www.w3.org/WAI/WCAG21/Understanding/page-titled

### 意味と構造を伝えるための見出しを使おう
関連する段落をグループ化し、セクションを明確に説明するための短い見出しを使用します。良い見出しは、コンテンツの概要を提供します。

多くの HTML タグは、フォーマットを提供するのではなく、文書の構造階層に関する情報を提供するために開発されました。アクセシビリティと Web 標準を促進するためには、情報階層における本来の目的のために使うのが良いでしょう。多くの場合、そうすることで文書の編集も容易になります。
3、4 段落より長いドキュメントでは、見出しと小見出しは、読者がドキュメント全体のアウトラインを判断し、ページ上の特定の情報にナビゲートするための重要なアクセシビリティ戦略です。

#### 見出しを使用してコンテンツを整理する
| 見出しを使用する場合 | 見出しと小見出しを使用する場合 |
|------------|---|
|            |   |

https://accessibility.psu.edu/headings/

https://www.w3.org/WAI/WCAG21/quickref/#headings-and-labels
https://www.w3.org/WAI/WCAG21/Understanding/headings-and-labels

https://www.w3.org/WAI/WCAG21/quickref/#section-headings
https://www.w3.org/WAI/WCAG21/Understanding/section-headings

https://www.w3.org/WAI/WCAG21/quickref/#info-and-relationships
https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships

### リンクテキストを意味を持たせる
リンクテキストを記述するときは、リンクのターゲットの内容を記述するようにします。
曖昧なリンクテキスト、例えば「ここをクリック」とか「もっと読む」などの使用を避けます。
リンクのターゲットに関する関連情報としてドキュメントの種類やサイズなどを示します。例：「提案文書（RTF, 20MB)」。

| 意味がないテキストリンク | 意味を持たせたテキストリンク |
|--------------|----------------|
|              |                |


https://www.w3.org/WAI/WCAG21/quickref/#link-purpose-in-context
https://www.w3.org/WAI/WCAG21/Understanding/link-purpose-in-context

https://www.w3.org/WAI/WCAG21/quickref/#link-purpose-link-only
https://www.w3.org/WAI/WCAG21/Understanding/link-purpose-link-only

### マルチメディアのためのトランスクリプトとキャプションを作成する
ポッドキャストのような音声のみのコンテンツの場合、トランスクリプトを提供します。研修ビデオのような音声と視覚のコンテンツの場合、キャプションも提供します。トランスクリプトとキャプションには、コンテンツの理解に重要な話し言葉や音、例えば「ドアがきしむ」といった情報を含めます。ビデオのトランスクリプトでは、重要な視覚的コンテンツも説明します。例：「アサンが部屋を出る」。

https://www.w3.org/WAI/media/av/

https://www.w3.org/WAI/WCAG21/quickref/#captions-prerecorded
https://www.w3.org/WAI/WCAG21/Understanding/captions-prerecorded

https://www.w3.org/WAI/WCAG21/quickref/#audio-description-or-media-alternative-prerecorded
https://www.w3.org/WAI/WCAG21/Understanding/audio-description-or-media-alternative-prerecorded

### 明確な指示を提供する
指示、ガイドライン、エラーメッセージがクリアであり、不必要に技術的な言語を避けるようにします。日付形式など、入力要件を説明します。

- 例： 指示がユーザーに提供すべき情報を伝える
- 例： エラーが何であるかを示し、可能であれば修正方法も示す

https://www.w3.org/WAI/WCAG21/quickref/#labels-or-instructions
https://www.w3.org/WAI/WCAG21/Understanding/labels-or-instructions

### コンテンツを簡潔に保つ
文脈に適した、シンプルな言語とフォーマットを使用します。

- 短く、明確な文と段落で書く。
- 不必要に複雑な言葉やフレーズの使用を避ける。
- 最初の使用時に略語を展開する。例えば、Web Content Accessibility Guidelines (WCAG)。
- 読者が知らない可能性のある用語のための用語集を提供することを検討する。
- 適切な場合にリストのフォーマットを使用する。
- 意味を明確にするために、画像、イラスト、ビデオ、オーディオ、シンボルの使用を検討する。

| コンテンツを不必要に複雑にする | コンテンツを読みやすく理解しやすくする |
|-----------------|--|
|                 |  |

https://www.w3.org/WAI/WCAG21/quickref/#reading-level
https://www.w3.org/WAI/WCAG21/Understanding/reading-level

https://www.w3.org/WAI/WCAG21/quickref/#unusual-words
https://www.w3.org/WAI/WCAG21/Understanding/unusual-words

https://www.w3.org/WAI/WCAG21/quickref/#abbreviations
https://www.w3.org/WAI/WCAG21/Understanding/abbreviations