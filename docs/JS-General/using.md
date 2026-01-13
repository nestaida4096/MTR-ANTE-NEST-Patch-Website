# スクリプトを用いた例
速度制御メソッドやサーバー通信機能を用いたスクリプトの例を紹介します。
!!! warning "動作保証について"
    以下の例で必ずしも動く保証はありません。修正が必要な場合もありますので、JavaScript経験者向けとなります。

## 段階的な速度制御を行う簡易的なATC
```js
function create(ctx, state, train) {
	state.oldSpeed = train.speed()*3.6*20;
	state.first = true;
	state.sendedRouteId = false;
}


function render(ctx, state, train) {
	train.getBlockDistance(train.id());
	let stoppingIndex = train.getServerResponse(train.id(), "getBlockDistance");
	
	if (!state.sendedRouteId) {
		try {
			let routeId = train.getThisRoutePlatforms().get(0).route.id;
			train.sendRequestToServer(train.id(), "setRouteIdToServer", routeId);
			state.sendedRouteId = true;
			state.routeId = routeId;

		} catch {}
	}
	let speed = train.speed()*3.6*20;
	train.sendRequestToServer(train.id(), "getNextPlatformIndex");
	let railIndex = train.getRailIndex(train.railProgress(), true);
	let nextPlatformIndex = train.getServerResponse(train.id(), "getNextPlatformIndex");
	let nextPlatformIndexSubRailIndex = train.getServerResponse(train.id(), "getNextPlatformIndex") - railIndex;
	let safeDistance = stoppingIndex - railIndex;
	ctx.setDebugInfo("NextStoppingIndex", stoppingIndex);
	ctx.setDebugInfo("railIndex", railIndex);
	ctx.setDebugInfo("TEMP", state.routeId);
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

## さらにきめ細やかな速度制御を行う簡易的なATC

!!! warning "負荷について"
    このスクリプトでは、列車から離れても処理が動き続けるよう設計しています。そのため、ゲームが非常に重くなる場合がありますのでご注意ください。

```js
importPackage(java.util.concurrent);

let activeLCDs = [];
let asyncPool = null;
	
function create(ctx, state, train) {
	state.oldSpeed = train.speed()*3.6*20;
	state.first = true;
	state.sendedRouteId = false;
	state.oldSafeDistance = [];
	let onLoaded = false;
	for (let i = 0; i < activeLCDs.length; i++) {
		let selected = activeLCDs[i];
		if (selected["trainId"] == train.id()) {
			onLoaded = true;
			break;
		}
	}
	activeLCDs.push({ trainId: train.id(), ctx: ctx, state: state, blockEntity: train });
    

    if (Resources.ANTE_FLAG && asyncPool === null) {
        asyncPool = Executors.newScheduledThreadPool(1);
        asyncPool.scheduleAtFixedRate(new java.lang.Runnable({
            run: () => {
                for (let i = 0; i < activeLCDs.length; i++) {
                    let entry = activeLCDs[i];
                    try {
                        tick(entry.ctx, entry.state, entry.blockEntity);
                    } catch (e) {
                      print("描画中に例外が発生しました: " + e);
                      print("描画中に例外が発生しました: " + e.stack);
					  print("=== DISPOSE ===")
					  close(entry.ctx, entry.state, entry.blockEntity);
                    }
                }
            }
        }), 0, 1000 / 12, TimeUnit.MILLISECONDS);
    }
}

function close(ctx, state, blockEntity) {
    print("Dispose")
    for (let i = activeLCDs.length - 1; i >= 0; i--) {
        if (activeLCDs[i].blockEntity.equals(blockEntity)) {
            activeLCDs.splice(i, 1);
            break;
        }
    }

    if (activeLCDs.length === 0 && asyncPool !== null) {
        asyncPool.shutdown();
		print("Closed asyncPool");
        asyncPool = null;
    }
    
}

