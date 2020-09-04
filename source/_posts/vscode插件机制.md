---
title: vscode插件机制
tags: 前端
categories: 前端
abbrlink: 20807
date: 2019-11-28 08:48:23
---
## 前言

Visual Studio Code（VS Code）近年来获得了爆炸式增长，成为广大开发者工具库中的必备神器。它作为一个开源项目，也吸引了无数第三方开发者和终端用户，成为顶尖开源项目之一。它在功能上做到了够用，体验上做到了好用，更在拥有海量插件的情况下做到了简洁流畅，实属难能可贵。通过插件来扩展功能的做法已经是司空见惯了，**但如何保证插件和原生功能一样优秀呢？历史告诉我们：不能保证。**

大家可以参考Eclipse，插件模型可以说是做得非常彻底了，功能层面也是无所不能，但存在几个烦人的问题：不稳定、难用、慢，所以不少用户转投IntelliJ的怀抱。可谓成也插件，败也插件。

**问题的本质在于信息不对称，它导致不同团队写出来的代码，无论是思路还是质量，都不一致**。最终，用户得到了一个又乱又卡的产品。所以要让插件在稳定性、速度和体验的层面都做到和原生功能统一，只能是一个美好的愿望。

来看看其他IDE是怎么做的，**Visual Studio自己搞定所有功能，并且做到优秀，让别人无事可做**，这也成就了其“宇宙第一IDE”的美名；IntelliJ与之相仿，开箱即用，插件可有可无。这么看起来，自己搞定所有的事情是个好办法，但大家是否知道，Visual Studio背后有上千人的工程团队，显然，这不是VS Code这二十几号人能搞定的。他们选择了让大家来做插件，那怎么解决Eclipse所遇到的问题呢？

这里分享一个小知识——Eclipse核心部分的开发者就是早期的VS Code团队。嗯，所以他们没有两次踏入同一条河流。**与Eclipse不同，VS Code选择了把插件关进盒子里**。这样做首先解决的问题就是**稳定性**，这个问题对于VS Code来说尤为重要。都知道VS Code基于Electron，实质上是个Node.js环境，单线程，任何代码崩了都是灾难性后果。所以VS Code干脆不信任任何人，把插件们放到单独的进程里，任你折腾，主程序妥妥的。
<!-- more -->
## 进程模型

VSCode中包含主进程，渲染进程，同时因为VSCode提供了插件的扩展能力，又出于安全稳定性的考虑，图中又多了一个Extension Host，其实这个Extension Host也是一个独立的进程，用于运行我们的插件代码。并且同渲染进程一样，彼此都是独立互不影响的。Extension Host Process暴露了一些VSCode的API供插件开发者去使用。

