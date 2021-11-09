[vscode插件-初学者](https://blog.csdn.net/weixin_42278979/category_9032760.html)

## vscode插件 WebView

### extenstion.ts

```typescript
import * as vscode from 'vscode';
import * as path from 'path';
const fs = require('fs')

export function activate(context: vscode.ExtensionContext) {
	let disposable = vscode.commands.registerCommand('plugin-demo1.helloWorld', () => {
		vscode.window.showInformationMessage('Hello World from plugin-demo1!');
	});

	/**
 * 获取某个扩展文件相对于webview需要的一种特殊路径格式
 * 形如：vscode-resource:/Users/toonces/projects/vscode-cat-coding/media/cat.gif
 * @param context 上下文
 * @param relativePath 扩展中某个文件相对于根目录的路径，如 images/test.jpg
 */
	function getExtensionFileVscodeResource(context:vscode.ExtensionContext, relativePath:any) {
		const diskPath = vscode.Uri.file(path.join(context.extensionPath, relativePath));
		return diskPath.with({ scheme: 'vscode-resource' }).toString();
	}

	/**
	 * 从某个HTML文件读取能被Webview加载的HTML内容
	 * @param {*} context 上下文
	 * @param {*} templatePath 相对于插件根目录的html文件相对路径
	 */
	function getWebViewContent(context:vscode.ExtensionContext, templatePath:string) {
		const resourcePath = path.join(context.extensionPath, templatePath);
		const dirPath = path.dirname(resourcePath);
		let html = fs.readFileSync(resourcePath, 'utf-8');
		// vscode不支持直接加载本地资源，需要替换成其专有路径格式，这里只是简单的将样式和JS的路径替换
		html = html.replace(/(<link.+?href="|<script.+?src="|<img.+?src=")(.+?)"/g, (m:any, $1:any, $2:any) => {
			return $1 + vscode.Uri.file(path.resolve(dirPath, $2)).with({ scheme: 'vscode-resource' }).toString() + '"';
		});
		return html;
	}


	let webviews = vscode.commands.registerCommand('extension.demo1.openWebview',function(uri){
		
		// 创建webView
		const panel = vscode.window.createWebviewPanel(
			'testWebview',  // viewType
			'WebView',   // 视图标题
			vscode.ViewColumn.One,  //显示在编辑器的哪个部位
			{
				enableScripts:true,  //启用js，默认禁用
				retainContextWhenHidden:true // webview被隐藏时保持状态，避免被重置
			}
		)

		const imgUrl = getExtensionFileVscodeResource(context,'src/media/pic.jpg')
		// panel.webview.html = `<html>
		// 	<div>你好，我是WebView</div>
		// 	<img src="${imgUrl}" />
		// 	</body></html>
			
		// `
		panel.webview.html = getWebViewContent(context, 'src/media/demo.html');
	})

	context.subscriptions.push(disposable);
	context.subscriptions.push(webviews);
}

export function deactivate() {}

```

### package.json

```json
{
  "name": "plugin-demo1",
	"displayName": "plugin-demo1",
	"description": "",
	"version": "0.0.1",
	"engines": {
		"vscode": "^1.54.0"
	},
	"categories": [
		"Other"
	],
	"activationEvents": [
        "onCommand:plugin-demo1.helloWorld",
		"onCommand:extension.demo1.openWebview"
	],
	"main": "./dist/extension.js",
	"contributes": {
		"commands": [
			{
				"command": "plugin-demo1.helloWorld",
				"title": "Hello World"
			},
			{
                "command": "extension.demo1.openWebview",
                "title": "打开WebView"
            }
		]
	},
}

```


## vscode插件 TreeView

### treeViewProvider.ts

```ttypescript
import { TreeItem,TreeItemCollapsibleState,TreeDataProvider,Uri,window, Event, ProviderResult } from "vscode";
import {join} from 'path'

// 创建一个Map集合
const ITEM_MAP = new Map<string,string>([
    ['pic','pic.svg'],
    ['pic1','pic1.svg'],
    ['pic2','pic2.svg'],
    ['pic3','pic3.svg'],
])

export class TreeItemNode extends TreeItem{
    constructor(public readonly label:string,public readonly collapsibleState:TreeItemCollapsibleState){
        super(label,collapsibleState)
    }

    // 为每项添加点击事件的命令
    command = {
        title:this.label,  //标题
        command:'itemClick',  //命令ID
        tooltip:this.label,   //鼠标覆盖时的小小提示框
        arguments:[    //向registerCommand传递参数
            this.label
        ]
    }

    static getIconUriForLabel(label:string):Uri{
        return Uri.file(join(__filename,'..','..','src','media',ITEM_MAP.get(label)+''))
    }

    iconPath = TreeItemNode.getIconUriForLabel(this.label)
}

export class TreeViewProvider implements TreeDataProvider<TreeItemNode>{
    onDidChangeTreeData?: import ('vscode').Event<TreeItemNode | null | undefined> | undefined;
    getTreeItem(element: TreeItemNode): TreeItem | Thenable<TreeItem> {
        return element
    }
    getChildren(element?: TreeItemNode): ProviderResult<TreeItemNode[]> {
       return ['pic','pic1','pic2','pic3'].map(item=>{
           return new TreeItemNode(
               item as string,
               TreeItemCollapsibleState.None as TreeItemCollapsibleState
           )
       })
    }


