---
title: "CSS Anchor Positioning 仕様の紹介"
emoji: "⚓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [css, html]
published: false
---

CSS Anchor Positioning は、要素の配置を制御する新しい仕様で、指定した要素（アンカー要素）の位置を基準に、要素を配置できます。
ポップオーバーやツールチップ、トーストなどをはじめとした、さまざまな UI コンポーネントの実装に利用できます。

特に CSS Anchor Positioning による要素の配置には、「配置する要素が表示領域に収まらない場合の挙動を制御できる」「Popover API などによって Top Layer に配置される要素であっても、アンカー要素を基準にその位置を制御できる」といった特徴があります。
これまでは、JavaScript を用いて要素の座標やサイズを計算し、それを元に要素を配置する方法が一般的でした。CSS Anchor Positioning を用いることで、そのような処理を CSS で行うことが可能になります。

CSS Anchor Positioning の基本的な使い方は、1枚のシートにまとめました。よろしければ参考にしてください。

![anchor-name プロパティ、position-anchor プロパティ、position-area プロパティ、position-try プロパティについてのそれぞれの説明。記事内でプロパティごと同等の内容が存在するので、そちらを参照ください。](/images/css-anchor-positioning-294aa71a7f77fc/image0-1.png)

以下、この記事では CSS Anchor Positioning の仕様内の各プロパティや値などを、より詳しく紹介します。

## アンカー要素の設定

### anchor-name ... アンカー要素の宣言

![](/images/css-anchor-positioning-294aa71a7f77fc/image1-1.png =400x)

この要素が、指定したアンカー名のアンカー要素になることを宣言します。これにより、この要素はアンカー名を通じて他の要素から参照されるようになります。
アンカー名は、`--`から始まる必要があります。

#### anchor-name の値

- `none`（初期値） ... このプロパティによる効果はない
- *`アンカー名`* ... この要素が、指定した値のアンカー名のアンカー要素となる。`--`から始まる値を指定する。

#### anchor-name の使用例

```css
.my-anchor {
  anchor-name: --my-anchor;
}
```


### position-anchor ... アンカー先の指定

![](/images/css-anchor-positioning-294aa71a7f77fc/image1-2.png =400x)

この要素がアンカー先とするアンカー要素を、アンカー名により指定します。これにより、この要素は指定したアンカー要素を基準に配置されることとなります。

#### position-anchor の値

- `auto`（初期値） ... 暗黙的なアンカー要素（後述）がある場合、その要素をこの要素のアンカー先として指定する
- *`アンカー名`* ... 指定したアンカー名の要素を、この要素のアンカー先として指定する

#### position-anchor の使用例

```css
.my-anchor {
  anchor-name: --my-anchor;
}
.my-popover {
  position-anchor: --my-anchor;
}
```


### anchor 属性（HTML） ... 暗黙的なアンカー要素の指定

