# Medixor 営業PDCA — 公開手順

所要 20〜30分。1〜7を上から順にやれば動きます。

---

## 0. ファイルの役割

| ファイル | どこに置くか |
|---|---|
| `index.html` | GitHub リポジトリ（公開される） |
| `supabase-setup.sql` | Supabase の SQL Editor に貼るだけ。リポジトリに置いてもよい |
| `medixor-leads.json` | **リポジトリに置かない。** 258件の個人情報。自分のPCに置いて、初回の取り込みにだけ使う |

`index.html` は GitHub Pages 上で誰でもソースを見られます。名簿はそこに入っていません。名簿は Supabase 側にあり、ログインした人だけが読めます。

---

## 1. Supabase プロジェクトを作る

1. https://supabase.com → New project
2. Region は **Northeast Asia (Tokyo)**
3. データベースのパスワードは控えておく

## 2. テーブルを作る

1. 左メニュー **SQL Editor** → New query
2. `supabase-setup.sql` の中身を全部貼って **Run**
3. 最後に出る表が3行とも `rowsecurity = true` になっていることを確認

`true` になっていない場合は先に進まないでください。名簿が誰でも読める状態です。

## 3. メンバーのアカウントを作る

1. 左メニュー **Authentication → Users → Add user**
2. **Create new user** を選び、メールアドレスとパスワードを入力
3. **Auto Confirm User** に必ずチェック（確認メールを飛ばさずに使えるようになります）
4. 使う人数ぶん繰り返す

表示名を出したいときは、ユーザーを開いて `User Metadata` に `{"name": "上京"}` を入れてください。入れなければメールアドレスの @ より前が表示されます。

新規登録を勝手にされないよう、**Authentication → Sign In / Providers → Email** の **Allow new users to sign up** は **オフ**にしておいてください。

## 4. 接続先を index.html に書く

1. 左メニュー **Project Settings → Data API**
2. `Project URL` と `anon public` キーをコピー
3. `index.html` を開き、いちばん上の CFG を書き換える

```js
const CFG = {
  url: 'https://xxxxxxxx.supabase.co',
  key: 'eyJhbGciOi...'
};
```

`anon public` キーは公開されて問題ないキーです。手順2でRLSを有効にしたので、このキー単体では何も読めません。**`service_role` キーは絶対に貼らないでください。**あれを貼るとRLSが素通しになります。

## 5. GitHub に上げる

```bash
mkdir medixor-pdca && cd medixor-pdca
git init

# 名簿を絶対にコミットしないための保険
printf 'medixor-leads.json\n*.xlsx\n' > .gitignore

cp /path/to/index.html .
cp /path/to/supabase-setup.sql .
git add .
git status          # ← medixor-leads.json が出ていないことを確認
git commit -m "Medixor 営業PDCA"
git branch -M main
git remote add origin https://github.com/<ユーザー名>/medixor-pdca.git
git push -u origin main
```

`git status` に `medixor-leads.json` が出ていたら、push せずに `.gitignore` を作り直してください。

## 6. GitHub Pages を有効にする

1. リポジトリ → **Settings → Pages**
2. Source: **Deploy from a branch** / Branch: **main** / フォルダ: **/ (root)** → Save
3. 1〜2分で `https://<ユーザー名>.github.io/medixor-pdca/` が開きます

## 7. 名簿を取り込む（初回だけ、1人がやれば十分）

1. 公開されたURLを開いて、手順3で作ったアカウントでログイン
2. **計画タブ → 設定 → 名簿を取り込む（初回のみ）**
3. `medixor-leads.json` を選ぶ
4. 「258件を取り込みました」と出たら完了

以降、他のメンバーは URL を開いてログインするだけで同じ名簿と進捗が見えます。

---

## 使うときの挙動

- 進捗は操作するたび自動保存されます。画面右上に「保存済み HH:MM」と出ます
- 他のメンバーの操作は **20秒ごと**に自動で反映されます。すぐ見たいときは設定の「いま同期する」
- 同じ相手を2人が同時に触った場合は、**あとから保存したほうが残ります**。担当を分けて動いてください
- 「保存できていません」と赤く出たら通信の問題です。その状態で画面を閉じると、その操作は消えます

## 修正したくなったら

`index.html` を直して push するだけです。名簿と進捗は Supabase 側にあるので消えません。

## 後回しにしている宿題

- 会社名・差出人が `【会社名】【担当者名】` のまま（計画タブの設定で入れれば全通に反映）
- 誰がどの相手を担当するかの割り当て機能がまだない。いまは口頭で分けてください
- 送信前に、薬機法の観点で文面とリスト運用を専門家に通すこと。計画タブの最上段に出しています
