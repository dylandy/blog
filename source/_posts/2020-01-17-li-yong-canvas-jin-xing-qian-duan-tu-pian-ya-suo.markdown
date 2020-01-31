---
layout: post
title: "利用 canvas 進行前端圖片壓縮"
date: 2020-01-17 22:24:19 +0800
comments: true
categories: ["javascript", "front-end", "canvas", "image processing"]
---

最近工作上剛好遇到一個需要調校效能的站，在過程中我們運用了許多不同的方法將整體的營運效能往上大幅度的提升，在未來幾篇文章我會討論有關如何提升前端效能的方法，雖然都有些老生常談，但是希望可以給初學者一些方向，當網站效能需要最佳化時可以想到這些方法。


許多人在實作 [CMS（Content Management System）](https://en.wikipedia.org/wiki/Content_management_system)時，對於儲存文章內部的圖片時，常常很直觀的透過 JavaScript 在前端將其轉成 base64 編碼之後，與文章本文一起變成一巨大字串傳至後端後直接儲存在資料庫內，這樣的會嚴重的造成效能上的問題，首先是無法進行 [lazy loading](https://developers.google.com/web/fundamentals/performance/lazy-loading-guidance/images-and-video)，其次是會使得 api request 封包非常巨大，需要花非常多時間下載，造成使用者等待時間較長，在這樣的狀況下，若是只能調校前端使其效能能夠提升上去，我們可以從壓縮圖片著手，將圖片壓縮之後，api request 容量減少，可以有效的提升下載效率與畫面渲染時間，提升使用者體驗。

<!-- more -->

以下是一個簡單透過 canvas 來進行圖片壓縮的方法。


首先，加入一個我們工作的 canvas tag，這個 canvas 可見與不可見(`display: none`)皆行，視情形調整是否需要顯示給使用者檢視，並加上一個需要使用者上傳的 input tag，我們會需要一個 image tag 用來暫存使用者上傳的檔案，一樣，這個 image tag 也可以設定為使用者不可見。

{% highlight html %}
<img src="" class="before" style="display: none"/>
<input type="file" name="upload-image" id="upload-image" required />
<canvas style="display: none"></canvas>
{% endhighlight %}

我們需要針對這個 input tag 進行事件的設定，當有檔案被上傳之後，我們會將其用 [Base64](https://en.wikipedia.org/wiki/Base64) 的方式寫入 canvas 裡面，並將其壓縮成其他不同的格式，下面我們示範轉換成 [WebP 格式](https://developers.google.com/speed/webp)，你可以視需求轉換成你所需要的格式。


{% highlight js %}
document.querySelector("input[name=upload-image]")
  .addEventListener("change", (event) => ReadAndCompress(event));
{% endhighlight %}

注意到我們上面呼叫了另外一個 `function ReadAndCompress` 並將 event 傳入，我們在另外一個地方定義了此 function，透過 `FileReader` 這個內建的檔案讀取 `class` 將檔案讀出之後，將並將其寫入 canvas 內，並將整個 canvas 的內容透過 canvas 的內容轉成我們所想要的檔案格式，這邊我們轉成 WebP。


{% highlight js %}
const ReadAndCompress = (e) => {
  console.log(`Before Compression: ${(e.target.files[0].size/(1000*1024)).toFixed(2)} MB`);
{% endhighlight %}

首先透過 `e.target.files[0].size` 可以取得檔案的大小（注意單位為 K)，我們先把它印出來作為壓縮前的大小對照。

{% highlight js %}
 const reader = new FileReader();
 reader.readAsDataURL(e.target.files[0])
{% endhighlight %}

新增一個 FileReader 並指定檔案以 Base64 的方式讀取。

{% highlight js %}
reader.onload = event => {
    const img = document.querySelector("img.before");
    img.src = event.target.result;
    img.onload = () => {
      const width = img.width;
      const height = img.height;
      const elem = document.querySelector('canvas');
      elem.width = width;
      elem.height = height;
      const ctx = elem.getContext('2d');
      ctx.drawImage(img, 0, 0, width, height);
      const webp = ctx.canvas.toDataURL("image/webp", 0.5);
{% endhighlight %}

接著我們新增一個 onload 事件，並在確定 loading 結束之後，將圖片寫進 image tag 裡面，這時候，圖片會以剛剛提到的 Base64 格式寫入，並在這個圖片的 onload 事件被觸發，也就是已經確定載入完成之後，我們可以將這張圖片寫入 canvas 來進行壓縮，為了確保壓縮的圖片大小仍舊與原本圖片相同，我們 canvas tag 本身不設定寬高，寬高在圖片載入後設定上去，我們可以使用 [`drawImage`](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage) 函式將 Base64 圖片直接寫入 canvas 裡面，之後，我們就能透過 [`ctx.canvas.toDataURL`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toDataURL) 函式將 canvas 的內容轉成 Base64 輸出，`toDataURL` 的格式如下： `toDataURL(type, option)` ，一般 `option` 即是輸出品質，在 [MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toDataURL) 上有提到預設 type 為 png ，若是選定的格式在這個瀏覽器上不支援的話，輸出則會自動變為 png，以上就是一個簡單的利用前端來壓縮圖片的小技巧，下面是整個範例 code，提供參考。


<p class="codepen" data-height="265" data-theme-id="dark" data-default-tab="js,result" data-user="dylandy" data-slug-hash="YzPovQz" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Frontend Image Compress">
  <span>See the Pen <a href="https://codepen.io/dylandy/pen/YzPovQz">
  Frontend Image Compress</a> by dylandy.chang (<a href="https://codepen.io/dylandy">@dylandy</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

{% highlight html %}
<!doctype html>
<html>
  <head>
  </head>
  <body>
    <input type="file" name="upload-image" id="upload-image" required />
    <p name="before-compression"></p>
    <p name="after-compression"></p>
    <img src="" class="before" style="display:none;"/>
    <canvas style="display: none;"></canvas>
    <img src="" class="after" style="display:none;"/>
    <script src="./index.js"></script>
  </body>
</html>
{% endhighlight %}

{% highlight js %}
const ReadAndCompress = (e) => {
  const size = `Before Compression: ${(e.target.files[0].size/(1000*1024)).toFixed(2)} MB`;
  document.querySelector("p[name=before-compression]").innerHTML = size;
  const reader = new FileReader();
  reader.readAsDataURL(e.target.files[0]);

  reader.onload = event => {
    const img = document.querySelector("img.before");
    img.src = event.target.result;
    //img.style = "display: true";
    img.onload = () => {
      const width = img.width;
      const height = img.height;
      const elem = document.querySelector('canvas');
      elem.width = width;
      elem.height = height;
      const ctx = elem.getContext('2d');
      ctx.drawImage(img, 0, 0, width, height);
      const webp = ctx.canvas.toDataURL("image/webp", 0.5);
      const imgAfter = document.querySelector("img.after");
      imgAfter.src = webp;
      //imgAfter.style = "display: true";
      const head = 'data:image/webp;base64,';
      const imgFileSize = (Math.round((webp.length - head.length)*3/4) / (1000)).toFixed(2);
      document.querySelector("p[name=after-compression]").innerHTML =
        `After Compression: ${imgFileSize} KB`;
    },
    reader.onerror = error => console.error(error);
  }
}

document.querySelector("input[name=upload-image]")
.addEventListener("change", (event) => ReadAndCompress(event));
{% endhighlight %}

