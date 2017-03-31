---
layout: post
title: "實做Signaling Server與WebRTC 連接前之交互流程"
date: 2017-03-31 16:06:34 +0800
comments: true
categories: 
---
最近拿Socket.io來實做Signaling Server, 做為WebRTC連接前溝通的信令伺服器, 這篇就來紀錄一下整個過程.
 
####先來一些簡單小問題解惑:

### 1. 什麼是WebRTC呢？
其實之前有寫過一篇 [Learn AppRTC/WebRTC](http://hank5000.github.io/blog/2015/09/15/learn-apprtc-slash-webrtc/) 可以先看看.

#### 他有包含三個部份：
1. RTCPeerConnection, 主要是在建立雙方連接的API, 連接完後就會使用下面兩個API傳輸資料
2. Network Stream API, 設定傳輸媒體流使用(Video/Audio)
3. Peer-to-Peer Data API, 這邊就是你可以拿來傳送資料的API, 在這條路上你要傳送什麼隨便你.
最重要的就是RTCPeerConnection在Android上就是PeerConnection這個Class

### 2. Signaling Server作用是什麼?
1. 因為WebRTC是一種P2P概念的東西, 在剛開始雙方要連線時, 根本不認識對方, 也不知道對方在何處, 所以中間一定要有一台Server幫雙方建立剛開始相互認識的橋樑
2. 深入一點說, 這邊Signaling Server要幫兩台設備轉傳一些資料, 例如 SessionDescription (SDP), Ice candidate...這些轉傳完兩台設備就會互向認識, 互相跨越對方防火牆, 但不是所有NAT Type都試用, 若至少有一人是NAT Type 4 (對稱型的NAT)的話就必須要多一台Turn Server, 不然基本上Stun Server就夠了.

### 3. 為什麼會拿Socket.io來實做Signaling Server呢? 
1. 因為他的Library非常好上手, 而且有支持各個平台(瀏覽器/IOS/Android/...), 所以就拿來玩玩看, 這邊是使用Socket.io官方提供的[Chat Demo](https://github.com/socketio/socket.io/tree/master/examples/chat)來改, 有興趣也可以去玩玩看這個Demo很好玩.
2. 另外我實在懶的去找其他的工具來用了...

---

###開始我們的主題, 
先講一下WebRTC要怎麼樣雙方才會對上.
首先有兩個Peer, 假設是Peer A 跟 Peer B好了

總是有個人會先上線吧?
#### Peer A 先上線了!! 
1. Peer A call createOffer.
2. Peer A 等待onCreateSuccess
3. Peer A 等到onCreateSuccess後會拿到 SDP, 把SDP 拿去setLocalSdp.( 自己的嘛, 當然是setLocal囉)
4. Peer A 把Sdp送到**server**上放著
5. Peer A 收集onIceCandidate的IceCandidates, 然後等待Peer B出現

####Peer B上線了!!!
6. Peer B 到Server上找尋Peer A的身影, 拿取Sdp, 將Sdp拿去setRemoteSdp(別人的嘛, 當然是setRemote囉)
7. 因為Peer B已經拿到Offer了, 要給點回應 所以 call createAnswer
並且等待onCreateSucess
8. Peer B有Offer Sdp了想要更進一步拿到Peer A的IceCandidates, 於是發訊息跟Server請Peer A把IceCandidates都透過Server送過來
9. 等待Peer B收到Peer A的IceCandidates把他全部addIceCandidate.
10. Peer B 等到onCreateSuccess後也會拿到Sdp, 把Sdp拿去setLocalSdp. 並且透過Server送給Peer A,  Peer A收到後要 setRemoteSdp.
11. Peer B 在onIceCandidate時將IceCandidate透過Server轉發給Peer A, 然後Peer A要 addIceCandidate.

#### 接下來兩個人就不用在透過Server了...(Server表示傷心)

---

理論上是還有一台Server要幫忙配對, 這台在官方就叫做Room Server,兩個人進同一間房間就是一種配對, 這邊直接在Signaling Server中讓雙方都帶同一個username去取代這間事情.