    public static initTreeViewItem(){
        // 实例化 TreeViewProvider
        const treeViewProvider = new TreeViewProvider()

        window.registerTreeDataProvider('treeView-item',treeViewProvider)
    } 
    
}
```

### webview.ts
```typescript
import {ExtensionContext,ViewColumn,WebviewPanel,window,commands,Uri} from 'vscode'
import * as path from 'path'
const fs = require('fs')

// 创建一个全局变量
let webviewPanel:WebviewPanel | undefined


// 创建一个可到出的方法，并且带上参数
export function createWebView(context:ExtensionContext,viewColumn:ViewColumn,label:string){
    if(webviewPanel === undefined){
        webviewPanel = window.createWebviewPanel(
            'webView',   //标识webview面板类型
            label,   //小组的标题
            viewColumn,  //在编辑器这显示webview的位置
            {
                retainContextWhenHidden:true,
                enableScripts:true
            }
        )
        webviewPanel.webview.postMessage({label:label})
        // webviewPanel.webview.html = getWebViewContent(context,`src/media/${label}.html`)
        webviewPanel.webview.html = getIframehtml(label)

    }else{
        webviewPanel.title = label
        // webview面板一次只能显示在一列中。如果它已经显示，则此方法将其移到新列
        webviewPanel.reveal() 
        // 向html传递一个标签为label的messgae
        webviewPanel.webview.postMessage({label:label})
        // webviewPanel.webview.html = getWebViewContent(context,`src/media/${label}.html`)
    }


    // 如果关闭该面板，将webviewPanel设置为undefined
    webviewPanel.onDidDispose(()=>{
        webviewPanel = undefined
    })

    return webviewPanel
}

function getWebViewContent(context:ExtensionContext, templatePath:string) {
    const resourcePath = path.join(context.extensionPath, templatePath);
    const dirPath = path.dirname(resourcePath);
    let html = fs.readFileSync(resourcePath, 'utf-8');
    // vscode不支持直接加载本地资源，需要替换成其专有路径格式，这里只是简单的将样式和JS的路径替换
    html = html.replace(/(<link.+?href="|<script.+?src="|<img.+?src=")(.+?)"/g, (m:any, $1:any, $2:any) => {
        return $1 + Uri.file(path.resolve(dirPath, $2)).with({ scheme: 'vscode-resource' }).toString() + '"';
    });
    return html;
}


export function getIframehtml(label:string){
    return `
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>百度</title>
        <style>
            html,body{
                margin: 0;
                padding: 0;
                width: 100%;
                height: 100%;
            }

            .iframe{
                width: 100%;
                height: 100%;
            }
        </style>
    </head>
    <body>
        <span id="btn">点击</span>
        <iframe class="iframe" src="http://www.baidu.com/" frameborder="0"></iframe>
        <script>
            window.addEventListener('message',e=>{
            console.log(e,'e')
            })

            document.getElementById('#btn').addEventListener(function(){
                window.parent.postMessage({ifarmeLabel:'pig1'},'*')
            })
        </script>
    </body>
    </html>
    `
}
```

### extension.ts
```typescript
import * as vscode from 'vscode';
import {TreeViewProvider} from './treeViewProvider'
import {createWebView} from './webView'
import {DecorationNumber} from './decorationNumber'

export function activate(context: vscode.ExtensionContext) {

	let decorationNumber = new DecorationNumber()
	
	let disposable = vscode.commands.registerCommand('plugin-treeview.helloWorld', () => {
		vscode.window.showInformationMessage('Hello World from plugin-treeView!');
	});

	TreeViewProvider.initTreeViewItem()

	context.subscriptions.push(disposable);
	context.subscriptions.push(vscode.commands.registerCommand('itemClick',(label)=>{
		// vscode.window.showInformationMessage(label)

		const webview = createWebView(context,vscode.ViewColumn.Active,label)
		context.subscriptions.push(webview)
	}));

	context.subscriptions.push(vscode.commands.registerCommand('extension.decorationNumber',()=>{
		decorationNumber.DecNumber()
	}));

}

export function deactivate() {}
```

### package.json
```json
{
  "name": "plugin-treeview",
	"displayName": "plugin-treeView",
	"description": "",
	"version": "0.0.1",
	"engines": {
		"vscode": "^1.54.0"
	},
	"categories": [
		"Other"
	],
	"activationEvents": [
        "onCommand:plugin-treeview.helloWorld",
		"onView:treeView-item",
		"onCommand:itemClick",
		"onCommand:extension.decorationNumber"
	],
	"main": "./dist/extension.js",
	"contributes": {
		"commands": [
			{
				"command": "plugin-treeview.helloWorld",
				"title": "Hello World"
			},
			{
				"command":"extension.decorationNumber",
				"title":"decNumber"
			}
		],
		"viewsContainers":{
			"activitybar":[
				{
					"id":"treeView",
					"title":"treeView",
					"icon":"src/media/pic.svg"
				}
			]
		},
		"views":{
			"treeView":[
				{
					"id":"treeView-item",
					"name":"item",
					"when":""
				}
			]
		}
	},


```
