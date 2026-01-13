# メソッドの一覧と説明
## 注意事項

!!! warning "警告"
    これらはMixin(ミックスイン)を用いた少し強引な実装であり、異常動作やクラッシュの潜在的な危険性があるかもしれません。
!!! warning "マルチプレイでの導入について"
    マルチプレイ環境で速度制御メソッドを用いる際、異常な列車の挙動になりかねませんので、十分テストをしてください。**問題がある場合は、使用しないでください。**

## メソッドの一覧と説明
※`id`には`train.id()`が入ります。

| メソッド名 | 説明 |
| - | - |
| `train.setSpeed(long id, float speed): void` | 速度を変更します。単位はm/tickです。|
| `setAcceleration(long id, float acceleration): void` | 加速度(減速度)を変更します。km/h/sからの値の変換は以下のコードを確認してください。|
```js
/**
 * 加速度（km/h/s）を内部値に変換する
 *
 * 前提:
 *  - 0.4 m/s^2 → 内部値 0.001
 *  - 比例関係と仮定
 *
 * @param {number} kmhPerSec  加速度 [km/h/s]
 * @returns {number} 内部値
 */
function calcInternalValue(kmhPerSec) {
  // km/h/s → m/s^2
  const ms2 = kmhPerSec * 1000 / 3600;

  // 基準値
  const baseMs2 = 0.4;
  const baseInternal = 0.001;

  return baseInternal * (ms2 / baseMs2);
}
```
| `train.setRailSpeed(long id, float speed): void` | レール速度を設定します。単位はm/tickです。|
| `train.getBlockDistance(long id): void` | 前方列車までの閉塞単位の距離を取得するようサーバーに送信します。結果は、`train.getServerResponse(long id, "getBlockDistance")`を使用することで取得できます。|
| `train.sendRequestToServer(long id, String requestType): void` | idとrequestTypeをサーバーに送信します。結果は、`train.getServerResponse(long id, String [先程のrequestType])`を使用することで取得できます。|
| `train.sendRequestToServer(long id, String requestType, long body): void` | idとrequestType、さらにbodyをサーバーに送信します。bodyはlong型でなければなりません。|
| `train.sendRequestToServer(long id, String requestType, String body): void` | idとrequestType、さらにbodyをサーバーに送信します。bodyはString型でなければなりません。|
| `train.getServerResponse(long id, String responseType): Object` | サーバーへ送信したリクエストの結果を返します。原則、`responseType`は`requestType`と共通になります。 |
| `train.runServerScript(long id, String scriptText): void` | 利用可能な場合、サーバー側で`scriptText`の内容を実行します。**危険なコードは実行しないようにしてください。** |

## `requestType`の一覧と説明
| タイプ名 | 説明 |
| - | - |
| `"getBlockDistance"` | 前方列車までの閉塞単位の距離を取得します。|
| `"getNextPlatformIndex"` | 次駅までの閉塞単位の距離を取得します。|
| `"setRouteIdToServer"` | routeIdをサーバーに送信します。|
| `"allTrainPositions"` | サーバー内にあるすべての列車のrailIndexを取得します。\n`Map<Long(Siding ID), Map<Long(Train ID), Integer(railIndex)>>`の状態で取得できます。|