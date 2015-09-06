# senz-sdk-android
最新版sdk使用请参考 README_2_0_0.md
---

### Android studio 中引用
---
在工程目录 *build.gradle* 文件中添加如下内容：
```
    repositories {
			jcenter()
			maven {
				url "https://raw.githubusercontent.com/petchat/senz.sdk.android.maven/master"
			}
		}
	packagingOptions {
		        exclude 'META-INF/LICENSE'
		        exclude 'META-INF/LICENSE-FIREBASE.txt'
		        exclude 'META-INF/NOTICE'
    }
	dependencies {
		compile 'io.petchat:senzsdk:1.0.3'
		compile 'com.firebase:firebase-client-android:2.3.1+'
	}
```


### 使用SENZ SDK
===
### 初始化
在MainActivity 的 onCreate()中（必须在UI主线程中调用），调用：
```java
//please register on senz server to get your own appId and developerId 
    Senz.initialize(MainActivity.this,appId,developerId);
```
### Senz核心api接口

- ***用户属性UserInfo类***
  ---  
调用例子：
```java
    UserInfo userInfo = new UserInfo(this);
    userInfo.getUserInfos(new UserInfo.InfoCallback() {
        @Override
        public void success(HashMap<String, Double> hashMap) {
        // user info is stored in hashMap, you can use it by yourself
        }
        @Override
        public void failed(String s) {
        
        }
    });
```
*hashMap数据格式说明*

用户属性名称保存在了hashMap的key中，对应的值保存在了value中，
senz会根据不同的用户，计算不同的属性值，所以返回结果hashMap中的
key是不固定的，使用者可以自行遍历hashMap来获得各个属性值。
数据模版：
```
has_car:0.10048
online_shopping:0.20056
current_news:0.10076
...
```


- ***用户的情景识别 context***

调用例子：

 ```java
     UserContext userContext = new UserContext(this);
     userContext.getUserContext(new UserContext.ContextCallback() {
        @Override
            public void success(HashMap<String, HashMap<String, Double>> hashMap) {
                if (hashMap != null) {
                HashMap<String, Double> mapPoiProbLv1 = hashMap.get("poiProbLv1");
                HashMap<String, Double> mapPoiProbLv2 = hashMap.get("poiProbLv2");
                HashMap<String, Double> mapSoundProb = hashMap.get("soundProb");
                HashMap<String, Double> mapMotionProb = hashMap.get("motionProb");
                //... you'd better check the map key sets to make sure that the map contains the
                //key(poiProbLv1,poiProbLv1,soundProb,motionProb);
                //then you can use the map yourself.
                }
            }
            @Override
            public void failed(String s) {
              
            }
        });
 ```
*数据格式说明*
 
 mapPoiProbLv1和mapPoiProLv2为用户location的情景识别，mapSoundPro为用户的所处场景识别，mapMotionProb为
 用户的动作情景识别。
 属性保存在了key中，对应的值保存在了value中，
 键值不是固定的，请使用者自行遍历相应的map。
 数据模版：
 ```
 "home": 0.00005734417729001384,
 "dining": 0.0014762602549603316,
 "scenic_spot": 0.0001,
 "unknown": 0.014440114964888793,
  "exhibition": 0.0007881301274801661,
  ...
```

 - ***用户事件识别 event***
 
 调用例子：

 ```java
        UserEvent userEvent = new UserEvent(this);
        userEvent.getUserEvent(1,new UserEvent.EventCallback() {
            @Override
            public void success(HashMap<String, HashMap<String, Double>> hashMap) {
                StringBuffer sb = new StringBuffer();
                if (hashMap != null) {
                    for (int i = 0; i < hashMap.size(); i++) {
                        HashMap<String, Double> tmp = hashMap.get("" + i);
                    }
                }


            }

            @Override
            public void failed(String s) {
            }
        });
 ```
 *数据格式说明*
 返回结果hashMap中包含n个HashMap<String,Double>数据，HashMap的键值为 n ，保存了用户的event预测数据。
 键值不是固定的，请自行遍历各个map。
 数据模版：
```
"shopping.mall": 0.5534016044686336,
"dining.chineseRestaurant": 1.0065577298294253e-12,
"work.office": 0.02321910395062181,
"fitness.running": 0.42337929157973797
...
```

#另一种调用方法(beta版)：注册监听

 - 1.
<!-- Declare your own receiver with the events you would like to receive from the SDK -->
```xml
<receiver android:name=".YourBroadcastReceiverClassNameHere">
    <intent-filter>
        <action android:name="senz.intent.action.CONTEXT_LOCATION"/>
        <action android:name="senz.intent.action.CONTEXT_MOTION"/>
        <action android:name="senz.intent.action.CONTEXT_SCENE"/>
        <action android:name="senz.intent.action.EVENT"/>
        <action android:name="senz.intent.action.STATUS"/>
    </intent-filter>
</receiver>
```

 - 2.
write your broadcast receiver to catch the broadcast, data stored in the bundle 
you need to resolve the bundle, the bundle key is "location", "motion", "scene", "event", "status" 
like this:
```java
public class SenzReceiver extends BroadcastReceiver {
    private static String TAG = "SenzReveiver";

    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals("senz.intent.action.CONTEXT_LOCATION")) {
            Bundle bundle = intent.getExtras();
            Serializable data = bundle.getSerializable("location");
            HashMap<String, Double> map = (HashMap<String, Double>) data;
            .....
        }
	}
}
```
description:

 - CONTEXT_LOCATION: HashMap \<String,Double\> store the prediction of the user location
 - CONTEXT_MOTION: HashMap \<String,Double\> 
 - CONTEXT_SCENE: HashMap \<String,Double\>
 - EVENT: List\<SenzEvent\>
   SenzEvent类的成员：
```
long startTime，
long endTime，
String eventType，
double probability;
```
 - STATUS: HashMap \<String,Boolean\>
 
 
 
 