:::message
2024年9月現在、anchor 属性の仕様は議論中です。
[whatwg/html #9144 Add anchor attribute](https://github.com/whatwg/html/pull/9144)
:::

HTML の `anchor` 属性を使用することでも、アンカー先を指定できます。`anchor` 属性の値には、アンカー先とする要素の `id` を指定します。これにより指定された要素は、この要素の暗黙的なアンカー要素として機能します。
`position-anchor` プロパティでアンカー先を指定しない場合、この暗黙的なアンカー要素がアンカー先となります。

#### anchor 属性の使用例

```html
<div class="my-anchor" id="anchor">⚓︎</div>

<div class="my-popover" anchor="anchor">...</div>
```


### anchor-scope ... アンカー名のスコープを指定

:::message
2024年9月現在、anchor-scope プロパティが動作するブラウザはありません。
:::

![DOMの木構造を用いて、値が "none" のときはアンカー要素がどこからでも参照できること、"アンカー名" もしくは "all" のときはアンカー要素のサブツリーからしか参照できないことを示した図。](/images/css-anchor-positioning-294aa71a7f77fc/image1-3.png)

アンカー名の参照されるスコープを指定します。このプロパティで指定されたアンカー名は、サブツリー内からのみ参照されます。
アンカー要素をコンポーネントを再利用する場合などに、アンカー名の衝突を避けられます。

#### anchor-scope の値

- `none`（初期値） ... アンカー名はどこからでも参照される
- *`アンカー名`* ... この要素のサブツリー内で宣言される指定したアンカー名は、この要素のサブツリー内からのみ参照される
- `all` ... この要素のサブツリー内で宣言されるすべてのアンカー名は、この要素のサブツリー内からのみ参照される

#### anchor-scope の使用例

```css
.my-anchor {
  /* このアンカー名のアンカー要素は、それぞれ自分の子孫からのみ参照される */
  anchor-name: --my-anchor;
  anchor-scope: --my-anchor;
}
.my-anchor .popover {
  position-anchor: --my-anchor;
}
```

![複数のアンカー要素とポップオーバーがあり、anchor-scope を指定しない場合と、 anchor-scope でスコープを制限した場合の2通りの表示のされ方のイメージが示されている。前者は最後のアンカー要素にポップオーバーがすべて集まって重なってしまっており、後者はそれぞれのアンカー要素にそれぞれのポップオーバーが配置されている](/images/css-anchor-positioning-294aa71a7f77fc/image1-4.png)

アンカー要素のコンポーネントを再利用する際など、同じアンカー名の要素が複数存在することがあります。
アンカー名のスコープを制限しない場合、そのアンカー名を通じて同名のアンカー要素すべてが参照されます。その結果、最後のアンカー要素を基準に配置されることとなります。
一方で、アンカー名のスコープを制限する場合、いずれのアンカー要素も自分の子孫からのみ参照されます。これにより、それぞれ自分の祖先のアンカー要素を基準に配置されます。


## 配置位置の制御

### anchor() 関数 ... アンカー要素の位置を参照

![](/images/css-anchor-positioning-294aa71a7f77fc/image2-1.png =480x)

`anchor()` 関数は、アンカー要素の指定した箇所の位置を参照する関数です。
`top`、`right`、`bottom`、`left` といったプロパティの値として使用でき、「配置したい要素のどの部分が、アンカー要素のどこと対応するか。」という指定ができます。

例えば、要素をアンカー要素の右上に配置する場合、次のように「下辺をアンカー要素の上辺と同じ位置に」「左辺をアンカー要素の左辺と同じ位置に」という指定をします。

![](/images/css-anchor-positioning-294aa71a7f77fc/image2-2.png)

```css
.my-popover {
  position-anchor: --my-anchor;
  position: absolute;
  bottom: anchor(top); /* 要素の下辺を、アンカー要素の上辺と同じ位置にする */
  left: anchor(right); /* 要素の左辺を、アンカー要素の左辺と同じ位置にする */
}
```

#### anchor() 関数の引数

`anchor()` は、順に「アンカー名（省略可）」「参照する箇所」「フォールバック値（省略可）」の3つの引数を指定できます。

「アンカー名」は、`--`から始まる値で、省略可能です。
省略した場合、`position-anchor` プロパティで指定されたアンカー要素、もしくは暗黙的なアンカー要素が参照されます。指定した場合、そのアンカー名のアンカー要素が参照されます。

「参照する箇所」は、次の値が使用できます。
- `top`,`right`,`bottom`,`left` ... それぞれアンカー要素の上辺、右辺、下辺、左辺の位置を参照する
- `start`, `end` ... 論理的な値。包含ブロックの書字モードに従って、指定されたプロパティと同じ軸（top, bottom なら垂直軸、left, right なら水平軸）について、開始位置もしくは終了位置を参照する。
- `self-start`, `self-end` ... 論理的な値。自身の書字モードに従って、`srart`, `end` と同様に開始位置または終了位置を参照する。
- *`パーセント値`* ... 指定されたプロパティと同じ軸について、アンカー要素の辺の長さに対する位置を参照する
- `center` ... アンカー要素の辺の中央を参照する。`50%` と指定した場合と同等。
- `inside`, `outside` ... 指定されたプロパティと同じ側（`top` ならアンカー要素の上辺）、または反対側（`top` ならアンカー要素の下辺）を参照する。

「フォールバック値」は、参照する箇所の位置が取得できない場合に使用される値です。省略可能です。
アンカー要素が削除された場合などに、この値が使用されます。

#### anchor() 関数についての補足
anchor() 関数が使用できるのは、`top`, `right`, `bottom`, `left` プロパティと、論理的な値の `inset-block-start`, `inset-block-end`, `inset-inline-start`, `inset-inline-end` プロパティ、これらを一括して指定する `inset`, `inset-block`, `inset-inline` プロパティ内です。

`anchor()` 関数は *`<length>`* を返すため、`calc()` 関数や`clamp()` 関数と組み合わせて使用できます。

Popover API で展開される要素は、ユーザーエージェントスタイルシートによって `inset: 0;` が適用されることがあります。この場合、 `inset: auto;` などを指定してスタイルをリセットする必要があります。

#### anchor() 関数の使用例

```css
.my-anchor {
  anchor-name: --my-anchor;
}

.my-popover {
  position-anchor: --my-anchor;  
  position: absolute;
  top: anchor(bottom);
  left: anchor(right);
}

.my-popover2 {
  position-anchor: --my-anchor;  
  position: absolute;
  top: anchor(10%); /* パーセント値を指定する */
  left: calc(anchor(left) - 10px); /* calc 関数を用いる */
}

/* anchor 関数内でアンカー要素を指定する */
.my-popover3 {
  position: absolute;
  top: anchor(--my-anchor, center);
  left: anchor(--my-anchor, right);
}

/* フォールバックする値を指定する */
.my-popover2 {
  position: absolute;
  top: anchor(--my-anchor, bottom, 10%);
  left: anchor(--my-anchor, right, 200px);
}
```

### anchor-center 値 ... アンカー要素を基準とした中央寄せ

![](/images/css-anchor-positioning-294aa71a7f77fc/image2-3.png =480x)

`anchor-center` 値は、`justify-self`, `align-self`, `justify-items`, `align-items` プロパティに指定できる値で、要素をアンカー要素の中央を基準に配置します。
これにより、アンカー要素を基準とした中央寄せが簡単に行えます。

#### anchor-center 値の使用例

```css
.my-popover {
  position: absolute;
  position-anchor: --my-anchor;
  top: anchor(bottom);
  justify-self: anchor-center; /* アンカー要素の中央に配置 */
}
```

### position-area ... 配置される領域の指定

![](/images/css-anchor-positioning-294aa71a7f77fc/image2-4.png =480x)

`position-area` プロパティは、要素をアンカー要素の周囲のどの領域に配置するか指定するプロパティです。
アンカー要素の周囲を「右上」「真上」「左上」...の8つの領域に区切り、要素が占める領域を指定します。1つもしくは隣り合う複数の領域を指定できます。
top, right 等のプロパティを記述するよりもシンプルに、アンカー要素を基準にした位置を指定できます。

このプロパティは以前、`inset-area` という名前で提案されていました。

#### position-area の値

- none（初期値） ... このプロパティによる効果はない
- *`キーワード値`* ... 要素はキーワード値の当てはまる領域に配置される。主な指定のしかたは次を参照。

![アンカー要素に対してどの位置に要素を配置するかによって、それぞれの値が示されている。全部で20通り。左上・上・右上は "top"、左上・上は "top span-left"、上・右上は "top span-right"、上のみは "top center"。右上と右と右下は "right"、右上と右は "right span-top"、右と右下は "right span-bottom"、右のみは "right center"。左下と下と右下は "bottom"、左下と下は "bottom span-left"、下と右下は "bottom span-right"、下のみは "bottom center"。左上と左と左下は "left"、左上と左は "left span-top"、左と左下は "left span-bottom"、左のみは "left center"。左上のみは "top left"、右上のみは "top right"、右下のみは "bottom right"、左下のみは "bottom left"。](/images/css-anchor-positioning-294aa71a7f77fc/image3-2.png)

#### position-area についての補足

値は、次のキーワードの1つか2つの適当な組み合わせになります。
> `start` `end`, `self-start`, `self-end`, `top`, `bottom`, `left`, `right`, `y-start`, `y-end`, `y-self-start`, `y-self-end`, `x-start`, `x-end`, `x-self-start`, `x-self-end`, `block-start`, `block-end`, `block-self-start`, `block-self-end`, `inline-start`, `inline-end`, `inline-self-start`, `inline-self-end`, `center`, `span-start`, `span-end`, `span-top`, `span-bottom`, `span-y-start`, `span-y-end`, `span-x-start`, `span-x-end`, `span-block-start`, `span-block-end`, `span-inline-start`, `span-inline-end`, `span-block-self-start`, `span-block-self-end`, `span-inline-self-start`, `span-inline-self-end`, `span-all`

`span-**` というキーワードは、中央も含めた隣接する2つの領域を占めることを意味します。例えば `top span-right` はアンカー要素の真上とその右側の2つの領域を指します。
`span-all` というキーワードは、隣接する3つの領域すべてを占めることを意味します。さらに、単に `top`, `right` と指定することは `top span-all`, `right span-all` と同じ意味になります。

論理的な値について、`x-start`, `span-y-end` といった`x` や `y` が含まれるキーワードは、ブロック方向・インライン方向を指定する代わりに、方向を直接指定するキーワードです。
`inline-self-start`, `span-block-self-end` といった `self` が含まれるキーワードは、自身の書字モードを参照するキーワードです。

#### position-area の使用例

```css
.my-popover {
  position-anchor: --my-anchor;
  position: absolute;
  position-area: top span-right; /* アンカー要素の右上に配置 */
}
```

## 表示領域に収まらない時の制御

### position-try-fallbacks プロパティ ... 表示領域に収まらない時の挙動の指定

![](/images/css-anchor-positioning-294aa71a7f77fc/image3-1.png =480x)

`position-try-fallbacks` プロパティは、要素が包含ブロックに収まらない場合、どのように表示位置をフォールバックするか指定します。
初期表示時だけでなく、スクロールやリサイズ等で収まらなくなった場合も、表示位置がフォールバックされます。

このプロパティは以前、`position-try-options` という名前で提案されていました。

#### position-try-fallbacks の値

- `none`（初期値） ... このプロパティによる効果はない
- `flip-block`, `flip-inline`, `flip-start` ... それぞれブロック方向、インライン方向、両方向について表示位置を反転させる。
- *`position-area に指定できる値`* ... 指定した position-area の値が適用される。
- *`@position-try で作成したルール`* ... 指定した @position-try ルール（後述）が適用される。さらに、ルール名の後に`flip-block`, `flip-inline`, `flip-start` を指定すると、それぞれの軸について反転される。

`none` 以外の値について、カンマ `,` 区切りで複数の値を指定できます。指定した順に、収まる位置が見つかるまでフォールバックが試行されます。（この順序は、後述の `position-try-order` プロパティで制御できます。）

`flip-**` を指定した際の挙動のイメージは、次のとおりです。
!["flip-inline"、"flip-block"、"flip-start" の3つの場合について、要素の表示位置がどのように変わるか示した図。"flip-inline" の場合、表示位置がインライン方向に反転されている。"flip-block" の場合、表示位置がブロック方向に反転されている。"flip-start" の場合、表示位置がブロック方向とインライン方向の両方に反転されている。](/images/css-anchor-positioning-294aa71a7f77fc/image3-3.png)

#### position-try-fallbacks の使用例

```css
.my-popover {
  position-anchor: --my-anchor;
  position: absolute;
  position-area: top span-right;
  position-try-fallbacks: flip-inline; /* インライン方向について反転させる */
}

.my-popover2 {
  position-anchor: --my-anchor;
  position: absolute;  
  position-area: top span-right;
  /* まずインライン方向について反転させ、それでも収まらない場合ブロック方向について反転させる */
  position-try-fallbacks: flip-inline, flip-block;
}

.my-popover3 {
  position-anchor: --my-anchor;
  position: absolute;
  position-area: top span-right; 
  /* position-area に指定できる値を指定して、アンカー要素の下側に配置する */
  position-try-fallbacks: bottom;
}
```

### position-try-order

![](/images/css-anchor-positioning-294aa71a7f77fc/image3-4.png =480x)

`position-try-order` プロパティは、`position-try-fallbacks` プロパティで指定したフォールバックの試行順序を指定します。
この要素の包含ブロックのサイズを、 `position-try-fallbacks` プロパティで指定したそれぞれの値について計算し、その大きさによって適用される順序を制御します。

#### position-try-order の値

- `normal`（初期値） ... `position-try-fallbacks` プロパティで指定した順序で試行される
- `most-width` ... 幅が大きくなるような順序で試行される
- `most-height` ... 高さが大きくなるような順序で試行される
- `most-block-size` ... 論理的な値。ブロック方向のサイズが大きくなるような順序で試行される。
- `most-inline-size` ... 論理的な値。インライン方向のサイズが大きくなるような順序で試行される。

#### position-try-order の使用例

```css
.my-popover {
  position-anchor: --my-anchor;
  position: absolute;
  position-area: top span-right;
  position-try-fallbacks: flip-block, flip-inline;
  position-try-order: most-width; /* 幅が大きくなるような順序で試行 */
}
```

![position-try-order を指定しない場合と "most-width" を指定した場合の2通りのイメージが示されている。双方、アンカー要素の右上、右下、左上のポップオーバーが配置されており、それぞれA、B、Cのラベルが付けられている。このうちAとBのポップオーバーは、Cに比べて横幅が狭くなっている。前者のイメージでは、Aのポップオーバーが最も前面に表示されており、後者のイメージではCのポップオーバーが最も前面に表示されている。](/images/css-anchor-positioning-294aa71a7f77fc/image3-5.png)

上記の例では、要素を配置する位置として、
A. もとの指定（`top span-right`）
B. `flip-inline` した結果（`top span-left`）
C. `flip-block` した結果（`bottom span-right`）
のいずれかになることが考えられます。
`position-try-order` プロパティを指定しない場合、記述順に従って、A, B, C の順に試行されます。つまり、もとの指定 A が表示領域に収まればそれが採用され、そうでなければ B, それも収まらなければ C が採用されます。
`position-try-order: most-width` が指定されている場合では、右側に十分なスペースが無いなどで C の幅が最も大きくなるときは、C, A, B の順に試行されます。結果、C が表示領域に収まればそれが採用されます。

### position-try ... position-try-fallbacks と position-try-order をまとめて指定

`position-try` プロパティは、`position-try-fallbacks` プロパティと `position-try-order` プロパティをまとめて指定する短縮プロパティです。

#### position-try の値

`position-try-order` の値、`position-try-fallbacks` の値の順に指定します。
`position-try-order` の値は省略できます。

#### position-try の使用例

```css
.my-popover {
  position-anchor: --my-anchor;
  position: absolute;
  position-area: top span-right;
  position-try: most-width flip-block, flip-inline;
}
```

### @position-try ルール

`@position-try` ルールは、要素の配置に関するプロパティをまとめて、名前付きで定義したものです。定義したルールは、`position-try-fallbacks` プロパティの値として指定できます。
`flip-**` や `position-area` の値では対応できないような複雑な配置にフォールバックできます。

ルールの名前は、 `--` から始まる値を指定します。

#### @position-try ルール内で使用できるプロパティ

- `position-area`
- `position-anchor`
- `top`, `right`, `bottom`, `left` と、 `inset-block-start` などそれに対応する論理プロパティ
- `margin`（`margin-top` などを含む）
- `width`, `height`（`min-width` などを含む）
- `align-self`, `justify-self`

#### @position-try ルールの使用例

```css
.my-popover {
  position-anchor: --my-anchor;
  position: absolute;
  position-area: right;
  position-try: --popover-top-rule, --popover-bottom-rule;
}

@position-try --popover-top-rule {
  margin-bottom: 10px;
  position-area: bottom;
}

@position-try --popover-bottom-rule {
  margin-top: 10px;
  position-area: bottom;
}
```

## その他

### anchor-size() 関数 ... アンカー要素のサイズを取得

![](/images/css-anchor-positioning-294aa71a7f77fc/image4-1.png =540x)

`anchor-size()` 関数は、アンカー要素のサイズを取得する関数です。要素のサイズをアンカー要素のサイズに関連付けて指定する際などに使用します。
`anchor-size()` 関数は *`<length>`* を返すため、`calc()` 関数や`clamp()` 関数と組み合わせて使用できます。

#### anchor-size() 関数の引数

`anchor-size()` 関数は、順に「アンカー名（省略可）」「参照する対象（省略可）」「フォールバック値（省略可）」の3つの引数を指定できます。

「アンカー名」は、`--`から始まる値で、省略可能です。
省略した場合、`position-anchor` プロパティで指定されたアンカー要素、もしくは暗黙的なアンカー要素が参照されます。指定した場合、そのアンカー名のアンカー要素が参照されます。

「参照する対象」は、次の値が使用できます。
- `width`, `height` ... それぞれアンカー要素の幅、高さを参照する
- `block`, `inline` ... 論理的な値。包含ブロックの書字モードをもとに、ブロック方向、インライン方向のサイズを参照する。
- `block-start`, `block-end`, `inline-start`, `inline-end` ... 論理的な値。自身の書字モードをもとに、ブロック方向、インライン方向のサイズを参照する。

「参照する対象」も省略可能で、省略した場合はプロパティに対応する軸が指定されたものとして扱われます。例えば `width: anchor-size();` は `width: anchor-size(width);` と同じ意味になります。

「フォールバック値」は、アンカー要素のサイズが取得できない際に使用される値です。省略可能です。
アンカー要素が削除された場合などに、この値が使用されます。

#### anchor-size() 関数の使用例

```css
.my-popover {
  position-anchor: --my-anchor;
  width: calc(anchor-size(width) * 4); /* アンカー要素の幅の4倍の幅 */
  height: anchor-size(height); /* アンカー要素の高さ */
}

/* アンカー名、フォールバック値をともに指定 */
.my-popover2 {
  width: anchor-size(--my-anchor, width, 40px);
  height: anchor-size(--my-anchor, height, 200px);
}
```

### position-visibility ... 表示・非表示の制御

`position-visibility` プロパティは、スクロールなどによりアンカー要素が表示領域に収まらなくなった場合や、自身が表示領域に収まらない場合の、要素の表示・非表示を制御します。

#### position-visibility の値

- `always` ... このプロパティによる効果はない。この要素は、アンカー要素が隠された場合や、自身が表示領域に収まらない場合も表示される。
- `anchors-valid` ... アンカー先となるアンカー要素が存在しない場合、この要素は非表示になる
- `anchors-visible`（初期値） ... アンカー要素が表示領域に収まらない場合、この要素は非表示になる
- `no-overflow` ... この要素が、`position-try` の試行後も表示領域に収まらない場合、この要素は非表示になる

次の図は、 `always`, `anchors-visible`, `no-overflow` の挙動の違いのイメージです。

![](/images/css-anchor-positioning-294aa71a7f77fc/image4-2.png)

#### position-visibility の使用例

```css
.my-popover {
  position-anchor: --my-anchor;
  position-visibility: no-overflow;
}
```

## 考慮事項

ここまで紹介したように、CSS Anchor Positioning は要素を配置する便利な機能を提供します。
しかし、これらはあくまで要素を配置するための機能であり、アンカー要素とそれを基準に配置される要素の間になにかしらの意味的な関係性を提供するわけではありません。また、アンカー要素と配置される要素間のフォーカスの移動、キーボード操作など、考慮すべき点は多くあります。
HTML（必要に応じて WAI-ARIA）で適切にマークアップし、これらのセマンティクスや機能を適切に提供することを忘れないようにしましょう。

## おわりに

この記事では、CSS Anchor Positioning の仕様について紹介しました。
この仕様の策定が進む背景には、Dialog 要素 や Popover API の仕様が挙げられます。 現状では CSS Anchor Positioning の各ブラウザの実装状況は揃っていませんが、この dialog や popover の進み具合を鑑みるに、実用できる日も遠くなさそうに感じます。
ポップオーバーやツールチップなど、この仕様が活かせそうな UI コンポーネントを実装する際には、ぜひ活用してみてください。

## 資料

- [CSS Anchor Positioning Editor’s Draft](https://drafts.csswg.org/css-anchor-position-1/)
- [CSS Anchor Positioning Editor’s Draft（日本語訳）](https://triple-underscore.github.io/css-anchor-position-ja.html)
- [Chrome for Developers | Blog | CSS アンカー ポジショニング API のご紹介](https://developer.chrome.com/blog/anchor-positioning-api?hl=ja)
- [ポップアップが画面内に収まらない場合に自動的に表示位置を調整する CSS Anchor Positioning](https://azukiazusa.dev/blog/css-anchor-positioning-294aa71a7f77fc/)
