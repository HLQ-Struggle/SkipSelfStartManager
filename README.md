> LZ-Says：话说现在流氓软件多不胜数，在反感的同时，我们确实应该从技术的角度去分析，为什么他们能够做到所谓的流氓呢？技术不分错对，关键在于使用技术的人去如何运用相关技术~

欢迎关注LZ个人公众号~ 不定期发布博文~

![](https://github.com/HLQ-Struggle/MaterialDesignStudy/blob/master/image/hlq_gzh.jpg)

博文地址如下，欢迎点击~

http://blog.csdn.net/u012400885/article/details/65515941

#### 前言
话说，最近项目要求软件自启动，脑子一想，静态注册广播，监听用户开机不就得了么。按照思路编写好代码，却发现怎么都监听不到这个开机权限。LZ很是郁闷。经过几天的咨询度娘和各种脑洞大开的测试后，发现有个东东叫做**自启动管理**，经过简单测试后发现，用户如果给定软件自启动权限后，我们只需要静态注册开机广播就可以监听到用户开机，并可以针对这一情况做相关操作。

#### 问题延伸

但是问题又来了，那么我们怎么知道用户是否给定软件自启动的权限呢？找了好久，没找到可以解决的办法，倒是有以下几个办法可以曲线救国，让我们一块来看看吧。

> **1.  静态注册广播，本地保存标识。** 主要监听用户开机和关机权限，如果监听到开机权限，本地保存一个标识，证明用户已给定这个权限；而监听到关机的同时将这个标识置为空或者其他一个标识，表明认为用户取消自启动权限。但是这个有个问题我一直想不明白的是，加入一开始用户设置了自启动，但是通过其他手段又禁止自启动，这时候怎么办？有点恶心。。。 
> 而关于广播，有的人又提出监听系统打电话，发短信，闹钟，等等。。。一系列监听，感觉有点头大。。。
>    
> **2. 弹框提示用户** 
> 用户首次登陆的时候，通过引导用户去给本软件设置自启动权限。
> 
> 。。。 。。。
> 
> 还有一些其他的办法，个人感觉不靠谱，就不一一细说了，感兴趣的同志们可以自己找找~

找了好久，恶心的不要不要的，最后索性直接给用户提示吧，通过引导，让用户设置自启动。那么，让我们先看一下效果图吧~

小的们，上图~

![这里写图片描述](http://img.blog.csdn.net/20170323234337518?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjQwMDg4NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#### 设置友好提示，引导用户
由于Android旗下各大手机厂商种类很多，我们第一步需要获取到用户当前使用的手机型号，然后根据相应的手机型号，去跳转不同的自启动界面，从而引导用户设置我们的软件自启动权限。

#### 介绍实现关键类

> **1.获取手机型号(Build)**
> 
> **所属包：** android.os.Build
> **作用(含义)：** 从系统属性中提取设备硬件和版本信息
> **静态属性：**
>  1.1 BOARD 主板：The name of the underlying board, like goldfish.
1.2 BOOTLOADER 系统启动程序版本号：The system bootloader version number.
1.3 BRAND 系统定制商：The consumer-visible brand with which the product/hardware will be associated, if any.
1.4 CPU_ABI cpu指令集：The name of the instruction set (CPU type + ABI convention) of native code.
1.5 CPU_ABI2 cpu指令集2：The name of the second instruction set (CPU type + ABI convention) of native code.
1.6 DEVICE 设备参数：The name of the industrial design.
1.7 DISPLAY 显示屏参数：A build ID string meant for displaying to the user
1.8 FINGERPRINT 唯一识别码：A string that uniquely identifies this build. Do not attempt to parse this value.
1.9 HARDWARE 硬件名称：The name of the hardware (from the kernel command line or /proc).
1.10 HOST
1.11 ID 修订版本列表：Either a changelist number, or a label like M4-rc20.
**1.12 MANUFACTURER 硬件制造商：The manufacturer of the product/hardware.（我们目前只需要关注这个静态属性即可）**
1.13 MODEL 版本即最终用户可见的名称：The end-user-visible name for the end product.
1.14 PRODUCT 整个产品的名称：The name of the overall product.
1.15 RADIO 无线电固件版本：The radio firmware version number. 在API14后已过时。使用 getRadioVersion()代替。
1.16 SERIAL 硬件序列号：A hardware serial number, if available. Alphanumeric only, case-insensitive.
1.17 TAGS 描述build的标签,如未签名，debug等等。：Comma-separated tags describing the build, like unsigned,debug.
1.18 TIME
1.19 TYPE build的类型：The type of build, like user or eng.
1.20 USER



> **2.打开其他应用程序中的Activity或服务(ComponentName)**
> **所属包：** android.content.ComponentName
> **构造方法使用方式如下：**
> 2.1 传递当前上下文和将要跳转的类名；
> 2.2 传递一个String包名和String类名；
> 2.3 传递一个Parcel数据容器。
> **需要关注的方法**：unflattenFromString("传递将要跳转的地址，格式为包名/跳转Activity Name")

当然以上还包括一些未介绍的方法属性，大家有兴趣可以看一下~
基本了解之后，让我们一起开启愉快的编码之路吧~

##### 创建MobileInfoUtils工具类
此工具类作用是获取用户手机型号，以及通过不同手机型号跳转相应管理界面。

同时也为大家通过几种不同方式去实现跳转，大家可以仔细查看代码~

```
package cn.hle.skipselfstartmanager.util;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.net.Uri;
import android.os.Build;
import android.provider.Settings;
import android.util.Log;

/**
 * Mobile Info Utils
 * create by heliquan at 2017年3月23日
 */
public class MobileInfoUtils {

    /**
     * Get Mobile Type
     *
     * @return
     */
    private static String getMobileType() {
        return Build.MANUFACTURER;
    }

    /**
     * GoTo Open Self Setting Layout
     * Compatible Mainstream Models 兼容市面主流机型
     *
     * @param context
     */
    public static void jumpStartInterface(Context context) {
        Intent intent = new Intent();
        try {
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            Log.e("HLQ_Struggle", "******************当前手机型号为：" + getMobileType());
            ComponentName componentName = null;
            if (getMobileType().equals("Xiaomi")) { // 红米Note4测试通过
                componentName = new ComponentName("com.miui.securitycenter", "com.miui.permcenter.autostart.AutoStartManagementActivity");
            } else if (getMobileType().equals("Letv")) { // 乐视2测试通过
                intent.setAction("com.letv.android.permissionautoboot");
            } else if (getMobileType().equals("samsung")) { // 三星Note5测试通过
                componentName = new ComponentName("com.samsung.android.sm_cn", "com.samsung.android.sm.ui.ram.AutoRunActivity");
            } else if (getMobileType().equals("HUAWEI")) { // 华为测试通过
                componentName = new ComponentName("com.huawei.systemmanager", "com.huawei.systemmanager.optimize.process.ProtectActivity");
            } else if (getMobileType().equals("vivo")) { // VIVO测试通过
                componentName = ComponentName.unflattenFromString("com.iqoo.secure/.safeguard.PurviewTabActivity");
            } else if (getMobileType().equals("Meizu")) { //万恶的魅族
                // 通过测试，发现魅族是真恶心，也是够了，之前版本还能查看到关于设置自启动这一界面，系统更新之后，完全找不到了，心里默默Fuck！
                // 针对魅族，我们只能通过魅族内置手机管家去设置自启动，所以我在这里直接跳转到魅族内置手机管家界面，具体结果请看图
                componentName = ComponentName.unflattenFromString("com.meizu.safe/.permission.PermissionMainActivity");
            } else if (getMobileType().equals("OPPO")) { // OPPO R8205测试通过
                componentName = ComponentName.unflattenFromString("com.oppo.safe/.permission.startup.StartupAppListActivity");
            } else if (getMobileType().equals("ulong")) { // 360手机 未测试
                componentName = new ComponentName("com.yulong.android.coolsafe", ".ui.activity.autorun.AutoRunListActivity");
            } else {
                // 以上只是市面上主流机型，由于公司你懂的，所以很不容易才凑齐以上设备
                // 针对于其他设备，我们只能调整当前系统app查看详情界面
                // 在此根据用户手机当前版本跳转系统设置界面
                if (Build.VERSION.SDK_INT >= 9) {
                    intent.setAction("android.settings.APPLICATION_DETAILS_SETTINGS");
                    intent.setData(Uri.fromParts("package", context.getPackageName(), null));
                } else if (Build.VERSION.SDK_INT <= 8) {
                    intent.setAction(Intent.ACTION_VIEW);
                    intent.setClassName("com.android.settings", "com.android.settings.InstalledAppDetails");
                    intent.putExtra("com.android.settings.ApplicationPkgName", context.getPackageName());
                }
            }
            intent.setComponent(componentName);
            context.startActivity(intent);
        } catch (Exception e) {//抛出异常就直接打开设置页面
            intent = new Intent(Settings.ACTION_SETTINGS);
            context.startActivity(intent);
        }
    }

}


```

写完之后，大家可能会有疑问，LZ你是通过什么方式得知的具体包名呢？
哈，有些是Android小伙伴友情赞助，有些是通过adb命令获取，再次为大家介绍下通过adb获取跳转包名。

##### 通过adb获取跳转包名路径
adb为我们提供了一个可以打印出当前系统所有service信息，在后面可加上具体的服务名的牛掰命令，那就是如下：
>**adb shell dumpsys**

在此为大家拓展一些常用的基于dumpsys命令：

> **获取设备电池信息：adb shell dumpsys battery**
> 
> **获取cpu信息：adb shell dumpsys cpuinfo** 
> 
> **获取内存信息：adb shell dumpsys meminfo**
**要获取具体应用的内存信息，可加上包名 **
**adb shell dumpsys meminfo PACKAGE_NAME** 

>**获取Activity信息：adb shell dumpsys activity**
>
>**获取package信息：adb shell dumpsys package**
**加上-h可以获取帮助信息**
**获取某个包的信息：adb shell dumpsys package PACKAGE_NAME** 

>**获取通知信息：adb shell dumpsys notification**
>
>**获取wifi信息：adb shell dumpsys wifi**
**可以获取到当前连接的wifi名、搜索到的wifi列表、wifi强度等** 

>**获取电源管理信息：adb shell dumpsys power**
**可以获取到是否处于锁屏状态：mWakefulness=Asleep或者mScreenOn=false**

>**获取电话信息：adb shell dumpsys telephony.registry**
**可以获取到电话状态，例如mCallState值为0，表示待机状态、1表示来电未接听状态、2表示电话占线状态**
**mCallForwarding=false #是否启用呼叫转移**
**mDataConnectionState=2 #0：无数据连接 1：正在创建数据连接 2：已连接mDataConnectionPossible=true  #是否有数据连接mDataConnectionApn=   #APN名称等**



而我们将通过以下命令获取当前在栈顶Activity包名：

> **adb shell dumpsys activity top**

看命令行字面的意思也就是**获取当前在栈顶Activity信息**(就是咱能看到的界面的具体信息)。

那么接下来拿LZ红米Note4为大家仔细讲解，我们如何通过命令行获取位置界面的包名路径。

###### 使用adb获取位置界面包名路径详情

1.连接手机，手动打开系统自启动管理界面，之后打开Android Studio，点击下方Terminal，输入以上命令，查看信息，如下图：

![这里写图片描述](http://img.blog.csdn.net/20170324004947392?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjQwMDg4NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

我们看到打印出n条信息，但是对我们有用的只是红色区域内，大家这次懂了吧~

##### 在相应的Activity调用，引导用户设置自启动
在此，简单写为点击按钮，弹框，确认跳转自启动管理界面。兄弟们有具体需求在具体考虑实现吧~
```
package cn.hle.skipselfstartmanager;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.DialogInterface;
import android.os.Bundle;
import android.view.View;

import cn.hle.skipselfstartmanager.util.MobileInfoUtils;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void gotoSelfStartingManager(View view){
        jumpStartInterface();
    }

    /**
     * Jump Start Interface
     * 提示是否跳转设置自启动界面
     */
    private void jumpStartInterface() {
        try {
            AlertDialog.Builder builder = new AlertDialog.Builder(this);
            builder.setMessage(R.string.app_user_auto_start);
            builder.setPositiveButton("立即设置",
                    new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            MobileInfoUtils.jumpStartInterface(MainActivity.this);
                        }
                    });
            builder.setNegativeButton("暂时不设置",
                    new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            dialog.dismiss();
                        }
                    });
            builder.setCancelable(false);
            builder.create().show();
        } catch (Exception e) {
        }
    }

}
```
##### 附上三星Note5以及魅族4运行跳转的结果
工具类中只有360手机未测试，其他的测试通过。

1. 三星Note5点击后跳转如下：

![这里写图片描述](http://img.blog.csdn.net/20170324090839548?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjQwMDg4NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2.魅族4点击后跳转如下：

![这里写图片描述](http://img.blog.csdn.net/20170324090855236?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjQwMDg4NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 查阅及下载demo地址

> **1. 查看Demo源码地址：https://github.com/HLQ-Struggle/SkipSelfStartManager**
> 
> **2.CSDN下载地址：http://download.csdn.net/detail/u012400885/9791924**

#### 结束语

> 努力了，还不够，不放弃，才为真。愿大家干的顺心，撸的开心，玩的舒心~
> 我们一起努力！

===============这是一条神奇的分割线===================

[w3812127](http://blog.csdn.net/w3812127)大兄弟指出**在oppo r9s 自启动项的包位置发生变化，并给出了更新后的coding，感谢~在此做一下补充。**

> ("OPPO")) { // OPPO R8205测试通过
componentName =ComponentName.unflattenFromString("com.oppo.safe/.permission.startup.StartupAppListActivity");
Intent intentOppo = new Intent();
intentOppo.setClassName("com.oppo.safe/.permission.startup", "StartupAppListActivity");
if (context.getPackageManager().resolveActivity(intentOppo, 0) == null) {
componentName =ComponentName.unflattenFromString("com.coloros.safecenter/.startupapp.StartupAppListActivity");
}

#### 参考资料
感谢如下提供资料~
1. http://www.2cto.com/kf/201503/379741.html
2. https://testerhome.com/topics/1462

如觉得对你有帮助，不妨赞助LZ抽根烟~

![这里写图片描述](http://upload-images.jianshu.io/upload_images/5566153-adc793ed3316f1eb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
