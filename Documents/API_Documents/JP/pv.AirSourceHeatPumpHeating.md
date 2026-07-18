## pv.AirSourceHeatPumpHeating(filename='equipment_spec.xlsx', sheet_name='AirSourceHeatPumpHeating')
空冷ヒートポンプの温水モード（暖房運転）。性能曲線（外気乾球温度×負荷率のCOP表）に基づく運転点の計算
  
### モデル化の前提
- 除霜運転による効率低下はCOP表に織り込む前提とし、明示的な除霜補正は行わない（COP表の値は負荷率・外気温に対して単調でなくてもよいため、着霜域の凹みも表現可能）
- 最大加熱能力は外気温によらず定格一定とする（実機では低外気温時に最大能力が低下するため、低外気温域では能力を過大評価し得る）
- COP表の外気温範囲外の入力は表の端の値を参照する（外挿はしない。Chillerの冷却水温度軸・冷房版 AirSourceHeatPump と同じ扱い、flag 6/7）
- 入力ファイル例の定格消費電力は、暖房標準定格条件相当（外気乾球温度7℃・負荷率1.0）のCOP表補間値（COP=3.36）と整合する値としている（冷房版 AirSourceHeatPump の入力ファイル例と同じ流儀）
  
入力ファイル例（外気温は上から下へ昇順、負荷率は左から右へ昇順であること）  
  
|rated hot water outlet temp. ['C]|rated hot water inlet temp. ['C]|rated hot water flow [m3/min]|rated power [kW]|hot water pressure loss coefficient [kPa/(m3/min)^2]|  |
| ---- | ---- | ---- | ---- | ---- | ---- |
|45|40|0.215|22.321180555555557|13.9|  |
|  |  |  |  |  |  |
|outdoor air temp.-load factor|0.2|0.4|0.6|0.8|1|
|-15|1.9|2.1|2.2|2.1|1.9|
|-10|2.2|2.4|2.5|2.4|2.2|
|-5|2.5|2.8|2.9|2.8|2.5|
|0|2.8|3.1|3.2|3.1|2.8|
|5|3.2|3.5|3.6|3.5|3.2|
|10|3.6|4.0|4.1|3.9|3.6|
|15|4.0|4.4|4.5|4.3|4.0|
  
### Parameters:
|  name  |  type  | description |
| ---- | ---- | ---- |
|filename|str|機器情報ファイル名（エクセルファイル）|
|sheet_name|str|対象機器情報が記載されたシート名|
|q_h_d|float|定格加熱能力 [kW]|
|pw_d|float|定格消費電力 [kW]|
|tin_h_d|float|定格温水入口温度 ['C]|
|tout_h_d|float|定格温水出口温度 ['C]|
|tin_h|float|温水入口温度 ['C]|
|tout_h|float|温水出口温度 ['C]|
|tout_h_sp|float|温水出口温度設定値 ['C]|
|g_h|float|温水流量 [m3/min]|
|q_h|float|加熱熱量 [kW]|
|pw|float|消費電力 [kW]|
|pl|float|部分負荷率 (0.0~1.0)|
|cop|float|COP|
|tdb|float|外気乾球温度 ['C]|
|kr_h|float|温水圧力損失係数 [kPa/(m3/min)^2]|
|dp_h|float|機器による温水圧力損失 [kPa]|
|flag|int|運転状態フラグ（下表）|
  
|  flag  | description |
| ---- | ---- |
|0|正常|
|1|要求熱量が定格能力超過（温水出口温度が設定値より低下）|
|2|負荷率がCOP表の上限を超過|
|3|負荷率がCOP表の下限未満|
|4|加熱熱量ゼロ（停止）|
|5|加熱熱量が負（tout_h_sp < tin_h）|
|6|外気温度がCOP表の下限未満（下限値で参照）|
|7|外気温度がCOP表の上限超過（上限値で参照）|
  
## pv.AirSourceHeatPumpHeating.cal(tout_h_sp, tin_h, g_h, tdb)
温水出口温度設定、温水入口温度、温水流量、外気乾球温度に基づいて出口温度と消費電力を算出する。
また、温水における圧力損失も算出する。
COPはCOP表の補間値に対し、逆カルノーサイクル（暖房COP: (273.15+tout_h)/(tout_h-tdb)）に基づく定格温水出口温度との差の補正を行う。
COP表参照と補正は同じ有効外気温度で評価する。温度リフト（温水出口温度-外気温度）が小さくなると補正係数が発散するため、有効外気温度は（温水出口温度-5℃）で頭打ち（飽和）とする（5℃は発散防止のために仮に設定した暫定値）。
これにより外気温度が上がるほどCOPが上がり、飽和点以降は一定となる（単調性が保たれる）。
  
### returns:
なし（変数の値が更新される）
  
## サンプルコード  
```
import phyvac as pv

AHPH1 = pv.AirSourceHeatPumpHeating(filename='equipment_spec.xlsx', sheet_name='AirSourceHeatPumpHeating')  # AHPH1の定義
AHPH1.cal(tout_h_sp=45.0, tin_h=40.0, g_h=0.15, tdb=5.0)  # 運転条件の入力と計算
print(AHPH1.pw, AHPH1.pl, AHPH1.tout_h)  # 計算結果例のプリント
```
> phyvac: ver20260716  
> 14.734610512153344 0.6976744186046512 45.0
  
