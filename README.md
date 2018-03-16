# BuildSrc

## 1、概述
为统一工程结构，依赖关系，所构建的项目基础结构

## 2、配置文件
按项目需要配置cc.ini  
文件分两部分属性project、cc  

### 2.1、project：
直接生成apk的项目
#### 属性：
repository：仓库地址  
branch：分支  

路径：project/
### 2.2、cc：
project所依赖的project，由若干大分组组成，每个分组下面可包含多个仓库
#### 路径
cc/
#### 分组属性
directory：分组目录名  
repositories：分组下的仓库列表  
路径：cc/${directory}/  

#### 仓库属性
name：仓库目录名  
repository：仓库地址  
branch：分支名  
路径：cc/${directory}/${name}/  

**cc.ini example**

    {
        "project": {
            "repository": "ssh://source.skyworth.com/skyworth/Application/SkyCCMall",
            "branch": "feature/v2.5-orderpay"
        },
        "cc": [
            {
                "directory": "sdk",
                "repositories": [
                    {
                        "name": "AppCore",
                        "repository": "http://source.skyworth.com:3000/MoneyApps/AppCore.git",
                        "branch": "master"
                    },
                    {
                        "name": "Framework",
                        "repository": "ssh://source.skyworth.com/skyworth/CoocaaOS/Framework",
                        "branch": "CCOS/Rel6.0"
                    }
                ]
            },
            {
                "directory": "sdk1",
                "repositories": [
                    {
                        "name": "SkyAppUISDK",
                        "repository": "ssh://source.skyworth.com/skyworth/Application/SkyAppUISDK",
                        "branch": "operate_framework"
                    },
                    {
                        "name": "AF_UpgraderSDK",
                        "repository": "http://source.skyworth.com:3000/MoneyApps/AF_UpgraderSDK.git",
                        "branch": "master"
                    },
                    {
                        "name": "AF_Coupon",
                        "repository": "http://source.skyworth.com:3000/MoneyApps/AF_Coupon.git",
                        "branch": "master"
                    },
                    {
                        "name": "AF_SkyLogSDK",
                        "repository": "http://source.skyworth.com:3000/MoneyApps/AF_LogSDK.git",
                        "branch": "master"
                    }
                ]
            }
        ]
    }

## 3、gradle相关内容
### 3.1、gradle plugin
有两个gradle plugin

#### 3.1.1、com.coocaa.gradle.plugin.config
这个plugin默认在根下的build.gradle中apply了，所以不用再特别关注的去依赖，但是，这里有若干的属性和task需要关注。

**环境属性**  
gradle.ext.androidJar           ：android.jar路径  
gradle.ext.layoutlibJar         ：layoutlib.jar路径  
gradle.ext.api                  ：对应android modual的build.gradle中的compileSdkVersion取值  
gradle.ext.buildTools           ：对应android modual的build.gradle中的buildToolsVersion取值  
gradle.ext.minSdkVersion        ：对应android modual的build.gradle中的minSdkVersion取值  
gradle.ext.targetSdkVersion     ：对应android modual的build.gradle中的targetSdkVersion取值  

**example:**

    apply plugin: 'com.android.library'
    apply plugin: 'kotlin-android'

    android {
        compileSdkVersion gradle.ext.api
        buildToolsVersion gradle.ext.buildTools

        defaultConfig {
            minSdkVersion gradle.ext.minSdkVersion
            targetSdkVersion gradle.ext.targetSdkVersion
            versionCode 1
            versionName "1.1"
        }
        buildTypes {
            release {
                minifyEnabled false
                proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            }
        }
        useLibrary 'org.apache.http.legacy'

        sourceSets {
            main {
                manifest.srcFile 'AndroidManifest.xml'
                java.srcDirs = ['src']
                resources.srcDirs = ['src']
                aidl.srcDirs = ['src']
                renderscript.srcDirs = ['src']
                res.srcDirs = ['res']
                assets.srcDirs = ['assets']
                jniLibs.srcDirs = ['lib']
            }
        }

    }

    dependencies {
        compile fileTree(dir: 'libs', include: ['ok*.jar'])
        compile project(':AppCore')
        provided files(gradle.ext.layoutlibJar)
    }

**签名属性**  
gradle.ext.sign_plaform         ：当前通用的系统签名  
gradle.ext.sign_cloudtv801      ：801cloudtv的系统签名  
gradle.ext.sign_superx          ：应用圈特有签名  
gradle.ext.sign_tianci801       ：天赐801系统签名  
gradle.ext.sign_android_default ：不知名的签名

**example:**

    buildTypes {
        debug {
            minifyEnabled true
            zipAlignEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig gradle.ext.sign_plaform
            ndk {
                abiFilter "armeabi-v7a"
            }
        }
        release {
            minifyEnabled true
            zipAlignEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig gradle.ext.sign_plaform
            ndk {
                abiFilter "armeabi-v7a"
            }
        }
    }
**task**  

**gradlew ccinit**  
根据配置的cc.ini文件，clone仓库到指定目录，并初始化cc_settings.gradle依赖。

#### 3.1.2、com.coocaa.gradle.plugin.builder
这个plugin需要在最终生成apk的modual的build.gradle中apply。此plugin的作用，做编译、指定目录且按规则命名输出的apk、自增编译版本号。


./gradlew.bat ccinit

注意：每个模块下都需要配置cc_settings.gradle