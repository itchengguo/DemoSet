**cordova.js中个模块的说明**

平台相关的：

- ​
 ```
  - src/android/android/nativeapiprovider.js JS->Native的具体交互形式
  - src/android/android/promptbasednativeapi.js 通过prompt()和Native交互（[Android2.3 simulator的Bug](https://code.google.com/p/android/issues/detail?id=12987)）
  - src/android/exec.js ****执行JS->Native交互
  - src/android/platform.js ***bootstrap处理
  - src/android/plugin/android/app.js 清缓存、loadUrl、退出程序等
 ```

通用的： 

- ```
  - src/common/argscheck.js 用于plugin中校验参数，比如argscheck.checkArgs('fFO', 'Camera.getPicture', arguments); 参数应该是2个函数1个对象
  - src/common/base64.js JS->Native交互时对ArrayBuffer进行uint8ToBase64（WebSockets二进制流）
  - src/common/builder.js 对象属性操作，比如把一个对象的属性Merge到另外一个对象
  - src/common/channel.js ****控制事件调用
  - src/common/exec/proxy.js 用于Plugin中往已经有的模块上添加方法
  - src/common/init.js ****初期处理
  - src/common/modulemapper.js ***把定义的模块clobber到一个对象，在初期化的时候会赋给window
  - src/common/pluginloader.js ***加载所有cordova_plugins.js中定义的模块，执行完成后会触发onPluginsReady
  - src/common/urlutil.js 获取绝对URL，InAppBrowser中会用到
  - src/common/utils.js 工具类
  ```

核心： 

- ```
  - src/cordova.js ****事件的处理和回调，外部访问cordova.js的入口
  - src/scripts/require.js  *****模块化系统
  - src/scripts/bootstrap.js 启动处理（只调用了初期处理require('cordova/init');），注意和platform的bootstrap处理不一样
  ```

  ​

**核心代码**

1.`cordova.js`模块系统`require/define`

- `src/scripts/require.js `自定义的模块系统

2.`cordova.js`事件通道`pub/sub`

- `src/common/channel.js` 发布/订阅模式的事件通道

3.`cordova.js`导入,初始化,启动,加载插件

- `src/cordova.js`  事件的处理和回调，外部访问`cordova.js ` 的入口
- `src/common/init.js`  初始化处理
- `src/android/platform.js`  平台启动处理
- `src/common/pluginloader.js ` 加载所有`cordova_plugins.js` 中定义的模块，执行完成后会触发`onPluginsReady` 

4.`cordova.js`本地交互`JS<->Native`

- `src/android/android/nativeapiprovider.js `JS->Native的具体交互形式
- `src/android/android/promptbasednativeapi.js` 通过`prompt()`和Native交互`（Android2.3 simulator的Bug）`
- `src/android/exec.js` 执行JS->Native交互