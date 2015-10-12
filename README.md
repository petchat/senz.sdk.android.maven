
# senz-sdk-android


## 1、 Android studio 中引用

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
		compile 'io.petchat:senzsdk:2.0.1'
		compile 'com.firebase:firebase-client-android:2.3.1+'
	}
```


## 2、 使用SENZ SDK

### 2.1 初始化
在MainActivity 的 onCreate()中（必须在UI主线程中调用），调用：
```java
//please register on senz server to get your own appId
    Senz.initialize(MainActivity.this,appId);
```
### 2.2 Senz核心api接口


####2.2.1 用户属性UserInfo类
该类主要用于分析用户的属性，如：年龄、性别等。

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
#####hashMap数据格式说明

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


####2.2.2 用户的情景识别 context

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
#####数据格式说明
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
 


####2.2.3 用户事件识别 event
 event指用户发生的事件，如某个时间段，用户在看电影或者逛商场。
 
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
 
#####数据格式说明
 SenzEvent 类：
 ```java
 public class SenzEvent implements Serializable {
    public long startTime;       //事件开始时间
    public long endTime;         //事件结束时间
    public String eventType;     //事件类型
    public double probability;   //事件发生的概率（测试值，有待优化）
    }
 ```

####2.2.4 用户的 home 和 office 的识别和监听
 senz可以自动的识别用户的家庭和工作地点，也可以检测出用户离开、进入家或者公司。
 检测出这些变化时，senz会发出相应的广播。
 
 监听action：
 
 action： senz.intent.action.HOME_OFFICE_STATUS
 
 调用例子：

 ```java
        BroadcastReceiver statusReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            if ("senz.intent.action.HOME_OFFICE_STATUS".equals(intent.getAction())) {
                Bundle bundle = intent.getExtras();
                Serializable data = bundle.getSerializable("home_office_status");
                SenzOHStatus status = (SenzOHStatus) data;
                //...
            }
        }
    };
 ```
#####数据格式说明
 SenzOHStatus 类：
 ```java
 public class SenzOHStatus implements Serializable {
    public long updateTime;  //状态更新时间
    public int stautsType;       //状态值
    }
    
    //对应的状态值
 public class OHStatusType {
    public final static int UNKNOWN = -1;
    public final static int ARRIVING_HOME = 0;
    public final static int LEAVING_HOME = 1;
    public final static int ARRIVING_OFFICE = 2;
    public final static int LEAVING_OFFICE = 3;
    public final static int GOING_HOME = 4;
    public final static int GOING_OFFICE = 5;
    public final static int USER_HOME_OFFICE_NOT_YET_DEFINED = 6;
    }
    
    //我们提供了解析工具类 ParseUtils:
    public static String parseStatusType(final int stautsType)
 ```

####2.2.5 用户 motion 的识别和 motion 改变的监听

senz可以识别出用户当前的动作，如 坐、跑步、走路等。当检测到用户的动作发生改变时
senz会发出相应的广播。

 监听action：
 
 action： senz.intent.action.MOTION_CHANGED
 
 调用例子：

 ```java
        BroadcastReceiver motionReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            if ("senz.intent.action.MOTION_CHANGED".equals(intent.getAction())) {
                Bundle bundle = intent.getExtras();
                Serializable data = bundle.getSerializable("motion_changed");
                SenzMotion motion = (SenzMotion) data;
                //...
            }
        }
    };
 ```
#####数据格式说明
 SenzMotion 类：
 ```java
 public class SenzMotion implements Serializable {
    public int motionType;		//motion类型
    public double similarity;   	//该类型的相似度
    }
    
    //对应的状态值
 public class MotionType {
    public final static int UNKNOWN = -1;
    public final static int SITTING = 0;
    public final static int DRIVING = 1;
    public final static int RIDING = 2;
    public final static int WALKING = 3;
    public final static int RUNNING = 4;
    }
    
    //我们提供了解析工具类 ParseUtils:
    public static String parseMotionType(final int motionType)
 ```
##2.3 设置监听的另一种方法

 - 1.
Declare your own receiver with the events you would like to receive from the SDK
```xml
<receiver android:name=".YourBroadcastReceiverClassNameHere">
    <intent-filter>
        <action android:name="senz.intent.action.HOME_OFFICE_STATUS"/>
        <action android:name="senz.intent.action.MOTION_CHANGED"/>
        <action android:name="senz.intent.action.EVENT"/>
        
    </intent-filter>
</receiver>
```

 - 2.
write your broadcast receiver to catch the broadcast, data stored in the bundle ,
you need to resolve the bundle, the bundle key is "motion_changed", "event", "home_office_status".
use like this:
```java
public class SenzReceiver extends BroadcastReceiver {
    private static String TAG = "SenzReveiver";

    @Override
    public void onReceive(Context context, Intent intent) {
        if ("senz.intent.action.MOTION_CHANGED".equals(intent.getAction())) {
                Bundle bundle = intent.getExtras();
                Serializable data = bundle.getSerializable("motion_changed");
                SenzMotion motion = (SenzMotion) data;
                //...
            }
	}
}
```
