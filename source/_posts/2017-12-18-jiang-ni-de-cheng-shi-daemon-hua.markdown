---
layout: post
title: "將你的程式 Daemon 化"
date: 2017-12-18 21:21:00 +0800
comments: true
categories: ["ruby", "linux", "tutorial"]
published: false
---

實務上我們常常會希望可以在離開 terminal 的狀況下，程式仍舊持續得執行，有些比較小、比較不需要在意系統資源耗費的程式，我們常可以看到用一些奇怪的手法來達到類似的效果，像是使用 screen 或是 tmux 來將程式跑在背景中，這樣下次登入的時候程式仍舊能夠可以被叫出來，雖然看似方便，但是仍然存在許多問題，例如 tmux 和 screen 所需的記憶體不少，且我們的程式相依於 screen 或 tmux ，當 screen 或 tmux 有任何不可預期的狀況 process 被 kill 掉了的話，我們的程式也會被連帶的受到波及，因此完全獨立的背景執行勢必是需要我們追尋的目標，以下是最近工作上簡單 survey 後的一些有關於如何 Daemon 化程式進行簡單的整理。

本文章主要知識來源於 [Daemon Processes in Ruby](https://www.jstorimer.com/blogs/workingwithcode/7766093-daemon-processes-in-ruby) 的教導，希望能夠用較易理解的文字呈現出來。在寫 Daemon 之前，讓我們回頭看看一個知名的 daemonize web server 程式 [rack](https://github.com/rack/rack)。我們來看看他將目標 web server daemon 化的程式碼片段，讓我們透過這個程式碼片段來看看是否能夠從中學到如何 daemon 一支程式。

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

我們可以從 else 片段開始看一下，在 ruby 1.9 以後提供了一個 `Process.daemon` 的 method 來 daemon 化目前的程式，或許有些人會覺得，「嘖！又是一個 Ruby的黑魔法，什麼功能都要包裝在抽象的黑盒子裡面。」，如果你有興趣了解 `Process daemon` 是如何實作的話，這邊是 [Ruby MRI 的 C code](https://github.com/ruby/ruby/blob/c852d76f46a68e28200f0c3f68c8c67879e79c86/process.c#L4817-4860) 可以研究一下，在跳下去研究了一番之後，你會發現其實這段 C code 做的事情跟上面這個 if 片段做的事情本質上是一模一樣的。

ok 那我們來逐一了解一下這個程式碼片段到底做了些什麼。

## Daemonize Process

```ruby
exit if fork
```
這行 code 巧妙的運用了 fork 的特性：他會 return 兩次，一次是在父程式 (parent process) 回應子程式 (child process) 的 pid，而在子程式的部分回應 nil，因此我們可以知道在父程式的時候 fork 會是 true 而子程式會是 false ，因此父程式會被關閉，而子程式會被保留，變成孤兒程式，那麼問題來了，一個孤兒程式，既然他的父程式已經被砍掉了，那他的 ppid (parent process id) 會是多少呢？答案是 1 ，所有的孤兒程式都會被系統核心接管，而因為被系統核心接管著，我們可以保證他的生命週期是與系統核心綁在一起了，不會因為其他的程式因素而造成此程式無故的終止。

這個步驟可以讓 terminal 認為程式已經被終止了，父程式是使用者執行的程式，子程式為變成孤兒的準 daemon ，因此控制權從程式身上交還給使用者，然後執行到下面這一行：

```ruby
Process.setsid
```
執行 `Process.setsid` 會進行以下三件事情：

* 系統新增一個新的 session 並目前的程式指定為新 session 的 leader 
* 系統新增一個新的 process group 並指定目前程式為新 process group 的 group leader 
* 因為此程式與 terminal 本身處於不同的 process group ，因此此程式對於 terminal 沒有控制能力

為了了解上面三件事情，我們需要從 Linux 系統更深的方向理解起。

## Process Groups and Session Groups



## Reference
1. Daemon Processes in Ruby, https://www.jstorimer.com/blogs/workingwithcode/7766093-daemon-processes-in-ruby
1. 程序管理與 SELinux 初探, http://linux.vbird.org/linux_basic/0440processcontrol.php 

