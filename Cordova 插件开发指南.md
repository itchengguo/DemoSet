###### 插件开发指南

​       插件是一个注入代码的包，它允许应用程序呈现的Cordova webview与它运行的本机平台进行通信。插件提供了对基于web的应用程序通常不可用的设备和平台 功能的访问。所有主要的Cordova API特性都是作为插件实现的，还有许多其他的功能可以支持诸如条形码扫描器、NFC通信或定制日历接口等功能。您可以在Cordova插件搜索页面上搜索可用的插件。

​      插件包括一个单独的JavaScript接口，以及每个支持的平台对应的本地代码库。本质上，这隐藏了一个公共JavaScript接口背后的各种原生代码实现。

​      本节介绍一个简单的echo插件，它将一个字符串从JavaScript传递到本机平台，然后返回，您可以使用它作为模型来构建更复杂的特性。本节讨论基本的插件结构和面向外的JavaScript接口。对于每个对应的Native接口，请参见本节末尾的列表。

​     除了这些指令之外，在准备编写插件时，最好先查看现有的插件以获得指导。

###### 构建一个插件

​     应用程序开发人员使用CLI的插件添加命令将插件添加到项目中。该命令的参数是包含插件代码的git存储库的URL。这个例子实现了Cordova的设备API:

```
cordova plugin add https://git-wip-us.apache.org/repos/asf/cordova-plugin-device.git
```

插件库必须有一个顶级的`plugin.xml`清单文件。有很多方法可以配置这个文件，详细信息可以在插件规范中找到。这个简化版的设备插件(`device plugin`)提供了一个简单的例子来作为一个模型:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<plugin xmlns="http://apache.org/cordova/ns/plugins/1.0"
        id="cordova-plugin-device" version="0.2.3">
    <name>Device</name>
    <description>Cordova Device Plugin</description>
    <license>Apache 2.0</license>
    <keywords>cordova,device</keywords>
    <js-module src="www/device.js" name="device">
        <clobbers target="device" />
    </js-module>
    <platform name="ios">
        <config-file target="config.xml" parent="/*">
            <feature name="Device">
                <param name="ios-package" value="CDVDevice"/>
            </feature>
        </config-file>
        <header-file src="src/ios/CDVDevice.h" />
        <source-file src="src/ios/CDVDevice.m" />
    </platform>
</plugin>
```

顶级`plugin`标签的`id`属性使用相同的反向域格式来(`对咱们来说就是package名`)识别插件包，因为它们添加了应用程序。`js-module`标记指定了公共JavaScript接口的路径。`platform`标签指定了相应的本地代码，在本例中为`IOS`平台。`The config-file tag encapsulates a feature tag that is injected into the platform-specific config.xml file to make the platform aware of the additional code library.`  `header-file`和`source-file`标记指定了库的组件文件的路径。

###### 使用Plugman验证插件

   您可以使用`plugman`实用程序来检查插件是否为每个平台安装正确。使用以下`node`命令安装`plugman`:

```
npm install -g plugman
```

您需要一个有效的应用程序源目录，如在[创建您的第一个应用程序](http://cordova.axuer.com/docs/zh-cn/latest/guide/cli/index.html)指南中所描述的默认`clic`生成项目中包含的顶级www目录。

然后运行以下命令，以测试iOS依赖项是否正确加载:

```
plugman install --platform ios --project /path/to/my/project/www --plugin /path/to/my/plugin
```

有关`plugman`选项的详细信息，请参见使用`plugman来`管理插件。有关如何实际调试插件的信息，请参见本页面底部列出的每个平台的本机接口。

###### JavaScript接口

   JavaScript提供了前置接口，使其成为插件中最重要的部分。你可以随心所欲地构造你的插件的JavaScript，但是你需要调用`cordova.exec`使用以下语法与Native平台进行通信:

```javascript
cordova.exec(function(winParam) {},
             function(error) {},
             "service",
             "action",
             ["firstArgument", "secondArgument", 42, false]);
