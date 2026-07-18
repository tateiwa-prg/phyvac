## pv.AirSourceHeatPump(filename='equipment_spec.xlsx', sheet_name='AirSourceHeatPump')
冷凍機の性能曲線に基づく運転点の計算  
  
<img src="https://github.com/ShoheiMiyata/phyvac/assets/27459538/5ad1f231-fa98-4c6c-a09e-64674bdcc7f7" width=60%>  
  
入力ファイル例  
  
### Parameters:
|  name  |  type  | description |
| ---- | ---- | ---- |
|filename|str|機器情報ファイル名（エクセルファイル）|
|sheet_name|str|対象機器情報が記載されたシート名|
|q_ch_d|float|定格能力 [kW]|
|pw_d|float|定格消費電力 [kW]|
|tin_ch_d|float||
|tout_ch_d|float|定格冷水出口温度 ['C]|
|tin_ch|float||
|tout_ch|float||
|tout_ch_sp|float||
|g_ch|float||
|q_ch|float||
|pw|float||
|pl|float||
|cop|float||
|tdb|float|外気乾球温度 ['C]|
|kr_ch|float||
|dp_ch|float||
|flag|int|運転状態フラグ（下表）|
  
|  flag  | description |
| ---- | ---- |
|0|正常|
|1|要求熱量が定格能力超過（冷水出口温度が設定値より上昇）|
|2|負荷率がCOP表の上限を超過|
|3|負荷率がCOP表の下限未満|
|4|冷凍熱量ゼロ（停止）|
|5|冷凍熱量が負（tout_ch_sp > tin_ch）|
|6|外気温度がCOP表の下限未満（下限値で参照）|
|7|外気温度がCOP表の上限超過（上限値で参照）|
  
## pv.AirSourceHeatPump.cal(tout_ch_sp, tin_ch, g_ch, tdb)
冷水出口温度設定、冷水入口温度、冷水流量、外気乾球温度に基づいて出口温度と消費電力を算出する。
また、冷水における圧力損失も算出する。
COPはCOP表の補間値に対し、逆カルノーサイクル（冷房COP: (273.15+tout_ch)/(tdb-tout_ch)）に基づく定格冷水出口温度との差の補正を行う。
COP表参照と補正は同じ有効外気温度で評価する。外気温度がCOP表の範囲外の場合は表の端の値を参照する（flag 6/7）。
温度リフト（外気温度-冷水出口温度）が小さくなると補正係数が発散するため、有効外気温度は（冷水出口温度+5℃）で下側頭打ち（飽和）とする（5℃は発散防止のために仮に設定した暫定値）。
これにより外気温度が下がるほどCOPが上がり、飽和点以降は一定となる（単調性が保たれる）。
  
### returns:
なし（変数の値が更新される）
  
## サンプルコード  
```
import phyvac as pv

ASHP1 = pv.AirSourceHeatPump(filename='equipment_spec.xlsx', sheet_name='AirSourceHeatPump')  # ASHP1の定義
ASHP1.cal(tout_ch_sp = 7.0, tin_ch = 12.0, g_ch=0.15, tdb=25.0)  # 運転条件の入力と計算
print(ASHP1.pw, ASHP1.pl, ASHP1.tout_ch)  # 計算結果例のプリント
```
> phyvac: ver20231116  
> 5.579090966913902 0.6976744186046512 7.0
  
