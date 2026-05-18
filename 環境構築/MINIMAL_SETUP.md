# LaTeX最小限環境構築ガイド（高速版）

## 必須パッケージ一覧

本テンプレートの**完全な運用**に必要な最小限パッケージ：

| パッケージ | 用途 | 代替なし |
|-----------|------|---------|
| `luatexja` | 日本語対応 | ✓ 必須 |
| `geometry` | ページレイアウト | ✓ 必須 |
| `titlesec` | セクション見出し | ✓ 必須 |
| `amsmath` | 数式環境 | ✓ 必須 |
| `amssymb` | 数学記号 | ✓ 必須 |
| `graphicx` | 画像挿入 | ✓ 必須 |
| `float` | 図表位置固定 [H] | - 推奨 |
| `hyperref` | ハイパーリンク | - 推奨 |
| `siunitx` | 単位表示 | - 推奨 |
| `pgfplots` | グラフ描画 | - グラフ使用時 |
| `tikz` | 図形描画 | - グラフ使用時 |

## インストール方法

### 方法1：TeX Live フルインストール（推奨）

```bash
# インストーラをダウンロード
# https://www.tug.org/texlive/acquire-netinstall.html

# install-tl-windows.exe を実行し、「Full」を選択
# インストール時間：30分〜1時間
# ディスク容量：約6〜7GB
```

**利点**：パッケージの追加导入が不要、安定した運用

### 方法2：カスタムインストール（上級者向け）

TeX Live インストーラの「Advanced」オプションで以下のみ選択：

```
- Base: latexbin, etc
- Essential packages: 
  - Japanese support (luatexja)
  - amsmath, amssymb
  - graphicx, geometry
  - pgfplots, tikz（グラフ使用時）
```

**利点**：ディスク容量削減（2〜3GB）  
**欠点**：依存パッケージの追加導入が必要な場合がある

## 最小限 main.tex テンプレート

```latex
% !TEX program = lualatex
\documentclass[11pt]{ltjarticle}
\usepackage{luatexja}
\usepackage[margin=20truemm]{geometry}
\usepackage{titlesec}
\usepackage{amsmath}
\usepackage{amssymb}
\usepackage{graphicx}
\usepackage{float}
\usepackage[hidelinks]{hyperref}
\usepackage{siunitx}

% 句読点の置換
\catcode`、=\active
\protected\def、{，}
\catcode`。=\active
\protected\def。{．}

\titleformat*{\section}{\large\bfseries}

\title{レポートタイトル}
\author{著者名}
\date{\today}

\begin{document}
\maketitle
\section{はじめに}
% ここにレポート内容を記載
\end{document}
```

## グラフが不要な場合

`figure.tex` ファイルを削除するか、`main.tex` から以下の行をコメントアウト：

```latex
% \input{figure.tex}
```

## VS Code 設定（settings.json）

```json
"latex-workshop.latex.tools": [
  {
    "name": "lualatex",
    "command": "lualatex",
    "args": ["-interaction=nonstopmode", "-halt-on-error", "%DOC%"]
  }
],
"latex-workshop.latex.recipes": [
  {
    "name": "lualatex",
    "tools": ["lualatex"]
  }
]
```

## コンパイル確認

```bash
# PowerShell で実行
cd "C:\Users\takut\OneDrive\ドキュメント\レポート\Latex_template\sample"
& "C:\texlive\2024\bin\windows\lualatex.exe" -interaction=nonstopmode -halt-on-error main.tex
```

出力に `Output written on main.pdf` と表示されれば成功。

## トラブル対応

| 問題 | 原因 | 解決策 |
|-----|------|--------|
| Package not found | パッケージが未インストール | TeX Live を再インストール（Full） |
| 文字化け | エンコーディング設定 | ファイルを UTF-8 で保存 |
| グラフが表示されない | pgfplots 未インストール | TeX Live 更新: `tlmgr update --all` |

## 今後の追加パッケージ

必要に応じて以下を追加可能：

- **化学分野**：`\usepackage[version=4]{mhchem}` 
- **電気分野**：`\usepackage{circuitikz}`
- **表の拡張**：`\usepackage{booktabs}` or `\usepackage{array}`

## ディスク容量

- TeX Live フルインストール：約 **6〜7GB**
- テンプレート実運用：約 **3GB**（フル機能使用時）
- 最小限（グラフなし）：約 **2GB**

## 参考リンク

- TeX Live: https://www.tug.org/texlive/
- LaTeX Workshop: https://github.com/James-Yu/LaTeX-Workshop
- 日本語 LaTeX: https://www.latex.or.jp/
