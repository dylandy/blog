---
layout: post
title: "使用 inotify 進行資料夾監控"
date: 2018-06-23 14:04:20 +0800
comments: true
categories: ["inotify", "ruby", "linux", "file-monitoring", "shellscript"]
published: false
---

在撰寫系統時，要監視一個資料夾的變化情形，我們常會寫一個小程式跑一個無限迴圈，隨時監視目標資料夾的狀態，但是這樣的程式不僅浪費系統資源，佔用珍貴的 CPU 時間、對於硬碟 IO 的使用率提昇，也會因為這個程式佔用有限的硬碟頻寬，下面是一個使用無限迴圈實作的簡單資料夾監視程式。

```sh
while true
do
  PASS="$(ls -la | wc -l)"
  sleep 5
  CURRENT="$(ls -la | wc -l)"
  if test ${CURRENT} -gt  ${PASS}; then
    echo "file has been added"
  fi
done
```
<!-- more -->
在 Linux kernel 2.6 以後推出了一個稱為 inotify 的子系統 (sub-system)，他延伸了系統對於檔案系統 (filesystem) 的掌控能力，並提供 API 讓使用者的程式能夠了解到檔案系統的變化情形，因此我們若是想要監控使用者對於某個資料夾的變化情形，我們可以用各種 inotify 的 client 端 library 來進行設計，下面是一個使用 Ruby inotify wrapper - rb-inotify 寫的小程式，我們可以看到，我們可以知道使用者這個動作是什麼樣的動作，被做動的檔名與路徑，因此，我們就可以對這些檔案進行操作。

```ruby
require "rb-inotify"
notifier = INotify::Notifier.new
EVENT = [:recursive, :create, :moved_to]
notifier.watch(app_folder, *EVENT) do |event|
  puts event.name
  puts event.flags
end
notifier.run
```
因為 inotify 是 Linux Kernel 的子系統，使用 inotify 進行系統的監控不只是節省系統效能，也能將重要的 CPU 時間還給需要的程式使用，亦能減少硬碟 IO 頻寬，因此對於有監控資料夾內容的需求者，這會是一個不錯的選擇，一點小分享，提供給大家參考。

## Reference
https://en.wikipedia.org/wiki/Inotify <br>
https://github.com/guard/rb-inotify