function checkLast10Values(arr) {
  if (!Array.isArray(arr) || arr.length < 10) {
    return false; // 10個未満なら判定不可
  }

  const last10 = arr.slice(arr.length - 10);

  // 出現回数カウント
  const countMap = {};

  for (let i = 0; i < last10.length; i++) {
    const v = last10[i];
    if (countMap[v] === undefined) {
      countMap[v] = 1;
    } else {
      countMap[v]++;
    }
  }

  // 最頻値を探す
  let normalValue = null;
  let maxCount = 0;

  const keys = Object.keys(countMap);
  for (let i = 0; i < keys.length; i++) {
    const value = keys[i];
    const count = countMap[value];
    if (count > maxCount) {
      maxCount = count;
      normalValue = Number(value);
    }
  }

  // 最頻値以外が含まれていれば異常
  for (let i = 0; i < last10.length; i++) {
    if (last10[i] !== normalValue) {
      return false;
    }
  }

  return true;
}



function tick(ctx, state, train) {
	train.getBlockDistance(train.id());
	let stoppingIndex = train.getServerResponse(train.id(), "getBlockDistance");
	
	if (!state.sendedRouteId) {
		try {
			let routeId = train.getThisRoutePlatforms().get(0).route.id;
			train.sendRequestToServer(train.id(), "setRouteIdToServer", routeId);
			state.sendedRouteId = true;
			state.routeId = routeId;

		} catch {}
	}
	let speed = train.speed()*3.6*20;
	train.sendRequestToServer(train.id(), "getNextPlatformIndex");
	let railIndex = train.getRailIndex(train.railProgress(), true);
	let nextPlatformIndex = train.getServerResponse(train.id(), "getNextPlatformIndex");
	let nextPlatformIndexSubRailIndex = train.getServerResponse(train.id(), "getNextPlatformIndex") - railIndex;
	let safeDistance = stoppingIndex - railIndex;

	ctx.setDebugInfo("NextStoppingIndex", stoppingIndex);
	ctx.setDebugInfo("railIndex", railIndex);
	ctx.setDebugInfo("前方車両(赤信号や駅)までの閉塞単位距離", safeDistance);
	ctx.setDebugInfo("駅までの閉塞単位距離", nextPlatformIndexSubRailIndex);
	let signalSpeed;
	if (safeDistance <= 2) signalSpeed = 20;
	else if (safeDistance <= 3) signalSpeed = 30;
	else if (safeDistance <= 5) signalSpeed = 40;
	else if (safeDistance <= 7) signalSpeed = 50;
	else if (safeDistance <= 9) signalSpeed = 60;
	else if (safeDistance <= 11) signalSpeed = 70;
	else if(safeDistance <= 13) signalSpeed = 80;
	else if(safeDistance <= 15) signalSpeed = 90;
	else if(safeDistance <= 17) signalSpeed = 100;
	else if(safeDistance <= 19) signalSpeed = 110;
	else if(safeDistance <= 21) signalSpeed = 120;
	else if(safeDistance <= 23) signalSpeed = 130;
	else signalSpeed = maxSpeed;
	ctx.setDebugInfo("信号速度", signalSpeed);
	state.signalSpeed = signalSpeed;
	train.setRailSpeed(train.id(), state.signalSpeed/3.6/20);
	state.oldSpeed = train.speed()*3.6*20;
	state.oldSafeDistance.push(safeDistance);
}
```

## レールによる制限速度にも対応する簡易的なATC／加速度のカスタマイズ

!!! warning "負荷について"
    このスクリプトでは、列車から離れても処理が動き続けるよう設計しています。そのため、ゲームが非常に重くなる場合がありますのでご注意ください。

```js

importPackage(java.util.concurrent);

let activeLCDs = [];
let asyncPool = null;
	

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


