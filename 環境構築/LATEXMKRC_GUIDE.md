# latexmk 設定ガイド

## 概要

`latexmk` は LaTeX コンパイルを自動化・監視するツールです。本プロジェクトで提供されている `.latexmkrc` ファイルは、LuaTeX + 日本語対応の自動ビルド環境を実現します。

## インストール

### 前提条件

- **TeX Live 2024** 以降（`latexmk` を含む）
- **LuaLaTeX** コンパイラ
- **Perl**（latexmk はPerl製）

### インストール確認

```powershell
# PowerShell で実行
latexmk -version
```

バージョン情報が表示されればインストール完了です。

## 配置場所と階層構造

```
C:\Users\takut\OneDrive\ドキュメント\レポート\
├── .latexmkrc                    # グローバル設定（全プロジェクト共通）
├── Latex_template\
│   ├── .latexmkrc               # テンプレート用設定
│   ├── sample\main.tex
│   └── chemical_sample\main.tex
├── 蒸留\
│   ├── .latexmkrc               # プロジェクト固有設定
│   └── main.tex
└── 流動\
    ├── .latexmkrc               # プロジェクト固有設定
    └── main.tex
```

**優先順位**：プロジェクト固有設定 > グローバル設定

## 基本的な使用方法

### 1. 単一コンパイル

```powershell
cd "C:\Users\takut\OneDrive\ドキュメント\レポート\蒸留"
latexmk main.tex
```

**動作**：
- `main.tex` を LuaTeX でコンパイル
- 必要に応じて複数回コンパイル（参考文献参照用）
- 補助ファイルを削除
- PDF を生成

### 2. 自動監視モード（推奨）

```powershell
latexmk -pvc main.tex
```

**動作**：
- ファイルシステムを監視
- `.tex` ファイルが変更されると自動で再コンパイル
- PDF を自動で開く/更新
- **メリット**：編集中に常に最新 PDF を表示

**終了方法**：`Ctrl+C` キーを押す

### 3. 強制再コンパイル

```powershell
latexmk -f main.tex
```

キャッシュを無視して強制的に再コンパイルします。

### 4. クリーンアップ

```powershell
latexmk -c
```

補助ファイル（`.aux`, `.log` など）を削除します。

### 5. 完全クリーンアップ

```powershell
latexmk -C
```

補助ファイル **および** PDF を削除します。

## .latexmkrc の主要設定項目

### コンパイラ設定

```perl
$pdf_mode = 4;          # 4 = LuaTeX → PDF 直接出力
$lualatex = 'lualatex %O %S';
```

| 値 | 説明 |
|-----|------|
| 0 | DVI モード（出力なし） |
| 1 | DVI → PS 変換 |
| 2 | DVI → PDF 変換 |
| 3 | pdfLaTeX 使用 |
| **4** | **LuaLaTeX 使用（推奨）** |
| 5 | XeLaTeX 使用 |

### ビルドプロセス

```perl
$max_repeat = 5;        # 最大 5 回再コンパイル
$bibtex_use = 0;        # bibtex は不使用
$makeindex = 0;         # makeindex は不使用
```

**必要な回数のコンパイル**：
- 通常：1 回
- 参考文献参照あり：2 回
- 複雑な参照：3 回以上

### 出力制御

```perl
@generated_exts = qw(aux log out idx ilg glo acn acr alg);
```

コンパイル成功後に削除するファイル拡張子を指定。

### プレビュー設定

```perl
$pdf_previewer = 'start %O %S';      # Windows のデフォルト PDF ビューアー
$preview_continuous_mode = 1;        # 自動監視モード有効
$new_viewer_always = 0;              # 既存ビューアーを再利用
```

## グラフ（pgfplots）対応

`latexmk` を使用する場合、グラフの依存関係を正しく認識させるために、`.latexmkrc` に以下を追加：