```

下面是每个参数的工作方式:

- `function(winParam) {}`  一个成功的回调函数.假设您的`exec`调用成功完成，此函数将执行您传递给它的任何参数。
- `function(error) {}`  一个错误的回调函数。如果操作未成功完成，此函数将执行一个可选的错误参数。
- `"service" `调用原生一侧的服务名称。这对应于一个原生类，在下面列出的原生指南中提供了更多的信息。
- `"action" ` 调用原生一侧的方法名称。调用本机一侧的动作名称。他通常与原生类方法相对应。参见下面列出的本地向导。
- `[/* arguments */]` 传入原生环境的参数数组。

###### 示例JavaScript

这个例子展示了实现插件JavaScript接口的一种方法:

```javascript
window.echo = function(str, callback) {
    cordova.exec(callback, function(err) {
        callback('Nothing to echo.');
    }, "Echo", "echo", [str]);
};
```

在本例中，插件将自己附加到`window`对象作为`echo`函数，插件用户将调用如下:

```javascript
window.echo("echome", function(echoValue) {
    alert(echoValue == "echome"); // should alert true.
});
```

查看`cordova.exec`函数的最后三个参数.第一个调用是Echo服务，一个类名。第二个请求是echo操作，它是该类中的一个方法。第三个参数是包含echo字符串的参数数组，它是`window.echo`函数是第一个参数。

成功回调传递到执行仅仅是一个参考的回调函数`window.echo`。如果本地平台触发错误回调，它只调用成功回调并将其传递为默认字符串。

###### Native Interfaces

一旦你为你的插件定义了JavaScript，你就需要用至少一个本地实现来补充它。下面列出了每个平台的详细信息，并且每个平台都建立在上面的简单的Echo插件示例中:

- [Android Plugins](http://cordova.axuer.com/docs/zh-cn/latest/guide/platforms/android/plugin.html)
- [iOS Plugins](http://cordova.axuer.com/docs/zh-cn/latest/guide/platforms/ios/plugin.html)
- [BlackBerry 10 Plugins](http://cordova.axuer.com/docs/zh-cn/latest/guide/platforms/blackberry10/plugin.html)
- [Windows Phone 8 Plugins](http://cordova.axuer.com/docs/zh-cn/latest/guide/platforms/wp8/plugin.html)
- [Windows Plugins](http://cordova.axuer.com/docs/zh-cn/latest/guide/platforms/win8/plugin.html)



##### Android插件开发指南

本节提供关于如何在Android平台上实现本地插件代码的详细信息。在阅读本文之前，请参阅插件开发指南，以了解插件的结构及其常见的JavaScript接口。本节继续演示从Cordova webview到本机平台和返回的示例echo插件。对于另一个示例，请参见`CordovaPlugin.java`中的注释。

Android插件是基于Cordova-Android的，它是用一个本地桥的Android WebView构建的。Android插件的本机部分由至少由一个继承CordovaPlugin的java类组成，并覆盖了它的一个`exec`方法。

###### Plugin类映射

插件的JavaScript接口使用了`cordova.exec `方法如下:

```
exec(<successFunction>, <failFunction>, <service>, <action>, [<args>]);
```

这将从WebView到Android本机端的请求，有效地调用服务类上的action方法，并在args数组中传递额外的参数。

无论您是将插件作为Java文件还是作为一个jar文件分发，插件都必须在您的Cordova-Android应用程序的`res/xml/config.xml `文件中指定。有关如何使用插件的更多信息，请参见注入该特性元素的`plugin.xml`文件:

```xml
<feature name="<service_name>">
    <param name="android-package" value="<full_name_including_namespace>" />
</feature>
```

服务名称与JavaScript` exec`调用中使用的名称相匹配。value该值是Java类的完全限定名称空间标识符。 (java类的全名,包括包名).否则，该插件可能会编译通过，但cordova仍然无法使用。

###### 插件初始化和生命周期

一个插件对象的实例是为每个WebView的生命创建的。除非在`config.xml` 中设置为“true”，否则，除非`param ` `name` 属性为`onload` 的value被设置为“true”，否则插件不会被实例化，直到它们被来自JavaScript的调用第一次引用。例如,

```xml
<feature name="Echo">
    <param name="android-package" value="<full_name_including_namespace>" />
    <param name="onload" value="true" />
</feature>
```

插件应该使用`initialize` 方法来进行启动逻辑

```java
@Override
public void initialize(CordovaInterface cordova, CordovaWebView webView) {
    super.initialize(cordova, webView);
    // your init code here
}
```

插件还可以访问Android生命周期事件，并可以通过扩展提供的方法之一(onResume、onDestroy等)来处理它们。具有长时间运行请求的插件、后台活动(如媒体播放、侦听器或内部状态)应该实现onReset()方法。当WebView导航到一个新页面或刷新时，它会执行重新加载JavaScript。

###### 编写一个Android Java插件

一个JavaScript调用向原生触发一个插件请求，相应的Java插件在config.xml被正确映射。但是最终的Android Java插件类是什么样的呢?无论如何将JavaScript的exec函数发送给插件，都将传递到plugin类的execute方法中。

大多数`execute`实现如下所示

```java
@Override
public boolean execute(String action, JSONArray args, CallbackContext callbackContext) throws JSONException {
    if ("beep".equals(action)) {
        this.beep(args.getLong(0));
        callbackContext.success();
        return true;
    }
    return false;  // Returning false results in a "MethodNotFound" error.
}
```

`The JavaScript exec function's action parameter corresponds to a private class method to dispatch with optional parameters.`

