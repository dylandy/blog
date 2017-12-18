---
layout: post
title: "設定 public key 進行 SSH 連線"
date: 2016-05-20 19:06:07 +0800
comments: true
categories: ["Linux","server admin"]

---

&nbsp;&nbsp;&nbsp;&nbsp;身為一個宅宅工程師，對於使用 ssh 連線登入遠端主機是家常便飯的事情，但是也常常會因為 ssh port 被一些惡意程式攻擊，導致主機的帳號密碼被猜到，從而被迫換機器或是全機重灌等等的悲劇。
有些系統工程師為了避免這樣的風險，把 ssh port 從預設的 22 port 搬移到其他的 port 上，希望能夠躲過大部分的機器人掃描，並且要求使用者的密碼要足夠複雜，但是相對於這些方法，使用 public key 來登入 ssh 主機似乎是最安全的作法，首先，足夠長度的 key length 能夠有效防止攻擊者的攻擊，並且以目前的科技與已知的演算法，並沒有有效的方法進行 key 破解，[畢竟 key 的拆解](https://en.wikipedia.org/wiki/Integer_factorization)是一個 [NP-hard 的問題](https://en.wikipedia.org/wiki/NP-hardness)，因此能夠使用 public key 來進行連線登入的話是一個較為安全的連線方法。
<!--more-->
*  要使用 public key 登入遠端伺服器的第一步，就是先產生 client 端的 public key，可以參考 [Github 的範例文件](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)
<script src="https://gist.github.com/dylandy/56e5ed043d616a892d0e82109910b99b.js"></script>
這邊建議使用 2048 bit 以上的安全加密，目前現行的商用網站加密層級至少都達到了 2048 bit ，因此，為了安全性考量，4096 bit 是很有必要的。<br>

* 使用 ssh-copy-id 工具將我們剛剛產生的 public key 上傳到伺服器上，這個程式會在伺服器上使用者的家目錄下 .ssh 資料夾新增一個 authorized_keys 檔案，並將 client 的 public key 存在裡面，因此如果不想使用 ssh-copy-id 的人，可以自己新增一個 authorized_keys 檔案也是可以的。
<script src="https://gist.github.com/dylandy/571ace32c9ad3d548c6ae87c002ba278.js"></script>
使用 Mac 的朋友如果要使用 ssh-copy-id 的話，需要另外安裝，這個程式沒有包含在 ssh 裡面，可以透過 brew 來安裝。
<script src="https://gist.github.com/dylandy/bade9ef40829f51499a5a1839dec8403.js"></script>

* 最後要修改 /etc/ssh/sshd_conf 這個檔案，找到 PubkeyAuthentication 將 no 改成 yes，並將 PasswordAuthentication 由 yes 改成 no，限制只能透過 public key 來進行登入
<script src="https://gist.github.com/dylandy/7b2877ce6aa9f075312b99a23ddefdcd.js"></script>
設定完成以後，請記得將 ssh 服務重開。

這樣往後就可以透過 public key 來登入了，也省下每次登入都要打一大串密碼的困擾了。
<script src="https://gist.github.com/dylandy/636aa794d2cf02e33b3e72b8721b2240.js"></script>

PS: 可以透過在 .ssh 資料夾下新增一個 config 檔案，將常用的登入設定寫進去，也可免去每次登入都要輸入一大堆 ip 和 domain 的麻煩，下面是一個簡單的範例。
<script src="https://gist.github.com/dylandy/2aa177ffcff15e10d5657f869a0270c2.js"></script>

