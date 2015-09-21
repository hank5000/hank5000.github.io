---
layout: post
title: Following Google Drive Quick Start (Android)
date: 2015-09-22 00:25:41 +0800
comments: true
categories: Android GoogleDrive
---

* * *

這其實就是Follow官網的[Android Quickstart](https://developers.google.com/drive/web/quickstart/android)下去實作，畢竟前一陣子我耐不下心看官網教學我就一直做不出來...，寫成**<font color='red'>中文</font>**的應該相對簡單。

*   ### Step1. 獲得 SHA1 FingerPrint

    使用keytool (keytool一般就在java_path/bin下面)，找到他然後開cmd(terminal)，在有keytool的目錄下輸入command
    
    - **windows** 要下：(ex: 我的電腦名稱是dell11，理論上你可以放在你想要的位置，我是乖乖照著官網想要的路徑擺放)
    
            $keytool -exportcert -alias androiddebugkey -keystore "C:\Users\dell11\.android\debug.keystore" -list -v
    
    - **Mac/Linux** 要下：
    
            $keytool -exportcert -alias androiddebugkey -keystore "~/.android/debug.keystore" -list -v
            
    差別大概就是在斜線方向不同，還有作業系統file system的一些差異，輸入了之後他會叫你輸入password，此時輸入 **android** 便可，然後應該可以看到一串東西如下:
    
        輸入金鑰儲存庫密碼:
        別名名稱: androiddebugkey
        建立日期: 2015/4/14
        項目類型: PrivateKeyEntry
        憑證鏈長度: 1
        憑證 [1]:
        擁有者: CN=Android Debug, O=Android, C=US
        發出者: CN=Android Debug, O=Android, C=US
        序號: 552bf95a
        有效期自: Tue Apr 14 01:14:02 CST 2015 到: Thu Apr 06 01:14:02 CST 2045
        憑證指紋:
	         MD5:  D8:C1:94:74:C7:0F:91:D8:EC:06:54:33:43:D3:EC:9F
	         SHA1: 27:82:C6:F3:FB:9A:82:2D:A5:93:36:3B:31:A9:F5:98:4A:0D:33:2C
	         SHA256: 95:A0:D3:69:DA:E6:37:BB:B0:66:9E:F2:D3:16:0D:8F:F4:E6:92:0D:B0:90:1D:A7:51:0C:87:7A:98:70:40:05
	         簽章演算法名稱: SHA1withRSA
	         版本: 3    
    
    把SHA1那行給記起來(<font color='red'>D8:AA:43:97:59:EE:C5:95:26:6A:07:EE:1C:37:8E:F4:F0:C8:05:C8</font>)。
*   ### Step2. 開啟Google API
    然後**[到這裡](https://console.developers.google.com/flows/enableapi?apiid=drive)**，選擇建立 <font color='green'>新專案 </font>=> <font color='green'>繼續</font> => <font color='green'>前往憑證</font> => <font color='green'>新增憑證</font> => <font color='green'>選擇OAuth2.0用戶端編號</font> => 你會發現不能選擇應用程式類型，因為Google說 **您必須先在同意畫面中設定產品名稱，才能建立 OAuth 用戶端編號**，所以先去<font color='green'>設定產品名稱</font>之後，就可以回來選擇應用程式類型， 這邊<font color='green'>選擇Android</font>:
    - **用戶端名稱** : 打什麼應該都可以。
    - **簽署憑證的指紋** : 就填上剛剛keytool拿到的SHA1後面那一長串。
    - **套件名稱** : 這個很重要，應用程式建構後要跟這個值對到才可以使用google drive api，例如你的應用程式是叫做 com.example.drivequickstart，這邊也要輸入com.example.drivequickstart. (為了實作方便，直接用這個吧，Android 提供的Source Code也是用這個)
    然後<font color='green'>選擇建立</font>。
*   ### Step3. 建立一個新的Android 應用程式專案
    打開Android Studio，**開一個新專案(Company Domain : com.example.drivequickstart)** ，然後把build.gradle(app)內容換成：
    
        apply plugin: 'com.android.application'
        
    	android {
         	compileSdkVersion 22
         	buildToolsVersion "22.0.1"

    		defaultConfig {
        		applicationId "com.example.drivequickstart"
        		minSdkVersion 11
        		targetSdkVersion 22
        		versionCode 1
        		versionName "1.0"
    		}
    		buildTypes {
        		release {
            		minifyEnabled false
            		proguardFiles getDefaultProguardFile('proguard-android.txt'),
                		'proguard-rules.pro'
        		}
    		}
		}

		dependencies {
    		compile fileTree(dir: 'libs', include: ['*.jar'])
    		compile 'com.android.support:appcompat-v7:22.1.1'
    		compile 'com.google.android.gms:play-services:7.3.0'
    		compile 'com.google.api-client:google-api-client:1.20.0'
         	compile 'com.google.api-client:google-api-client-android:1.20.0'
         	compile 'com.google.api-client:google-api-client-gson:1.20.0'
         	compile 'com.google.apis:google-api-services-drive:v2-rev170-1.20.0'
         	} 


    然後在上方列點選 <font color='green'>Android</font>=> <font color='green'>Sync Project with Gradle Files</font>. 讓他把依賴的sdk加入project中，然後分別覆蓋下列幾個檔案，如果不存在就開新檔 :
- AndroidManifest.xml

{% codeblock lang:xml AndroidManifest.xml %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.example.drivequickstart">
    
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.MANAGE_ACCOUNTS" />
    <uses-permission android:name="android.permission.USE_CREDENTIALS" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="Drive API Android Quickstart"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".MainActivity"
            android:label="Drive API Android Quickstart" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <meta-data
            android:name="com.google.android.gms.version"
            android:value="@integer/google_play_services_version" />
    </application>
</manifest>  
{% endcodeblock %}

- MainActivity.java

{% codeblock lang:java MainActivity.java %}
package com.example.drivequickstart;

import com.google.android.gms.common.ConnectionResult;
import com.google.android.gms.common.GooglePlayServicesUtil;
import com.google.api.client.extensions.android.http.AndroidHttp;
import com.google.api.client.googleapis.extensions.android.gms.auth.GoogleAccountCredential;
import com.google.api.client.http.HttpTransport;
import com.google.api.client.json.JsonFactory;
import com.google.api.client.json.gson.GsonFactory;
import com.google.api.client.util.ExponentialBackOff;


import com.google.api.services.drive.DriveScopes;

import android.accounts.AccountManager;
import android.app.Activity;
import android.app.Dialog;
import android.app.ProgressDialog;
import android.content.Context;
import android.content.Intent;
import android.content.SharedPreferences;
import android.graphics.Typeface;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.os.Bundle;
import android.text.TextUtils;
import android.text.method.ScrollingMovementMethod;
import android.view.ViewGroup;
import android.widget.LinearLayout;
import android.widget.TextView;

import java.util.Arrays;
import java.util.List;

public class MainActivity extends Activity {
    /**
     * A Drive API service object used to access the API.
     * Note: Do not confuse this class with API library's model classes, which
     * represent specific data structures.
     */
    com.google.api.services.drive.Drive mService;

    GoogleAccountCredential credential;
    private TextView mStatusText;
    private TextView mResultsText;
    ProgressDialog mProgress;
    final HttpTransport transport = AndroidHttp.newCompatibleTransport();
    final JsonFactory jsonFactory = GsonFactory.getDefaultInstance();

    static final int REQUEST_ACCOUNT_PICKER = 1000;
    static final int REQUEST_AUTHORIZATION = 1001;
    static final int REQUEST_GOOGLE_PLAY_SERVICES = 1002;
    private static final String PREF_ACCOUNT_NAME = "accountName";
    private static final String[] SCOPES = { DriveScopes.DRIVE_METADATA_READONLY };

    /**
     * Create the main activity.
     * @param savedInstanceState previously saved instance data.
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        LinearLayout activityLayout = new LinearLayout(this);
        LinearLayout.LayoutParams lp = new LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.MATCH_PARENT);
        activityLayout.setLayoutParams(lp);
        activityLayout.setOrientation(LinearLayout.VERTICAL);
        activityLayout.setPadding(16, 16, 16, 16);

        ViewGroup.LayoutParams tlp = new ViewGroup.LayoutParams(
                ViewGroup.LayoutParams.WRAP_CONTENT,
                ViewGroup.LayoutParams.WRAP_CONTENT);

        mStatusText = new TextView(this);
        mStatusText.setLayoutParams(tlp);
        mStatusText.setTypeface(null, Typeface.BOLD);
        mStatusText.setText("Retrieving data...");
        activityLayout.addView(mStatusText);

        mResultsText = new TextView(this);
        mResultsText.setLayoutParams(tlp);
        mResultsText.setPadding(16, 16, 16, 16);
        mResultsText.setVerticalScrollBarEnabled(true);
        mResultsText.setMovementMethod(new ScrollingMovementMethod());
        activityLayout.addView(mResultsText);

        mProgress = new ProgressDialog(this);
        mProgress.setMessage("Calling Drive API ...");

        setContentView(activityLayout);

        // Initialize credentials and service object.
        SharedPreferences settings = getPreferences(Context.MODE_PRIVATE);
        credential = GoogleAccountCredential.usingOAuth2(
                getApplicationContext(), Arrays.asList(SCOPES))
                .setBackOff(new ExponentialBackOff())
                .setSelectedAccountName(settings.getString(PREF_ACCOUNT_NAME, null));

        mService = new com.google.api.services.drive.Drive.Builder(
                transport, jsonFactory, credential)
                .setApplicationName("Drive API Android Quickstart")
                .build();
    }


    /**
     * Called whenever this activity is pushed to the foreground, such as after
     * a call to onCreate().
     */
    @Override
    protected void onResume() {
        super.onResume();
        if (isGooglePlayServicesAvailable()) {
            refreshResults();
        } else {
            mStatusText.setText("Google Play Services required: " +
                    "after installing, close and relaunch this app.");
        }
    }

    /**
     * Called when an activity launched here (specifically, AccountPicker
     * and authorization) exits, giving you the requestCode you started it with,
     * the resultCode it returned, and any additional data from it.
     * @param requestCode code indicating which activity result is incoming.
     * @param resultCode code indicating the result of the incoming
     *     activity result.
     * @param data Intent (containing result data) returned by incoming
     *     activity result.
     */
    @Override
    protected void onActivityResult(
            int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch(requestCode) {
            case REQUEST_GOOGLE_PLAY_SERVICES:
                if (resultCode != RESULT_OK) {
                    isGooglePlayServicesAvailable();
                }
                break;
            case REQUEST_ACCOUNT_PICKER:
                if (resultCode == RESULT_OK && data != null &&
                        data.getExtras() != null) {
                    String accountName =
                            data.getStringExtra(AccountManager.KEY_ACCOUNT_NAME);
                    if (accountName != null) {
                        credential.setSelectedAccountName(accountName);
                        SharedPreferences settings =
                                getPreferences(Context.MODE_PRIVATE);
                        SharedPreferences.Editor editor = settings.edit();
                        editor.putString(PREF_ACCOUNT_NAME, accountName);
                        editor.commit();
                    }
                } else if (resultCode == RESULT_CANCELED) {
                    mStatusText.setText("Account unspecified.");
                }
                break;
            case REQUEST_AUTHORIZATION:
                if (resultCode != RESULT_OK) {
                    chooseAccount();
                }
                break;
        }

        super.onActivityResult(requestCode, resultCode, data);
    }

    /**
     * Attempt to get a set of data from the Drive API to display. If the
     * email address isn't known yet, then call chooseAccount() method so the
     * user can pick an account.
     */
    private void refreshResults() {
        if (credential.getSelectedAccountName() == null) {
            chooseAccount();
        } else {
            if (isDeviceOnline()) {
                mProgress.show();
                new ApiAsyncTask(this).execute();
            } else {
                mStatusText.setText("No network connection available.");
            }
        }
    }

    /**
     * Clear any existing Drive API data from the TextView and update
     * the header message; called from background threads and async tasks
     * that need to update the UI (in the UI thread).
     */
    public void clearResultsText() {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                mStatusText.setText("Retrieving data…");
                mResultsText.setText("");
            }
        });
    }

    /**
     * Fill the data TextView with the given List of Strings; called from
     * background threads and async tasks that need to update the UI (in the
     * UI thread).
     * @param dataStrings a List of Strings to populate the main TextView with.
     */
    public void updateResultsText(final List<String> dataStrings) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                if (dataStrings == null) {
                    mStatusText.setText("Error retrieving data!");
                } else if (dataStrings.size() == 0) {
                    mStatusText.setText("No data found.");
                } else {
                    mStatusText.setText("Data retrieved using" +
                            " the Drive API:");
                    mResultsText.setText(TextUtils.join("\n\n", dataStrings));
                }
            }
        });
    }

    /**
     * Show a status message in the list header TextView; called from background
     * threads and async tasks that need to update the UI (in the UI thread).
     * @param message a String to display in the UI header TextView.
     */
    public void updateStatus(final String message) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                mStatusText.setText(message);
            }
        });
    }

    /**
     * Starts an activity in Google Play Services so the user can pick an
     * account.
     */
    private void chooseAccount() {
        startActivityForResult(
                credential.newChooseAccountIntent(), REQUEST_ACCOUNT_PICKER);
    }

    /**
     * Checks whether the device currently has a network connection.
     * @return true if the device has a network connection, false otherwise.
     */
    private boolean isDeviceOnline() {
        ConnectivityManager connMgr =
                (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo networkInfo = connMgr.getActiveNetworkInfo();
        return (networkInfo != null && networkInfo.isConnected());
    }

    /**
     * Check that Google Play services APK is installed and up to date. Will
     * launch an error dialog for the user to update Google Play Services if
     * possible.
     * @return true if Google Play Services is available and up to
     *     date on this device; false otherwise.
     */
    private boolean isGooglePlayServicesAvailable() {
        final int connectionStatusCode =
                GooglePlayServicesUtil.isGooglePlayServicesAvailable(this);
        if (GooglePlayServicesUtil.isUserRecoverableError(connectionStatusCode)) {
            showGooglePlayServicesAvailabilityErrorDialog(connectionStatusCode);
            return false;
        } else if (connectionStatusCode != ConnectionResult.SUCCESS ) {
            return false;
        }
        return true;
    }

    /**
     * Display an error dialog showing that Google Play Services is missing
     * or out of date.
     * @param connectionStatusCode code describing the presence (or lack of)
     *     Google Play Services on this device.
     */
    void showGooglePlayServicesAvailabilityErrorDialog(
            final int connectionStatusCode) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Dialog dialog = GooglePlayServicesUtil.getErrorDialog(
                        connectionStatusCode,
                        MainActivity.this,
                        REQUEST_GOOGLE_PLAY_SERVICES);
                dialog.show();
            }
        });
    }

}
{% endcodeblock %}

