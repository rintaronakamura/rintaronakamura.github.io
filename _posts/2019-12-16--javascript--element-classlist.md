---
layout: post
title:  "[JavaScript] Element.classListでクラスの除去と追加を行う"
date:   2019-12-16 03:00:00 +0900
categories: javascript
---

JavaScriptでクラスの除去と追加を簡単に行えたので、コードを載せとく。

```javascript
document.addEventListener('DOMContentLoaded', () => {
  const el = document.getElementById('toggle-btn')
  el.addEventListener('click', () => {
    if (document.getElementById('hidden-contents').classList.toggle("is-hidden")) {
      el.textContent = "＋"
      return
    }
    el.textContent = "ー"
  })
})
```

```html
<button type="button" id="toggle-btn">＋</button>

<div id="hidden-contents" class="is-hidden">
  <p>hello world!</p>
</div>
```

```css
.is-hidden {
  display: none;
}
```

「＋」ボタンがクリックされるとリスナーが反応し、`document.getElementById('hidden-contents').classList.toggle("is-hidden")`で指定のDOMに`is-hidden`クラスが存在する場合は除去し、存在しない場合は追加する。戻り値はtrue,falseである。

ちなみに、今回特に画像に関する処理を行っていないのでページの初期化に`DOMContentLoaded`を利用している。
`DOMContentLoaded`イベントはDOMツリーの構築が完了した時点で発火される。


# 参考サイト

- [Element.classList - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/Element/classList)
- [DOMContentLoadedイベントとloadイベントの違い[タイミング]](https://noumenon-th.net/programming/2017/06/11/domcontentloaded/)
