# Core
[Overview](https://github.com/Microsoft/vscode-docs/blob/master/docs/extensions/overview.md)

- Render process <=> Debug Adaptor(app) <=> real debugger
- Extenstion Host <=> Language Server(other process)

All VS Code extensions share a common model of contribution (registration), activation (loading) and access to the VS Code extensibility API. There are however two special flavors of VS Code extensions, language servers and debuggers, which have their own additional protocols and are covered in their own sections of the documentation.

In order to avoid problems with local firewalls, VS Code communicates with the adapter through stdin/stdout instead of using a more sophisticated mechanism (e.g. sockets).

- When create LSP(language server protol), also create a client and a server.
- When create a debugger, it can be extension and invoke debugger adaptor.

[Language Server Protol](https://code.visualstudio.com/blogs/2016/06/27/common-language-protocol)

[Example](https://github.com/Microsoft/vscode-extension-samples/):

[Doc](https://github.com/Microsoft/vscode-docs/tree/master/docs)

# Others
## file
.vscode/{launch,settings,tasks}.json
extension: package.json, extension.ts // the source of the extension entry point

``` launch.json
{
	"version": "0.2.0",
	"configurations": [
		
		{
			"type": "extensionHost",
			"request": "launch",
			"name": "Extension", // debug extension
			"preLaunchTask": "npm",
			"runtimeExecutable": "${execPath}",
			"args": [
				"--extensionDevelopmentPath=${workspaceFolder}"
			],
			"outFiles": [ "${workspaceFolder}/out/**/*.js" ]
		},
		{
			"type": "node",
			"request": "launch",
			"name": "Server", // debug debug adaptor
			"cwd": "${workspaceFolder}",
			"program": "${workspaceFolder}/src/debugAdapter.ts",
			"args": [ "--server=4711" ],
			"outFiles": [ "${workspaceFolder}/out/**/*.js" ]
		},
		{
			"type": "node",
			"request": "launch",
			"name": "Tests",
			"cwd": "${workspaceFolder}",
			"program": "${workspaceFolder}/node_modules/mocha/bin/_mocha",
			"args": [
				"-u", "tdd",
				"--timeout", "999999",
				"--colors",
				"./out/tests/"
			],
			"outFiles": [ "${workspaceFolder}/out/**/*.js" ],
			"internalConsoleOptions": "openOnSessionStart"
		},
		{
			"type": "mock",
			"request": "launch",
			"name": "Mock Sample",
			"program": "${workspaceFolder}/${command:AskForProgramName}",
			"stopOnEntry": true
		}
	],
	"compounds": [
		{
			"name": "Extension + Server",
			"configurations": [ "Extension", "Server" ]
		}
	]
}


// package.json
"debuggers": [{
    "type": "gdb",
    "windows": {
        "program": "./bin/gdbDebug.exe",
    },
    "osx": {
        "program": "./bin/gdbDebug.sh",
    },
    "linux": {
        "program": "./bin/gdbDebug.sh",
    }
}]
```
## Extension API
VS Code does not expose DOM/Web to extension
Define Extension Manifest
Protocol based extensions
Pattern: 
promise，
cancellation token，
Disposables: For instance, the setStatusBarMessage(value: string) function returns a Disposable which upon calling dispose removes the message again.
Events: event as function, regsiter and get Disposable, onWill/DidVerbNoun

Contribution Points：
Extension Manifest File 
Activation Events 