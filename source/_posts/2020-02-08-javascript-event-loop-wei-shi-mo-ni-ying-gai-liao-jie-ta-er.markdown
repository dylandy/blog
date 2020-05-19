---
layout: post
title: "JavaScript Event Loop - 為什麼你應該了解它（二）"
date: 2020-02-08 17:45:50 +0800
comments: true
categories: ["javascript", "event loop", "vm"]
---

上一章，我們知道 JavaScript 是一個需要跑在虛擬機器上的程式語言，而在 JavaScript 虛擬機器執行 JavaScript 程式時就像一般的 CPU ，每個指令都會有一個額定的時間區間可以執行，在真實的 CPU 上是 clock，在 JavaScript 虛擬機器裡面則是 tick，不同的指令會有不同長度的 ticks，有了以上的認知之後，我們就可以開始討論我們的題目，Event Loop 了。在上一章我們提到，Event Loop 是 JavaScript 虛擬機器在執行不同程式片段的決策方法，也是 JavaScript 的非同步行為的主要造成原因，下面就讓我們來好好聊聊 JavaScript Event Loop 與還有非同步的問題。