function create(ctx, state, train) {
	state.oldSpeed = train.speed()*3.6*20;
  train.setAcceleration(train.id(), calcInternalValue(3.3));
	state.first = true;
	state.sendedRouteId = false;
	state.oldSafeDistance = [];
	let onLoaded = false;
  
  
  state.oldPos = Vector3f.ZERO;
  state.trainPos = Vector3f.ZERO;
  
	for (let i = 0; i < activeLCDs.length; i++) {
		let selected = activeLCDs[i];
		if (selected["trainId"] == train.id()) {
			onLoaded = true;
			break;
		}
	}
  for (let i = 0; i < activeLCDs.length; i++) {
    let trainId = activeLCDs["trainId"];
    if (train.id() == trainId) {
      activeLCDs.splice(i, 1);
    }
  }
	activeLCDs.push({ trainId: train.id(), ctx: ctx, state: state, blockEntity: train });
    

    if (Resources.ANTE_FLAG && asyncPool === null) {
        asyncPool = Executors.newScheduledThreadPool(2);
        asyncPool.scheduleAtFixedRate(new java.lang.Runnable({
            run: () => {
                for (let i = 0; i < activeLCDs.length; i++) {
                    let entry = activeLCDs[i];
                    try {
                        tickCustomSpeed(entry.ctx, entry.state, entry.blockEntity);
                    } catch (e) {
                      print("描画中に例外が発生しました: " + e);
                      print("描画中に例外が発生しました: " + e.stack);
                      print("=== DISPOSE ===")
                      close(entry.ctx, entry.state, entry.blockEntity);
                    }
                }
            }
        }), 0, 1000 / 12, TimeUnit.MILLISECONDS);
        
        asyncPool.scheduleAtFixedRate(new java.lang.Runnable({
            run: () => {
                for (let i = 0; i < activeLCDs.length; i++) {
                    let entry = activeLCDs[i];
                    try {
                        atc_point_detect(entry.ctx, entry.state, entry.blockEntity);
                    } catch (e) {
                      print("描画中に例外が発生しました: " + e);
                      print("描画中に例外が発生しました: " + e.stack);
                      print("=== DISPOSE ===")
                      close(entry.ctx, entry.state, entry.blockEntity);
                    }
                }
            }
        }), 0, 1000 / 12, TimeUnit.MILLISECONDS);
    }
}

function atc_point_detect(ctx, state, train) {
  try {
    let railTypes = {
      "WOODEN": 20,
      "STONE": 40,
      "EMERALD": 60,
      "IRON": 80,
      "OBSIDIAN": 120,
      "BLAZE": 160,
      "QUARTZ": 200,
      "DIAMOND": 300,
      "PLATFORM": 80,
      "SIDING": 40,
      "TURN_BACK": 80,
      "CABLE_CAR": 30,
      "CABLE_CAR_STATION": 2,
      "RUNWAY": 300,
      "AIRPLANE_DUMMY": 900,
      "NONE": 20
    }
    let railIndex = train.getRailIndex(train.railProgress(), true);
    let path = train.path().get(railIndex);
    let railType = String(path.rail.railType);
    if (railType.startsWith("P")) {
      let railMaxSpeed = Number(String(railType).substring(1));
      state.atcMaxSpeed = railMaxSpeed;
    } else {
      state.atcMaxSpeed = railTypes[railType];
    }
    
    ctx.setDebugInfo("railType", railType);
  } catch(e) {
    ctx.setDebugInfo("error_atc_point", e + e.stack);
  }
}

function close(ctx, state, blockEntity) {
    print("Dispose");
    for (let i = activeLCDs.length - 1; i >= 0; i--) {
        if (activeLCDs[i].blockEntity.equals(blockEntity)) {
            activeLCDs.splice(i, 1);
            break;
        }
    }

    if (activeLCDs.length === 0 && asyncPool !== null) {
        asyncPool.shutdown();
		print("Closed asyncPool");
        asyncPool = null;
    }
    
}

function checkLast10Values(arr) {
  if (!Array.isArray(arr) || arr.length < 10) {
    return false; // 10個未満なら判定不可
  }

  const last10 = arr.slice(arr.length - 10);

  // 出現回数カウント
  const countMap = {};

  for (let i = 0; i < last10.length; i++) {
    const v = last10[i];
    if (countMap[v] === undefined) {
      countMap[v] = 1;
    } else {
      countMap[v]++;
    }
  }

  // 最頻値を探す
  let normalValue = null;
  let maxCount = 0;

  const keys = Object.keys(countMap);
  for (let i = 0; i < keys.length; i++) {
    const value = keys[i];
    const count = countMap[value];
    if (count > maxCount) {
      maxCount = count;
      normalValue = Number(value);
    }
  }

  // 最頻値以外が含まれていれば異常
  for (let i = 0; i < last10.length; i++) {
    if (last10[i] !== normalValue) {
      return false;
    }
  }

  return true;
}