JavaScript exec函数的参数对应于一个私有类方法，可以使用可选参数进行调度。

在捕获异常和返回错误时，重要的是为了清晰起见，错误返回到JavaScript匹配Java的异常名。

###### Threading

插件的JavaScript不会在WebView接口的主线程中运行;相反，它在WebCore线程上运行，`execute`方法也是如此。如果需要与用户界面交互，那么应该使用`Activity`的`runOnUiThread`方法:

```java
@Override
public boolean execute(String action, JSONArray args, final CallbackContext callbackContext) throws JSONException {
    if ("beep".equals(action)) {
        final long duration = args.getLong(0);
        cordova.getActivity().runOnUiThread(new Runnable() {
            public void run() {
                ...
                callbackContext.success(); // Thread-safe.
            }
        });
        return true;
    }
    return false;
}
```

如果您不需要在UI线程上运行，但是也不希望阻塞WebCore线程，您应该使用`cordova.getThreadPool()`获得的`Cordova ExecutorService`执行您的代码.像这样

```java
@Override
public boolean execute(String action, JSONArray args, final CallbackContext callbackContext) throws JSONException {
    if ("beep".equals(action)) {
        final long duration = args.getLong(0);
        cordova.getThreadPool().execute(new Runnable() {
            public void run() {
                ...
                callbackContext.success(); // Thread-safe.
            }
        });
        return true;
    }
    return false;
}
```

```
## Adding Dependency Libraries[**](http://cordova.axuer.com/docs/zh-cn/latest/guide/platforms/android/plugin.html#adding-dependency-libraries)

If your Android plugin has extra dependencies, they must be listed in the `plugin.xml` in one of two ways.

The preferred way is to use the `<framework />` tag (see the [Plugin Specification](http://cordova.axuer.com/docs/zh-cn/latest/plugin_ref/spec.html#framework) for more details). Specifying libraries in this manner allows them to be resolved via Gradle's [Dependency Management logic](https://docs.gradle.org/current/userguide/dependency_management.html). This allows commonly used libraries such as *gson*, *android-support-v4*, and *google-play-services* to be used by multiple plugins without conflict.

The second option is to use the `<lib-file />` tag to specify the location of a jar file (see the [Plugin Specification](http://cordova.axuer.com/docs/zh-cn/latest/plugin_ref/spec.html#lib-file) for more details). This approach should only be used if you are sure that no other plugin will be depending on the library you are referencing (e.g. if the library is specific to your plugin). Otherwise, you risk causing build errors for users of your plugin if another plugin adds the same library. It is worth noting that Cordova app developers are not necessarily native developers, so native platform build errors can be especially frustrating.
```



###### 添加依赖库

如果你的Android插件有额外的依赖项，它们必须在`plugin.xml`中列出，有两种方式。

​      首选的方法是使用标记(更多细节见插件规范)。以这种方式指定库可以通过Gradle的依赖管理逻辑来解决它们。这使得通常使用的库如gson、android- supportv4和google-play-服务可以被多个插件使用，而不需要冲突。

​      第二个选项是使用标记来指定jar文件的位置(更多细节请参见插件规范)。只有在确定没有其他插件依赖于你所引用的库(例如，如果库是特定于你的插件)时，才应该使用此方法。否则，如果另一个插件添加了相同的库，那么您可能会为插件的用户造成构建错误。值得注意的是，Cordova应用程序开发人员不一定是本地开发人员，因此本地平台构建错误尤其令人沮丧。

###### echo Android插件的例子

要匹配应用程序插件中描述的JavaScript接口的echo特性，请使用plugin.xml为本地平台的`config.xml`

```xml
<platform name="android">
    <config-file target="config.xml" parent="/*">
        <feature name="Echo">
            <param name="android-package" value="org.apache.cordova.plugin.Echo"/>
        </feature>
    </config-file>

    <source-file src="src/android/Echo.java" target-dir="src/org/apache/cordova/plugin" />
</platform>
```

