# Minecraft Mod 翻訳手順書 (ローカルLLM/Ollama活用)

Minecraftの一部のModは日本語に対応していないことがあります。本手順書では、ローカルLLM（Ollama）とマルチモーダルAIモデル（画像認識対応モデル）を組み合わせて、ゲーム画面のスクリーンショットからテキストを抽出し、Minecraftの公式用語に沿った日本語へ翻訳する手順を解説します。

---

## 1. 前提条件

- **Ollama** がシステムにインストールされ、バックグラウンドで実行されていること。
- 画像認識（Vision）に対応したモデル（例: `qwen3-vl` や `qwen2-vl` など）が利用可能であること。
  - ※モデルのサイズや種類は、実行環境のスペックに合わせて適宜選択してください。

---

## 2. カスタムモデルの作成

Minecraft向けの翻訳指示（システムプロンプト）や、公式用語の定義を組み込んだカスタムモデル（`mc-translator`）を作成します。

### ステップ 1: 設定用ディレクトリの作成

Modelfileを配置するためのディレクトリを作成します。

```bash
mkdir -p ~/.config/ollama
```

### ステップ 2: Modelfileの作成

設定ファイルである `Modelfile` を作成します。

```bash
touch ~/.config/ollama/Modelfile
```

作成した `~/.config/ollama/Modelfile` に、以下の内容を記述します。

```dockerfile
FROM qwen3-vl:8b

PARAMETER temperature 0.5
PARAMETER top_p 0.5
PARAMETER stop "<|endoftext|>"
PARAMETER stop "<|im_end|>"

SYSTEM """
You are a Minecraft translation assistant. Please extract the text from the screenshot and output it in the following format. 

CRITICAL RULE FOR TRANSLATION:
Use official Minecraft terminology for the translation. 
- 'water source' -> '水源'
- 'full sources' -> '無限水源'
- 'bonemeal' -> '骨粉'
- 'flowing water' -> '水流'
- 'bowl' -> 'ボウル'
- 'materials' -> '素材' or 'ブロック'
- 'fluids' -> '液体'

---
[Original English]:
(extracted text here)

[Japanese Translation]:
(translated text here)
---
"""
```

### ステップ 3: カスタムモデルのビルド

作成した `Modelfile` を指定して、Ollamaにカスタムモデル `mc-translator` を作成（登録）します。

- Modelfile の配置先に移動

```bash
cd ~/.config/ollama
```

- mc-translator モデルをビルド

```bash
ollama create mc-translator -f ~/.config/ollama/Modelfile
```

---

## 3. 翻訳の実行

ゲーム内で翻訳したい画面のスクリーンショットを撮影し、以下のコマンドを実行します。

```bash
ollama run mc-translator "Translate this" /path/to/screenshot.png
```

実行すると、モデルが画像から英語テキストを自動的に抽出し、Minecraft公式用語を適用した日本語訳を出力します。

---

## 4. カスタマイズと用語の追加

翻訳の精度をより向上させるため、Mod特有の用語や新しい翻訳ルールを追加したい場合は、`~/.config/ollama/Modelfile` の `SYSTEM` プロンプト内にある用語リストを適宜編集し、再度 **ステップ 3** のビルドコマンドを実行してください。