- ApiAsyncTask.java

{% codeblock lang:java ApiAsyncTask.java %}
package com.example.drivequickstart;

/**
 * Created by dell11 on 2015/9/19.
 */

import android.os.AsyncTask;

import com.google.api.client.googleapis.extensions.android.gms.auth.GooglePlayServicesAvailabilityIOException;
import com.google.api.client.googleapis.extensions.android.gms.auth.UserRecoverableAuthIOException;
import com.google.api.services.drive.model.File;
import com.google.api.services.drive.model.FileList;


import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * An asynchronous task that handles the Drive API call.
 * Placing the API calls in their own task ensures the UI stays responsive.
 */
public class ApiAsyncTask extends AsyncTask<Void, Void, Void> {
    private MainActivity mActivity;

    /**
     * Constructor.
     * @param activity MainActivity that spawned this task.
     */
    ApiAsyncTask(MainActivity activity) {
        this.mActivity = activity;
    }

    /**
     * Background task to call Drive API.
     * @param params no parameters needed for this task.
     */
    @Override
    protected Void doInBackground(Void... params) {
        try {
            mActivity.clearResultsText();
            mActivity.updateResultsText(getDataFromApi());

        } catch (final GooglePlayServicesAvailabilityIOException availabilityException) {
            mActivity.showGooglePlayServicesAvailabilityErrorDialog(
                    availabilityException.getConnectionStatusCode());

        } catch (UserRecoverableAuthIOException userRecoverableException) {
            mActivity.startActivityForResult(
                    userRecoverableException.getIntent(),
                    MainActivity.REQUEST_AUTHORIZATION);

        } catch (Exception e) {
            mActivity.updateStatus("The following error occurred:\n" +
                    e.getMessage());
        }
        if (mActivity.mProgress.isShowing()) {
            mActivity.mProgress.dismiss();
        }
        return null;
    }

