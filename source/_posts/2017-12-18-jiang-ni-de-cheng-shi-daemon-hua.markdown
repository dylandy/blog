---
layout: post
title: "將你的程式 Daemon 化"
date: 2017-12-18 21:21:00 +0800
comments: true
categories: ["ruby", "linux", "tutorial"]
---

實務上我們常常會希望可以在離開 terminal 的狀況下，程式仍舊持續得執行，有些比較小、比較不需要在意系統資源耗費的程式，我們常可以看到用一些奇怪的手法來達到類似的效果，像是使用 screen 或是 tmux 來將程式跑在背景中，這樣下次登入的時候程式仍舊能夠可以被叫出來，雖然看似方便，但是仍然存在許多問題，例如 tmux 和 screen 所需的記憶體不少，且我們的程式相依於 screen 或 tmux ，當 screen 或 tmux 有任何不可預期的狀況 process 被 kill 掉了的話，我們的程式也會被連帶的受到波及，因此完全獨立的背景執行勢必是需要我們追尋的目標，以下是最近工作上簡單 survey 後的一些有關於如何 Daemon 化程式進行簡單的整理。

本文章主要知識來源來自於 [Daemon Processes in Ruby](https://www.jstorimer.com/blogs/workingwithcode/7766093-daemon-processes-in-ruby) 的教導，希望能夠用較易理解的文字呈現出來。在寫 Daemon 之前，讓我們回頭看看一個知名的 daemonize web server 程式 rackup。我們來看看他將目標 web server daemon 化的程式碼片段，讓我們透過這個程式碼片段來看看是否能夠從中學到如何 daemon 一支程式。

<!-- more -->

```ruby
def daemonize_app
  if RUBY_VERSION < "1.9"
    exit if fork
    Process.setsid
    exit if fork
    Dir.chdir "/"
    STDIN.reopen "/dev/null"
    STDOUT.reopen "/dev/null" , "a"
    STDERR.reopen "/dev/null" , "a"
  else
    Process.daemon
  end
end
```