然后将以下内容添加到` src/android/Echo.java`文件:

```java
package org.apache.cordova.plugin;

import org.apache.cordova.CordovaPlugin;
import org.apache.cordova.CallbackContext;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

/**
* This class echoes a string called from JavaScript.
*/
public class Echo extends CordovaPlugin {

@Override
public boolean execute(String action, JSONArray args, CallbackContext callbackContext) throws JSONException {
    if (action.equals("echo")) {
        String message = args.getString(0);
        this.echo(message, callbackContext);
        return true;
    }
    return false;
}

private void echo(String message, CallbackContext callbackContext) {
    if (message != null && message.length() > 0) {
        callbackContext.success(message);
    } else {
        callbackContext.error("Expected one non-empty string argument.");
    }
}
}
```

```
The necessary imports at the top of the file extends the class from `CordovaPlugin`, whose `execute()` method it overrides to receive messages from `exec()`. The `execute()` method first tests the value of `action`, for which in this case there is only one valid `echo` value. Any other action returns `false` and results in an `INVALID_ACTION` error, which translates to an error callback invoked on the JavaScript side.

Next, the method retrieves the echo string using the `args` object's `getString` method, specifying the first parameter passed to the method. After the value is passed to a private `echo` method, it is parameter-checked to make sure it is not `null` or an empty string, in which case `callbackContext.error()` invokes JavaScript's error callback. If the various checks pass, the `callbackContext.success()` passes the original `message` string back to JavaScript's success callback as a parameter.
```

这个文件的类必须是继承CordovaPlugin的类，它的execute()方法可以覆盖从exec()接收消息。execute()方法首先测试操作的值，在本例中只有一个有效的echo值。任何其他操作返回false并导致INVALID_ACTION错误，这将转换为JavaScript端调用的错误回调。

接下来，该方法使用args对象的getString方法检索echo字符串，并指定传递给该方法的第一个参数。在将值传递给私有的echo方法后，它将被参数检查以确保它不是null或空字符串，在这种情况下，callbackContext.error()调用了JavaScript的错误回调

###### 安卓系统集成

Android有一个intent系统，允许进程彼此通信。插件可以访问CordovaInterface对象，该对象可以访问运行该应用程序的Android Activity.这是启动一个新的Android意图所需要的上下文

```
Android features an [Intent](http://developer.android.com/reference/android/content/Intent.html) system that allows processes to communicate with each other. Plugins have access to a`CordovaInterface` object, which can access the Android [Activity](http://developer.android.com/reference/android/app/Activity.html) that runs the application. This is the [Context](http://developer.android.com/reference/android/content/Context.html) required to launch a new Android [Intent](http://developer.android.com/reference/android/content/Intent.html). The `CordovaInterface` allows plugins to start an [Activity](http://developer.android.com/reference/android/app/Activity.html) for a result, and to set the callback plugin for when the [Intent](http://developer.android.com/reference/android/content/Intent.html) returns to the application.

As of Cordova 2.0, Plugins can no longer directly access the [Context](http://developer.android.com/reference/android/content/Context.html), and the legacy `ctx` member is deprecated. All `ctx` methods exist on the [Context](http://developer.android.com/reference/android/content/Context.html), so both `getContext()` and `getActivity()` can return the required object.
```

###### 安卓系统权限

直到最近，Android的权限都是在安装时而不是运行时处理的。这些权限需要在使用权限的应用程序上声明，这些权限需要添加到Android清单中。这可以通过使用config.xml在AndroidManifest.xml文件中注入这些权限。

```xml
<config-file target="AndroidManifest.xml" parent="/*">
    <uses-permission android:name="android.permission.READ_CONTACTS" />
</config-file>
```

###### 运行时的权限(Cordova-Android 5.0.0 +)

Android 6.0“Marshmallow”引入了一个新的权限模型，用户可以根据需要打开和关闭权限。这意味着应用程序必须处理这些权限更改才能成为未来的证据，这是Cordova-Android 5.0.0版本的重点。

他的权限需要在运行时处理，可以在Android开发人员文档中找到。

As far as a plugin is concerned, the permission can be requested by calling the permission method, which signature is as follows:

```java
cordova.requestPermission(CordovaPlugin plugin, int requestCode, String permission);

```

To cut down on verbosity, it's standard practice to assign this to a local static variable:

```java
public static final String READ = Manifest.permission.READ_CONTACTS;

```