function tickCustomSpeed(ctx, state, train) {
  state.atc_stopped = false;
  try {
    let railIndex = train.getRailIndex(train.railProgress(), true);
    train.getBlockDistance(train.id());
    let stoppingIndex = train.getServerResponse(train.id(), "getBlockDistance");
    
    if (!state.sendedRouteId) {
      try {
        let routeId = train.getThisRoutePlatforms().get(0).route.id;
        train.sendRequestToServer(train.id(), "setRouteIdToServer", routeId);
        state.sendedRouteId = true;
        state.routeId = routeId;

      } catch {}
    }
    let speed = train.speed()*3.6*20;
    train.sendRequestToServer(train.id(), "getNextPlatformIndex");
    let nextPlatformIndex = train.getServerResponse(train.id(), "getNextPlatformIndex");
    let nextPlatformIndexSubRailIndex = train.getServerResponse(train.id(), "getNextPlatformIndex") - railIndex;
    let safeDistance = stoppingIndex - railIndex;

    ctx.setDebugInfo("NextStoppingIndex", stoppingIndex);
    ctx.setDebugInfo("railIndex", railIndex);
    ctx.setDebugInfo("前方車両(赤信号や駅)までの閉塞単位距離", safeDistance);
    ctx.setDebugInfo("駅までの閉塞単位距離", nextPlatformIndexSubRailIndex);

    if (safeDistance == nextPlatformIndexSubRailIndex && train.getThisRoutePlatformsNextIndex() == train.getThisRoutePlatforms().size()-1) {
      state.station_safe_stopping = true;
    } else {
      state.station_safe_stopping = false;
    }

    let signalSpeed;
    if (safeDistance != nextPlatformIndexSubRailIndex || state.station_safe_stopping) {
      if (safeDistance < 1) signalSpeed = 25;
      else if (safeDistance <= 2) signalSpeed = 40;
      else if (safeDistance <= 3) signalSpeed = 45;
      else if (safeDistance <= 5) signalSpeed = 50;
      else if (safeDistance <= 7) signalSpeed = 60;
      else if (safeDistance <= 9) signalSpeed = 70;
      else if (safeDistance <= 11) signalSpeed = 80;
      else if(safeDistance <= 13) signalSpeed = 90;
      else if(safeDistance <= 15) signalSpeed = 100;
      else if(safeDistance <= 17) signalSpeed = 110;
      else if(safeDistance <= 19) signalSpeed = 120;
      else signalSpeed = maxSpeed;
    } else signalSpeed = maxSpeed;
    if (signalSpeed > maxSpeed) {
      signalSpeed = maxSpeed;
    }
    if (signalSpeed > state.atcMaxSpeed) {
      signalSpeed = state.atcMaxSpeed;
    }
    ctx.setDebugInfo("信号速度", signalSpeed);
    if (signalSpeed != state.signalSpeed) {
      train.setRailSpeed(train.id(), signalSpeed/3.6/20);
      let sound = TickableSound(Resources.id("mtr:atc_speed_changed"));
      sound.setData(1, 1, train.lastCarPosition[0]);
      sound.play();
      let sound2 = TickableSound(Resources.id("mtr:atc_speed_changed"));
      sound2.setData(1, 1, train.lastCarPosition[train.trainCars()-1]);
      sound2.play();
      
      state.signalSpeed = signalSpeed;
    }
    state.oldSpeed = train.speed()*3.6*20;
    state.oldSafeDistance.push(safeDistance);

    train.getServerResponse(train.id(), "")
  } catch {
    state.atc_stopped = true;
  }
}
```