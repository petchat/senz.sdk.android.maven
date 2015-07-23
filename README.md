# senz-sdk-android
---
senz android端的sdk，用github做的maven仓库。
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
		compile 'io.petchat:senzsdk:0.0.1'
		compile 'com.firebase:firebase-client-android:2.3.1+'
	}
```


### 使用SENZ SDK
===
### 初始化
在MainActivity 的 onCreate()中（必须在UI主线程中调用），调用：
```java
    Senz.initialize(this);
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
                
                }
            }
            @Override
            public void failed(String s) {
              
            }
        });
 ```
*数据格式说明*
 
 mapPoiProbLv1和mapPoiProLv2为用户location的情景识别，mapSoundPro为用户的声音情景识别，mapMotionProb为
 用户的动作情景识别。
 属性保存在了key中，对应的值保存在了value中，
 键值不是固定的，请使用者自行遍历相应的map。
 
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
 返回结果hashMap中包含n个HashMap<String,Double>数据，保存了用户的event预测数据。
 键值不是固定的，请自行遍历各个map。
 
 
 
 
 