It is also standard practice to define the requestCode as follows:

```java
public static final int SEARCH_REQ_CODE = 0;

```

Then, in the exec method, the permission should be checked:

```java
if(cordova.hasPermission(READ))
{
    search(executeArgs);
}
else
{
    getReadPermission(SEARCH_REQ_CODE);
}

```

In this case, we just call requestPermission:

```java
protected void getReadPermission(int requestCode)
{
    cordova.requestPermission(this, requestCode, READ);
}
```

This will call the activity and cause a prompt to appear asking for the permission. Once the user has the permission, the result must be handled with the `onRequestPermissionResult` method, which every plugin should override. An example of this can be found below:

```java
public void onRequestPermissionResult(int requestCode, String[] permissions,
                                         int[] grantResults) throws JSONException
{
    for(int r:grantResults)
    {
        if(r == PackageManager.PERMISSION_DENIED)
        {
            this.callbackContext.sendPluginResult(new PluginResult(PluginResult.Status.ERROR, PERMISSION_DENIED_ERROR));
            return;
        }
    }
    switch(requestCode)
    {
        case SEARCH_REQ_CODE:
            search(executeArgs);
            break;
        case SAVE_REQ_CODE:
            save(executeArgs);
            break;
        case REMOVE_REQ_CODE:
            remove(executeArgs);
            break;
    }
}
```

The switch statement above would return from the prompt and depending on the requestCode that was passed in, it would call the method. It should be noted that permission prompts may stack if the execution is not handled correctly, and that this should be avoided.

In addition to asking for permission for a single permission, it is also possible to request permissions for an entire group by defining the permissions array, as what is done with the Geolocation plugin:

```java
String [] permissions = { Manifest.permission.ACCESS_COARSE_LOCATION, Manifest.permission.ACCESS_FINE_LOCATION };

```

Then when requesting the permission, all that needs to be done is the following:

```java
cordova.requestPermissions(this, 0, permissions);

```

This requests the permissions specified in the array. It's a good idea to provide a publicly accessible permissions array since this can be used by plugins that use your plugin as a dependency, although this is not required.

## Debugging Android Plugins[**](http://cordova.axuer.com/docs/zh-cn/latest/guide/platforms/android/plugin.html#debugging-android-plugins)

Android debugging can be done with either Eclipse or Android Studio, although Android studio is recommended. Since Cordova-Android is currently used as a library project, and plugins are supported as source code, it is possible to debug the Java code inside a Cordova application just like a native Android application.

## Launching Other Activities[**](http://cordova.axuer.com/docs/zh-cn/latest/guide/platforms/android/plugin.html#launching-other-activities)

