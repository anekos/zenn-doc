---
title: "Neovimで画像を即座にプレビューする - Preview on Console プラグイン"
emoji: "🖼️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "vim", "terminal"]
published: false
---

# Neovimで画像を即座にプレビューする - Preview on Console プラグイン

ターミナル上でNeovimを使っていて、「この画像ファイルどんな内容だっけ？」と思ったことはありませんか？毎回別のアプリケーションで開くのは面倒ですよね。

そんな悩みを解決する **Preview on Console** プラグインを紹介します。このプラグインを使えば、Neovim上でファイルパスにカーソルを移動するだけで、ターミナル上に自動的に画像がプレビュー表示されます。

**プラグインのリポジトリ**: https://github.com/anekos/preview-on-console.nvim

![プラグインのスクリーンショット](/images/preview-on-console-screenshot.jpg)

## 特徴

- **リアルタイムプレビュー**: カーソルがファイルパスに移動すると自動的に画像を表示
- **豊富なフォーマット対応**: JPG、JPEG、PNG、GIF、BMP、WEBP、SVG、PDFをサポート  
- **キャッシュ機能**: 変換済みファイルをキャッシュして高速化
- **デバウンス処理**: カーソル移動時の過度な更新を防止
- **切り替え機能**: プレビュー機能のオン/オフ切り替え
- **[Liname](https://github.com/anekos/liname-hs)形式対応**: タブ区切りの行番号とファイルパス形式に特別対応

## 仕組み

このプラグインは2つの部分から構成されています：

1. **Neovimプラグイン**: カーソル移動を監視し、ファイルパスを抽出
2. **プレビューデーモン**: FIFOを通してファイルパスを受け取り、実際の表示を行う

### 処理の流れ

1. Neovimでカーソルがファイルパス上に移動
2. プラグインがパスを抽出してFIFO経由でデーモンに送信
3. デーモンがファイル形式に応じて処理：
   - **画像**: `kitty icat`で直接表示
   - **PDF**: `pdftoppm`でPNGに変換してから表示
   - **SVG**: `rsvg-convert`でPNGに変換してから表示
4. 変換済みファイルは`/tmp/preview_on_console_cache`にキャッシュ

## 必要な環境

- **Neovim**: 任意のバージョン
- **Kittyターミナル**: 画像表示に必要
- **pdftoppm** (poppler-utils): PDF表示用
- **rsvg-convert** (librsvg): SVG表示用

### 依存関係のインストール

```bash
# Ubuntu/Debian
sudo apt install poppler-utils librsvg2-bin

# Arch Linux  
sudo pacman -S poppler librsvg

# macOS
brew install poppler librsvg
```

## インストール

### lazy.nvim使用時

```lua
{
  "anekos/preview-on-console.nvim",
  config = function()
    require('preview-on-console').setup()
  end
}
```

### packer.nvim使用時

```lua
use {
  'anekos/preview-on-console.nvim',
  config = function()
    require('preview-on-console').setup()
  end
}
```

## 使い方

### 基本的な使用方法

1. **別ターミナルでプレビューデーモンを起動**:
   ```bash
   ./preview-on-console
   ```

2. **Neovimでプレビューを有効化**:
   ```
   :POCEnable
   ```

3. **ファイルパス上にカーソルを移動** - デーモンを実行しているターミナルに自動的にプレビューが表示されます

### コマンド

- `:POCEnable` - プレビュー機能を有効化
- `:POCDisable` - プレビュー機能を無効化
- `:POCToggle` - プレビュー機能の切り替え
- `:POCLinameEnable` - [Liname](https://github.com/anekos/liname-hs)形式（タブ区切りの行番号:パス）モードを有効化

### [Liname](https://github.com/anekos/liname-hs)モード

以下のようなタブ区切り形式のファイルに対応：

```
123	/path/to/image.jpg
456	/path/to/document.pdf
```

`:POCLinameEnable`を使用すると、行番号とファイルパスのマッピングをキャッシュして高速化します。

## 対応ファイル形式

| 形式 | 拡張子 | 必要な依存関係 |
|------|--------|----------------|
| 画像 | jpg, jpeg, png, gif, bmp, webp | Kittyターミナル |
| PDF | pdf | pdftoppm, Kittyターミナル |
| SVG | svg | rsvg-convert, Kittyターミナル |

## パフォーマンス最適化

- **デバウンス処理**: 200msの遅延でカーソル移動時の過度な更新を防止
- **キャッシュ機能**: 変換済みファイルを`/tmp/preview_on_console_cache`にキャッシュ
- **効率的なファイルパス抽出**: 正規表現を使った高速なパス検出

## トラブルシューティング

### プレビューが表示されない場合

1. **Kittyターミナルの確認**: `which kitty`
2. **依存関係の確認**: `which pdftoppm rsvg-convert`  
3. **デーモンの確認**: `preview-on-console`スクリプトが別ターミナルで実行されているか
4. **FIFO権限の確認**: `/tmp/preview_on_console_fifo`の権限を確認

## まとめ

Preview on Consoleプラグインを使えば、Neovimでの開発効率が大幅に向上します。特に：

- 画像ファイルの内容を素早く確認したい時
- PDFドキュメントの概要を把握したい時  
- SVGアイコンの見た目を確認したい時

などに非常に便利です。

ターミナル環境でのNeovim使用がより快適になること間違いなしです！ぜひ試してみてください。

## ライセンス

MIT License

## 貢献

改善、バグ修正、新機能の追加など、プルリクエストは大歓迎です！
