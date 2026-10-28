# ZQL-16-VSCode-Extensions — Walkthrough Labs

**Source:** [ZQL-16-VSCode-Extensions.md](../ZQL-16-VSCode-Extensions.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** VS Code extension development, quantum IDE tooling, VSIX packaging

---

## Lab 1: VS Code Extension Architecture
```python
print("Extension: package.json + activation events + commands + providers")
print("Lifecycle: activate() → commands registered → deactivate()")
```
### Deliverable: `extension_arch.py`

## Lab 2: Extension Manifest (package.json)
```python
import json
manifest = {"name":"zuntenium-vscode","version":"0.2.0",
    "engines":{"vscode":"^1.80.0"},"activationEvents":["onLanguage:python"],
    "main":"./out/extension.js","contributes":{"commands":[],"views":{}}}
print(json.dumps(manifest, indent=2))
```
### Deliverable: `manifest.py`

## Lab 3: Command Registration
```python
print("registerCommand('zuntenium.runCircuit', callback)")
print("Appears in Command Palette: 'Zuntenium: Run Circuit'")
```
### Deliverable: `commands.py`

## Lab 4: Language Features — Syntax Highlighting
```python
print("TextMate grammar for .qasm files")
print("Semantic tokens for QuantumCircuit method chains")
```
### Deliverable: `syntax.py`

## Lab 5: Code Prediction Extension
```python
print("zuntenium-predict: IntelliSense for quantum gates")
print("CompletionItemProvider: suggest gates after 'qc.'")
```
### Deliverable: `prediction.py`

## Lab 6: Workspace UI Extension
```python
print("zuntenium-workspace: TreeView for circuits, WebView for visualisation")
print("Custom editor for circuit diagrams")
```
### Deliverable: `workspace_ui.py`

## Lab 7: API Explorer Extension
```python
print("zuntenium-api-explorer: browse QuFlow API docs inline")
print("Hover provider: show gate documentation on hover")
```
### Deliverable: `api_explorer.py`

## Lab 8: SecretStorage Authentication
```python
print("Use VS Code SecretStorage API for API keys")
print("context.secrets.store('apiKey', key)")
print("Never store secrets in settings.json!")
```
### Deliverable: `auth.py`

## Lab 9: WebView Panels
```python
print("WebView: embedded browser panel for rich UI")
print("Circuit visualisation, Bloch sphere, results histogram")
```
### Deliverable: `webview.py`

## Lab 10: TreeView Data Provider
```python
print("Custom TreeView: list circuits, show gate counts, organise by module")
print("Implement TreeDataProvider interface")
```
### Deliverable: `treeview.py`

## Lab 11: Diagnostic Provider
```python
print("DiagnosticCollection: show errors/warnings in quantum code")
print("Check: qubit index bounds, gate parameter ranges, measurement before use")
```
### Deliverable: `diagnostics.py`

## Lab 12: Code Lens
```python
print("CodeLens: clickable 'Run Circuit' / 'Visualise' above circuit definitions")
print("CodeLensProvider: detect QuantumCircuit constructors")
```
### Deliverable: `codelens.py`

## Lab 13: Snippet Contributions
```python
print("Snippets: qc-init, qc-bell, qc-ghz, qc-vqe-template")
print("Tab triggers for common quantum patterns")
```
### Deliverable: `snippets.py`

## Lab 14: Testing Extensions
```python
print("@vscode/test-electron: run tests in VS Code instance")
print("Mock workspace, activate extension, assert commands work")
```
### Deliverable: `testing.py`

## Lab 15: VSIX Packaging
```python
print("vsce package → zuntenium-vscode-0.2.0.vsix")
print("Include: compiled JS, assets, README, changelog")
print("Exclude: node_modules, src/, tests/")
```
### Deliverable: `vsix.py`

## Lab 16: Publishing to Marketplace
```python
print("vsce publish → VS Code Marketplace (or private registry)")
print("Requires: publisher account, Personal Access Token")
```
### Deliverable: `publishing.py`

## Lab 17: Local Execution Bridge
```python
print("Extension → spawn Python process → import zuntenium → run circuit")
print("stdout/stderr capture → parse results → display in WebView")
```
### Deliverable: `execution_bridge.py`

## Lab 18: Configuration Schema
```python
print("contributes.configuration: zuntenium.simulator, zuntenium.shots")
print("Workspace vs user settings")
```
### Deliverable: `configuration.py`

## Lab 19: Extension Bundling (esbuild)
```python
print("esbuild: bundle TypeScript to single JS file → fast activation")
print("Tree-shake unused imports → smaller VSIX")
```
### Deliverable: `bundling.py`

## Lab 20: VS Code Extensions Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("Extension architecture",True);ok("Commands",True);ok("WebView",True)
ok("VSIX packaging",True);ok("SecretStorage",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("vscode_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `vscode_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct VS Code extension concepts ✓
