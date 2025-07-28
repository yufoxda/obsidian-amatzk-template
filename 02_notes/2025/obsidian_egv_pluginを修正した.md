---
tags:
  - note
  - obsidian
  - plugin
  - typescript
  - oss
---
[[obsidianグラフweb]]
これをやるためにExport Graph View Pluginを導入してみた。
無事ファイルは出力できたが、グラフ表示ができなかった。
vscodeのdotファイルプレビューアドオンを入れたりやwebサイトで試してみたが、表示できなかった。
ファイルの中身をで見ると
```dot
digraph "obsidian-amatzk-template-main" {
    rankdir=LR;
    node [shape=box, style=rounded];
    "01_diary_2025_2025_07_28_md" [label="2025-07-28", fillcolor="#e3f2fd", style="filled,rounded"]; 
```
となっていた。
なんか```-``` が入っているのが怪しいとおもい、消してみると表示できることが判明。
[pluginのgithub](https://github.com/seantiz/obsidian_egv_plugin)を見てみるとmain関数内に[dotfileを作成していそうな関数](https://github.com/seantiz/obsidian_egv_plugin/blob/main/src/main.ts#L101)が運よくわかった。
この方、かなり見やすいコードを書いてくれている。
ここから使用している関数に飛んでみると
[obsidian\_egv\_plugin/src/whisperer.ts at main · seantiz/obsidian\_egv\_plugin · GitHub](https://github.com/seantiz/obsidian_egv_plugin/blob/main/src/whisperer.ts#L749)
```TS
	runDotPrinter(graph: Graph): string {
		const vaultName = this.app.vault.getName()
		let dot = `digraph ${vaultName} {\n`
		dot += '    rankdir=LR;\n'
		dot += '    node [shape=box, style=rounded];\n'
/*
省略
*/
// Inject nodes without clustering
			graph.nodes.forEach((node) => {
				const nodeId = cleanId(node.id)
				const nodeStyle =
					node.type === 'attachment'
						? 'fillcolor="#ffcc80", style="filled,rounded"'
						: 'fillcolor="#e3f2fd", style="filled,rounded"'

				dot += `    "${nodeId}" [label="${safeEscape(node.name)}", ${nodeStyle}];\n`
			})
```
となっていた。
```ts
dot += `    "${nodeId}" [label="${safeEscape(node.name)}", ${nodeStyle}];\n`
```
と同じようにやれば``` -``` も文字列として扱われてできるのではないかと考えた。
そこで
```ts
let dot = `digraph "${vaultName}" {\n`
```
としてみた。

どうやら``` <vault path>.obsidian\plugins\```にプラグインのプログラムがあるようなのでビルドしたものをそこにおいてみるとうまく動いてしまった。
オープンソースやプラグインの不具合を見つけて修正できたことは初めてだったが、AIに文章を考えてもらってissueとprを作成した。
[Errors with special characters in dot file export · Issue #25 · seantiz/obsidian\_egv\_plugin](https://github.com/seantiz/obsidian_egv_plugin/issues/25)
[fix:dotfile export defect #25 by yufoxda · Pull Request #26 · seantiz/obsidian\_egv\_plugin · GitHub](https://github.com/seantiz/obsidian_egv_plugin/pull/26)
マージしてくれると嬉しいな。