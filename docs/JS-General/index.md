# メソッドの一覧と説明
※`id`には`train.id()`が必ず入ります。

!!! warning "警告"
    これらはMixin(ミックスイン)を用いた少し強引な実装であり、異常動作やクラッシュの潜在的な危険性があるかもしれません。
!!! warning "マルチプレイでの導入について"
    マルチプレイ環境で速度制御メソッドを用いる際、異常な列車の挙動になりかねませんので、十分テストをしてください。**問題がある場合は、使用しないでください。**
| メソッド名 | 説明 |
| - | - |
| `train.setSpeed(long id, float speed): void` | 速度を変更します。単位はm/tickです。|
| `train.setRailSpeed(long id, float speed): void` | レール速度を設定します。単位はm/tickです。|
| `train.getBlockDistance(long id): void` | 前方列車までの閉塞単位の距離を取得するようサーバーに送信します。結果は、`train.getServerResponse(long id, "getBlockDistance")`を使用することで取得できます。|
| `train.sendRequestToServer(long id, String requestType): void` | idとrequestTypeをサーバーに送信します。結果は、`train.getServerResponse(long id, String [先程のrequestType])`を使用することで取得できます。|
| `train.sendRequestToServer(long id, String requestType, long body): void` | idとrequestType、さらにbodyをサーバーに送信します。bodyはlong型でなければなりません。|
| `train.sendRequestToServer(long id, String requestType, String body): void` | idとrequestType、さらにbodyをサーバーに送信します。bodyはString型でなければなりません。|
| `train.getServerResponse(long id, String responseType): Object` | サーバーへ送信したリクエストの結果を返します。原則、`responseType`は`requestType`と共通になります。 |

## `requestType`の一覧と説明
| タイプ名 | 説明 |
| - | - |
| `"getBlockDistance"` | 前方列車までの閉塞単位の距離を取得します。|
| `"getNextPlatformIndex"` | 次駅までの閉塞単位の距離を取得します。|
| `setRouteIdToServer` | routeIdをサーバーに送信します。|