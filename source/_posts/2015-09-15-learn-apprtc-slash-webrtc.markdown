---
layout: post
title: "Learn AppRTC/WebRTC"
date: 2015-09-15 23:59:02 +0800
comments: true
categories: Android WebRTC AppRTC
 
---

最近在使用WebRTC做些Android上的應用，一開始被Google的這些東西快要搞死，為了方便還是把資料整理一下，丟個教學出來，以便複習跟學習，如果寫的有誤請不吝告知。

#####都不知道是甚麼的可以先去玩一下 [AppRTC Demo](https://apprtc.appspot.com)#####

- [WebRTC](http://www.webrtc.org/)是Web Real-Time Communication的縮寫，其本意就是可以在Web上實行即時溝通的一個免費且開放的project，其project提供了簡易的api讓開發者便於開發應用.

- WebRTC的目標為在browser/mobile device/IoT device上開創一個豐富且高品質且統一的溝通協定架構。

#####一個私有的WebRTC基本上要有#####

- **Room Server :** 主要用來創建跟管理通訊狀況。
(Google原生是使用Nodejs實現，用python驅動)

- **Signaling Server :** 主要是管理以及協助終端做點對點的一個角色，負責了控制通訊發起/結束/錯誤的連線消息控制，建立安全連接的關鍵數據，管理雙方在外界所能看到的網路上數據(public ip, port,...)
(Google原生使用go language寫的**[collider](https://github.com/webrtc/apprtc/tree/master/src/collider)**)

- **Turn Server :** 主要用來協助防火牆穿透的部分，給予Ice candidate等...資訊。
(Google推薦使用**[coTurn](https://code.google.com/p/coturn/)**/**[rfc5766-turn-server](https://code.google.com/p/rfc5766-turn-server/)**)

基本上的架設方法在[AppRTC Demo Code](https://github.com/webrtc/apprtc)可以找到如何架設，如果架設起來應該會長得像他們做出來的[AppRTC Demo](https://apprtc.appspot.com)，不過我在實作的過程中遇到了一些問題就是我的Turn Server架設不起來。

#####附上找到的一些架設資訊:#####

- **[第六章 WebRTC服務器搭建](http://io.diveinedu.com/2015/02/05/%E7%AC%AC%E5%85%AD%E7%AB%A0-WebRTC%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA.html) by Diveinedu**
- **[基於webrtc的apprtc服務器的搭建](http://www.cnblogs.com/cther/p/4604599.html) by 小曉陳**
- **[自建AppRTC](http://www.jianshu.com/p/c55ecf5a3fcf) by ETiv ，比較推薦看這個，寫得挺詳細的。**

AppRTC也有**Mobile device**的**Application**的版本，在**Android**跟**IOS**上都有，官方連結 **[WebRTC-NativeCode](http://www.webrtc.org/native-code)**可以進去看看，但是要使用Google官網把整份Source Code弄下來會非常痛苦，而且非常耗時，好在有好心的開發者把已經把相關的library compile (native code...)，應該可以讓其他的開發者只要注重其他java的部分，不過當然可以動的地方就比較少，以下是連結:

- **[IOS版的AppRTC](https://github.com/ISBX/apprtc-ios) by ISBX**

- **[Android版的AppRTC](https://github.com/njovy/AppRTCDemo) by njovy，好心的njovy還一直有在更新，此專案是在[Android Studio](https://developer.android.com/sdk/index.html)上開發，下載下來之後直接run就行了**


_____

###NOTE:###
由於我參考了Etiv的文章後還是有些不懂，所以目前我所有Server都是使用Google的，然後去實現我的Android Application(讓兩隻android device透過WebRTC中的**Data Channel**來傳輸資料，點對點前的溝通都是透過google架設的Server，目前使用Google的Server都堪用)，如果有人整個架設成功還請教一下小弟。







