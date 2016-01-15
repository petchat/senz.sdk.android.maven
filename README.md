
# senz-sdk-android


## 1、 Android studio 中引用

在工程目录 `build.gradle` 文件中添加如下内容：
```
buildscript {
    repositories {
	jcenter()
	maven {
                url "https://raw.githubusercontent.com/petchat/senz.sdk.android.maven/master"
        }
        maven { 
                url "https://jitpack.io" 
                }
	}
}

allprojects {
    repositories {
        jcenter()
        maven {
            url "https://raw.githubusercontent.com/petchat/senz.sdk.android.maven/master"
        }
        maven { url "https://jitpack.io" }
    }
}
```
在引用模块下的`build.gradle`

```
android {
	packagingOptions {
		exclude 'META-INF/LICENSE'
		exclude 'META-INF/NOTICE'
    }
	dependencies {
		compile 'io.petchat:senzsdk:2.0.16'
		compile 'com.wilddog:wilddog-client-android:0.5.1+'
        	compile 'com.github.zishell:MotionClassifyLib:v1.0.5'
		    
	}
}
```


## 2、 使用SENZ SDK

### 2.1 初始化
在MainActivity 的 onCreate()中（必须在UI主线程中调用），调用：
```java
//please register on senz server to get your own appId
    Senz.initialize(MainActivity.this,appId);
```
appId为您在Dashboard注册新建的应用id。
### 2.2 Senz核心api接口


####2.2.1 用户属性UserInfo类
该类主要用于分析用户的属性，如：年龄、性别、消费能力、社会属性等。

调用例子：
```java
    UserInfo userInfo = new UserInfo(this);
        userInfo.getUserInfo(new SenzCallback() {
            @Override
            public void done(Exception e, Object object) {
                if (e == null) {
                    HashMap<String, Object> map = (HashMap<String, Object>) object;
                    //...
                   
                } else {
                   //...
                }
            }
        });
```
#####hashMap数据格式说明
UserInfo 包含用户属性和用户喜好，
用户属性：

```json
{
    "age": [
        "16down",
        "16to35",
        "35to55",
        "55up"
    ],
    "has_pet": [
        "positive",
        "negative"
    ],
    "has_car": [
        "positive",
        "negative"
    ],
    "occupation": [
        "engineer",
        "salesman",
        "teacher",
        "student",
        "soldier",
        "official",
        "supervisor",
        "freelancer",
        "others"
    ],
    "pregnant": [
        "positive",
        "negative"
    ],
    "field": [
        "infotech",
        "commerce",
        "law",
        "athlete",
        "medical",
        "human_resource",
        "financial",
        "architecture",
        "humanities",
        "natural",
        "manufacture",
        "agriculture",
        "service"
    ],
    "marriage": [
        "positive",
        "negative"
    ],
    "consumption": [
        "5000down",
        "5000to10000",
        "10000to20000",
        "20000up"
    ],
    "gender": [
        "positive",
        "negative"
    ]
}
```

用户喜好：

```json
{
    "sport": [
        "jogging",
        "fitness",
        "basketball",
        "football",
        "badminton",
        "bicycling",
        "table_tennis"
    ],
    "sports_news": [
        "positive",
        "negative"
    ],
    "variety_show": [
        "positive",
        "negative"
    ],
    "tvseries_show": [
        "positive",
        "negative"
    ],
    "study": [
        "positive",
        "negative"
    ],
    "social": [
        "positive",
        "negative"
    ],
    "offline_shoppng": [
        "positive",
        "negative"
    ],
    "indoorsman": [
        "positive",
        "negative"
    ],
    "gamer": [
        "positive",
        "negative"
    ],
    "sports_show": [
        "positive",
        "negative"
    ],
    "online_shopping": [
        "positive",
        "negative"
    ],
    "current_news": [
        "positive",
        "negative"
    ],
    "business_news": [
        "positive",
        "negative"
    ],
    "game_news": [
        "positive",
        "negative"
    ],
    "health": [
        "positive",
        "negative"
    ],
    "game_show": [
        "positive",
        "negative"
    ],
    "acg": [
        "positive",
        "negative"
    ],
    "tech_news": [
        "positive",
        "negative"
    ],
    "entertainment_news": [
        "positive",
        "negative"
    ]
}
```
例如，如果您想获取用户的消费信息：

```java
if (map.containsKey("consumption")) {
        HashMap<String, Double> consumption = (HashMap<String, Double>) map.get("consumption");
        }
```
返回一个HashMap<String, Double>：
```
10000to20000 : 0.24214
20000up : 0.15570
5000down : 0.15272
5000to10000 : 0.13414
```
如果想获取用户在tech_news的喜好，则返回一个Double值。即，如果key下有子类会返回HashMap，如果没有子类则返回相应的预测值（double）

####2.2.2 用户的情境识别 context(*deprecated*)

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
 
 poiProbLv1 和 poiProbLv2 为用户location的情境识别， sceneProb 为用户的所处场景识别， motionProb 为
 用户的动作情境识别。
 属性保存在了key中，对应的值保存在了value中，
 键值不是固定的，请使用者自行遍历相应的map。
 


####2.2.3 用户事件识别 event
 event指用户发生的事件，如某个时间段，用户在看电影或者逛商场。
 event所包含的所有事件类型：
```
0. 在家: contextAtHome
1. 上班路上: contextCommutingWork
2. 在公司: contextAtWork
3. 回家路上: contextCommutingHome
4. 商圈工作中: contextWorkingInCBD
5. 学校上课中: contextStudyingInSchool {mapping to contextAtSchool.png}
6. 学校工作中: contextWorkingInSchool {mapping to contextAtSchool.png}
7. 户外锻炼: contextOutdoorExercise
8. 室内锻炼 contextIndoorExercise
9. 在餐厅吃饭: contextDinningOut
10. 旅游: contextTravelling
11. 出行: contextGoingOut
12. 聚会: contextInParty
13. 逛街: contextWindowShopping 
14. 看电影: contextAtCinema
15. 展览会: contextAtExhibition
16. 演唱会: contextAtPopsConcert
17. 戏剧: contextAtTheatre
18. 音乐会: contextAtClassicsConcert
19. 在家休息：contextRelaxing
```
 
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
                SenzEvent event = (SenzEvent) data;
                //...
            }
        }
    };

 ```
 
#####数据格式说明
 SenzEvent 类：
 ```java
 public class SenzEvent implements Serializable {
    public long timestamp;     //更新时间
    public String eventType;   //event类型
    public double probability;  //可信度
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
    public long updateTime;   //更新时间
    public String stautsType;  //状态类型
    public double probability;  //可信度
    }

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
    public String motionType;   //motion类型
    public double probability;   //可信度
    public long timestamp;      //更新时间
    }

 ```
##2.3 设置监听的另一种方法

 - 1.
在manifest中注册想要监听的事件：
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
编写自己的类，继承自 BroadcastReceiver，监听相应的action，并从intent中提取相应的值。
三种事件的值，通过序列化后的键分别为："motion_changed", "event", "home_office_status"。
请参考下面的例子：
```java
public class YourReceiver extends BroadcastReceiver {

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
