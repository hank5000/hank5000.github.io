---
layout: post
title: "How to use Octopress"
date: 2015-06-16 23:30
comments: true
external-url:
categories: octopress
published: true

---

#Something about Octopress
===

##紀錄如何使用github + octopress架個人網站，順便讓自己熟悉一下Markdown語法。

+ 如果是第一次聽到git的人可能要先去熟悉一下git的操作方法，可以到[CodeSchool](https://www.codeschool.com/paths/git)學一下基本的操作方法。
+ octopress的介紹可以看看[高手的介紹](http://blog.xdite.net/posts/2011/10/07/what-is-octopress/) 
+ 關於markdown的好處跟介紹可以參考[此連結](http://markdown.tw/ "markdown.tw")
+ 環境似乎是在 Mac or Ubuntu 會相對比 Windows方便

---
##1. 創建github repository (前提是你已經有github帳號，如果沒有就[註冊](https://github.com/)一個囉)
這邊介紹我使用的方法，登入github之後選擇 + New repository，並且在Repository name的地方填上 _username.github.io_，之後你就會生出一個叫做username.github.io的Repository，這個就是以後作為你的github page使用。


##2. 架設Octopress環境
如何架設Ocotpress環境可以參考[官方網站](http://octopress.org/docs/setup/)
大概就是

+ 安裝 git
+ 安裝 ruby 1.9.3 ，較建議使用 rvm 安裝
+ 最後一個 Install one of the ExecJS supported JavaScript runtimes. (我沒安裝)

當這些環境都弄好之後，當然就是要把octopress 從git clone下來囉
    
    $ git clone https://github.com/imathis/octopress
    $ cd octopress
    $ rake install
    $ rake setup_github_pages

在這之後他會要求你的github repository, 你可以進到你的github repository右邊有個 SSH clone URL 裡面框框的內容直接複製貼上就可以，如果你第一步跟我是一樣的話，你的這串字應該是 _git@github.com:username/username.github.io.git_，然後再輸入指令如下：

	$ rake generate
	$ rake deploy
	
他就會把目前你的這些東西都推到你的github page上了，這時你可以連到你的github page看看囉(如果沒有生效可能要等一下下)

然後你要把目前目錄下的所有東西push到git上面做個備份
	
	$ git add .
	$ git commit -m "github first commit"
	$ git push origin source

此時你的github repo會有兩個branch一個是source 一個是master，source是你目前這個octopress資料夾下的所有檔案除了\_deploy，而master呢就是你實際的網頁內容(_deploy資料夾裡面的部分)


##3. 使用Octopress發文/刪文

###發文

	$ rake new_post['文章標題']
Octopress會生成個 *.markdown 到你的./source/_posts/ 下面，就我理解這個有點像是原始檔(Markdown Source Code)，然後你可以在 *.markdown中用markdown語法寫文章存檔，然後執行
	
	$ rake generate
他會去編譯 *.markdown變成 *.html 放到 _deploy的某個目錄中，然後再執行

	$ rake deploy
他會把_deploy新增的或者是刪除的push到github做修改，在po好文章之後你只有把html的部份上傳，可是*.markdown的部分並未上傳做備份，記得再輸入
	
	$ git add .
or
	
	$ git add "新的file path"

then

	$ git commit -m "一些敘述"
	$ git push


+ ###刪文

有了剛剛些概念後應該可以大概知道，使用
	
	$ rake new_post["post_name"]

他其實是幫你把 *.markdown生成到source/_posts/...，然後再透過 
	
	$ rake generate
去把html檔生成出來，也就是說如果要刪文的話，你可以到source/_posts/...，去找你想要刪掉的文章

	$ rm source/_posts/XXXXXX.markdown
然後在執行
 	
	$ rake generate
	$ rake deploy
這樣他就會去把在github上面的相關 html給刪除

或者是你可以直接在*.markdown上面有一串字描述這篇文章的屬性的

	---
	layout: post
	title: "How to use Octopress"
	date: 2015-06-16 23:30
	comments: true
	external-url:
	categories: octopress
	published: true
	---
加入pulished屬性，然後把它改成false。

##4. 更換Octopress theme
來來來來看[這篇](http://tommy351.github.io/Octopress-Theme-Slash/index_tw.html)你大概就知道怎麼更換Octopress theme囉，連結裡面那篇是一個叫做Slash的主題，非常的簡約風格，就是我現在使用的這個，照著他教學做，你也可以跟我的風格一樣囉

##5. 如何在別台電腦上clone你做好的octopress環境
這件事是困擾我最久的，也是我搞最久的，剛好有找到一個很好的教學，前提你架設過octopress然後在github也有紀錄，你是想在另外一條電腦上發文章，你該怎麼複製出一份一樣的環境呢（octopress最基本的ruby跟git是一定要有的）

	$ git clone -b source git@github.com:username/username.github.com.git octopress
	$ cd octopress
	$ git clone git@github.com:username/username.github.com.git _deploy 
	$ gem install bundler
	$ bundler install
	
不用再執行
	
	$ rake setup_github_pages
直接可以使用囉。


###參考來源：

+ ####[Why Octopress?](http://blog.xdite.net/posts/2011/10/07/what-is-octopress/)
+ ####[Octopress 官網](http://octopress.org/)
+ ####[國王的耳朵是驢耳朵](http://wen00072-blog.logdown.com/posts/258497-octopress-installed-and-deployed-on-the-github-pages#oct-dl)
+ ####[Restore Octopress at a New Computer](http://cn.soulmachine.me/blog/20140128/)
