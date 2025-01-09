# android studio多渠道打包基础配置

![API](https://img.shields.io/badge/API-15%2B-green) ![license](https://img.shields.io/badge/License-Apache%202.0-blue)

android studio使用gradle作为构建工具，这里简单记录一下使用gradle进行多渠道打包的基础知识

## 笔记
笔记是这个：220209_gradle教程笔记

## 配置flavorDimensions
这个是配置多维度，必须配置，可以是一个或多个维度
如果只有1个维度，在productFlavors里可以不指定这个维度，如果有多个维度，必须指定是哪个维度
而且如果有多个维度productFlavors里面的配置必须要每个维度都有所指定，比如你配置了2个维度，但是在productFlavors
里面却只使用了一个，则会报错。
flavorDimensions配置如下：
```Groovy
    flavorDimensions "company", "version"
```
android 3.0只有也可以配置flavorDimensions "default"，这样将不需要在productFlavors配置dimension这个维度，
相当于直接忽略了flavorDimensions，只有productFlavors这个维度了。

## 配置signingConfigs
这个是配置签名文件的，signingConfigs的配置需要在productFlavors前面，否则productFlavors
里面使用signingConfigs会报错
```Groovy
    signingConfigs {
        test1 {
            storeFile file('../jks/test1')
            storePassword "123456"
            keyAlias "key0"
            keyPassword "123456"
        }
        test2 {
            storeFile file('../jks/test2')
            storePassword "123456"
            keyAlias "key0"
            keyPassword "123456"
        }
    }
```
## 配置buildTypes
这里可以理解为编译类型，例如debug，release，当然也可以添加自定义的类型
```Groovy
    buildTypes {
        other {
            debuggable true
            jniDebuggable true
            zipAlignEnabled true
            minifyEnabled false
            shrinkResources false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-project.txt'
        }

        debug {
            debuggable true
            jniDebuggable true
            zipAlignEnabled true
            minifyEnabled false
            shrinkResources false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-project.txt'
        }

        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
```
buildTypes常用配置字段的含义：<br/>
debbuggable	否生成一个可调式的apk<br/>
minifyEnabled	打开混淆<br/>
multiDexEnabled	是否可以分包<br/>
proguardFiles	指定插件使用的混淆文件<br/>
zipAlignEnabled	是否使用zipAlign优化apk,Android sdk包里面的工具，能够对打包的应用程序进行优化，让整个系统运行的更快<br/>
shrinkResources 打开资源压缩

## 配置productFlavors
在这里进行配置多渠道
```Groovy
    productFlavors {
        c_vivo {
            dimension "company"
        }

        v_test1 {
            signingConfig signingConfigs.test1
            dimension "version"
        }

        v_test2 {
            signingConfig signingConfigs.test2
            dimension "version"
        }
    }
```
指定dimension，其中这里面配置的渠道必须要覆盖了flavorDimensions指定的dimension,否则会报错
另外在这里指定各个渠道的签名

这样配置完成之后，就可以点击工具栏上面的sync project按钮同步project了，同步完成后可以再Build Variants里面查看
配置的渠道

## 添加debug配置
debug加入debug相关的代码和资源，release则是另外的资源

![variant](./docs/images/variant.png)

## 添加多渠道打包用法
1.添加自定义apk文件名
2.复制mapping文件到固定目录，实现备份功能
3.备份aab
4.备份aab mapping文件

## 注意问题
1.添加自定义输出apk文件名的时候，发现执行一次gradlew assembleC_vivoV_test1Release输出文件名后，再次执行不会输出了
然后重新删除app下的build目录，再次执行才可以输出。
可能原因是，打包时app/build/output/apk/c_vivoV_test1/release目录下有个output-metadata.json的文件在文件里面记录了
新的apk路径
      "outputFile": "..\\..\\..\\..\\..\\..\\outputApk\\22_08_12\\Alomet_v1.0_1_2022-08-12_14-00-07_test001_Debug.apk"
打包的时候gradle应该也有这个outputFile的缓存，如果缓存一样，gradle直接就不输出文件了，如果修改上面outputFile的值，
即使不删除build目录，gradle打包也会有新的apk文件输出


## 修改成gradle 8.0遇到问题
1.编译报错  
Namespace not specified. Specify a namespace in the module's build file. 
See https://d.android.com/r/tools/upgrade-assistant/set-namespace for information about setting the namespace.
If you've specified the package attribute in the source AndroidManifest.xml, you can use the AGP Upgrade 
Assistant to migrate to the namespace value in the build file. Refer to https://d.android.com/r/tools/upgrade-assistant/agp-upgrade-assistant 
for general information about using the AGP Upgrade Assistant.
解决办法
在每个module的build.gradle上添加namespace，这个namespac就是AndroidManifest.xml下的package，并且把
AndroidManifest.xml的package去掉
android {
      namespace "com.cold.flavorsdemo"
      ...
}

2.编译报错
Product Flavor 'ayome' contains custom BuildConfig fields, but the feature is disabled.
To enable the feature, add the following to your module-level build.gradle:
`android.buildFeatures.buildConfig true`
解决办法：
在gradle.properties文件中增加
android.defaults.buildfeatures.buildconfig=true

参考文档：
https://blog.csdn.net/zxl1173558248/article/details/131143566

3.编译报错
Gradle JVM version incompatible.
This project is configured to use an older Gradle JVM that supports up to version 11 but the current AGP requires a Gradle JVM that supports version 17.
需要把jdk版本修改为17

4.项目编译报错：
A problem occurred configuring project ':hyk_frame'.
> Could not create task ':hyk_frame:greendaoPrepare'.
> Cannot use @TaskAction annotation on method DetectEntityCandidatesTask.execute() because interface 
> org.gradle.api.tasks.incremental.IncrementalTaskInputs is not a valid parameter to an action method.

解决办法：
把classpath 'org.greenrobot:greendao-gradle-plugin:3.3.0'修改为：
classpath 'org.greenrobot:greendao-gradle-plugin:3.3.2'

参考文档：
AS中 Gradle8.0 配置greendao插件
https://blog.csdn.net/xiaodongvtion/article/details/133708470

5. 编译报错
   Execution failed for task ':hyk_base:kaptGenerateStubsDebugKotlin'.
> 'compileDebugJavaWithJavac' task (current target is 1.8) and 'kaptGenerateStubsDebugKotlin' task (current target is 17) jvm target compatibility should be set to the same Java version.
Consider using JVM toolchain: https://kotl.in/gradle/jvm/toolchain

解决办法：
把
compileOptions {
sourceCompatibility JavaVersion.VERSION_1_8
targetCompatibility JavaVersion.VERSION_1_8
}
改为 17
compileOptions {
sourceCompatibility = rootProject.ext.android.javaVersion
targetCompatibility = rootProject.ext.android.javaVersion
}

参考文档：
https://blog.csdn.net/weixin_45681365/article/details/138167808

6.编译报错
* Exception is:
  org.gradle.internal.execution.WorkValidationException: A problem was found with the configuration of task ':hyk_frame:compileDebugKotlin' (type 'KotlinCompile').
    - Gradle detected a problem with the following location: 'D:\temp5\tuantuan\hyk_frame\src\main\java'.

      Reason: Task ':hyk_frame:compileDebugKotlin' uses this output of task ':hyk_frame:greendao' without declaring an explicit or implicit dependency. This can lead to incorrect results being produced, depending on what order the tasks are executed.

      Possible solutions:
        1. Declare task ':hyk_frame:greendao' as an input of ':hyk_frame:compileDebugKotlin'.
        2. Declare an explicit dependency on ':hyk_frame:greendao' from ':hyk_frame:compileDebugKotlin' using Task#dependsOn.
        3. Declare an explicit dependency on ':hyk_frame:greendao' from ':hyk_frame:compileDebugKotlin' using Task#mustRunAfter.

解决办法：
tasks.whenTaskAdded { task ->
    if (task.name.matches("compile\w*Kotlin")) {
        task.dependsOn('greendao')
    }
}

参考文档：
greenDAO适配AGP 8.0+版本
https://juejin.cn/post/7255224807321681978

7.子模块访问父模块资源失败
解决办法：
gradle.properties添加设置
android.nonTransitiveRClass=false

参考文档：
Android子模块资源文件找不到
https://www.jianshu.com/p/f176563ceca8


5 种方法，来为 Android Studio Flamingo（火烈鸟）版本，准备 你的app构建
https://blog.csdn.net/cnzzs/article/details/140703286

8.使用case R.id.的地方都提示：错误: 需要常量表达式
解决办法：
在gradle.properties中添加下面代码
android.nonFinalResIds=false
参考文档：
Android Constant expression required (case R.id.xxx)
https://blog.csdn.net/GracefulGuigui/article/details/140296210


9.直接运行通过，打包需要升级jdk版本，把11升级到17

10.升级android-junk-code:1.3.3

11.报错ResourceException
com.android.tools.r8.ResourceException: com.android.tools.r8.internal.Sb: I/O 
exception while reading '/home/andrew/StudioProjects/Project/Module/build/intermediates/merged_java_res/netCleanRelease/base.jar':
解决办法：
build.gradle下面的写法：
tasks.whenTaskAdded { task ->
    if (task.name.matches("compile\\w*Kotlin")) {
        task.dependsOn('greendao')
    }
}
替换为：
tasks.configureEach {
    if(name.startsWith("connectedNet")) {
        enabled = false
    }
}

12.打包aab修改aab名称的逻辑修改

13.打包的aap安装运行报错
gradle.properties添加下面配置：
android.enableR8.fullMode=false
一开始是报presenter通过反射创建，初始化失败，启动页就退出了，然后我把启动页的Presenter添加到混淆脚本设置不混淆之后，就报下面的错误
Missing type parameter
参考文档：
使用AGPv8代码混淆后Gson解析报错：Missing type parameter
https://github.com/liangjingkanji/Net/issues/203

14.去掉ButterKnife

15.一部分ext放到app下的build.gradle会提示找不到renameTaskName
把它放到flavorsConfig.gradle之后就可以了




## license

    Copyright 2019 wjianchen13

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.







