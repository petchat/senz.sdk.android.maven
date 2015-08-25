
# senz-sdk-android
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
		compile 'io.petchat:senzsdk:2.0.0'
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
        userInfo.getUserInfos(new SenzCallback() {
            @Override
            public void done(Exception e, Object object) {
                if (e == null) {
                    HashMap<String, Double> hashMap = (HashMap<String, Double>) object);
                    //...
                   
                } else {
                   //...
                }
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
        userContext.getUserContext(new SenzCallback() {
            @Override
            public void done(Exception e, Object object) {
                if (e == null) {
                    SenzContext senzContext = (SenzContext) object;
                    //...
                } else {
                    //...
                }
            }
        });
 ```
*数据格式说明*
SenzContext类：
```java
public class SenzContext {
    public HashMap<String, Double> poiProbLv1;
    public HashMap<String, Double> poiProbLv2;
    public HashMap<String, Double> motionProb;
    public HashMap<String, Double> sceneProb;
    public String locationUpdatedAt;
    public String motionUpdatedAt;
    public String sceneUpdatedAt;
    }
```
 
 poiProbLv1 和 poiProbLv2 为用户location的情景识别， sceneProb 为用户的所处场景识别， motionProb 为
 用户的动作情景识别。
 属性保存在了key中，对应的值保存在了value中，
 键值不是固定的，请使用者自行遍历相应的map。
 
```

 - ***用户事件识别 event***
 event值的获取需要注册监听：
 
 action： senz.intent.action.EVENT
 
 调用例子：

 ```java
        BroadcastReceiver eventReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            if ("senz.intent.action.EVENT".equals(intent.getAction())) {
                Bundle bundle = intent.getExtras();
                Serializable data = bundle.getSerializable("event");
                List<SenzEvent> events = (List<SenzEvent>) data;
                //...
            }
        }
    };
 ```
 *数据格式说明*
 SenzEvent 类：
 ```java
 public class SenzEvent implements Serializable {
    public long startTime;       //事件开始时间
    public long endTime;         //事件结束时间
    public String eventType;     //事件类型
    public double probability;   //事件发生的概率（测试值，有待优化）
    }
 ```
