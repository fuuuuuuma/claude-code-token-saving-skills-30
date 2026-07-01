# トークン節約Skills集30選 — Claude Codeで今日から効く一次情報チェック

> Claude Codeを使っているとトークン（＝料金・レート制限）が増えがちな理由と、公式ドキュメントで実際に裏が取れた節約テクニック・Skills活用術を30項目にまとめた、撮影用の解説リサーチノートです。すべての項目を`code.claude.com/docs`等の一次情報に当たって再検証し、公式に確認できなかった項目は正直に⚠️と明記しています。

📊 [スライド資料を見る](./slides.html) ｜ 📄 [1枚まとめ資料を見る](./onepager.html)

## TL;DR（3行）

- Claude Codeは毎メッセージで会話全履歴を読み直す仕組み上、コンテキストが大きいほど1リクエストのコストが増える ✅。だから「小さく保つ」ことが節約の核心。
- `/clear`・`/compact`・CLAUDE.mdの200行以下維持・Skillsへの移行・サブエージェント委任の5つが、公式ドキュメントで裏付けられた効果の大きい打ち手 ✅。
- ネット上でよく語られる「`.claudeignore`」は、今回Claude Code公式ドキュメントを直接確認した限りでは**存在が確認できなかった**⚠️。実在する代替は`permissions.deny`（Read/Edit拒否ルール）。

## 目次

