# ブラック or ホワイト？企業体験クイズ

労働法・ブラック企業知識を問うクイズゲームです。  
単一HTMLファイルで動作し、Firebaseでリアルタイムランキングを管理します。

---

## ファイル構成

```
bw-quiz.html   ← これ1つをサーバーに置くだけで動く
README.md      ← このファイル
```

---

## 初期設定手順

### Step 1 ｜ Firebase プロジェクトを作る

1. [https://console.firebase.google.com](https://console.firebase.google.com) を開く
2. **「プロジェクトを追加」** をクリック
3. プロジェクト名を入力（例: `bw-quiz`）→ 続行
4. Google アナリティクスは「**有効にする**」を選択（後でGA4と連携できる）
5. **「プロジェクトを作成」** をクリック → 完了まで待つ

---

### Step 2 ｜ Firestore Database を有効にする

1. 左メニューの **「Firestore Database」** をクリック
2. **「データベースを作成」** をクリック
3. ロケーションを選択（`asia-northeast1`（東京）推奨）
4. セキュリティルールは **「テストモードで開始」** を選択
   - ⚠️ テストモードは30日間のみ誰でも読み書き可能。本番運用前にルールを設定すること
5. **「有効にする」** をクリック

#### Firestore セキュリティルール（推奨設定）

テストモードの30日が過ぎる前に、以下のルールを設定してください。  
Firestore コンソール → **「ルール」タブ** に貼り付けて「公開」をクリック。

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rankings/{userId} {
      // 誰でも読める（ランキング表示用）
      allow read: if true;
      // 書き込みは自分のドキュメントのみ
      allow write: if true; // 簡易版。認証導入後は条件を絞る
    }
  }
}
```

---

### Step 3 ｜ Firebase の設定値をコピーする

1. Firebase コンソールのトップページで **歯車アイコン → 「プロジェクトの設定」** をクリック
2. 下にスクロールして **「マイアプリ」** → **`</>`（ウェブ）アイコン** をクリック
3. アプリのニックネームを入力（例: `bw-quiz-web`）→ **「アプリを登録」**
4. 表示された `firebaseConfig` の値をコピーする

```js
// コピーする値の例（実際の値は各自異なる）
const firebaseConfig = {
  apiKey: "AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  authDomain: "bw-quiz-12345.firebaseapp.com",
  projectId: "bw-quiz-12345",
  storageBucket: "bw-quiz-12345.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abcdefabcdefabcdef",
};
```

5. `bw-quiz.html` をテキストエディタで開き、以下の箇所を差し替える

```html
<!-- bw-quiz.html 内のこの部分を差し替える -->
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",          ← ここ
  authDomain: "YOUR_PROJECT...",   ← ここ
  projectId: "YOUR_PROJECT_ID",    ← ここ
  storageBucket: "YOUR_PROJECT...",← ここ
  messagingSenderId: "YOUR_...",   ← ここ
  appId: "YOUR_APP_ID",            ← ここ
};
```

---

### Step 4 ｜ Google Analytics 4（GA4）を設定する

#### GA4 プロパティの作成

1. [https://analytics.google.com](https://analytics.google.com) を開く
2. **「管理」（左下の歯車）→「プロパティを作成」** をクリック
3. プロパティ名を入力 → タイムゾーンと通貨を設定 → **「次へ」**
4. ビジネスの規模・目標を選択 → **「作成」**
5. **「ウェブ」** を選択 → サイトのURLとストリーム名を入力
6. **「ストリームを作成」** をクリック

#### 測定 ID をコピーして貼り付ける

1. 作成したウェブストリームの詳細画面で **「測定 ID」** をコピー（`G-XXXXXXXXXX` 形式）
2. `bw-quiz.html` 内の以下2箇所を差し替える

```html
<!-- 1箇所目: scriptタグのsrc -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>

<!-- 2箇所目: gtag config -->
gtag('config', 'G-XXXXXXXXXX');
```

`G-XXXXXXXXXX` を自分の測定IDに書き換えるだけでOK。

---

### Step 5 ｜ HTMLファイルを公開する

ローカルでも動きますが、ランキングをみんなで共有するにはウェブサーバーに置く必要があります。

#### おすすめの無料ホスティング

| サービス | 難易度 | 特徴 |
|---|---|---|
| **Firebase Hosting** | ★★☆ | Firebase と同じコンソールで管理できる |
| **GitHub Pages** | ★★☆ | GitHubにpushするだけ。無料 |
| **Netlify** | ★☆☆ | HTMLをドラッグ&ドロップするだけ。最簡単 |

#### Netlify での公開手順（最速）

1. [https://app.netlify.com](https://app.netlify.com) にアクセス → 無料登録
2. ダッシュボードの **「Sites」タブ** を開く
3. `bw-quiz.html` をそのまま**ドラッグ&ドロップ**
4. 数秒でURLが発行される（例: `https://amazing-quiz-abc123.netlify.app`）

---

## ゲームの使い方

### プレイヤー向け

1. ニックネームを入力して **「SET」** を押す
2. ゲームを選ぶ
   - **ブラック企業診断**: ブラック/ホワイトを○×で判定（難易度選択あり）
   - **求人広告のウソを見抜け**: 求人票の危険箇所を4択で選ぶ
3. ゲーム終了後、スコアがランキングに自動保存される

### 管理者向け（問題の追加・削除）

1. ホーム画面の **「問題管理」** ボタンを押す
2. パスワードを入力（デフォルト: **`0308`**）
3. 問題の追加・削除が可能

#### パスワードの変更方法

`bw-quiz.html` 内の以下の行を編集する。

```js
const SETTINGS_PASSWORD = "0308"; // ← ここを変更
```

---

## Google Analytics で計測できるイベント

| イベント名 | タイミング |
|---|---|
| `page_view` | ホーム画面を表示したとき |
| `set_nickname` | ニックネームをセットしたとき |
| `quiz_start` | クイズを開始したとき |
| `answer` | 問題に回答したとき（正誤・問題IDも送信） |
| `quiz_complete` | クイズを完了したとき（スコア・合計問題数も送信） |

GA4 コンソール → **「レポート」→「エンゲージメント」→「イベント」** から確認できます。

---

## トラブルシューティング

### ランキングが表示されない

- Firebase の `firebaseConfig` が正しく設定されているか確認
- Firestore の **セキュリティルール** が `allow read: if true;` になっているか確認
- ブラウザの開発者ツール（F12）の「Console」タブでエラーを確認

### スコアが保存されない

- Firestore の **セキュリティルール** が `allow write: if true;` になっているか確認
- テストモードの30日期限が切れていないか確認

### ローカルで開くとランキングが動かない

Firebase は `file://` プロトコルでは動作しません。  
VS Code の **Live Server** 拡張機能などでローカルサーバーを立てるか、Netlify などにデプロイしてください。

---

## 無料枠について（Firebase）

Firebaseの無料枠（Spark プラン）の主な制限：

| 項目 | 無料枠 |
|---|---|
| Firestore 読み取り | 50,000回/日 |
| Firestore 書き込み | 20,000回/日 |
| ホスティング転送量 | 10GB/月 |

このゲームは読み書きを最小限に設計しているので、通常の利用では無料枠を超えません。
