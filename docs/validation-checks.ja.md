# リポジトリ検証項目

[English](validation-checks.md) · [README に戻る](../README.ja.md)

`scripts/validate_repo.py` は、スキル梱包、対訳ドキュメント、リンク、テキスト
衛生、プライバシー、asset ライセンスを一括で確認する唯一のゲートです。
[validate workflow](../.github/workflows/validate.yml) が Linux と Windows の
両方で実行し、[CONTRIBUTING](../CONTRIBUTING.ja.md) は PR 前の実行を必須と
しています。このページは実際の検証内容を一覧化したものです。

## 実行方法

```
python -m pip install "PyYAML==6.0.2"
PYTHONUTF8=1 python scripts/validate_repo.py
```

各検証は共通のエラー一覧へ追記するため、最初の失敗で停止せず全件を報告します。
終了コード `0` で要約 1 行、`1` で `Repository validation failed:` と
`- メッセージ` 行を stderr に出力します。

## 検証グループ

| 順序 | 関数 | 対象 |
|---|---|---|
| 1 | `validate_required` | 必須ファイル 23 件 |
| 2 | `validate_skills` | `.agents/skills/` のスキル 8 件 |
| 3 | `validate_links` | 全 `*.md` の相対リンク |
| 4 | `validate_text_and_privacy` | 文字コード、空白、禁止文字列、プライバシー |
| 5 | `validate_assets` | 事例 asset と banner のライセンス表記 |

## 1. 必須ファイル

いずれもファイルとして存在が必要で、欠落は
`missing required file: <path>` として報告されます。

| 区分 | ファイル |
|---|---|
| ルート | `README`、`SUPPORT`、`CONTRIBUTING` の英日対、`ASSET-LICENSES.md` |
| ライセンス | `LICENSE`、`LICENSES/CC-BY-SA-4.0.txt` |
| ガイド | `docs/installation`、`docs/choose-a-skill`、`docs/prompts`、`docs/artifact-contracts`、`docs/case-study-nescart` の各 `.md` と `.ja.md` |
| 記録 | `docs/privacy-review.md`（英語のみ） |
| asset | `assets/banner.png` |
| インストーラ | `scripts/install-skills.ps1`、`scripts/install-skills.sh` |

## 2. スキルパッケージ

`.agents/skills/` 直下のディレクトリ集合は `manage-pcba-program`、
`plan-electronic-product`、`qualify-pcba-sourcing`、
`design-and-review-circuit`、`schematic-humanizer`、`pcb-layout-review`、
`release-pcba-fabrication`、`operate-jlcpcb-order` と完全一致が必要で、過不足は
集合不一致として報告されます。

| 検証 | 規則 |
|---|---|
| 構成ファイル | `SKILL.md`、`agents/openai.yaml`、単独 `LICENSE` が揃う。欠けた場合そのスキルの以降の検証は省略 |
| 単独ライセンス | スキルの `LICENSE` が前後空白を除いてリポジトリ MIT `LICENSE` と一致 |
| 分量 | `SKILL.md` は 500 行以内 |
| 未完了記述 | `SKILL.md` に大文字小文字を問わず `TODO` を含まない |
| frontmatter | `SKILL.md` は解析可能な YAML frontmatter で始まり、キーは `name` と `description` のみ |
| 本文 | frontmatter 以降の本文が空でない |
| 名前 | `name` がディレクトリ名と一致し、小文字ケバブケース |
| 説明 | `description` が前後空白を除き 40 文字以上 |
| interface | `openai.yaml` の `interface` に空でない文字列 `display_name`、`short_description`、`default_prompt` がある |
| 短い説明 | `short_description` が 25〜64 文字 |
| 既定プロンプト | `default_prompt` に `$<スキル名>` を含む |

## 3. Markdown リンク

`.git` を除く全 `*.md` のインラインリンクと画像が対象です。`http://`、
`https://`、`mailto:`、`#` で始まる参照は対象外とし、それ以外は
パーセントデコードとフラグメント除去を行い、記載ファイルの位置を基準に解決して
実在を確認します。未解決は `broken link in <file>: <target>` になります。

## 4. テキスト衛生とプライバシー

拡張子 `.md`、`.yml`、`.yaml`、`.py`、`.ps1`、`.sh`、`.json`、`.csv`、`.svg`、
`.txt` のファイルを UTF-8 として読み、行単位で検査します。

| 検証 | 規則 |
|---|---|
| キャッシュ | git 追跡下の `.pyc`、`.pyo`、`__pycache__` 配下が存在しない |
| 文字コード | 対象ファイルが UTF-8 として復号できる |
| 空白 | 行末に空白や tab がない。報告は `<file>:<line>` 形式 |
| 禁止文字列 | 旧リポジトリ URL を含まない。バックスラッシュ正規化後も判定 |
| メールアドレス | `@example.com` 以外のアドレスを含まない |
| アクセストークン | `gh[pousr]_` や `sk-` 形式で 20 文字以上の値を含まない |
| 認証情報 | `api_key`、`access_token`、`secret`、`password`、`cookie`、`authorization` への 12 文字以上の代入を含まない |
| 注文識別子 | `pcbfile`、`quote`、`order`、`project` の識別子へ 8 文字以上を代入していない |
| 注文 URL 識別子 | `pcbFileNo`、`quoteId`、`orderId` のクエリ値が 8 文字以上でない |
| Windows 絶対パス | ドライブレター付きパスは説明用の `c:\path\to\project` 配下のみ許可 |
| POSIX ホームパス | `/home/user/` と `/Users/user/` 配下のみ許可 |

`example-token`、`redacted`、`placeholder`、`fixture-id` は安全な仮値として
除外し、プライバシー検出は 1 パターンにつき 1 ファイル 1 件までを報告します。
`scripts/validate_repo.py` 自身はパターン定義を含むため、禁止文字列、
プライバシー、絶対パスの検査対象外です。文字コードと行末空白は検査されます。

## 5. asset とライセンス

`assets/case-studies/nescart/` が存在し、その中の全ファイルが
`ASSET-LICENSES.md` にリポジトリ相対パスで記載され、`assets/banner.png` も
記載が必要です。事例ディレクトリ自体が無い場合、このグループは打ち切ります。

## 一覧の維持

対訳ガイドを追加する際は同じ変更で英日 2 ファイルを `REQUIRED` に追加し、
このページと対訳版を更新し、[プライバシーレビュー](privacy-review.md)が公開
asset を網羅しているか確認します。画像のプライバシーはテキスト検査で証明でき
ないため、追加・差し替え時は原寸の目視レビューが必要です。
