---
layout: post
title: "讓我們討論一下：JavaScript Event Loop"
date: 2019-12-26 15:28:23 +0800
comments: true
categories: ["javascript", "event loop", "vm"]
published: false
---

最近在工作上面頻繁的使用 Node.js，雖然以前在開發前端有使用 JavaScript 的經驗，但是對於 JavaScript 的一些特性在這幾年中仍是不求甚解的，對於 JavaScript 的[一些奇怪的行為](https://www.google.com/search?q=javascript+what+the+fuck&client=firefox-b-d&sxsrf=ACYBGNRxxqeZNR62CPMhSK4QvPbxv6xMOg:1578734050768&source=lnms&tbm=isch&sa=X&ved=2ahUKEwiJvanSmvvmAhVAx4sBHUgvCqAQ_AUoAXoECAsQAw&biw=1280&bih=703&dpr=2)，總是用其他語言的想法來思考最後總是無奈的 workaround，前一段時間，花了一點空閒時間稍微了解了一下 JavaScript 底層到底做了什麼，這篇文章將討論 JavaScript 底層最重要的機制之一 Event Loop 的流程如何影響到 JavaScript 的執行行為。

<!-- more -->

相信很多人在剛開始接觸 JavaScript 的時候，對於異步行為是非常陌生與困惑的，到底為什麼一個號稱[「單執行緒」](https://dev.to/steelvoltage/if-javascript-is-single-threaded-how-is-it-asynchronous-56gd)的程式與語言，能夠以不卡住主執行緒的方式來執行，對於其他程式語言的使用者來說，這是非常不合理的行為，到底 JavaScript 的底層是怎麼處理這樣的行為呢？

首先，讓我們先看一下下面這段 code。

```js
console.log("before time out");
setTimeout(() => { console.log("hello")}, 0);
console.log("after time out");
```

讓我們來作個小測驗吧，以上的程式碼的執行結果如何呢？
<div name="first_ans" style="background: #ebebeb; display: flex; padding: 1em; margin-bottom: 10px; justify-content: space-around; cursor: pointer">
  <code style="display: inline-flex; background: none; border: 0">
    => before time out<br>
    => hello<br>
    => after timeout
  </code>
</div>
<div name="second_ans" style="background: #ebebeb; display: flex; padding: 1em; justify-content: space-around; cursor: pointer">
  <code style="display: inline-flex; background: none; border: 0" >
    => before time out<br>
    => after time out<br>
    => hello
  </code>
</div>

你答對了嗎？如果是對於 JavaScript 不太熟的朋友，這邊幫你解釋一下上面的程式片段。[`console.log`](https://developer.mozilla.org/en-US/docs/Web/API/Console/log) 是 JavaScript 中內建的將文字印在終端機上的功能，若是在瀏覽器上，則會將文字印在 debug 工具裡，而 [`setTimeout`](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout) 則是在 JavaScript 中的一個內建 timer ，它可以設定一個定時器（以 ms 為單位），並在時間結束之後執行所傳入的 lambda function，因此，上面的範例片段中的 `setTimeout` 片段 `setTimeout(() => console.log("hello"), 0)` 意味著「在 0ms 之後，至終端機上印出 "hello" 字串」


對於從其他同步行為語言出身的人來說，直覺的會想說是第一個選項是很正常的，像是 JavaScript 這樣的執行行為，在前面跟我們說了這是「單執行緒」的程式語言來說，會覺得非常奇怪，因為同步行為語言來說，第二個選項的行為幾乎都是建立在多執行緒的條件下才能完成。嚴格來說，這樣的想法，也沒有錯啦，只是這個多執行緒發生的地點有所不同，下面就讓我試著解釋看看了。


JavaScript 是個 [JIT](https://en.wikipedia.org/wiki/Just-in-time_compilation) 的程式語言，程式語言在執行的時候會先編譯過後，被執行在其虛擬機器上，

<script>
  // simple quiz
  const first_ans = document.querySelector("div[name=first_ans]");
  const second_ans = document.querySelector("div[name=second_ans]");
  first_ans.addEventListener("click", (e) => {
    first_ans.style.backgroundColor = "#d9534f";
    second_ans.style.backgroundColor = "#ebebeb";
  });
  second_ans.addEventListener("click", (e) => {
    first_ans.style.backgroundColor = "#ebebeb";
    second_ans.style.backgroundColor = "#5cb85c";
  });
</script>
