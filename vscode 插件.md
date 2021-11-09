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
	"scripts": {
		"vscode:prepublish": "npm run package",
		"compile": "webpack",
		"watch": "webpack --watch",
		"package": "webpack --mode production --devtool hidden-source-map",
		"compile-tests": "tsc -p . --outDir out",
		"watch-tests": "tsc -p . -w --outDir out",
		"pretest": "npm run compile-tests && npm run compile && npm run lint",
		"lint": "eslint src --ext ts",
		"test": "node ./out/test/runTest.js"
	},
	"devDependencies": {
		"@types/vscode": "^1.54.0",
		"@types/glob": "^7.1.4",
		"@types/mocha": "^9.0.0",
		"@types/node": "14.x",
		"@typescript-eslint/eslint-plugin": "^5.1.0",
		"@typescript-eslint/parser": "^5.1.0",
		"eslint": "^8.1.0",
		"glob": "^7.1.7",
		"mocha": "^9.1.3",
		"typescript": "^4.4.4",
		"ts-loader": "^9.2.5",
		"webpack": "^5.52.1",
		"webpack-cli": "^4.8.0",
		"@vscode/test-electron": "^1.6.2"
	}
}

```