```perl
add_cus_dep('tex', 'pdf', 0, 'graph_pdf');

sub graph_pdf {
  my $file = shift;
  system("lualatex -interaction=nonstopmode -halt-on-error $file");
}
```

これにより、`figure.tex` が変更された場合、自動的に再コンパイルされます。

## トラブルシューティング

### latexmk が見つからない

```powershell
# latexmk のパスを確認
Get-Command latexmk
```

出力がない場合、TeX Live が正しくインストールされていません。

**解決策**：
```powershell
# TeX Live の完全インストール
# または
# PATH に TeX Live bin ディレクトリを追加
$env:PATH += ";C:\texlive\2024\bin\windows"
```

### Perl スクリプトエラー

```
Can't open perl script "latexmk": No such file or directory
```

**原因**：Perl が インストールされていない

**解決策**：TeX Live インストール時に Perl も同時にインストール

### ファイルが更新されない（-pvc モード）

```powershell
# latexmk を終了
Ctrl+C

# 強制再コンパイル
latexmk -f main.tex

# 再び監視モード開始
latexmk -pvc main.tex
```

### PDF が開かない

**原因**：デフォルト PDF ビューアーが設定されていない

**解決策**：`.latexmkrc` を編集：

```perl
# Adobe Reader を使用
$pdf_previewer = 'start "C:\Program Files\Adobe\Acrobat DC\Acrobat.exe" %S';

# Sumatra PDF を使用
$pdf_previewer = 'start "C:\Program Files\SumatraPDF\SumatraPDF.exe" %S';
```

## VS Code との連携

### LaTeX Workshop 設定

`.vscode/settings.json` に以下を追加：

```json
{
  "latex-workshop.latex.tools": [
    {
      "name": "lualatex",
      "command": "lualatex",
      "args": ["-interaction=nonstopmode", "-halt-on-error", "%DOC%"]
    }
  ],
  "latex-workshop.latex.recipes": [
    {
      "name": "latexmk",
      "tools": ["lualatex"]
    }
  ]
}
```

### コマンドラインでの実行

VS Code のターミナルから直接実行可能：

```powershell
cd "C:\Users\takut\OneDrive\ドキュメント\レポート\蒸留"
latexmk -pvc main.tex
```

## パフォーマンス最適化

### キャッシュの活用

```powershell
# 初回：完全ビルド
latexmk main.tex

# 2 回目以降：キャッシュから高速化
latexmk main.tex
```

### 並列コンパイル（高度な設定）

大規模プロジェクトの場合：

```perl
# .latexmkrc に以下を追加
$lualatex = 'lualatex -synctex=1 %O %S';
```

## 運用例

### 開発フロー（推奨）

```powershell
# 1. プロジェクトディレクトリに移動
cd "C:\Users\takut\OneDrive\ドキュメント\レポート\蒸留"

# 2. 自動監視モード開始
latexmk -pvc main.tex

# 3. エディタで main.tex を編集
# （.tex ファイル保存時に自動再コンパイル）

# 4. 完了後、Ctrl+C で終了
```

### CI/CD パイプライン

自動テストサーバーで使用：

```powershell
# エラーで停止
latexmk main.tex

# 成功時のみ PDF を出力
if ($?) {
  Copy-Item "main.pdf" "output/"
}
```

## 参考

- **latexmk マニュアル**：`perldoc latexmk`
- **TeX Live**：https://www.tug.org/texlive/
- **LaTeX Workshop**：https://github.com/James-Yu/LaTeX-Workshop

## まとめ

| 使用場面 | コマンド |
|--------|---------|
| 単一コンパイル | `latexmk main.tex` |
| 開発中（自動監視） | `latexmk -pvc main.tex` |
| 強制再コンパイル | `latexmk -f main.tex` |
| クリーンアップ | `latexmk -c` |
| 完全削除 | `latexmk -C` |

`.latexmkrc` の設定により、複雑なコンパイルプロセスが自動化され、開発効率が大幅に向上します。
