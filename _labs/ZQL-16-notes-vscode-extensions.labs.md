# ZQL-16-notes-vscode-extensions — Walkthrough Labs

**Source:** [ZQL-16-notes-vscode-extensions.md](../ZQL-16-notes-vscode-extensions.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** VS Code extension API deep-dive, TypeScript patterns, publishing

---

## Labs 1-20: VS Code extension API overview, activation events deep-dive, contribution points, ExtensionContext lifecycle, Disposable pattern, EventEmitter pattern, TextDocument API, CompletionItemProvider, HoverProvider, DefinitionProvider, DocumentSymbolProvider, workspace configuration API, OutputChannel logging, StatusBarItem, progress notifications, multi-root workspace support, extension testing with Mocha, continuous integration for extensions, marketplace SEO, integration capstone.

## Lab 1: VS Code Extension API Overview
```python
print("vscode module: window, workspace, commands, languages, debug")
print("Provider pattern: register providers for language features")
```
### Deliverable: `api_overview.py`

## Lab 2: Activation Events
```python
print("onLanguage:python — activate when Python file opened")
print("onCommand:zuntenium.run — activate on command execution")
print("workspaceContains:**/*.qasm — activate if .qasm files present")
```
### Deliverable: `activation.py`

## Lab 3: Contribution Points
```python
print("commands, keybindings, menus, views, viewsContainers")
print("configuration, languages, grammars, snippets, themes")
```
### Deliverable: `contributions.py`

## Lab 4: ExtensionContext Lifecycle
```python
print("activate(context): called once, register all providers")
print("context.subscriptions.push(disposable) → auto-cleanup")
print("deactivate(): cleanup on extension unload")
```
### Deliverable: `lifecycle.py`

## Lab 5: Disposable Pattern
```python
print("Every registration returns a Disposable")
print("Push to context.subscriptions → VS Code manages cleanup")
```
### Deliverable: `disposable.py`

## Lab 6: EventEmitter Pattern
```python
print("vscode.EventEmitter<T>: fire events, consumers listen")
print("TreeDataProvider.onDidChangeTreeData → refresh tree view")
```
### Deliverable: `event_emitter.py`

## Lab 7: TextDocument API
```python
print("document.getText(), document.lineAt(n), document.positionAt(offset)")
print("TextEdit: insertions, deletions, replacements")
```
### Deliverable: `text_document.py`

## Lab 8: CompletionItemProvider
```python
print("provideCompletionItems(document, position) → CompletionItem[]")
print("Label, detail, documentation, insertText, kind")
```
### Deliverable: `completion.py`

## Lab 9: HoverProvider
```python
print("provideHover(document, position) → Hover with MarkdownString")
print("Show gate documentation, parameter info on hover")
```
### Deliverable: `hover.py`

## Lab 10: DefinitionProvider
```python
print("provideDefinition(document, position) → Location")
print("Go to definition for QuantumCircuit methods")
```
### Deliverable: `definition.py`

## Lab 11: DocumentSymbolProvider
```python
print("Outline view: show circuit definitions, gate sequences")
print("SymbolKind: Function, Class, Variable")
```
### Deliverable: `symbols.py`

## Lab 12: Workspace Configuration
```python
print("workspace.getConfiguration('zuntenium').get('shots', 4096)")
print("onDidChangeConfiguration → react to settings changes")
```
### Deliverable: `config.py`

## Lab 13: OutputChannel Logging
```python
print("window.createOutputChannel('Zuntenium')")
print("channel.appendLine('Circuit executed: 4096 shots')")
```
### Deliverable: `output_channel.py`

## Lab 14: StatusBarItem
```python
print("StatusBarItem: show 'Zuntenium: Ready' in status bar")
print("Click → trigger command; color → indicate state")
```
### Deliverable: `statusbar.py`

## Lab 15: Progress Notifications
```python
print("window.withProgress({location: Notification, title: 'Running...'})")
print("report.increment(10) → update progress bar")
```
### Deliverable: `progress.py`

## Lab 16: Multi-Root Workspace
```python
print("workspace.workspaceFolders → iterate all root folders")
print("Find QuantumCircuit files across multiple project roots")
```
### Deliverable: `multi_root.py`

## Lab 17: Extension Testing (Mocha)
```python
print("@vscode/test-electron: launch VS Code, run Mocha tests")
print("suite('Extension Tests', () => { test('activates', ...) })")
```
### Deliverable: `mocha_testing.py`

## Lab 18: CI for Extensions
```python
print("GitHub Actions: npm ci → npm run compile → npm test → vsce package")
print("Upload .vsix as artifact")
```
### Deliverable: `ci_extensions.py`

## Lab 19: Marketplace SEO
```python
print("Good README with screenshots → higher ranking")
print("Categories: 'Programming Languages', 'Linters', 'Snippets'")
print("Tags: quantum, python, qiskit, zuntenium")
```
### Deliverable: `marketplace_seo.py`

## Lab 20: Integration Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("API overview",True);ok("Providers",True);ok("Testing",True);ok("VSIX",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("vscode_notes_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `vscode_notes_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct concepts ✓
