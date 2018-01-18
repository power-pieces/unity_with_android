## 引言
> 最近为了实现Unity与Android之间的通信，在网络上发现了很多种实现方案。有打包Jar的，有打包aar的，有直接拷贝文件的。试了几种方案虽然都能解决需求，但是使用起来给我的感觉并不是很舒服。在各种尝试中，已了解了Unity和Android之间通信的底层原理。该方案为本人结合Java特性所给出，可以减少很多其它方案的一些不明确以及繁琐的步骤。

## 本文适用对象
* 有一定的Unity开发经验，会使用Unity
* 有一定的Android开发经验，会使用AndroidStudio

## 方案优势
* 不需要引用unity下的class.jar
* 不用在Unity的/Plugins/Android下放置AndroidManifest.xml文件
* Unity打包时PackageName不依赖于引用文件
* 发布简单，只需要导出arr并直接拷贝到/Plugins/Android目录下即可使用，不用对文件做任何修改


## 文章对应的IDE版本
* AndroidStudio 2.1
* Unity 5.4.3 
 

# 流程
### Android部分
##### 创建AndroidStudio项目
1. 首先我们打开AndroidStudio，并创建一个新项目，这里随便填写项目名、包名即可，因为这个项目我们后面并不会用到。
2. SDK我们选最低的就行。
3. Activity我们选个EmptyActivity也行。
![1.png](http://upload-images.jianshu.io/upload_images/9825434-eddd9988e0910fce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后我们点击「Finish」完成AndroidStudio项目创建。

##### 创建和unity交互的Moudle项目
1. 项目创建好以后开始我们的主菜，选中app然后新建一个moudle
![2.png](http://upload-images.jianshu.io/upload_images/9825434-b83f3359664ed5b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 类型选择「Android Library」
![3.png](http://upload-images.jianshu.io/upload_images/9825434-216b4310ce8e85d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. Application/Library name认真填写，之后为arr导出的名称，这里我们叫「MyUnityLib」。
4. Module name没有强迫症就不用管它
5. Package name认真填写，之后unity里会用到，不过它和unity导出的包名没有什么关系这里我们叫「com.jing.unity」好了
6. Minimum SDK能选多低选多低，反正不超过unity发布的版本就行
![4.png](http://upload-images.jianshu.io/upload_images/9825434-064469538ba3b5af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

7. 创建

8. 然后我们在com.jing.unity包下创建一个类，作为Unity和Android通信的核心类，名字尽量炫酷一点，这里我们叫「Unity2Android」
![6.png](http://upload-images.jianshu.io/upload_images/9825434-0b1f5ba4a446968f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 编写Android端代码
9. 然后我们直接粘贴该类的代码，讲解直接看注释。这里我们通过Java的反射原理来获取本来导入class.jar类才能引用到的com.unity3d.player.UnityPlayer包下的currentActivity上下文。同理给unity发消息也是反射原理。「getActivity」和「callUnity」这两个方法，有一定的开发经验应该很容易理解。
这里我们实现一个简单的接口「showToast」。

            package com.jing.unity;
            
            import android.app.Activity;
            import android.widget.Toast;
            
            import java.lang.reflect.InvocationTargetException;
            import java.lang.reflect.Method;
            
            /**
             * Created by Jing on 2018-1-18.
             */
            public class Unity2Android {
            
                /**
                 * unity项目启动时的的上下文
                 */
                private Activity _unityActivity;
                /**
                 * 获取unity项目的上下文
                 * @return
                 */
                Activity getActivity(){
                    if(null == _unityActivity) {
                        try {
                            Class<?> classtype = Class.forName("com.unity3d.player.UnityPlayer");
                            Activity activity = (Activity) classtype.getDeclaredField("currentActivity").get(classtype);
                            _unityActivity = activity;
                        } catch (ClassNotFoundException e) {
            
                        } catch (IllegalAccessException e) {
            
                        } catch (NoSuchFieldException e) {
            
                        }
                    }
                    return _unityActivity;
                }
            
                /**
                 * 调用Unity的方法
                 * @param gameObjectName    调用的GameObject的名称
                 * @param functionName      方法名
                 * @param args              参数
                 * @return                  调用是否成功
                 */
                boolean callUnity(String gameObjectName, String functionName, String args){
                    try {
                        Class<?> classtype = Class.forName("com.unity3d.player.UnityPlayer");
                        Method method =classtype.getMethod("UnitySendMessage", String.class,String.class,String.class);
                        method.invoke(classtype,gameObjectName,functionName,args);
                        return true;
                    } catch (ClassNotFoundException e) {
            
                    } catch (NoSuchMethodException e) {
            
                    } catch (IllegalAccessException e) {
            
                    } catch (InvocationTargetException e) {
            
                    }
                    return false;
                }
            
                /**
                 * Toast显示unity发送过来的内容
                 * @param content           消息的内容
                 * @return                  调用是否成功
                 */
                public boolean showToast(String content){
                    Toast.makeText(getActivity(),content,Toast.LENGTH_SHORT).show();
                    //这里是主动调用Unity中的方法，该方法之后unity部分会讲到
                    callUnity("Main Camera","FromAndroid", "hello unity i'm android");
                    return true;
                }
            }


##### 导出arr准备给unity使用
10. 代码写好了我们选中module然后选择「Build」「Rebuild Project」
![7.png](http://upload-images.jianshu.io/upload_images/9825434-feffddaaa8148784.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

11. 接着将这个arr文件找到，就是我们要导入到unity的文件了。

### Unity部分
1.  创建一个unity项目
2.  创建目录Assets/Plugins/Android，并将刚才导出的arr文件放到该文件夹下，我们的导入就算完成了。没错就是这么Easy，然后我们看看怎么来调用它。
![8.png](http://upload-images.jianshu.io/upload_images/9825434-89774cf4717820eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.  在界面上放一个按钮，并且创建一个Script绑定到「Main Camera」。用一个文本控件来展示Android发送过来的消息。
4.  Script的代码内容如下

        using UnityEngine;
        using UnityEngine.UI;
        
        public class Main : MonoBehaviour {
        
            /// <summary>
            /// 场景上的文本框用来显示android发送过来的内容
            /// </summary>
            public Text text;
        
            /// <summary>
            /// android原生代码对象
            /// </summary>
            AndroidJavaObject _ajc;
        
            void Start () {
                //通过该API来实例化导入的arr中对应的类
                _ajc = new AndroidJavaObject("com.jing.unity.Unity2Android");
            }
        	
        	void Update () {
        		
        	}
        
            /// <summary>
            /// 场景上按点击时触发该方法
            /// </summary>
            public void OnBtnClick()
            {
                //通过API来调用原生代码的方法
                bool success = _ajc.Call<bool>("showToast","this is unity");
                if(true == success)
                {
                    //请求成功
                }
            }
        
            /// <summary>
            /// 原生层通过该方法传回信息
            /// </summary>
            /// <param name="content"></param>
            public void FromAndroid(string content)
            {
                text.text = content;
            }
        }

5. 然后打包APK到我们的Android设备上进行测试。
![9.png](http://upload-images.jianshu.io/upload_images/9825434-4fb8f6021fede209.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
6. 点击按钮，查看效果
![10.png](http://upload-images.jianshu.io/upload_images/9825434-4c42ed411198c88a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## DEMO地址
* 「国外git」GitHub：
* 「国内git」Coding：[https://coding.net/u/jinglikeblue/p/unity_with_android/git](https://coding.net/u/jinglikeblue/p/unity_with_android/git)

## 结束语
- aar和jar的区别各位可以自行百度了解。
- 如果要对接第三方库，可以在moudle下对接，并打包aar给Unity使用。切记jar需要放到aar的libs下引用，才可以在打包的时候一并导出。通过gradle的网络下载编译方式是不会被打包到aar中的。gradle网络下载的文件的jar可以自行百度查看如何找到。