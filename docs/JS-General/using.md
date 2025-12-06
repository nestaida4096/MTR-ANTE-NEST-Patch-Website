# スクリプトを用いた例
速度制御メソッドやサーバー通信機能を用いたスクリプトの例を紹介します。
!!! warning "動作保証について"
    以下の例で必ずしも動く保証はありません。修正が必要な場合もありますので、JavaScript経験者向けとなります。
## 段階的な速度制御を行う簡易的なATCのようなもの
```js
function create(ctx, state, train) {
	state.oldSpeed = train.speed()*3.6*20;
	state.first = true;
}


function render(ctx, state, train) {
	train.getBlockDistance(train.id());
	let speed = train.speed()*3.6*20;
	let stoppingIndex = train.getServerResponse(train.id(), "getBlockDistance");
	train.sendRequestToServer(train.id(), "getNextPlatformIndex");
	let railIndex = train.getRailIndex(train.railProgress(), true);
	let nextPlatformIndex = train.getServerResponse(train.id(), "getNextPlatformIndex");
	let nextPlatformIndexSubRailIndex = train.getServerResponse(train.id(), "getNextPlatformIndex") - railIndex;
	let safeDistance = stoppingIndex - railIndex;
	ctx.setDebugInfo("NextStoppingIndex", stoppingIndex);
	ctx.setDebugInfo("railIndex", railIndex);
	ctx.setDebugInfo("前方車両(赤信号や駅)までの閉塞単位距離", safeDistance);
	ctx.setDebugInfo("駅までの閉塞単位距離", nextPlatformIndexSubRailIndex);
	signalSpeed = maxSpeed;
	if (safeDistance <= 1) signalSpeed = 25;
	else if (safeDistance <= 2) signalSpeed = 40;
	else if(safeDistance <= 3) signalSpeed = 55;
	else if(safeDistance <= 4) signalSpeed = 70;
	ctx.setDebugInfo("信号速度", signalSpeed);
	train.setRailSpeed(train.id(), signalSpeed/3.6/20);
	if (speed > signalSpeed && train.isCurrentlyManual()) {
		train.setSpeed(train.id(), speed/3.6/20 - (0.0005 * (speed / state.oldSpeed)));
	}
	state.oldSpeed = train.speed()*3.6*20;
}
```