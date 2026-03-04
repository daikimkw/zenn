---
title: "Web 標準動向 2026年2月版"
emoji: "👹"
type: "idea"
topics: ["frontend", "cybozuwebstandards"]
published: false
publication_name: "cybozu_frontend"
---

こんにちは！ サイボウズ株式会社 フロントエンドエンジニアの [daiki (@daikimkw)](https://x.com/daikimkw) です。

## はじめに

サイボウズは 2025 年 4 月より、W3C のメンバーに加入しました。

https://blog.cybozu.io/entry/joining-w3c

標準化プロセスに関わることができるようになるための最初の一歩として、フロントエンドエンジニアの一部のメンバーは積極的に Web 標準のキャッチアップを行っています。

そこで、毎月メンバーが興味を持った Web 標準に関する話題や、実際に標準化プロセスに関わることができた場合にはその報告などを 1 つの記事としてまとめ、紹介していきます。
また、ここでは W3C に限らず、TC39 や WHATWG などの標準化団体のトピックについても扱います。

:::message
今月の執筆者は以下の 5 名です。

- [Saji](https://x.com/sajikix)
  - ECMA402 (Intl) 周りのトピックを執筆
- [saku](https://x.com/sakupi01)
  - HTML, CSS に関連するトピックを執筆
- [くらっち](https://x.com/kuracchi04)
  - ECMA262 周りのトピックを執筆
- [daiki](https://x.com/daikimkw)
  - WinterTC 周りのトピックを執筆
- [mehm8128](https://x.com/mehm8128)
  - 主にアクセシビリティに関連するトピックを執筆

:::

## HTML

### The translatable-data-* attribute

https://github.com/whatwg/html/issues/12162

`alt` 属性や `aria-label` 属性など一部の属性は、ブラウザの翻訳機能などで翻訳の対象になります。

しかし、`data-*` 属性は翻訳の対象になりません。`data-*` 属性を翻訳したいケースとして、例えば CSS の `content` プロパティにおいて `attr()` を用いて参照したり、カスタム要素において `this.dataset.*` のような形で参照して DOM に埋め込むような例が紹介されています。

これを解決するために、`data-*` 属性のように動作しながら翻訳の対象になるような、`translatable-data-*` や `data-translatable-*` のような属性が提案されています。

一方で、content プロパティの値は基本的に全て翻訳されるべきという案や、カスタム要素で DOM に埋め込むパターンについては既に機能しているという話も出ています。

### Introduce "Declarative Overscroll Actions"

https://github.com/openui/open-ui/issues/1372

JS 不要でスワイプ系のインタラクションを実現する Declarative Overscroll Actions の提案です。

たとえばスワイプでサイドメニューを開く、リストアイテムをスワイプして削除ボタンを出す、pull-to-refresh といった操作において、HTML 属性のみで宣言的な実装が可能になります。

まだ Explainer の段階ですが、Invoker Commands API（`command`/`commandfor`）の仕組みに乗ることで実現されるようです。

[Declarative Overscroll Actions (Explainer) | Open UI](https://open-ui.org/components/overscroll-actions.explainer/)

## CSS

### Blink: Intent to Prototype: Margin-trim

https://groups.google.com/a/chromium.org/g/blink-dev/c/Ix14Xjg7R7g/m/SqENaABYAwAJ

[margin-trim](https://developer.mozilla.org/ja/docs/Web/CSS/Reference/Properties/margin-trim) は、コンテナー内の要素の先頭と末尾の margin を除去することができるプロパティです。

Safari では既にリリース済みなので、Blink に入ったら残りは Firefox のみになります。

### Ship: Web app scope system accent color

https://groups.google.com/a/chromium.org/g/blink-dev/c/uSDN66x2nHc

`accent-color: auto` はシステム（OS）のアクセントカラーを使える一方で、`AccentColor` / `AccentColorText` はフィンガープリンティング対策の観点から PWA 文脈での限定的な利用が許可されてきました。
しかし今回、この差をそろえるために `accent-color: auto` も PWA と同等のスコープに限定し、通常の Web ページでは UA 既定の色へフォールバックさせる方針が進んでいます。
もとから `accent-color: auto` に対してシステムアクセントカラーを利用していない Firefox からは、この動きに対して Positive なシグナルが出ています。

### Ship: The `revert-rule` Keyword

https://groups.google.com/a/chromium.org/g/blink-dev/c/ieQj5bt-wA8

`revert-rule` は、「カスケードで直前の CSS ルール」に戻すためのキーワードです。
`revert-layer` がレイヤー単位で戻すのに対して、`revert-rule` は CSS ルール単位で戻せる点が違いです。

例えば、以下のようなケースでは `--unknown-custom-property` がスタイル計算時に見つからない場合、color は仕様デフォルトにフォールバックしてしまいます。カスケードで 2 番目に優位な `color: cyan` はカスタムプロパティのスタイル計算前に棄却されてしまうからです。

```
p { color: cyan; color: var(--unknown-custom-property); }
```

この挙動は `var()` や `if()`、Custom Function（`@function()`）などの CSS 関数で起こる可能性があり、CSS WG ではこうした CSS 関数のフォールバックを改善する取り組みが進んでいます。

今回の `revert-layer` キーワードを用いて上記を `color: var(--unknown-custom-property, revert-layer);` のようにすることで、これまで棄却されていた `cyan` をフォールバックとして利用可能になります。

### Ship: CSSPseudoElement interface

https://groups.google.com/a/chromium.org/g/blink-dev/c/l20Dc3XMo0c

`Element.pseudo(type)` で疑似要素を `CSSPseudoElement` として取得できるようにする提案です。
現時点の対象は `::before` / `::after` / `::marker` で、`type` / `element` / `parent` / `pseudo()` などを通じて疑似要素を JS から扱いやすくします。
`::scroll-marker` など疑似要素中心の機能拡張が増える中で、疑似要素を「JS から参照可能な対象」として扱う必要性が高まってきた流れに沿う内容です。

[\[css-pseudo\] Can we make pseudo-elements first-class citizens in the DOM? · Issue #11559 · w3c/csswg-drafts](https://github.com/w3c/csswg-drafts/issues/11559)

関連して、疑似要素に直接イベントリスナーを登録する機能 – というより、イベントに `pseudoTarget` を追加して発火元の疑似要素を識別できるようにする提案も進んでいます。

[Intent to Ship: Pseudo target on events](https://groups.google.com/a/chromium.org/g/blink-dev/c/Xhg02ya-kro)

### First Public Working Draft: Selectors Level 5

https://www.w3.org/news/2026/first-public-working-draft-selectors-level-5/

Selectors Level 5 の FPWD が公開されました。
Level 5 での主な変更は以下のとおりです。

- `:local-link` and `:local-link()`
  - 現在のページ URL とリンク先を比較して、サイト内リンクかどうかを判定します
- `:state()`
  - カスタム要素の内部状態を外部に公開するための擬似クラスです
- `:heading` and `:heading()`
  - 見出し要素にマッチする擬似クラスです
- `:interest-source` and `:interest-target`
  - Interest Invokers 関連の擬似クラスです
- `:blank`
  - フォーム入力要素が空の状態にマッチする擬似クラスです
- `|| / :nth-col() / :nth-last-col()`
  - テーブルのような 2 次元グリッドで、列の関係をセレクタで表現するためのものです。`col.selected || td` で特定列のセルを選択したり、`:nth-col(An+B)` で偶数列・奇数列を指定したりが可能です
- `:current` / `:past` / `:future`
  - タイムライン上の現在・過去・未来の要素にマッチする擬似クラスです
  - 音声字幕のレンダリングや WebVTT での利用が考えられます

### [meta] Overview page on drafts.csswg.org not updating · Issue #12743 · w3c/csswg-drafts

https://github.com/w3c/csswg-drafts/issues/12743#issuecomment-3894298496

こちらは技術仕様そのものではなく、CSSWG の drafts/wiki サーバーの持続性に関するトピックです。
drafts/wiki サーバーは、長期間の個人負担（約 200 ドル/月）で運用されており、その費用・運用負荷の偏りへの不満が示され、今月末で支払いを止める方針が表明されました。これにより、サーバーの移行・代替運用の検討が急務になっています。

### CSS Snapshot 2026

https://www.w3.org/TR/2026/css-2026/

2026 年の CSS Snapshot が公開されました。
これは「どの CSS 仕様を参照すべきか」を安定度ベースで整理する年次スナップショットで、実装状況の一覧というより仕様参照の基準点として使うドキュメントとなっています。

## ARIA・WCAG

### Gecko: Intent to Ship: ariaNotify

https://groups.google.com/a/mozilla.org/g/dev-platform/c/UMLaNzRQ5MI/m/dAs9R4RMAgAJ

Gecko でも ariaNotify が Intent to Ship になりました。Firefox 150 から利用可能になるとのことです。

[以前の Web 標準動向](https://zenn.dev/cybozu_frontend/articles/web_standards_monthly_202509#aria-notify)でも紹介したように Chrome では 141 から利用可能になっており、Webkit でも実装済みとのことです。

### ACD End of Year Report and Roadmap

https://lolaslab.co/blog/2025/acd-2025-report/

ACD（Accessibility Compat Data）の、2025 年まとめと 2026 年の展望の記事です。

ACD プロジェクトは 2025 年に正式に始まりました。マイルストーンと KPI の設定から始まり、[Technical Gap Analysis and Scope](https://github.com/lolaslab/accessibility-compat-data/blob/master/technical-gap-analysis-scope.md) というドキュメントを書き、資金を調達し、[ACD Collector](https://github.com/lolaslab/acd-poc) の POC を実装したのが 2025 年でした。

2026 年には W3C の WebDX CG 及び ARIA-AT などと連携し、開発や既存の WPT のテストスイートを利用する方法の検討を進めていきたいとのことです。

### "Using ARIA" status

https://github.com/w3c/aria/issues/2708

Using ARIA のドキュメントは、古い Working Draft でした。

しかし多くの内容は APG に組み込まれたため、今回廃止されました。なお、たびたび参照・引用される「ARIA の第一のルール」及びそれに付随する第二から第四までのルールについては、ドキュメント上からは削除されずに残されています。

### web-platform-tests/interop-accessibility

wpt のアクセシビリティ関連のテストは現状、アクセシビリティツリーの accessible name と role しか見ることができません。しかし、accessible name と role 以外のプロパティについても見ることができるようにしたり、Accessibility API が提供している情報（＝よりユーザー目線の情報）も見ることができるようにしたりすることで core-aam と html-aam のテストをできるようにしようという Acacia プロジェクトが動いています。

TPAC2025 での資料やその他関連資料は [TPAC 2025 参加記 - アクセシビリティの話 - mehm8128 の Weblog](https://portfolio.hm8128.me/blog/tpac2025/#acacia) をご覧ください。

特に最近これに関連して 2 つの PR に動きがありました。

[Support for testing additional accessibility properties beyond name and role. by jcsteh · #55784](https://github.com/web-platform-tests/wpt/pull/55784) と [Create new test type for accessibility API testing by spectranaut · #57696](https://github.com/web-platform-tests/wpt/pull/57696) です。

前者はアクセシビリティツリーにおいて accessible name と role 以外のプロパティも見ることができるようにする PR です。以前からあったのですが、2 月に入ってから複数人に Approve されているようです。

後者はアクセシビリティツリーではなく、Accessibility API が提供する情報をテストできるようにするための PR です。以前から [Extend testdriver.js and wptrunner to test AAMs (platform accessibility APIs) by spectranaut · #53733](https://github.com/web-platform-tests/wpt/pull/53733) で作業が行われていたのですが、別のアプローチで再実装が行われているようです。

[直近の ARIA WG のミーティング](https://www.w3.org/2026/02/26-aria-minutes.html#848b)でも扱われており、間もなく追加で多くのアクセシビリティテストを wpt に導入できるようになりそうです。

詳細については後者の PR で引用されている [RFC](https://github.com/web-platform-tests/wpt/pull/57696) にて基本的な部分から解説されているようです。

## JavaScript

### ECMA402

#### IntlMV limit

https://github.com/tc39/ecma402/pull/1022

Intl.NumberFormat 等で扱われる仕様での内部的な値 Intl.MathematicalValue（Intl MV）の制限値を引き上げる提案についてです。現在高精度な 10 進浮動小数点を扱えるようにする Decimal などが提案されており、これらの提案を見据えて現状の上限を緩和することが検討されています。

今回の会議では、TG1 からのフィードバックや ICU 等のライブラリ側で対応可能かなどが確認され、元々提案されていた有効桁数 10,000 桁という形である程度合意が取れました。

#### Intl locale info

2025 年 11 月の会議で Stage 4 になった Intl Locale Info API について、アルゴリズムをより明示的にする修正が承認されました。

#### Intl keep-trailing-zeros

https://github.com/tc39/proposal-intl-keep-trailing-zeros/issues/9

Intl.NumberFormat や Intl.PluralRules などの数値を受け取る API で、数値の末尾のゼロを保持する提案である Intl keep-trailing-zeros の Stage について Stage 3 への進展が議論されました。

議論の結果、[最終レビュー](https://github.com/tc39/proposal-intl-keep-trailing-zeros/pull/10)を条件に TG2 としては Stage 3 に進めることができると判断されました。

#### Amount update

数値と単位を組み合わせて保持できる Amount 型について Stage 2 に進めるための状況報告がなされました。精度情報を持つアクセッサを削除しパフォーマンスを向上させる修正の紹介や、単位変換メソッドのオーバーライド構造などについて議論されました。

#### Intl Unit Protocol Handling conflicting units

Intl Unit Protocol Proposal は Intl.NumberFormat の format メソッドや Intl.PluralRules の select メソッドなどで単位を指定できるようにする提案です。この提案においてコンストラクタで指定した単位と、format/select メソッドに渡されたオブジェクトの単位が矛盾する場合の処理の定義が不足していることについて議論されました。

以前の会議では暗黙の変換やエラーなど複数の案が出されていましたが、今回の会議では当面 RangeError をスローすることで合意されました。また将来的に ECMA-262 側で単位変換機能が追加された場合は、暗黙の変換を導入する方針となりました。

さらに単位と合わせた指定の仕方として `{unit, value}` のようなオブジェクトで指定する方式を採用することで合意されました。

#### Temporal Stage 4

Temporal およびそれに関連する Intl Era Month Code の Stage 4 へ向けた最終確認が行われました。特に大きな懸念点もなく、TG2 としては両提案が Stage 4 へ進むことが承認されました。

#### dateStyle/timeStyle patterns

Intl.DateTimeFormat で dateStyle / timeStyle を使用した場合、個別のコンポーネントオプション（year, month 等）の組み合わせでは再現不可能なパターンが生成されることがあるという問題です。

この問題に関して、そもそもフォーマット結果は不透明なものであり制約を設けるべきではないという意見や、この問題が CLDR 側のデータ構造に起因すること、また単一の日付と期間でフォーマットが大きく異なる原因にもなっていることなどが指摘されました。

現状は [CLDR 側で上がっている問題](https://unicode-org.atlassian.net/browse/CLDR-1993)の解決を待って再び議論することになりました。

### ECMA262

#### Upsert to Stage 4

https://github.com/tc39/proposals/commit/131e53d6c9e658c6439831a167ed3f7897daf160

Map/WeakMap の getOrInsert/getOrInsertComputed が Stage 4 へ進みました。ES2026 に間に合う見込みです。

#### Error code property for Stage 1, 2, or 2.7

https://github.com/jasnell/proposal-error-code-property

ECMAScript の `Error` オブジェクトに、標準的な `code` プロパティを追加する提案です。

message の文字列に依存せず、特定の条件で分岐可能になります。

TC39 第 113 回の MTG（2026 年 3 月）で Stage 1, 2, 2.7 への昇格が審議予定です（現在 Stage 0）。

#### Iterator Includes for Stage 1, 2, or 2.7

https://github.com/michaelficarra/proposal-iterator-includes

イテレータが特定の値を含むかを確認する `includes` メソッドを追加する提案です。

現在は `some` メソッドとカスタムコンパレータで代替可能ですが、意図が直接的に表現されず煩雑という問題を解決します。

`Array.prototype.includes` と同様に `SameValueZero` 比較を使用し、第 2 引数で検索開始位置のスキップも可能です（ただし負のインデックスは不可）。

同 MTG で Stage 1, 2, 2.7 への昇格が審議予定です（現在 Stage 0）。

#### Thenable Curtailment for Stage 2

https://github.com/tc39/proposal-thenable-curtailment

`Promise.resolve()` などがオブジェクトの `then` プロパティをプロトタイプチェーン全体から探索することで、`Object.prototype.then` 経由で予期しないコード実行が発生し、セキュリティ脆弱性の原因となっている問題を解決する提案です。

Promise 解決時にユーザーコード実行の可能性がある場合は `MaybeDeferredPromiseResolve` 操作を導入し、新しい Job をキューに入れて処理を遅延させることで安全性が向上します。

同 MTG で Stage 2 への昇格が審議予定です（現在 Stage 1）。

#### TypedArray concat for Stage 2

https://github.com/tc39/proposal-typedarray-concat

TypedArray、ArrayBuffer、SharedArrayBuffer に、`concat` メソッドを追加する提案です。

手動でのオフセット計算やバッファ確保が不要になり、エンジンによる最適化が可能になります。

同 MTG で Stage 2 への昇格が審議予定です（現在 Stage 1）。

#### TypedArray find within for Stage 2

https://github.com/tc39/proposal-typedarray-findwithin

TypedArray 内でサブシーケンス（連続した要素列）を検索するネイティブ API を追加する提案です。

現在は単一要素の `indexOf` のみで、部分配列の検索には開発者が自前で実装する必要があり、最適化が難しいという問題を解決します。

`search`、`searchLast`、`contains` の 3 メソッドを `TypedArray.prototype` に追加し、実装固有の最適化されたアルゴリズムで高速な検索が可能になります。

同 MTG で Stage 2 への昇格が審議予定です（現在 Stage 1）。

#### Normative: make re-exporting a namespace object in 2 steps behave like 1 step

https://github.com/tc39/ecma262/pull/3715

ES Modules において、namespace object を 2 ステップで再エクスポートする場合と 1 ステップで再エクスポートする場合の挙動を統一する仕様変更です。

従来、以下の 2 つのパターンは通常のバインディングでは等価でしたが、namespace object の場合は異なる挙動をしていました：

```
// moduleA.js: 直接の再エクスポート export * as foo from "./mod.js"; // moduleB.js: import + export の2ステップ import * as foo from "./mod.js"; export { foo };
```

この違いにより、複数のモジュールから同じ namespace を `export * from` で再エクスポートする際、moduleA のパターン同士では問題なく動作するものの、moduleA と moduleB のパターンを混在させると「曖昧なエクスポート」エラーが発生するという不整合がありました。

```
// moduleC.js export * from './moduleA.js'; export * from './moduleB.js'; // 以前は競合エラーになる可能性があった
```

今回の変更では、moduleB の場合に `localExportEntries` ではなく `indirectExportEntries` へ追加するように修正されました。具体的には `[[ImportName]]: ~all~` を持つ `ExportEntry Record` として扱われるようになり、`ResolveExport` での衝突検出において「同じモジュールの同じバインディングであれば衝突とみなさない」というロジックが適用されるようになります。

SpiderMonkey、JavaScriptCore では実装済みで、V8 も Chrome 146 で対応します。バンドラーや minifier が `import * as ns; export { ns }` を `export * as ns from` に安全に変換できるようになる点も実用上のメリットです。

## Baseline

### 📃 February 2026 release notes

https://web-platform-dx.github.io/web-features-explorer/release-notes/february-2026/

### Interop 2026

Web プラットフォームの互換性を向上させる取り組み、Interop が今年も始まりました。Interop 2026 では、150 以上の提案から 20 の重点分野と 4 つの調査分野が選定されています。

- [Interop 2026: Continuing to improve the web for developers | Blog | web.dev](http://web.dev/blog/interop-2026)
- [Launching Interop 2026 – Mozilla Hacks - the Web developer blog](https://hacks.mozilla.org/2026/02/launching-interop-2026/)
- [Announcing Interop 2026 | WebKit](https://webkit.org/blog/17818/announcing-interop-2026/)
- [Microsoft Edge and Interop 2026 | Microsoft Edge Blog](https://blogs.windows.com/msedgedev/2026/02/12/microsoft-edge-and-interop-2026/)
- [Interop 2026 Focus Areas Announced | Igalia](https://www.igalia.com/news/interop-2026.html)

## Misc

### State of JavaScript 2025

https://2025.stateofjs.com/en-US

State of JavaScript 2025 の結果が公開されました。

詳細については[今月の Cybozu Frontend Monthly](https://cybozu.github.io/frontend-monthly/posts/2026-02/) で話しています。

### Group Note Draft: Text-to-Speech Rendering of Electronic Documents Containing Ruby: User Requirements

https://www.w3.org/news/2026/group-note-draft-ruby-tts-req/

日本語や中国語（CJK）における、ルビを含む電子文書の音声合成に関するドキュメントの Draft が公開されました。

ルビの様々な用例を提示し、どのように読み上げられるべきなのかという戦略がまとめられています。

### Interop 2025: A year of convergence | WebKit

https://webkit.org/blog/17808/interop-2025-review/

Interop 2025 のまとめ記事です。

2025 年の始めには、主要ブラウザで全て合格したテストが 29% だったのが、年末には 97% になり、伸び率としては Safari が最も大きかったとのことです。

特に成長のあった分野として、Anchor Positioning、View Transition、及び Navigation API がピックアップされています。

### Blink: Intent to Prototype: Allowing JS web-requesting APIs to modify the User-Agent header

https://groups.google.com/a/chromium.org/g/blink-dev/c/GH3D30pY6uQ/m/wc6dmQaKAwAJ

Blink の実装において、`fetch()` で `User-Agent` をヘッダーに設定できるようになりました。仕様自体は、2015 年の更新で禁止ヘッダーリストから `User-Agent` が削除されていたのですが、今までは CORS の実装に関する理由により、対応できていなかったとのことです。

### Open Web Docs Impact and Transparency Report 2025

https://openwebdocs.org/content/reports/2025/

Open Web Docs は外部組織と連携しながら、Web に関するドキュメントの管理や更新、改善などに取り組んでいます。

記事では MDN や Browser Compat Data での取り組みを始めとし、W3C の Documentation Community Group の立ち上げ、セキュリティに関するドキュメントの作成、CSS module プロジェクト、ACD など様々な活動が紹介されています。

### Web Frameworks Community Group が設立

https://www.w3.org/community/wfcg/

W3C に Web Frameworks Community Group が設立されました。

Web フロントエンドフレームワークに組み込める共通パターンについて議論し、ブラウザベンダーにフィードバックできるような場にするとのことです。

[設立者である Keith の投稿](https://bsky.app/profile/keithamus.social/post/3mfa2bfqac22b)

### Save the date for BlinkOn 21!

https://groups.google.com/a/chromium.org/g/blink-dev/c/k3qAz6ADbik

Chrome や Edge ブラウザのレンダリングエンジンである Blink に関するイベント、BlinkOn の開催が決定しました。今年は 4/20-21 で開催されるようです。

### The Power of 'No' in Internet Standards

https://www.mnot.net/blog/2026/02/13/no

Mark Nottingham による、インターネット標準における「No」の力についての記事です。
標準プロセスにおいて最も強力な権力行使は、仕様の方向性を誘導することではなく、チョークポイント（OS やブラウザ）を握る企業が仕様の実装を「拒否」できることだとしています。標準そのものには大した力はなく、力は実装・展開・利用の側にあるためです。
結果として Web は、成功したプロプライエタリなアプリを公共財として再発明する場になれておらず、低レベル API の上にサイロが積み上がる状態になっていると指摘しています。

### HTTP responses should be able to indicate that a resource cannot be prefetched/prerendered

https://github.com/WICG/nav-speculation/issues/138

Speculation Rules の文脈で、リソースの prefetch や preload のリクエストをサーバーが拒否していることを表す HTTP ステータスコードを定義したいという案です。

2022 年から議論されているようですが、最近議論が再開しているようです。詳しくは以下の記事をご覧ください。

[新しい HTTP ステータスコード「4xx Preliminary Request Denied」の提案仕様 - ASnoKaze blog](https://asnokaze.hatenablog.com/entry/2026/02/26/003647)
