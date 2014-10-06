class: center middle
# remark.js ことはじめ

---
class: center middle
# @harasou

---
# slide 1
１ページ目

```text
# slide 1
１ページ目
---
```

---
# slide 2
２ページ目

```text
# slide 1
１ページ目
---
# slide 2
２ページ目
 :
--
１行追記
--
１行を
--
分割して
--
追記
---
```

--

１行追記

--

１行を
--
分割して
--
追記

---
class: top left
.left[
```text
---
[未指定(class: top left)]
# slide 3
ページ全体の表示位置(左上：デフォルト)
---
```
]

# slide 3
ページ全体の表示位置(左上：デフォルト)

---
class: middle center
.left[
```text
---
class: middle center
# slide 4
ページ全体の表示位置(真ん中)
---
```
]

# slide 4
ページ全体の表示位置(真ん中)

---
class: bottom right
.left[
```text
---
class: bottom right
# slide 5
ページ全体の表示位置(右下)
---
```
]

# slide 5
ページ全体の表示位置(右下)
---
# slide 6

```text
.left[テキスト左寄せ]
```
.left[
テキスト左寄せ
]

--

```text
.center[テキスト中央]
```
.center[
テキスト中央
]

--

```text
.right[テキスト右寄せ]
```
.right[
テキスト右寄せ
]

---
layout: true
.center[ title ]
---

# slide 7

`layout: ture` にすると、そのページの内容(ここでは、title の文字列)が `layout: false` にするまでずっと表示される。

```text
---
layout: true
.center[ title ]
---

# slide 7
 :
---

# slide 8
 :
---
layout: false
# slide 9
 :
---
```

---

# slide 8

ここでも表示される

```text
---
layout: true
.center[ title ]
---

# slide 7
 :
---

# slide 8
 :
---
layout: false
# slide 9
 :
---
```
---
layout: false
# slide 9

ここで `layout: false` にしたので、title の文字列が表示されなくなる。

```text
---
layout: true
.center[ title ]
---

# slide 7
 :
---

# slide 8
 :
---
layout: false
# slide 9
 :
---
```

---
# slide 10

コードのハイライトもOK

```c
#include <stdio.h>
int main(void){
    printf("Hello World!\n");
    return 0;
}
```

---
# slide 11

独自の class も定義できる

.hoge[
hoge class が適用された文字列
]

markdown
```markdown
.hoge[
hoge class が適用された文字列
]
```

css
```css
.hoge {
  color: blue;
  font-size: 2em;
}
```

---
# slide 12
class の指定は、入れ子表記も、並列表記も可能

.left-column.g8[
.hoge[
入れ子
```text
.left-column[
.hoge[
入れ子
]
]
```
```html
<div class="left-column">
  <div class="hoge">
    <p>入れ子</p>
```
]
]
.right-column.g8.hoge[
並列
```text
.right-column.hoge[
並列
]
```
```html
<div class="right-column hoge">
  <p>並列</p>
```
]

---
# slide 13

css を記載すれば、２列表示も可能

.left-column.g8[
## markdown
```text
.left-column[
## markdown
 :
]
.right-column[
## css
 :
]
```
]

.right-column.g8[
## css
```css
/* Two-column layout */
.left-column {
  width: 50%;
  float: left;
}
.right-column {
  width: 45%;
  float: right;
}
```
]

---
class: center middle
# おしまい