There are special considerations to be made if your plugin launches an Activity that pushes the Cordova [Activity](http://developer.android.com/reference/android/app/Activity.html) to the background. The Android OS will destroy Activities in the background if the device is running low on memory. In that case, the `CordovaPlugin` instance will be destroyed as well. If your plugin is waiting on a result from the [Activity](http://developer.android.com/reference/android/app/Activity.html) it launched, a new instance of your plugin will be created when the Cordova [Activity](http://developer.android.com/reference/android/app/Activity.html) is brought back to the foreground and the result is obtained. However, state for the plugin will not be automatically saved or restored and the `CallbackContext` for the plugin will be lost. There are two methods that your `CordovaPlugin` may implement to handle this situation:

```
/**
 * Called when the Activity is being destroyed (e.g. if a plugin calls out to an
 * external Activity and the OS kills the CordovaActivity in the background).
 * The plugin should save its state in this method only if it is awaiting the
 * result of an external Activity and needs to preserve some information so as
 * to handle that result; onRestoreStateForActivityResult() will only be called
 * if the plugin is the recipient of an Activity result
 *
 * @return  Bundle containing the state of the plugin or null if state does not
 *          need to be saved
 */
public Bundle onSaveInstanceState() {}

/**
 * Called when a plugin is the recipient of an Activity result after the
 * CordovaActivity has been destroyed. The Bundle will be the same as the one
 * the plugin returned in onSaveInstanceState()
 *
 * @param state             Bundle containing the state of the plugin
 * @param callbackContext   Replacement Context to return the plugin result to
 */
public void onRestoreStateForActivityResult(Bundle state, CallbackContext callbackContext) {}

```

It is important to note that the above methods should only be used if your plugin launches an [Activity](http://developer.android.com/reference/android/app/Activity.html) for a result and should only restore the state necessary to handle that Activity result. The state of the plugin will *NOT* be restored except in the case where an Activity result is obtained that your plugin requested using the `CordovaInterface`'s `startActivityForResult()` method and the Cordova Activity was destroyed by the OS while in the background.

As part of `onRestoreStateForActivityResult()`, your plugin will be passed a replacement CallbackContext. It is important to realize that this CallbackContext *IS NOT* the same one that was destroyed with the Activity. The original callback is lost, and will not be fired in the javascript application. Instead, this replacement `CallbackContext` will return the result as part of the [`resume`](http://cordova.axuer.com/docs/zh-cn/latest/cordova/events/events.html#resume) event that is fired when the application resumes. The payload of the [`resume`](http://cordova.axuer.com/docs/zh-cn/latest/cordova/events/events.html#resume) event follows this structure:

```
{
    action: "resume",
    pendingResult: {
        pluginServiceName: string,
        pluginStatus: string,
        result: any
    }
}

```

- `pluginServiceName` will match the [name element](http://cordova.axuer.com/docs/zh-cn/latest/plugin_ref/spec.html#name) from your plugin.xml.
- `pluginStatus` will be a String describing the status of the PluginResult passed to the CallbackContext. See PluginResult.java for the String values that correspond to plugin statuses
- `result` will be whatever result the plugin passes to the CallbackContext (e.g. a String, a number, a JSON object, etc.)

This [`resume`](http://cordova.axuer.com/docs/zh-cn/latest/cordova/events/events.html#resume) payload will be passed to any callbacks that the javascript application has registered for the [`resume`](http://cordova.axuer.com/docs/zh-cn/latest/cordova/events/events.html#resume) event. This means that the result is going *directly* to the Cordova application; your plugin will not have a chance to process the result with javascript before the application receives it. Consequently, you should strive to make the result returned by the native code as complete as possible and not rely on any javascript callbacks when launching activities.

Be sure to communicate how the Cordova application should interpret the result they receive in the [`resume`](http://cordova.axuer.com/docs/zh-cn/latest/cordova/events/events.html#resume) event. It is up to the Cordova application to maintain their own state and remember what requests they made and what arguments they provided if necessary. However, you should still clearly communicate the meaning of `pluginStatus` values and what sort of data is being returned in the [`resume`](http://cordova.axuer.com/docs/zh-cn/latest/cordova/events/events.html#resume) field as part of your plugin's API.

The complete sequence of events for launching an Activity is as follows

1. The Cordova application makes a call to your plugin
2. Your plugin launches an Activity for a result
3. The Android OS destroys the Cordova Activity and your plugin instance
   - *onSaveInstanceState() is called*
4. The user interacts with your Activity and the Activity finishes
5. The Cordova Activity is recreated and the Activity result is received
   - *onRestoreStateForActivityResult() is called*
6. `onActivityResult()` is called and your plugin passes a result to the new CallbackContext
7. The [`resume`](http://cordova.axuer.com/docs/zh-cn/latest/cordova/events/events.html#resume) event is fired and received by the Cordova application

Android provides a developer setting for debugging Activity destruction on low memory. Enable the "Don't keep activities" setting in the Developer Options menu on your device or emulator to simulate low memory scenarios. If your plugin launches external activities, you should always do some testing with this setting enabled to ensure that you are properly handling low memory scenarios.



##### Deviceready事件

Cordova框架中第一个应该掌握的就是这个deviceready事件。采用Cordova开发的应用在运行的时候，Cordova提供的通过HTML5调用Native功能并不是立即就能使用的，Cordova框架在读入HTML5代码之后，要进行HTML5和Native建立桥接，在未能完成这个桥接的初始的情况下，是不能调用Native功能的。在Cordova框架中，当这个桥接的初始化完成后，会调用他自身特有的事件，即deviceready事件。

所以首先应该在HTML中注册deviceready的事件监听，在它的CallBack函数中再去使用Cordova的功能。

JS代码

`document.addEventListener('deviceready', function ()                                           {console.log('Device is Ready!');  ` 

 ` // ....your code  }, ` 

 `false); `

`document.addEventListener('deviceready', this.onDeviceReady.bind(this), false);`

需要注意的是，deviceready事件是在每回读入HTML的时候都会被调用，而不只是应用启动时调用。 

其实调用的顺序是：DOMContentLoaded -> load -> deviceready 
deviceready事件一定是在load事件之后，所以load事件的执行速度会影响到deviceready事件的调用。把一些不必要的资源可以在deviceready事件之后调用从而提高执行速度。 