1. [なぜトークンが増えるのか](#1-なぜトークンが増えるのか)
2. [節約の5軸](#2-節約の5軸)
3. [A軸：コマンド操作系（6項目）](#3-a軸コマンド操作系6項目)
4. [B軸：設定ファイル最適化系（4項目）](#4-b軸設定ファイル最適化系4項目)
5. [C軸：プロセス最適化系（7項目）](#5-c軸プロセス最適化系7項目)
6. [D軸：実在するClaude Code Skills活用（10項目）](#6-d軸実在するclaude-code-skills活用10項目)
7. [E軸：運用Tips（3項目）](#7-e軸運用tips3項目)
8. [よくある誤解の訂正（今回の一次情報確認で判明）](#8-よくある誤解の訂正今回の一次情報確認で判明)
9. [今日からやること（初心者向け3ステップ）](#9-今日からやること初心者向け3ステップ)
10. [出典](#10-出典)
11. [未確認・注意事項](#11-未確認注意事項)

確度マークの意味：✅公式ドキュメントで直接確認済み ／ 🔶一次情報の記載はあるが数値や運用は流動的・伝聞 ／ ⚠️未確認・推測・非公式。

---

## 1. なぜトークンが増えるのか

Claude Codeはリクエストのたびに会話履歴・CLAUDE.md・利用可能なSkill名／MCPツール名をまとめてモデルに送ります。会話が長くなるほど、それを維持するために毎ターン再送されるトークン量が積み上がっていく仕組みです。これはClaude Code固有の欠陥ではなく、LLMエージェント全般に共通する構造です。

だからこそ「コンテキストを小さく保つ」ことが、料金の節約と精度の維持を同時に達成する一番のレバーになります。以下30項目は、その「小さく保つ」ための具体的な打ち手を、公式ドキュメントで裏付けが取れた順に整理したものです。

## 2. 節約の5軸

- **A軸 コマンド操作**：会話やコンテキストを能動的に操作するコマンド群
- **B軸 設定ファイル最適化**：CLAUDE.md・権限・MCP・Skillsの構成をあらかじめ整える
- **C軸 プロセス最適化**：モデル選択やサブエージェント委任など、作業の進め方そのものを変える
- **D軸 実在するSkills活用**：このセッションでも実際に呼び出しているClaude Code純正Skillsの棚卸し
- **E軸 運用Tips**：習慣レベルの小さな工夫

---

## 3. A軸：コマンド操作系（6項目）

### 1. `/clear` ✅
新しい会話を空のコンテキストで開始します。それまでの会話は`/resume`で後から呼び戻せます。**なぜ節約になるか**：無関係な作業に切り替えても古い会話を持ち越すと、以後の全メッセージでその分のトークンを消費し続けます。公式ドキュメントも「古い会話は次に必要なファイルの居場所を奪い、毎メッセージでトークンを消費する」と明記しています。また`/clear`後は会話履歴レイヤーのキャッシュプレフィックスが引き継がれないため、プロンプトキャッシュの観点でも古い文脈を再利用しない形になります。
出典: [commands](https://code.claude.com/docs/en/commands)／[costs（日本語）](https://code.claude.com/docs/ja/costs)／[context-window](https://code.claude.com/docs/en/context-window)

### 2. `/compact [指示]` ✅
会話をここまでの要約に置き換えてコンテキストを解放するコマンドです。`/compact focus on the auth bug fix`のように引数でカスタム指示を渡し、要約時に何を残すかを指定できます（CLAUDE.mdでの恒常指定も可）。**なぜ節約になるか**：全履歴を毎ターン送る代わりに構造化要約1本に圧縮し、以後のリクエストの入力トークンを削減します。圧縮そのものはキャッシュ済みプレフィックスを読んで生成されるため要約生成コストは比較的小さく、圧縮後は会話レイヤーのみが再構築される、と公式に説明されています。
出典: [commands](https://code.claude.com/docs/en/commands)／[costs（日本語）](https://code.claude.com/docs/ja/costs)／[prompt-caching](https://code.claude.com/docs/en/prompt-caching)

### 3. `/status` `/context` `/usage` の使い分け ✅
3つは役割がまったく違います。**`/status`**は設定画面のStatusタブでバージョン・モデル・接続状況を表示するだけでトークン量は出ません。**`/context [all]`**は現在のコンテキスト使用量を色分けグリッドで可視化し、何が圧迫しているかの最適化提案まで出す「中身の診断ツール」です。**`/usage`**（エイリアス`/cost` `/stats`）はセッションのコストや利用上限を示す「会計ツール」で、Pro/Max/Team/Enterpriseではスキル・サブエージェント・MCPサーバー別の使用内訳も出ます。節約したいなら、まず`/context`で何が容量を食っているかを特定し、`/usage`で金額ペースを把握する、という順で使うのが実務的です。
出典: [commands](https://code.claude.com/docs/en/commands)／[costs（日本語）](https://code.claude.com/docs/ja/costs)

### 4. `/effort`（reasoning effort） ✅
モデルによって対応段階が異なります。Sonnet 5・Opus 4.8・Opus 4.7は`low`/`medium`/`high`/`xhigh`/`max`の5段階、Opus 4.6・Sonnet 4.6は`low`/`medium`/`high`/`max`の4段階（**`xhigh`が無く`max`はある**）です。対応外のレベルを指定すると、その時点で対応する最も高い段階へ自動的にフォールバックします（例：`xhigh`指定はOpus 4.6上では`high`として動作）。**なぜ節約になるか**：effortが低いほど「考えるかどうか・どれだけ考えるか」が抑制され、出力トークンとして課金される思考（thinking）トークンを減らせます。逆に`max`は公式に「トークン消費に制約なし」と明記されています。ルーティン作業は`low`、本当に難しい問題だけ`high`以上、が基本方針です。
出典: [model-config](https://code.claude.com/docs/en/model-config)（「Adjust effort level」表）／[commands](https://code.claude.com/docs/en/commands)／[effort（API仕様）](https://platform.claude.com/docs/en/build-with-claude/effort)

### 5. `/model`・`opusplan` ✅
`/model [model]`でモデルを切り替えられ、次回セッションのデフォルトとしても保存されます。`opusplan`はモデルエイリアスの一つで、公式説明は「plan modeでは`opus`を使い、実行フェーズでは自動的に`sonnet`に切り替える特殊モード」というシンプルな一文です。**なぜ節約になるか**：複雑な設計判断が要るplanフェーズだけ高性能・高コストのOpusを使い、実装フェーズは低コストのSonnetに自動で戻すことで、Opusの単価を計画部分だけに限定できます。ただしplanモード切替のたびにモデルが変わるため、そのたびにプロンプトキャッシュが再構築される（キャッシュミスが起きる）という副作用も公式に説明されています。
出典: [model-config](https://code.claude.com/docs/en/model-config)（`opusplan`項）／[prompt-caching](https://code.claude.com/docs/en/prompt-caching)

### 6. `/btw`（履歴に残さない質問） ✅
`/btw <question>`は、現在の会話履歴に一切追加せずに脇道の質問をするコマンドです。質問と回答は一時的なオーバーレイに表示されるだけで会話履歴には入りません。ファイル読み取りやコマンド実行などのツールアクセスはできず、既存コンテキストのみから回答します。公式には「サブエージェントの逆（サブエージェントはツールを持つが空コンテキストから開始、`/btw`はフルコンテキストを見えるがツールなし）」と説明されています。**なぜ節約になるか**：通常の質問のように会話履歴に恒久的に積み上がらないため、以後の全ターンで再送されるトークンが増えません。
出典: [commands](https://code.claude.com/docs/en/commands)／[interactive-mode](https://code.claude.com/docs/en/interactive-mode)（「Side questions with /btw」節）

---

## 4. B軸：設定ファイル最適化系（4項目）

### 7. CLAUDE.mdは200行以下 ✅
公式ドキュメントに独立した2箇所で明記されています。costsページ（日本語版）は「スキルはオンデマンドでのみ呼び出されたときに読み込まれるため、特殊な指示をスキルに移動することで、ベースコンテキストを小さく保ちます。CLAUDE.mdを200行以下に保つことを目指し、必須のみを含めます」と述べ、context-windowページ（英語版）も「Keep it under 200 lines. Move reference content to skills or path-scoped rules so it only loads when needed.」と同旨を記載しています。CLAUDE.mdはセッション開始時に毎回全読み込みされるため、これが固定コストとして効いてきます。200行という数値自体はAnthropicの推奨値であり、あらゆるプロジェクト規模での最適性が実証されているわけではない点は留意してください。
出典: [costs（日本語）](https://code.claude.com/docs/ja/costs)／[context-window](https://code.claude.com/docs/en/context-window)

### 8. `permissions.deny`（Read/Edit拒否ルール） ✅
`.claude/settings.json`の`permissions.deny`に`Read(./.env)`や`Read(./secrets/**)`のようなルールを書くと、Claudeはそのファイルを読めなくなります。公式ドキュメントは評価順序を「Rules are evaluated in order: deny, then ask, then allow. The first match in that order determines the outcome」と明記し、さらに「Permission rules are enforced by Claude Code, not by the model.」――つまりモデルの気まぐれではなく、ハーネス側が強制的にブロックする仕組みであると説明しています。**なぜ節約になるか**：直接の節約効果というより、不要なファイル（大きなログ・ビルド成果物・秘密情報）へのアクセスそのものを未然に断つことで、無駄な読み込みトークンと事故リスクを同時に防げます。
出典: [permissions](https://code.claude.com/docs/en/permissions)（「Manage permissions」「Read and Edit」節、原文確認済み）

### 9. MCP tool search（ツール名のみ起動時ロード） ✅
公式ドキュメントに「Tool search keeps MCP context usage low by deferring tool definitions until Claude needs them. Only tool names and server instructions load at session start, so adding more MCP servers has minimal impact on your context window.」と明記されています。これはデフォルトで有効な仕組みで、実際にこの資料を作っている本セッションでも、大量のMCPツールが「deferred tools」として名前だけ列挙され、実際に使う段になって初めてスキーマが読み込まれる挙動を直接観測しています。`/mcp`パネルは接続中サーバーごとのツール数を表示するため、どのMCPが重いかも確認できます。**なぜ節約になるか**：MCPサーバーを増やしても、使わないツールの詳細スキーマはコンテキストに乗らないため、サーバー数そのものが即コスト増にはなりません。
出典: [mcp](https://code.claude.com/docs/en/mcp)（「Scale with MCP tool search」節、原文確認済み）

### 10. Skillsのprogressive disclosure（段階的開示） ✅
Anthropic公式エンジニアリングブログとClaude Code公式ドキュメントの両方で、同じ設計思想が説明されています。ブログは「At startup, the agent pre-loads the `name` and `description` of every installed skill into its system prompt.」「If Claude thinks the skill is relevant to the current task, it will load the skill by reading its full `SKILL.md` into context.」と述べ、この仕組みを「progressive disclosureの第一段階」と呼んでいます。Claude Code公式ドキュメントも「Unlike CLAUDE.md content, a skill's body loads only when it's used, so long reference material costs almost nothing until you need it.」と明記しています（ただしサブエージェントに事前ロードされたSkillは例外的に起動時から本文がインジェクトされる、という注記も同じページにあります）。**なぜ節約になるか**：システムプロンプトに全Skillの手順を書き込む方式と違い、実際に使うSkillだけが本文コストを払う設計だからです。
出典: [Anthropic Engineering: Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)／[skills（Claude Code）](https://code.claude.com/docs/en/skills)

---

## 5. C軸：プロセス最適化系（7項目）

### 11. サブエージェントへの調査委任 ✅
公式ドキュメントに「Each subagent runs in its own context window with a custom system prompt, specific tool access, and independent permissions.」「the subagent does that work in its own context and returns only the summary」と明記されています。目的は「メイン会話にexploration・implementationの中身を持ち込まない」ことです。**なぜ節約になるか**：「認証システムを全部読んで」をメインセッションでやると読み込んだファイル全部が会話履歴に残りますが、サブエージェントに投げれば要約だけがメインに返り、探索過程のトークンはメインの履歴を汚しません。
出典: [sub-agents](https://code.claude.com/docs/en/sub-agents)

### 12. プランモード ✅
`Shift+Tab`を押すとパーミッションモードが循環し（`default → acceptEdits → plan`）、plan modeでは「Claude reads files and runs read-only shell commands to explore but doesn't edit your source files」という状態になります。目的はディスクへの変更前にレビューすることです。ネット上では「Shift+Tab×2」と語られがちですが、公式ドキュメントに「2回」という固定回数の明記はなく、現在のモード次第で到達までの回数は変わる点に注意してください。**なぜ節約になるか**：設計方針がズレたまま実装を進めて手戻りするコストの方が、プランを固めるコストより大きくなりやすいためです。
出典: [permission-modes](https://code.claude.com/docs/en/permission-modes)

### 13. worktree分離 ✅
「A git worktree is a separate working directory with its own files and branch, sharing the same repository history and remote as your main checkout. Running each Claude Code session in its own worktree means edits in one session never touch files in another」と明記されています。`claude --worktree <name>`でデフォルト`.claude/worktrees/<name>/`にworktreeが作られ、専用ブランチ上で動作します。サブエージェント側も`isolation: worktree`のfrontmatterで個別worktree分離が可能で、「変更が無ければ終了時に自動削除される」とも記載されています。**なぜ節約になるか**：直接のトークン節約というより、並列セッションのファイル競合を防ぎ、コンフリクト解消のための余分なやり取り（＝トークン）を発生させない効果です。
出典: [worktrees](https://code.claude.com/docs/en/worktrees)

### 14. モデル選択（Sonnetファースト） ✅
公式costsページに「Sonnet handles most coding tasks well and costs less than Opus. Reserve Opus for complex architectural decisions or multi-step reasoning.」と明記されています。エージェントチームの構成でも「Use Sonnet for teammates. It balances capability and cost for coordination tasks.」とあり、チームメンバーには基本Sonnetを、難しい設計判断が要る場面にだけOpusを、という使い分けが公式に推奨されています。参考までに同じcostsページには「average cost is around \$13 per developer per active day and \$150-250 per developer per month」という利用実績や、「Agent teams use approximately 7x more tokens than standard sessions when teammates run in plan mode」という記載もあり、これらは断定的な保証値ではなく公式ドキュメント内の参考値として紹介されています。
出典: [costs](https://code.claude.com/docs/en/costs)（「Choose the right model」節）／[model-config](https://code.claude.com/docs/en/model-config)

### 15. 並列 vs 逐次の判断 🔶
独立したタスクは並列実行（各セッション・エージェントが独立worktreeで動く）でよい一方、共有コンテキストが要るタスクを並列に投げると、各エージェントが同じ前提を個別に読み直すため合計トークンがかえって増えることがあります。この使い分けの原則自体は妥当ですが、「並列→逐次で40%削減」のような具体的な削減率は、今回確認した公式ドキュメントの範囲では裏付けが取れず、個人ブログ・実測ベンチマーク由来の参考値として扱うべきです。
出典: 公式ドキュメントに定量記載なし。数値は元記事（BSWEN実測ブログ）由来、未確認。

### 16. プロンプトを具体化する 🔶
「認証モジュールをよりセキュアにリファクタして」のような曖昧な指示は、関連範囲を広く探索させることになりやすく、「`/app/auth/login.py`の`authenticate()`にレート制限を追加。Redisクライアントで5回/15分/IP」のように対象ファイル・関数・条件を絞った指示は探索コストを抑えられます。この方向性自体は「サブエージェントは要約だけ返す」設計（項目11）とも整合的ですが、「60%削減」等の具体数値は公式ドキュメントでは確認できておらず、個人ブログの実測値として紹介するにとどめます。
出典: 公式ドキュメントに定量記載なし。数値は元記事由来、未確認。

### 17. 1プロジェクト×1スレッド原則 🔶
無関係なリポジトリを同じセッションで開きっぱなしにすると、片方の文脈がもう片方のタスクにも常時乗り続けます。項目1（`/clear`）・項目13（worktree分離）と組み合わせて、プロジェクトが変わったら会話も切る、という運用上の原則です。効果の大きさを示す公式な定量データは今回確認できていません。
出典: 公式ドキュメントに定量記載なし。運用上の原則として紹介。

---

## 6. D軸：実在するClaude Code Skills活用（10項目）

ここからは「Skills自体をどう使い分けるか」の実例です。Skillsはprogressive disclosure（項目10）の仕組み上、呼び出すまでは名前と説明しかコンテキストに乗らないため、種類を増やすこと自体はベースコストにほぼ影響しません。

### 18. `superpowers:using-superpowers`
関連スキルの発見・呼び出しの規律を統一する起点スキルです。「このスキルが1%でも当てはまるなら必ず使う」という規律を強制することで、自己流の場当たり対応（＝手戻り＝再生成トークン）を減らします。

### 19. `superpowers:brainstorming`
実装前に要件と設計意図を整理するスキルです。要件が曖昧なまま実装し、後から作り直す手戻りコストを防ぎます。

### 20. `superpowers:systematic-debugging`
バグ調査を体系立てて進めるスキルです。場当たり的な試行錯誤による無駄なファイル読み込みの繰り返しを抑えます。

### 21. `superpowers:test-driven-development`
実装前にテストを書く進め方を強制するスキルです。実装→動作確認→やり直しの往復回数を減らします。

### 22. `deep-research`
多段階のWeb検索・情報収集を1つのスキルに集約します。都度WebSearchを積み上げてメイン会話に結果を貼り続けるより、まとまった調査を1ステップに圧縮できます。

### 23. `research-doc`
この資料自体を作るのに使っているスキルです。README・スライド・1枚資料という3点セットの構成をテンプレ化しており、毎回ゼロから章立てを考えるコストを省きます。

### 24. `code-review`
effortレベル（low/medium/high〜ultra）を指定してレビューの深さを調整できます。軽量な確認はlow/mediumで済ませ、重量級のレビューが必要な時だけ深いレベルを使う、という使い分けが可能です。

### 25. `verify`
実装が実際に動くかを検証するスキルです。「動きます」「完成しました」という主張と実機確認の往復を1回で終わらせ、確認漏れによる再修正の往復を減らします。

### 26. `update-config`
`settings.json`などハーネス設定の変更を定型操作として扱うスキルです。都度、設定ファイルの構造を一から説明させずに済みます。

### 27. `claude-api`
Claude APIやモデルID・料金体系についての固定リファレンスです。モデル名や仕様を毎回聞き直す代わりに参照する用途で、本資料の`/effort`段階数の検証（項目4）でも実質的に同種の一次情報確認を行っています。

---

## 7. E軸：運用Tips（3項目）

### 28. 中間ファイルの削除
`.dump.txt`や`.review.txt`のような作業用中間ファイルをタスク完了後に削除する運用です。次回セッションでの誤読み込みや、不要な説明の往復を防ぎます。

### 29. セッション終了時にmemoryへ引き継ぎを書き残す
経緯や決定事項を会話履歴から再構築させるのではなく、要点だけをメモリファイルに残しておく運用です。次回セッションの立ち上がりで、同じ経緯説明を毎回コンテキストに積み直さずに済みます。

### 30. Skill定義は自分の記憶で代替しない
`using-superpowers`（項目18）が明示している規律で、「知っているスキルでも毎回ロードし直す（Skills evolve. Read current version）」という原則です。うろ覚えの手順で進めて後から訂正するより、都度正しい定義を読み込む方が、結果的に手戻りが少なくなります。

---

## 8. よくある誤解の訂正（今回の一次情報確認で判明）

この資料を作る過程で、ネット上でよく見かける説明の一部が、Claude Code公式ドキュメントの記載と食い違っていることが分かりました。

- **`.claudeignore`はClaude Code公式機能として確認できませんでした** ⚠️。`permissions`・`settings`・`context-window`・`mcp`の各公式ページを確認しましたが、`.claudeignore`という語自体が見当たりません。同種の目的（特定ファイルへのアクセス制限）を果たす実在の公式機能は`permissions.deny`（項目8）です。「`.claudeignore`で40〜60%削減」という説明を見かけた場合は、非公式な情報である可能性を念頭に置いてください。
- **「Shift+Tab×2でプランモード」という固定回数の説明は公式には明記されていません** ⚠️。公式には「Shift+Tabでモードを循環させる」としか書かれておらず、到達までの回数は現在のモード状態に依存します（項目12）。
- **`opusplan`の説明として出回る長文引用は、公式ページの実際の文言と異なります** ⚠️。公式の説明は「plan modeでは`opus`を使い、実行フェーズでは自動的に`sonnet`に切り替える」という短い一文のみです（項目5）。
- **`/effort`の段階数はモデルによって異なり、`xhigh`の有無を混同しやすい点に注意してください** ⚠️。Opus 4.6・Sonnet 4.6には`max`はありますが`xhigh`はありません（項目4）。

---

## 9. 今日からやること（初心者向け3ステップ）

専門用語が並ぶと身構えてしまいますが、最初にやることは3つだけです。

1. **タスクが変わったら`/clear`を打つ**（項目1）。「バグ修正が終わった」「別の機能を作り始める」というタイミングで、まず`/clear`を押す癖をつけるだけで、多くの場合十分な効果があります。
2. **CLAUDE.mdの行数を確認する**（項目7）。長い手順書のような内容が入っていたら、それはSkillへ移すべきサインです。
3. **`/context`を一度打ってみる**（項目3）。今の会話で何がコンテキストを占めているかを、色付きグリッドで見るだけでも「何を減らせばいいか」の勘所がつかめます。

## 10. 出典

**Claude Code公式ドキュメント**
- https://code.claude.com/docs/en/commands
- https://code.claude.com/docs/en/model-config
- https://code.claude.com/docs/ja/costs
- https://code.claude.com/docs/en/costs
- https://code.claude.com/docs/en/context-window
- https://code.claude.com/docs/en/interactive-mode
- https://code.claude.com/docs/en/permissions
- https://code.claude.com/docs/en/mcp
- https://code.claude.com/docs/en/skills
- https://code.claude.com/docs/en/sub-agents
- https://code.claude.com/docs/en/permission-modes
- https://code.claude.com/docs/en/worktrees
- https://code.claude.com/docs/en/prompt-caching
- https://platform.claude.com/docs/en/build-with-claude/effort

**Anthropic公式エンジニアリングブログ**
- https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

**参考にした既存リサーチ資料（数値ベンチマークは元記事側の確度表記のまま未検証扱い）**
- Claude Code編（No表記は本資料では割愛）: https://github.com/chaaaaarin/claudecode-channel-20260629
- Codex編（No表記は本資料では割愛）: https://github.com/chaaaaarin/kawaru-20260629

## 11. 未確認・注意事項

- ⚠️ `.claudeignore`はClaude Code公式ドキュメントの範囲では存在が確認できませんでした。「存在しない」と完全に断定できる悉皆調査ではなく、主要ページ（permissions・settings・context-window・mcp）を確認した範囲での結果です。
- ⚠️ 項目15〜17（並列/逐次判断・プロンプト具体化・1プロジェクト1スレッド）の具体的な削減率は、公式ドキュメントでは裏付けが取れず、個人ブログ・実測ベンチマーク由来の未確認情報です。
- ⚠️ 参考にした2つの既存リサーチ資料に含まれる数値（CLAUDE.md最適化91.9%削減、.claudeignore 40〜60%削減など）は、本資料では独自に再検証しておらず、元記事側の確度表記をそのまま踏襲しています。
- 🔶 Codex（OpenAI）側の情報は今回のセッションでは独自に再検証しておらず、既存のCodex編リサーチ資料が確認した公式情報をそのまま参照する形にとどめています。
- 本資料は非公式のリサーチノートです。Anthropic公式・OpenAI公式とは無関係で、仕様は変わるため最終確認は各社公式ドキュメントで行ってください（作成日: 2026-07-01）。
