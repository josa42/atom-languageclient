# Atom Language Server Protocol Client
[![macOS Build Status](https://travis-ci.org/atom/atom-languageclient.svg?branch=master)](https://travis-ci.org/atom/atom-languageclient) [![Windows Build Status](https://ci.appveyor.com/api/projects/status/xibqpw9h3lya87xs/branch/master?svg=true
)](https://ci.appveyor.com/project/Atom/atom-languageclient/branch/master) [![Dependency Status](https://david-dm.org/atom/atom-languageclient.svg)](https://david-dm.org/atom/atom-languageclient)

Provide integration support for adding Language Server Protocol servers to Atom.

## Background

[Language Server Protocol (LSP)](https://github.com/Microsoft/language-server-protocol/blob/master/protocol.md) is a JSON-RPC based mechanism whereby a client (IDE) may connect to an out-of-process server that can provide rich analysis, refactoring and interactive features for a given programming language.

## Implementation

This npm package can be used by Atom package authors wanting to integrate LSP-compatible language servers with Atom. It provides:

* Conversion routines between Atom and LSP types
* A FlowTyped wrapper around JSON-RPC for v2 of the LSP protocol (v3 to follow)
* All necessary FlowTyped input and return structures for LSP, notifications etc.
* A number of adapters to translate communication between Atom and Nuclide and the LSP's capabilities
* Automatic wiring up of adapters based on the negotiated capabilities of the language server
* Helper functions for downloading additional non-npm dependencies

## Capabilities

The language server protocol consists of a number of capabilities. Some of these already have a counterpoint we can connect up to today while others do not.  The following table shows each capability in v2 and how it is exposed via Atom;

| Capability                      | Atom interface                |
|---------------------------------|-------------------------------|
| window/showMessage              | Notifications package         |
| window/showMessageRequest       | Notifications package (TODO)  |
| window/logMessage               | Ignored                       |
| telemetry/event                 | Ignored                       |
| workspace/didChangeWatchedFiles | Atom file watch api (TODO)    |
| textDocument/publishDiagnostics | Linter v2 push/indie          |
| textDocument/completion         | AutoComplete+                 |
| completionItem/resolve          | AutoComplete+      (TBD)      |
| textDocument/hover              | Nuclide data tips             |
| textDocument/signatureHelp      | TBD                           |
| textDocument/definition         | Nuclide definitions           |
| textDocument/findReferences     | Nuclide findReferences        |
| textDocument/documentHighlight  | Nuclide definitions           |
| textDocument/documentSymbol     | Nuclide outline view          |
| workspace/symbol                | TBD                           |
| textDocument/codeAction         | TBD                           |
| textDocument/codeLens           | TBD                           |
| textDocument/formatting         | Format File command           |
| textDocument/rangeFormatting    | Format Selection command      |
| textDocument/onTypeFormatting   | TBD                           |
| textDocument/rename             | TBD                           |
| textDocument/didChange          | Send on save                  |
| textDocument/didOpen            | Send on open                  |
| textDocument/didSave            | Send on save                  |
| textDocument/didClose           | Send on close                 |

## Developing packages

The underlying JSON-RPC communication is handled by the [vscode-jsonrpc npm module](https://www.npmjs.com/package/vscode-jsonrpc).

### Minimal example

A minimal implementation can be illustrated by the Omnisharp package here which has only npm-managed dependencies.  You simply the scope name, language name and server name as well as start your process and AutoLanguageClient takes care of interrogating your language server capabilities and wiring up the appropriate services within Atom to expose them.

```javascript
const cp = require('child_process')
const {AutoLanguageClient} = require('atom-languageclient')

class OmnisharpLanguageServer extends AutoLanguageClient {
  getGrammarScopes () { return [ 'source.cs' ] }
  getLanguageName () { return 'C#' }
  getServerName () { return 'OmniSharp' }

  startServerProcess () {
    return cp.spawn('node', [ require.resolve('omnisharp-client/languageserver/server') ])
  }
}

module.exports = new OmnisharpLanguageServer()
```

You can get this code packaged up with the necessary package.json etc. from the [languageserver-csharp](https://github.com/atom/languageserver-csharp) provides C# support via [Omnisharp (node-omnisharp)](https://github.com/OmniSharp/omnisharp-node-client) repo.

Note that you will also need to add various entries to the `providedServices` and `consumedServices` section of your package.json (for now).  You can [obtain these entries here](https://github.com/atom/languageserver-csharp/tree/master/package.json).

### Using other connection types

The default connection type is *stdio* however both *ipc* and *sockets* are also.  

#### IPC

To use ipc simply return *ipc* from getConnectionType(), e.g.

```javascript
class ExampleLanguageServer extends AutoLanguageClient {
  getGrammarScopes () { return [ 'source.js' ] }
  getLanguageName () { return 'JavaScript' }
  getServerName () { return 'JavaScript Language Server' }

  getConnectionType() { return 'ipc' }

  startServerProcess () {
    const startServer = require.resolve('@example/js-language-server')
    return cp.spawn('node', [startServer, '--node-ipc'], {
      stdio: [null, null, null, 'ipc']
    })
  }
}
```

#### Sockets

Sockets are a little more complex bercause you need to allocate a free socket. The [languageserver-php package](https://github.com/atom/languageserver-php/blob/master/lib/main.js) contains an example of this.

### Debugging

Atom-LanguageClient can log all sent and received messages nicely formatted to the Developer Tools Console within Atom. To do so simply enable it with `atom.config.set('core.DebugLSP', true)`, e.g.


### Tips

Some more elaborate scenarios can be found in the [languageserver-java](https://github.com/atom/languageserver-java) package which includes:

* Downloading and unpacking non-npm dependencies (in this case a .tar.gz containing JAR files)
* Per-platform start-up configuration
* Wiring up custom extensions to the protocol (language/status to Atom Status-Bar, language/actionableNotification to Atom Notifications)

### Available packages

Right now we have the following experimental Atom LSP packages in development. They are mostly usable but are missing some features that either the LSP server doesn't support or expose functionality that is as yet unmapped to Atom (TODO and TBD in the capabilities table above).

* [languageserver-csharp](https://github.com/atom/languageserver-csharp) provides C# support via [Omnisharp (node-omnisharp)](https://github.com/OmniSharp/omnisharp-node-client)
* [languageserver-java](https://github.com/atom/languageserver-java) provides Java support via [Java Eclipse JDT](https://github.com/eclipse/eclipse.jdt.ls)

Additional LSP servers for consideration can be found at [LangServer.org](http://langserver.org)

## Contributing
Always feel free to help out!  Whether it's [filing bugs and feature requests](https://github.com/atom/atom-languageclient/issues/new) or working on some of the [open issues](https://github.com/atom/atom-languageclient/issues), Atom's [contributing guide](https://github.com/atom/atom/blob/master/CONTRIBUTING.md) will help get you started while the [guide for contributing to packages](https://github.com/atom/atom/blob/master/docs/contributing-to-packages.md) has some extra information.

## License
MIT License.  See [the license](LICENSE.md) for more details.
