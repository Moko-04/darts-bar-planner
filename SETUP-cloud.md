# クラウド同期（Googleログイン）の設定手順

図面を別のPCでも開けるようにするための、初回だけ必要な設定です。所要 10分程度、費用は無料枠の範囲で収まります。

---

## 1. Firebase プロジェクトを作る

1. https://console.firebase.google.com/ を開き、会社のGoogleアカウントでログイン
2. 「プロジェクトを追加」→ 名前は `darts-bar-planner` など任意 → 続行
3. Google アナリティクスは **オフでOK** → プロジェクトを作成

## 2. ウェブアプリを登録して firebaseConfig を取得

1. プロジェクトのトップで **ウェブアイコン `</>`** をクリック
2. アプリのニックネーム（例：`planner`）を入れて「アプリを登録」
3. 表示される `firebaseConfig` の **`{ ... }` の中身をコピー**しておく

```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "xxx.firebaseapp.com",
  projectId: "xxx",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

> この値は公開されても問題ない種類の情報です。実際のアクセス制御は手順4のルールで行います。

## 3. Google ログインを有効にする

1. 左メニュー「構築」→ **Authentication** →「始める」
2. 「Sign-in method」タブ → **Google** を選択 → 有効にする → 保存
3. 「Settings」タブ →「承認済みドメイン」に次を追加
   - `moko-04.github.io`

## 4. Firestore を作成してルールを設定

1. 左メニュー「構築」→ **Firestore Database** →「データベースの作成」
2. ロケーションは `asia-northeast1（東京）` → **本番環境モード**で開始
3. 「ルール」タブを開き、内容を次に置き換えて「公開」

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // 自分の物件は自分だけが読み書きできる
    match /users/{uid}/projects/{projectId} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }

    // 共有リンク：リンクを知っている人は読める。作成・削除は本人のみ
    match /shares/{shareId} {
      allow read: if true;
      allow create: if request.auth != null
                    && request.resource.data.ownerUid == request.auth.uid;
      allow update, delete: if request.auth != null
                    && resource.data.ownerUid == request.auth.uid;
    }
  }
}
```

> 社内メンバーだけに限定したい場合は、`allow read, write` の条件に
> `&& request.auth.token.email.matches('.*@dart-ace[.]com')` を足してください。

## 5. アプリに接続設定を貼り付ける

1. https://moko-04.github.io/darts-bar-planner/restaurant-planner-v2.html を開く
2. 右上の **⚙** を押す
3. 手順2でコピーした `firebaseConfig` を貼り付けて「保存して接続」
4. 「Googleでログイン」を押してログイン

これで完了です。以降その端末では設定不要になります。

### 全PCで設定を省きたい場合

手順2の config を教えていただければ、アプリ本体に埋め込んでコミットします。
そうすると、どのPCでも **開いてログインするだけ** で使えるようになります。

---

## 使い方

| やりたいこと | 操作 |
|---|---|
| 別のPCで続きを編集 | 同じGoogleアカウントでログイン → 最初の画面「☁ クラウドの物件」から開く |
| 相手にプランを送る | 編集画面の **🔗 共有リンクを作成** → 出てきたURLを送る |
| 送られたプランを見る | URLを開くだけ（受け取った側は自分のコピーとして自由に編集できます。元のプランは変わりません） |

- 編集内容は**このPC（ブラウザ内）とクラウドの両方に自動保存**されます
- 下絵のPDF画像も一緒に保存されます。容量が大きい場合は自動で圧縮し、それでも収まらない場合は下絵のみ除外して保存します（その旨が画面に表示されます）
- 共有リンクは**URLを知っている人なら誰でも開けます**。社外に渡らないようご注意ください

## 費用について

Firebase の無料枠（Spark プラン）は 1日あたり読み取り5万回・書き込み2万回、保存容量1GB です。
数人で図面を編集する用途であれば、無料枠を超えることはまずありません。
