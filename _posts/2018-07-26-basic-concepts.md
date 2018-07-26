# Basic Concept

### Nuts-and-Bolts Description


* 凡是掛載在網路上的任何裝置 (Webcam, automobiles, laptops, etc.) 都稱為 **end system** 或 **host**，一個以上的 end system / host 組成網路 (Internet)
* host 彼此之間以 connection links 和 packet switches 互相連結：
    * connection links 以型態分類，有分成有線的實體纜線、無線的訊號傳播，不同的連結方式傳輸速度有所不同，單位時間能傳輸的資料比率我們稱為 **transmittion rate**
    * Packet Switches 在之後的文章會有更詳細的說明，基本上 Packet Switches 是一種接收 Packet 之後轉送 Packet 到正確的 host / destination 去的裝置，具有很多形式，但一般而言最常見的就是 router 和 link-layer switch (像是 Cisco 的 layer-2 switch)，他們都是一種 packet switch。

![](https://i.imgur.com/PeZZ00p.png)


* 基本上將裝置互連廣義來說就是一個網路了，但是若要連上我們大家所在的網際網路，必須藉由網路服務提供者才能連上，而網路服務提供者簡稱為 **ISP (Internet Service Providers)**， ISP 依據他所能提供的網路服務規模有不同的等級分別：例如說在台灣的中華電信就是屬於 residential ISPs、大學的學術網路則是 University ISPs (粗略分類的話)
* 傳說中的網路分七層，指的是 OSI/ISO 規範的一種標準格式，分類在不同領域的網路服務協定，藉由分層達到易於維護的作用，但就現今我們常用的概念是由這個標準格式簡化而來的 TCP/IP 五層架構，**因為目前網路服務中最重要的協定主要由 TCP 協定和 IP 協定主宰**，之後會有更詳細的說明

### Protocols

* Protocol ，中文譯作協定，以簡略的方式說明就是一個描述資料傳輸時的方法、規定與限制的規範，像前文提到的 TCP 協定，裡面規範了許多規則以達到封包不易丟失的作用 (當然 TCP 不只保障封包不丟失喔，還有另外兩種功能)。
* 而每一種 Protocol 的規定內容會被寫入一個文件由 IETF 這個組織保存與維護，這個文件簡稱 **RFC (Request for contents)**，每個協定都會有一個專屬編號，格式為 RFC + 數字，像是 HTTP/1.1 就是 RFC 2616，因為愚人節而生的鴿子封包則是 RFC 1149，對任何協定有興趣都可以用他們的 RFC 編號搜尋，內容和 ISO 的 C Standard 長得很像。

### 常見的傳輸方式

* DSL (Digital Subscriber Line)：這是一種比較舊式的傳輸方式，是基於電話網路為架構，如果裝設分流，一條作為電話用的纜線、而一條作為電腦上網用的纜線 (電腦會先接上 modem ，俗稱小烏龜，轉換後的訊號才會與電話線匯流)，這樣就能在撥電話時也能同時上網，之前的 ADSL 就是其中一種，只不過礙於成本和技術發展的原因已經漸漸被淘汰
* Ethernet 和 Wifi：以太網路是目前有線連線比較常見的傳輸方式，通常會以同軸電纜當作媒介，在介紹 Link Layer 時會比較詳細的說明；而 Wifi 是定義無線傳輸的一個組織，但常常被一般人借指無線網路 (誤用？)，目前最新的 5GHz 傳輸標準被定義在 IEEE 802.11a

* Guided / Unguided Media：Guided Media 指的是訊號藉由實體媒介來傳播，像是有線網路就是 guided；反之沒有實體媒介的無線網路就是 unguided