![](http://alan-picpack.oss-cn-hangzhou.aliyuncs.com/1574653513904)



至此，我们了解到VS Code里至少有3个进程：

- Electron Main Process：App主进程
- Electron Renderer Process：UI进程
- Extension Host Process：插件宿主进程，给插件提供执行环境

其中Extension Host Process（每个VS Code窗体）只存在一个，所有插件都在该进程执行，*而不是每个插件一个独立进程*

注意，插件宿主进程是个*普通的Node进程*（`childProcess.fork()`出来的），并不是Electron进程，而且*被限制了不能使用electron。*

### 进程间通信方式

Extension Host与Main之间的通信是通过`fork()`内置的IPC来完成的，具体如下：

```javascript
// Support logging from extension host
this._extensionHostProcess.on('message', msg => {
 if (msg && (<IRemoteConsoleLog>msg).type === '__$console') {
   this._logExtensionHostMessage(<IRemoteConsoleLog>msg);
 }
});
```

这里只是单向通信（`插件 -> Main`），实际上可以通过`this._extensionHostProcess.send({msg})`完成另一半（`Main -> 插件`）。

P.S.关于进程间通信的更多信息，请查看Nodejs进程间通信。

## 扩展能力

VS Code插件**不适合做UI定制**，vscode为插件提供了丰富的扩展能力，但*不允许插件直接访问底层UI DOM*（也就是说插件难以改变IDE外观，UI定制受限），UI DOM这一层可能会随着优化频繁变动，VS Code不希望这些优化项受限于插件依赖，所以干脆把UI定制能力限制起来，除UI定制之外的，IDE相关的功能型特性都是支持扩展的，如基础的语法高亮/API提示、引用跳转（转到定义）/文件搜索、主题定制，高级的debug协议等等。

P.S.实际上，非要扩展UI，也是有办法的（逃出插件运行环境，但要费不少力气），具体见[access electron API from vscode extension](https://github.com/Microsoft/vscode/issues/3011)。

## 运行环境

为了性能与兼容性，*插件在独立的进程（称为extension host process）中运行*，并且不允许直接访问DOM，所以提供了一套内置的UI组件，比如智能提示（IntelliSense）

所以插件崩溃或无响应不影响IDE正常运行，例如：

```javascript
export function activate(context: vscode.ExtensionContext) {
  // hang up
  while (true);
}
```

一个插件的死循环并不影响IDE的正常使用和其它插件的加载/激活，但在进程列表能够看到Code Helper的CPU占用接近100%，*进程级沙箱*保证了插件机制的稳定性。

## 核心理念

### 稳定性：插件隔离

插件可能会影响启动性能和IDE自身的稳定性，所以通过进程隔离来解决这个问题，插件运行在独立的进程中，不影响IDE及其启动时间。这样做是从用户角度考虑的，希望*用户对IDE拥有完全的控制力*，无论插件在做什么，都不影响IDE基本功能的正常使用。

P.S.extension host process是个特殊的Node进程，能够访问VS Code扩展API，VS Code也对这种进程提供了debug支持。

### 性能：插件激活

**插件都是懒加载的**（as late as possible），只在特定场景才加载/激活，所有在此之前也不耗费内存等资源。实现上是插件注册特定激活事件（activation events），由IDE来触发执行，比如markdown插件只在用户代开md文件时才需要激活。

#### 激活事件(Activation Events)

vscode插件拥有6种激活方式：

* onLanguage:${language} 打开特定语言的文档
* onCommand:${command} 通过Command Palette执行特定命令
* onDebug 进入调试模式
* workspaceContains:${toplevelfilename} 打开的文件夹里含有特定文件
* onView:${viewId} 展开指定view
* \* 打开IDE就激活

除`"activationEvents": ["*"]`外都是条件激活，只在特定场景或满足特定条件时才加载/激活插件。

##### activationEvents.onLanguage

- 当某种语言的文件被打开时，该扩展就会被激活。
- 该扩展使用语言标记符（a language identifier value）。
- 可以在`activationEvents`数组中声明多种语言。

```
"activationEvents": [
    "onLanguage:json",
    "onLanguage:markdown",
    "onLanguage:typescript"
]
```

##### activationEvents.onCommand

调用命令就会激活该扩展。

```
"activationEvents": [
    "onCommand:extension.sayHello"
]
```

##### activationEvents.onDebug

启动调试之前会激活该事件。

还有两个更细粒度的激活事件：
- onDebugInitialConfigurations:在调用DebugConfigurationProvider的provideDebugConfigurations方法之前会被调用。
- onDebugResolve:type:在调用DebugConfigurationProvider的resolveDebugConfiguration方法之前会被调用。

经验法则：如果该调试扩展是轻量级的，就使用onDebug。 如果是重量级的，就使用onDebugInitialConfigurations或者onDebugResolve，使用哪一个取决于DebugConfigurationProvider是否实现了provideDebugConfigurations或者resolveDebugConfiguration方法。

##### activationEvents.workspaceContains

每当打开文件夹，并且该文件夹至少匹配一个与glob pattern有关的文件。

```
"activationEvents": [
    "workspaceContains:**/.editorconfig"
]
```

##### activationEvents.onFileSystem

读取到特定格式(specific scheme)的文件或者文件夹，就会激活该扩展，特定格式通常是指文件格式(file-scheme)。

```
"activationEvents": [
    "onFileSystem:sftp"
]
```

##### activationEvents.onView

打开指定ID的视图，该扩展就会被激活。

```
"activationEvents": [
    "onView:nodeDependencies"
]
```

##### activationEvents.*

- 只要vscode启动，该扩展就会被激活。
- 为了更好地用户体验，仅当使用其他激活方式组合无法实现功能时使用该激活事件。
- 扩展可以监听多个激活事件，这比使用`*`要好。

#### 插件清单文件(Extension Manifest File)

清单文件用来描述插件的meta信息，直接把`package.json`作为清单文件，并增加了一些特有字段，比如触发插件加载的激活事件（`activation events`）、插件想要增强的扩展点（`contribution points`）。

IDE在启动过程中扫一遍插件清单文件，UI相关的就扩展UI，UI无关的就把扩展点与插件功能关联起来。

另外，由于插件的执行环境是Node进程，所以npm package都是可用的，依赖模块同样声明在`package.json`里。*注意*，用户安装插件时*不会*自动`npm install`，所以需要在发布插件前把依赖模块打包进去，具体见[Installation and Packaging](https://code.visualstudio.com/docs/extensionAPI/patterns-and-principles#_installation-and-packaging)。

| 名称                  | 必须 | 类型                                 | 描述                                                         |
| --------------------- | ---- | ------------------------------------ | ------------------------------------------------------------ |
| name                  | Y    | string                               | 扩展的名称，全小写，没有空格                                 |
| version               | Y    | string                               | 兼容版本                                                     |
| publisher             | Y    | string                               | 发布名称                                                     |
| engines               | Y    | object                               | 扩展能够兼容的最小的VS Code代码版本，不能为`*`。例如^0.10.5表示扩展能够运行的最小的VS Code版本为0.10.5 |
| license               | N    | string                               | 许可                                                         |
| displayName           | N    | string                               | 市场中显示的扩展的名称                                       |
| description           | N    | string                               | 简短描述您的扩展是什么和做了什么                             |
| categories            | N    | string[]                             | 扩展所属的类别: [Programming Languages, Snippets, Linters, Themes, Debuggers, Formatters, Keymaps, SCM Providers, Other, Extension Packs, Language Packs] |
| keywords              | N    | array                                | 关键词，使扩展更容易被发现                                   |
| main                  | N    | string                               | 扩展入口                                                     |
| contributes           | N    | object                               | 扩展描述对象                                                 |
| activationEvents      | N    | array                                | 激活事件数组                                                 |
| badges                | N    | array                                | 显示在市场侧栏的徽标，分别表示url、href、description         |
| markdown              | N    | string                               | 控制扩展市场上使用的markdown渲染引擎，要么是`github`(default)要么是`standard` |
| qna                   | N    | marketplace (default), string, false | 控制市场上`Q & A`链接                                        |
| dependencies          | N    | object                               | 运行时需要的NodeJS依赖。                                     |
| devDependencies       | N    | object                               | 开发时需要的NodeJS依赖。                                     |
| extensionDependencies | N    | array                                | 此扩展所依赖的扩展的id集合。当该扩展被安装时，所依赖的扩展会一并被安装。扩展的id总是`${publisher}.${name}` |
| scripts               | N    | object                               | 与npm的脚本完全相同，但是带有额外的VS代码特定字段。          |
| icon                  | N    | string                               | 至少128x128像素的图标路径(视网膜屏幕256 x256)                |

#### 扩展点（Contribution Points）

即支持的扩展类型，都声明在`package.json/contributes`下，包括：

```
configuration 插件配置项，用户可以通过Settings设置
configurationDefaults 插件配置项默认值
commands  添加命令，用户可以通过Command Palette输入特定命令激活插件功能
menus     添加与命令关联的菜单项，用户点击菜单项时执行对应命令
keybindings 添加与命令关联的快捷键，用户按下特定快捷键时执行对应命令
languages 与文件类型建立关联或扩展新语言，用户打开（满足某些要求的）特定文件类型时执行对应命令
debuggers 添加debugger，通过VS Code debug协议与IDE通信
breakpoints 配合debuggers，声明对debugger支持的（编程）语言类型
grammars 新增TextMate语法描述，语法高亮
themes 添加定制主题
snippets 添加代码片段
jsonValidation 添加json格式校验
views 新增左侧文件查看器视图和调试视图分栏
problemMatchers 添加错误匹配，从lint结果解析出error，warning等
problemPatterns 配合problemMatchers，定义匹配模式
```

`menus`是*唯一的UI扩展官方途径*，支持扩展的菜单具体如下：

```
Command Palette搜索框下方菜单 commandPalette
文件查看器右键菜单 explorer/context
编辑器
  右键菜单 editor/context
  标题栏菜单 editor/title
  标题栏右键菜单 editor/title/context
调试视图
  调用栈右键菜单 debug/callstack/context
SCM（源码管理）视图
  标题栏菜单 scm/title
  文件分组菜单 scm/resourceGroup/context
  文件状态菜单 scm/resource/context
  文件变动菜单 scm/change/title
左侧视图
  文件查看器分栏 view/title
  调试视图分栏 view/item/context
```

#### 扩展API

##### 设计原则

- 基于Promise：异步操作都用Promise来描述
- 取消token：传入`CancellationToken`作为额外参数来检查取消状态，以及接收取消通知
- 可释放式资源管理：持有的资源都需要手动释放，例如事件监听，命令，UI交互等
- 事件API：调用订阅方法（`on[Will|Did]VerbNoun`）传入listener（接收`event`参数）返回Disposable
- 严格空检查：通过TypeScript严格区分`undefined`和`null`

##### api概览

API按命名空间组织，全局命名空间如下：

```
commands 执行/注册命令，IDE自身的和其它插件注册的命令都可以，如executeCommand
debug 调试相关API，比如startDebugging
env IDE相关的环境信息，比如machineId, sessionId
extensions 跨插件API调用，extensionDependency声明插件依赖
languages 编程语言相关API，如createDiagnosticCollection, registerDocumentFormattingEditProvider
scm 源码版本控制API，如createSourceControl
window 编辑器窗体相关API，如onDidChangeTextEditorSelection, createTerminal, showTextDocument
workspace 工作空间级API（打开了文件夹才有工作空间），如findFiles, openTextDocument, saveAll
```

比如可以通过`workspace.findFiles + languages.registerDefinitionProvider`实现Haste的*全局模块引用跳转支持*

另外，一些API以命令形式提供（即上面提到的“IDE自身的”命令），例如`vscode.previewHtml`、`vscode.openFolder`、`editorScroll`等等。

## 注入机制

在使用vscode提供的api时，需要引入`vscode`模块访问插件可用的API：

```
import * as vscode from 'vscode'
```

但是我们发现在`node_modules`下并没有`vscode`模块，而且`vscode`模块也名没被`define()`过，看起来我们`require`了一个不存在的模块，那么，这个东西是哪里来的？

对`require('vscode')`的过程进行debug，很容易发现做过手脚的地方：

```
// ref: src/vs/workbench/api/node/extHost.api.impl.ts
function defineAPI(factory: IExtensionApiFactory, extensionPaths: TernarySearchTree<IExtensionDescription>): void { // each extension is meant to get its own api implementation
 const extApiImpl = new Map<string, typeof vscode>();
 let defaultApiImpl: typeof vscode; const node_module = <any>require.__$__nodeRequire('module');
 const original = node_module._load;
 node_module._load = function load(request, parent, isMain) {
   if (request !== 'vscode') {
     return original.apply(this, arguments);
   }   // get extension id from filename and api for extension
   const ext = extensionPaths.findSubstr(parent.filename);
   if (ext) {
     let apiImpl = extApiImpl.get(ext.id);
     if (!apiImpl) {
       apiImpl = factory(ext);
       extApiImpl.set(ext.id, apiImpl);
     }
     return apiImpl;
   }   // fall back to a default implementation
   if (!defaultApiImpl) {
     defaultApiImpl = factory(nullExtensionDescription);
   }
   return defaultApiImpl;
 };
}
```

`Module._load()`方法被劫持了，遇到`vscode`返回一个虚拟模块，叫做`apiImpl`。*注意*，每个插件拿到的API都是独立的（可能是出于插件安全隔离考虑，避免劫持API影响其它插件）。

## 插件运行

以vscode.window.setStatusBarMessage('Hello World')为例：

前文我们提到所有的 API 被定义在 extHost.api.impl.ts 文件的 createApiFactory 里，例如 vscode.window.setStatusBarMessage 的实现：

```tsx
const window: typeof vscode.window = {
  /* 省略部分代码 */
  setStatusBarMessage(text: string, timeoutOrThenable?: number | Thenable<any>): vscode.Disposable 		{
			return extHostStatusBar.setStatusBarMessage(text, timeoutOrThenable);
   },
  /* 省略部分代码 */
}
```

实际调用的是 extHostStatusBar.setStatusBarMessage 函数，而 extHostStatusBar 则是 ExtHostStatusBar 的实例:

```tsx
const extHostStatusBar = new ExtHostStatusBar(rpcProtocol);
```

ExtHostStatusBar 包含了两个方法 createStatusBarEntry 和 setStatusBarMessage，createStatusBarEntry 返回了一个 ExtHostStatusBarEntry ，它被包装了一层代理，在 ExtHostStatusBar 被实例化化的同时也会产生一个 ExtHostStatusBarEntry 实例

```tsx
export class ExtHostStatusBar {

  private _proxy: MainThreadStatusBarShape;
  private _statusMessage: StatusBarMessage;

  constructor(mainContext: IMainContext) {
    // 获取代理
    this._proxy = mainContext.getProxy(MainContext.MainThreadStatusBar);
    // 传入 this, StatusBarMessage 中也随即实例化了一个 ExtHostStatusBarEntry
    this._statusMessage = new StatusBarMessage(this);
  }
  /* 省略部分代码 */
}

class StatusBarMessage {

  private _item: StatusBarItem;
  private _messages: { message: string }[] = [];

  constructor(statusBar: ExtHostStatusBar) {
    // 调用 createStatusBarEntry 
    this._item = statusBar.createStatusBarEntry(void 0, ExtHostStatusBarAlignment.Left, Number.MIN_VALUE);
  }
  /* 省略部分代码 */
}
```



而在setStatusBarMessage方法中，主要是调用的this._statusMessage.setMessage(text);

```tsx
setStatusBarMessage(text: string, timeoutOrThenable?: number | Thenable<any>): Disposable {

		const d = this._statusMessage.setMessage(text);// 这一句
		let handle: any;

		if (typeof timeoutOrThenable === 'number') {
			handle = setTimeout(() => d.dispose(), timeoutOrThenable);
		} else if (typeof timeoutOrThenable !== 'undefined') {
			timeoutOrThenable.then(() => d.dispose(), () => d.dispose());
		}

		return new Disposable(() => {
			d.dispose();
			clearTimeout(handle);
		});
	}
```

而 this._statusMessage.setMessage 方法经过层层调用，最终调用了 ExtHostStatusBarEntry 实例的 update 方法，也就是前面的 StatusBarMessage 构造函数中的 this._item.update，而这里就到了重头戏，update 方法中包含了一个 延时为 0 的 setTimeout ：

```tsx
private update(): void {
		if (this._disposed || !this._visible) {
			return;
		}

		clearTimeout(this._timeoutHandle);

		// Defer the update so that multiple changes to setters dont cause a redraw each
		this._timeoutHandle = setTimeout(() => {
			this._timeoutHandle = undefined;

			// Set to status bar
			this._proxy.$setEntry(this.id, this._statusId, this._statusName, this.text, this.tooltip, this.command, this.color,
				this._alignment === ExtHostStatusBarAlignment.Left ? MainThreadStatusBarAlignment.LEFT : MainThreadStatusBarAlignment.RIGHT,
				this._priority);
		}, 0);
}
```

这里的_proxy是通过依赖注入引入的mainThreadStatusBat, 具体则来自：

```tsx
constructor(mainContext: IMainContext) {
		this._proxy = mainContext.getProxy(MainContext.MainThreadStatusBar);
		this._statusMessage = new StatusBarMessage(this);
}
```

这里的 IMainContext 其实就是继承了 IRPCProtocol 的一个别名而已，new ExtHostStatusBar 的参数是一个 rpcProtocol 实例，它被定义在 src/vs/workbench/services/extensions/node/rpcProtocol.ts 中，我们重点看一下 getProxy 的实现

```tsx
public getProxy<T>(identifier: ProxyIdentifier<T>): T {
  // 这里只是根据对应的 identifier 生成对应的 scope 而已，插件调用和 API 的调用一模一样比较方便一些
  const rpcId = identifier.nid;
  // 例如 StatusBar 的 identifier.nid 就是 'MainThreadStatusBar'
  if (!this._proxies[rpcId]) {
    // 缓存中没有代理则生成新的代理
    this._proxies[rpcId] = this._createProxy(rpcId);
  }
  // 返回代理后的对象
  return this._proxies[rpcId];
}


// 创建代理
private _createProxy<T>(rpcId: number): T {
  let handler = {
    get: (target: any, name: string) => {
      // target 即表示 scope，name 即为被调用方法名
      if (!target[name] && name.charCodeAt(0) === CharCode.DollarSign) {
        target[name] = (...myArgs: any[]) => {
          // 插件中的 API 实际被代理到 remoteCall，因为这是一个 RPC 协议
	  		return this._remoteCall(rpcId, name, myArgs);
	};
      }
      return target[name];
    }
  };
  // 返回 API 代理
  return new Proxy(Object.create(null), handler);
}
```

_createProxy 返回的是一个代理对象，即它代理了主线程中真正实现这些 API 的对象，例如 'MainThreadStatusBar' 返回的是一个 MainThreadStatusBarShape 类型的代理。

```tsx
export interface MainThreadStatusBarShape extends IDisposable {
  $setEntry(id: number, extensionId: string, text: string, tooltip: string, command: string, color: string | ThemeColor, alignment: MainThreadStatusBarAlignment, priority: number): void;
  $dispose(id: number): void;
}
```

插件 API 定义中并没有实现这个接口，它只需要被主线程中对应的模块实现即可，前面我们说到 setStatusMessage 最终调用了 this._proxy.$setEntry。

_remoteCall 里会调用 RPCProcotol 的静态方法 serializeRequest 将 rpcId 方法名以及参数序列化成一个 Buffer 并发送给主线程。

```
const msg = MessageIO.serializeRequest(req, rpcId, methodName, args, !!cancellationToken, this._uriReplacer);
// 省略部分代码
this._protocol.send(msg);
```

关于主线程中接收到消息如何处理其实已经不用多说了，根据 rpcId 找到对应的 Services 以及方法，传入参数即可。

## 最后

VS Code像一颗耀眼的星星，吸引着成千上万开发者为其添砖加瓦。从VS Code的成功中，我们看到了好的设计和工程实践能创造多少奇迹。放眼软件产业，各个层面的模式不断被刷新，让人激动之余，也要求从业者不断提高技能水平。从个人学习的角度来看，了解这些模式诞生的前因后果，理解工程实践中的决策过程是非常有利于提高工程能力的。