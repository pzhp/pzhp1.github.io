# Extension
VS Code does not expose DOM/Web to extension
Define Extension Manifest
Protocol based extensions
Pattern: 
promise，
cancellation token，
Disposables: For instance, the setStatusBarMessage(value: string) function returns a Disposable which upon calling dispose removes the message again.
Events: event as function, regsiter and get Disposable, onWill/DidVerbNoun