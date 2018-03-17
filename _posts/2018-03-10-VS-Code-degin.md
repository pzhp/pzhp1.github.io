# Core Concept
Main process
Render process <=> Debug Adaptor(app) <=> real debugger
Extenstion Host <=> Language Server(other process)
ref https://github.com/Microsoft/vscode-docs/blob/master/docs/extensions/overview.md

![Extension](../images/extensibility-architecture.png)

In order to avoid problems with local firewalls, VS Code communicates with the adapter through stdin/stdout instead of using a more sophisticated mechanism (e.g. sockets).

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

## Debug Adaptor

## language server
client is in ExtensionHost and is also a extension.