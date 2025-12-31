# サーバー側スクリプト（ServerScripting）について
- サーバー側スクリプトでは、サーバー側で任意のコードが実行できます。
- このページでは、実行できるまでの設定方法を説明します。
!!! danger "危険性について"
    この機能は、サーバー側でコードが実行でき、**利用できるJavaパッケージやJavaクラスには制限がありません。**

    つまり、使い方によってはサーバー内のファイルを外部サーバーに送信できます。

    **そのようなサーバー管理者が不審に思われる可能性のある使い方は、サーバーのガイドラインを確認し、問題がないか今一度確認してください。**

    **サーバーに何かありましても、制作者は責任を負いません。自己責任でご利用ください。**

## コンフィグから有効にする
1. `config/mtr-ante-nest-patch.json`を開いてください。
2. 導入後編集していない場合、以下の状態になっていると思います。
```json
{
  "serverScriptingStatus": false
}
```
3. 以下の通り値を変更してください。
    1. `serverScriptingStatus`の値を`true`に設定
4. コンフィグデータを保存してください。
5. サーバーを再起動してください。

これでサーバー側スクリプト機能は利用可能になりました！
## 実行する
1. クライアント側でゲームを起動します。
2. 以下のコードを書き、`mtr:scripts/render.js`に保存し、`mtr_custom_resources.json`に登録します（登録方法は説明しません）。
```js
function render(ctx, state, train) {
    train.runServerScript(train.id(), Resources.readString(Resources.id("scripts/server.js")));
}
```
3. 以下のコードを書き、`mtr:scripts/server.js`に保存します。
```js
importPackage(java.lang);

System.out.println("serverScript is Working!");
```
4. また、上記のファイルをサーバー側の`serverScripts/server.js`にも保存します。
5. これでゲームをリロード(F3+T)すれば、サーバーコンソールに`serverScript is Working!`が連続して表示されます。
## 最後に
- この機能によりサーバーが破壊されても、**私は一切の責任を負いません。**
- それを理解していただいたうえでご利用ください。