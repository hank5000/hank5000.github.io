---
layout: post
title: JNI C register java callback
date: 2015-11-02 23:15:38 +0800
comments: true
categories:  callback interface
---
***

Java可以透過JNI去跟其他語言溝通，這邊要介紹跟Ｃ溝通的一種情境，這篇不是教人如何使用JNI去Call C的function，而是**反過來想要使用C 去 Call Java 的function**，而且還是當作Callback的情況下使用。

（今天你想要開發一個Java library, 但是你已經有C版本的library了，你現在想要使用C library當作核心，多寫一份JNI的code讓他可以延伸到Java的世界來，這時候有些callback function要接到java去就可以使用這種方法）

我在改Google的AppRTC的時候看到了libjingle_peerconnection.jar中有個DataChannel的Class 其中有個Interface是Observer，這個Observer是要留給使用者實作，而這個Observer中的

1. onMessage是在接收到DataChannel的資料的時候會被Call到
2. onStateChange是在DataChannel的狀態有改變的時候會被Call到

使用者要自行去Implement這個DataChannel.Observer，畢竟開發Library的人不知道你想要在DataChannel狀態改變的時候做什麼跟你想要把接收到的資料做什麼，所以就留了一個空間讓大家去發揮。

然而在implement完這個DataChannel.Observer的時候，我要如何去告知底層的C，跟他說Callback function設定成我Implement好的這個DataChannel.Observer Class呢。

這邊就使用了registerObserver這個function，這個function其實最後是通到了native，把implement好的實體帶往C，讓C記住之後，當C要用的時候就Call就好了想法就這麼簡單。

( Note: libjingle_peerconnection.jar只是libjingle.so在java層的一層殼子，所有的事情都是在native中完成，然後再通知java這邊做些應對，所以必須要跟native層說java層這邊準備好了什麼你記得來Call，用Interface的原因是因為要限制接口的樣子，畢竟他是library，使用者要依照他的形式下去使用。)

下面大概描述一下要如何使用這樣的機制：

{% codeblock lang:java library_name.java%}
public class library_name {
	static {
		// if library .so is named 'libname.so'
		System.loadLibrary("name")
	}
	//跟Native註冊interface_name實體的Native function.
	private native registerInterfaceNameNative(interface_name var1);
		
	//假設有一個叫做interface_name的 Interface
	public Interface interface_name {
		void interface_func(byte[]);
	}
	//java的接口，負責把interface_name的實體帶給Native function.
	public void registerInterfaceName(interface_name var1) {
		registerInterfaceNameNative(var1);
	}
}
{% endcodeblock %}


{% codeblock lang:C library_jni.c%}

static JavaVM *gJavaVM;
jobject callback_obj;
jmethodID callback_mid;

JNIEXPORT jint JNICALL Java_com_XXXXX_library_name_somethingInitFunction(JNIEnv* env, jobject obj) {
	(*env)->GetJavaVM(env,&gJavaVM);
}

JNIEXPORT jint JNICALL Java_com_XXXXX_library_name_registerInterfaceNameNative(JNIEnv* env, jobject obj, jobject instance) {
	callback_obj = (*env)->NewGlobalRef(env,instance);
	jclass clz = (*env)->GetObjectClass(env,callback_obj);
	if(clz == NULL) {
		//failed to find class
	}
	callback_mid = (*env)->GetMethodID(env,clz,"interface_func","([B)V");
}

// callback function in C
void cb_function(char* buf) {
	JNIEnv *env;
	if((*gJavaVM)->AttachCurrentThread(gJavaVM, &env, NULL) != JNI_OK) 	{
		// attachCurrentThread() failed.
	}
	else 
	{
		  // create byte[]
        jbyteArray arr = (*env)->NewByteArray(env,len);
        // set buf -> byte[]
        (*env)->SetByteArrayRegion(env,arr,0,len, (jbyte*)buf);
        // call java function, which is implemented by user
        (*env)->CallVoidMethod(env,callback_obj,callback_mid,arr);
	}
}

{% endcodeblock %}

實際使用上，要先實作 library_name.interface_name 然後把實作好的實體new出來後從registerInterfaceName一路帶下去給C.

大概會長成這樣 

{% codeblock lang:java used_library.java%}

public class interfaceImpl implements library_name.interface_name {
	@Override
	void interface_func(byte[] a) {
		// show new String(a);
	}
}

 main {
	library_name library = new library_name();
	// call some function to init library
	// ...
	
	//這行最重要，要把implmenet完的實體送下去給Native層知道
	library.registerInterfaceName(new interfaceImpl());

	...
}
{% endcodeblock %}

這樣就可以透過註冊的方式設定Native C可以用自己implement的Interface callback回來囉.

大致上是這樣子使用，我也還在學習中，

如果有問題歡迎交流:D


   