---
title: "Xaris（文章制作支援ツール）の技術構成"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Xaris", "AI", "Svelte", "Supabase", "ProseMirror"]
published: false
---


あらすじ

沖縄の宿を転々、その旅の途中、男は一つのゲストハウスに流れ着いた。宿サブスクユーザー、旅人、その地のコミュニティの人間、、様々な人々が行き交うその場で、男は幾らかの時を過ごした。東京勤務の後にその地に移住した家主は、そこに居を構えながらいくつかの事業を展開しているという。時を経て男は、家主の事業のWebアプリ開発をスポットで手伝ったりするようになっていた。そして2022年12月、男の元に家主から連絡があった。

「ヤッベ！これマジヤッベ！」

ChatGPTの世界的バズである。問いかけに対し、みたこともない精度で流暢に回答するAI。度肝ヌキヌキである。「アッカン！これマジアッカン！」家主の主たる事業が、こいつによりひどく影響を受ける未来が見えたという。これはこのまま何もしなかったらマズい、何か、、何かこいつをうまく活用したツールを作らねば！男、男ォー！！となり、男に連絡したという。男は家主から作りたいプロダクトの概要を聞くと、早速その開発に取り掛かった。

## Xaris の概要

[Xaris](https://xaris.ai/)（カリス）はPCで文章制作する際に使うエディタアプリである。画面左にChatGPT（のようなチャット画面）、画面右にNotion（のようなテキストエディタ）が付いている。裏で運営が多くのプロンプト（AIに送る指示文）を組んでおり、ユーザーは Xaris の指示に従いながら文章を仕上げていくことができる。

<video controls src="./xaris_seo_demo.mp4"></video>


### Case 1. SEO記事ライター、太郎🙂
今回の案件は「和菓子屋本舗「あゝる」の認知向上・集客力アップにつながるような記事を書く」というもの。「和菓子　種類」でネット検索された時に上位表示される記事を書くことに。太郎は Xaris の「SEO記事作成」モードを選択した。

Xaris「上位表示したい検索キーワードを入力してください。」
🙂太郎「和菓子　種類」
Xaris「（タイトル案） 1. ... 2. ... 3. ... （読者像・ニーズ）「和菓子　種類」で検索する人は...」
🙂太郎：気に入ったタイトルがあれば「挿入」ボタンでエディタに挿入、エディタでタイトルを編集、「次へ」ボタンを押す
Xaris「...」というタイトルからアウトラインを生成します。参考記事: ...、アウトライン： ...
🙂太郎：アウトラインが気に入れば挿入、エディタで編集、「次へ」を押す
Xaris「...」という小見出しから本文を生成します。...
🙂太郎：文章が気に入れば挿入、エディタで編集、次へ
...

☝️Xaris とやり取りを繰り返し、記事を仕上げていく。

### Case 2. インタビュー記事ライターふみ子
ふみ子：「インタビュー記事作成」モードを選択、エディタの「音声から文字起こし」でインタビュー音源を文字起こし
Xaris「インタビュー記事の目的を入力してください」
...
☝️インタビュー記事作成を Xaris で効率化できる。

### Case 3. のび太
のび太：「その他」モードを選択
Xaris「どんなことを書きたいですか？」
のび太「あさがおの観察日記」
Xaris「具体的にどのような内容を記録したいと考えていますか？例えば、観察期間、成長の過程、花の色や形、気象条件など」
のび太「成長の過程」
Xaris「あさがおの成長を観察する際、どのような段階（発芽、成長、開花など）に注目したいですか？」
...
☝️Xaris が書きたい事について色々ヒアリングしてくれる。


## デプロイ環境
xaris のリポジトリ内から、以下をそれぞれ別プロジェクトとして `Cloudflare Workers & Pages` にデプロイしている。

- `xaris.ai` - xaris 本体（Pages）
- xaris 開発者用ドキュメント（Pages）
- xaris 共同編集用サーバー（Workers）。なんやかんやで本体と分けた。


## 技術構成

- [Svelte/SvelteKit](https://svelte.dev/) - フロント・バックエンド。他フレームワークよりネストを浅く、概念数を少なく、簡素に実装できそうで採用。ただエンジニアの採用に困るくさい。確かに旅先で話すエンジニアに「Svelte 使ってます」と話しても「それおいしい？」とよく言われる。エンジニア求ム。
- [Supabase](https://supabase.com/) - データベース。Postgres を楽にいじるためのクライアントライブラリ、管理画面があるクラウドサービス。DB構造を管理画面から編集し、差分をマイグレーションファイルとして吐き出せる。DB周りで楽したいのと、Docker を自分で触りたくなくて採用。
- [Cloudflare Pages](https://pages.cloudflare.com/) - デプロイ先。エッジ環境（Node環境ではないので Node API 依存のライブラリが使えなかったりする）。なんか速いらしい。GitHub 連携で勝手にデプロイしてくれる。各ブランチごとにプレビュー環境を用意してくれる。Docker を自分で触りたくなくて採用。
- [VitePress](https://vitepress.dev/) - 開発者用ドキュメント

### UI
- [shadcn-svelte](https://shadcn-svelte.com/) - UIコンポ実装例集。キーボードナビゲーションに対応したくて採用（特に DropdownMenu）。コピペ・改変して使っている。shadcn-svelte を使い始めるまで [UnoCSS](https://unocss.dev/) & [daisyUI](https://daisyui.com/) で実装を進めていたので、それらと組み合わさった感じだったり、それらから移行気味だったりする。

### テキストエディタ
- [ProseMirror](https://prosemirror.net/) - エディタを実装できるライブラリ。拡張性が高く、あらゆる機能を実装できる。ただ学習コストが高い。[Tiptap](https://tiptap.dev) 経由で使っている。
  - [Tiptap](https://tiptap.dev) - ProseMirror を拡張したライブラリ。ProseMirror での実装を機能ごとの拡張としてまとめ、登録できる。ex. (Node) Heading, Paragraph, (Mark) Bold, Italic, (Extension) Toolbar, SlashMenu。また、拡張から登録したコマンドを Tiptap のインスタンスから呼び出せる。

(背景)
初めは [Editor.js](https://editorjs.io/) を使って実装していた。しかし細かい機能を実装するのに Editor.js だと厳しそう（ドラッグ&ドロップ、スラッシュコマンド、.etc）、TypeScript 未対応、などの理由から別のエディタライブラリに移行することにした。note が過去に何度もエディタを作り直していたらしく、現在採用しているのが ProseMirror である、ということなどを参考に、ProseMirror を採用した。

[noteのエディタを新バージョンに完全移行しました](https://engineerteam.note.jp/n/n48cf05ad9784)
  > 実はnoteのエディタは今回が4度目のリプレイスになります。

### 共同編集
- [Collaboration extension (Tiptap)](https://tiptap.dev/docs/editor/extensions/functionality/collaboration) - 内部で [yjs](https://github.com/yjs/yjs) が使われている。yjs 形式のデータのやり取りを処理するための共同編集サーバーを [DurableObject](https://developers.cloudflare.com/durable-objects/), WebSocket を使って実装し、Cloudflare Workers 上で動かしている。

### AI マネージャー
「パラメータに渡すモデル名を変えるだけで、各プロバイダのAIにリクエストを送れるやつ」が欲しい。LangChain のやつは複雑すぎるように思い、しかしいい感じのが見つけられないので自前で実装している。[Vercel AI SDK](https://sdk.vercel.ai/) がそういう感じに変化しているっぽいが、まだテキスト生成しか対応してなさそう & 好みじゃない。[portkey](https://portkey.ai/) みたく、「OpenAI SDK 経由で OpenAI 以外のプロバイダのAIにもリクエストできるようにする」がいいと思ったので、そんな感じのリファクタを企んでいる。portkey を契約するのは「管理プラットフォームが増えるのを避けたい、データが外部に溜まるのを避けたい」などの理由で避けている。

### Admin 画面
![xaris admin](/images/xaris-tech-stack/xaris_admin.png)

- [TanStack Table](https://tanstack.com/table/latest) - 多機能なテーブルが実装できるライブラリ。プロンプトの設定をする Excel 的なのを用意している。

### API
- [OpenAI](https://platform.openai.com/docs/overview), [Anthropic](https://www.anthropic.com/), [Gemini](https://ai.google.dev/gemini-api) - 生成AI
- [Whisper (OpenAI)](https://platform.openai.com/docs/models#whisper) - 文字起こし。「Xaris の文字起こしの精度が高い」と評判がいい。whisper に投げてるだけです。
- [Stripe](https://stripe.com) - 決済。[Polar](https://polar.sh/) が気になっている。
- [SendGrid](https://sendgrid.com) - メール配信。[Resend](https://resend.com/) が気になっていたが、SendGrid にはある「Contacts からフィルタしたリストを作る」画面がないのでやめた。


## 参考プロジェクト
### テキストエディタ
- [Google Docs](https://docs.google.com/)
- [Lark Docs](https://www.larksuite.com/en_us/product/creation) - 中華 Slack 的なアプリ「Lark」についてる Google Docs 的なやつ
- [Notion](https://www.notion.so/)
- [Outline](https://github.com/outline/outline) - Open Source Notion。コメント、共同編集の実装を参考にした。
- [AFFiNE](https://affine.pro/) - > open-source alternative for Notion & Miro.
- [Obsidian](https://obsidian.md/) - ローカルファーストのメモアプリ。Plugin API の設計を参考にした。（コマンドの登録、load, unload メソッド、など）

### 共同編集
- [PartyKit](https://www.partykit.io/) - ~~独自プラットフォームにデプロイする必要がある。Pro プランはサポートに連絡する必要がある（英語）。自由度が下がりそう、管理コストが上がりそうで不採用。~~ 色々アップデートされてそう、再検討の余地あり？
- [Hocuspocus](https://tiptap.dev/docs/hocuspocus/introduction) - Tiptap collab 用のサーバー実装。Node API 依存なので不採用。

### Copilot, AI チャット
- [GitHub Copilot](https://github.com/features/copilot)
- [Cody](https://sourcegraph.com/cody)
- [ChatGPT](https://chatgpt.com/)
- [Claude](https://claude.ai/new)
- [perplexity](https://www.perplexity.ai/)

### AI マネージャー
- [portkey AI Gateway](https://github.com/portkey-ai/gateway)
- [Vercel AI SDK](https://sdk.vercel.ai/)
- [litellm](https://github.com/BerriAI/litellm) - python 用。

### AI モニタリング
リクエストの集計、プロンプトを組む画面、などの参考
- [portkey](https://portkey.ai/)
- [Helicone](https://www.helicone.ai/)
- [Langfuse](https://langfuse.com/)

### ドキュメンテーション
- [AFFiNE Docs](https://docs.affine.pro/docs/development/quick-start)
- [tldraw Docs](https://tldraw.dev/quick-start)
- [portkey](https://portkey.ai/docs/introduction/what-is-portkey)


## やってもらいたいタスク
上のものほど軽いので、上からやっていくと効率良くコード理解が進む & モチベ保ちながら進められると思います。

- 「コメントがありません」を表示
- (CommandPalette) フォルダ名も含めて検索
- keydownmap.ts → keybinding.ts に移行
- Export
  - PDF
  - WordPress
- transcription
  - リファクタ
  - YouTube リンクから文字起こし
- 右サイドバー
  - 編集履歴
  - 参考情報
- Team Space


以上。
