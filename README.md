# github-gas-four-keys

## 概要

GoogleSpreadsheet で GitHub レポジトリの FourKeys の計測/可視化を行うツールです.
以下の特徴があります.

- PullRequest のソースブランチ名を利用して FourKeys 計測を行うため、導入が容易
- GAS を用いて自動化されるため、インフラコストが不要

![出力例](img/example.png)

## インストレーション

### GitHub アクセストークンの準備

https://github.com/settings/tokens/new から repo にチェックを入れアクセストークンを発行します(トークンは後ほど使います).

### GAS API の有効化

https://script.google.com/home/usersettings から Google Apps Script API をオンにします.

### Clasp による Spreadsheet と GAS の作成

1. 以下のシェルを実行. GoogleDrive 上に `Github-gas-four-keys` というスプレッドシートが作成されます.

```sh
npm install @google/clasp -g

git clone https://github.com/cosoji-jp/github-gas-four-keys.git
cd github-gas-four-keys
npm init -y

# ブラウザでGoogleアカウントのログインが求められます.
clasp login
# ログインしたアカウントのGoogleDriveのマイドライブのルートにSpreadsheetが作成されます.
clasp create --type sheets
clasp push
```

2. GoogleDrive 上に `Github-gas-four-keys` という名前のスプレッドシートが作られます.
3. 作成されたスプレッドシートの[Apps Script プロジェクト](https://developers.google.com/apps-script/guides/projects?hl=ja#create-from-docs-sheets-slides)を開きます.
4. 以下の[ScriptProperty](https://developers.google.com/apps-script/guides/properties?hl=ja#add_script_properties)を設定します.

| プロパティ        | 値                                                                           |
| ----------------- | ---------------------------------------------------------------------------- |
| GITHUB_API_TOKEN  | GitHub アクセストークンの準備で作成したトークン. 例) `ghp_` から始まる文字列 |
| GITHUB_REPO_NAMES | レポジトリ名の JSON 配列. 例) `["github-gas-four-keys"]`                     |
| GITHUB_REPO_OWNER | レポジトリの Owner あるいは Organization 名. 例) `cosoji-jp`                 |

5. [Apps Script プロジェクト](https://developers.google.com/apps-script/guides/projects?hl=ja#create-from-docs-sheets-slides)から、 `initialize` 関数を実行し、スプレッドシートを初期化します.
6. [Apps Script プロジェクト](https://developers.google.com/apps-script/guides/projects?hl=ja#create-from-docs-sheets-slides)から、 `getAllRepos` 関数を実行し、Github から PR 情報を取得します.
7. スプレッドシートの"FourKeys 計測結果"シートに FourKeys 分析結果が出力されます.

### 自動計測設定

上記手順で `initialize` 関数を実行すると毎週日曜日 00:00~01:00 で自動的に PR が取得される[トリガー](https://developers.google.com/apps-script/guides/triggers/installable?hl=ja#time-driven_triggers)が設定されます。カスタマイズし任意の時間帯や間隔で実施することもできます.

## 各 FourKeys 項目の計測手法

### デプロイ頻度

GitHub 上の PullRequest のマージ頻度を計測しています.
1 日あたりの平均マージ回数によって、Elite/High/Medium/Low の分類が行われます.

デフォルトでは 1 週間に 3 回以上(1 日あたり 0.4285714286 回以上)の場合に Elite と判定されます.
この値はスプレッドシート上の分析設定シート E5 セルで設定することができます.

### 変更リードタイム

PullRequest のソースブランチ(作業ブランチ)上で行われた初回コミットから、その PullRequest がマージされるまでの時間を計測しています.
マージされるまでの平均時間によって、Elite/High/Medium/Low の分類が行われます.

デフォルトでは平均 24 時間(1 日)以内の場合に Elite と判定されます.
この値はスプレッドシート上の分析設定シート E8 セルで設定することができます.

### 変更障害率

各 PullRequest のソースブランチのうち、障害対応を行ったブランチ(デフォルトではブランチ名に hotfix が付くブランチ)または巻き戻しを行ったブランチ(デフォルトではブランチ名に revert が付くブランチ)の割合を計測しています.

この割合によって、Elite/High/Medium/Low の分類が行われます.
デフォルトでは割合が 15%以下の場合 Elite と判定されます.
この値はスプレッドシート上の分析設定シート E11 セルで設定することができます.

障害対応を行ったブランチ名の判定ルールは、スプレッドシート上の分析設定シート E3 セルで設定することができます.

### 平均修復時間

障害対応を行った PullRequest のソースブランチ(デフォルトではブランチ名に hotfix が付くブランチ)の初回コミットから、その PullRequest がマージされるまでの時間を計測しています.
マージされるまでの平均時間によって、Elite/High/Medium/Low の分類が行われます.

デフォルトでは平均 24 時間(1 日)以内の場合に Elite と判定されます.
この値はスプレッドシート上の分析設定シート E14 セルで設定することができます.

## その他のカスタマイズ

### 計測範囲

FourKeys 計測結果シートの計測は移動平均に基づいて計測されます.
移動平均のウィンドウ幅(日単位)は分析設定シート E2 セルで設定することができます.
デフォルトでは 28 日(4 週間)に設定されています.

### 計測対象ユーザの設定

分析設定シートの A 列に PullRequest を作成したユーザ名が表示されます.
各ユーザの隣のセル(B 列)に`FALSE`を入力するとそのユーザの PullRequest を計測対象から除外することができます.
`FALSE`以外の値が入力されている、もしくはブランクの場合、計測対象になります.

## 詳細な PR データの取得

### プルリク ALL データ機能

`getAllPullRequestsDetailed` 関数を実行することで、より詳細な PR データを「プルリク ALL データ」シートに取得できます。
この機能では以下の情報が収集されます：

| 項目                    | 説明                                          |
| ----------------------- | --------------------------------------------- |
| メンバー名              | PR 作成者の名前                               |
| ブランチ名              | PR のソースブランチ名                         |
| PR 本文                 | PR の説明文                                   |
| マージ済                | マージ済みかどうか                            |
| 初コミット日時          | PR の最初のコミット日時                       |
| マージ日時              | PR がマージされた日時                         |
| マージまでの時間(hours) | 初コミットからマージまでの時間（時間単位）    |
| リポジトリ              | PR が属するリポジトリ名                       |
| 障害発生判定            | 障害発生と判定されたかどうか                  |
| 障害対応 PR             | 障害対応 PR と判定されたかどうか              |
| レビュー者              | PR のレビューを行ったユーザー（カンマ区切り） |
| 初レビュー日時          | 最初のレビューが行われた日時                  |

### 使用方法

1. [Apps Script プロジェクト](https://developers.google.com/apps-script/guides/projects?hl=ja#create-from-docs-sheets-slides)から、`getAllPullRequestsDetailed` 関数を実行します。
2. GAS の実行時間制限（1800 秒）を考慮して、取得処理が途中で終了した場合は再度関数を実行することで続きから処理が再開されます。
3. 進捗状況は「PR データ進捗管理」シートで確認・管理されます。

### 自動再開の仕組み

実行時間制限を考慮して、以下の情報がスプレッドシート内で管理されます：

- 現在処理中のリポジトリインデックス
- ページネーションカーソル位置
- 次回書き込み開始行

これにより、大量の PR データがある場合でも確実に全データを取得できます。