    /**
     * Fetch a list of up to 10 file names and IDs.
     * @return List of Strings describing files, or an empty list if no files
     *         found.
     * @throws IOException
     */
    private List<String> getDataFromApi() throws IOException {
        // Get a list of up to 10 files.
        List<String> fileInfo = new ArrayList<String>();
        FileList result = mActivity.mService.files().list()
                .setMaxResults(20)
                .execute();
        List<File> files = result.getItems();
        if (files != null) {
            for (File file : files) {
                fileInfo.add(String.format("%s (%s)\n",
                        file.getTitle(), file.getId()));
            }
        }
        return fileInfo;
    }
}
{% endcodeblock %}

然後就可以執行囉，這個app不能執行在虛擬機上，執行的Device必須要有Google Service，執行結果是把你的google drive的前十項名字列出來，這就是個很簡易的Android google drive 應用程式了。不過當然不只要這樣，我們還想要很多很多的功能，要怎麼實作呢，這邊提供一個別人寫的demo，裡面把該如何使用google drive的方法都寫好了，詳細請點連結吧 —>[連結在此](https://github.com/googledrive/android-demos)

我follow google的教學寫的google drive quick start也放到github了，想要的也可以點連結 —>[連結在此](https://github.com/hank5000/googledrive-quickstart/tree/master)，只是我比較背骨的把 Company Domain改成 com.example.dell11.googledrive，如果你要使用記得開啟Google API時的套件名稱要寫對.