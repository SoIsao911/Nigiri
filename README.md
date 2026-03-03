Polymer Foaming Process Predictor
v8.0 → v8.0b Update Report
2026-03-03 | Confidential
1. Executive Summary
本次更新 (v8.0 → v8.0b) 涵蓋六大方向的模型修正與功能增強，重點在於提升模型科學準確性、消除已知假陽性警告、並增強工廠端使用便利性。修正共涉及 16 項變更，已通過 JavaScript syntax 驗證與 Engineer Backtest 功能測試。
2. Bug Fixes (Critical)
2.1 Batch Scoring Inconsistency
問題：三個獨立的 form-fill 函數 (loadHistoryToForm, loadHistoryToBatch, runEngineerBacktest) 使用不同的欄位處理邏輯，導致同一筆配方在不同入口計算出不同分數。
修正：抽取共用函數 _fillFormFromRecord(r)，統一處理 bulk materials、Color MB 5 slots、additives (含 PEG)、tail material、VA%、process params。三個函數統一呼叫，確保一致性。
2.2 Engineer Backtest Mode — formFields Scope Error
問題：runEngineerBacktest 函數引用 formFields 但未定義，導致 ReferenceError。
修正：在 runEngineerBacktest scope 內加入 formFields 定義。
2.3 Comparison Panel Visibility
問題：Engineer Backtest 比較面板插入在 batchStep2 底部，scrollIntoView 無效，使用者看不到面板。
修正：改用 insertBefore(panel, summaryCards.nextSibling) 插入位置 + 300ms 延遲 scrollIntoView。
2.4 Duplicate CI Warning
問題：交聯指數 Too High 警告重複出現兩次 (line 5794 和 line 5729 各 push 一次)。
修正：加入 !warnings.includes(dcpEval.warning) 防重複檢查。
2.5 dispIdx Scope Chain Error
問題：dispersionIndex 在 evaluateDCPCrosslinking() 內部定義為 const dispIdx，但被外部 calculate() 的 warning 區塊直接引用，造成 'Cannot access before initialization' 錯誤。
修正：建立完整的資料流：calcShearHeating() → mixShearEffect.dispersionIndex → params 傳入 → evaluateDCPCrosslinking destructure (default=1.0) → return → dcpEval.dispersionIndex → warning 區塊讀取。
2.6 BOM Import v4.0 Format Detection
問題：Load BOM 按鈕對 v4.0 配方單 (生產總表) 格式無法正確解析，被當成 BOM Format A 處理，導致原料名稱被誤讀為 model name。
修正：加入 v4.0 自動偵測邏輯 (sheet name / A1 cell / C2 cell)，偵測到時轉交 importExcelFormula() 處理。多檔模式下解析為 BOM-compatible product entry。
 
3. Model Improvements
3.1 LDPE-Dominant CI Range (P0)
問題：EVA% < 15% 的 LDPE-dominant 配方 (如 1600NN-2特殊板，EVA%=1.8%) CI 值遠超 CI Range 上限。原因是 evaRatio ≤ 30% 分支使用 EVA-based target (1.02)，但 LDPE 交聯效率更高，CI 自然偏高。
EVA% Range	Target CI	Range %	Status
< 15% (NEW)	1.10	±25%	v8.0b 新增
15% ~ 30%	1.02	±28%	維持
> 30%, Exp < 20X	0.62	±22%	維持
> 30%, Exp 20-25X	0.68	±20%	維持
> 30%, Exp > 25X	0.68 + f(EVA,Exp)	±22%	係數修正 0.90→0.65

3.2 Continuous Sigmoid Timing Model (P1)
問題：timingQuality 使用 if-else 階梯函數，在邊界值 (如 temp1=140°C) 產生不自然的品質跳變。真實 DCP 反應動力學遵循連續的 Arrhenius 曲線。
修正：改用雙 sigmoid 乘積模型：
lowTempFactor = 1/(1 + exp(-0.35×(T - 137)))    — 低溫端 ramp
highTempFactor = 1 - sigmoid(T, 160, 0.4)×0.6    — 高溫端 penalty
timingQuality = lowTempFactor × highTempFactor    — 鐘形曲線，峰值 145-155°C
Time1 影響也改為連續 ramp：12min 以下嚴懲，12-22min 線性過渡，22min 以上全效。
3.3 Color MB DCP Absorption 文獻修正 (P1)
問題：原模型假設 Carbon Black 對 DCP 的 absorption 係數為 0.08~0.11，但文獻回顧顯示 CB 主要透過物理吸附 (restricted chain mobility) 影響表觀交聯度，而非化學消耗 DCP 自由基。
Color MB	原 Absorption	新 Absorption	Rationale
8503 (CB 37%)	0.09	0.05	Physical adsorption
8509 (CB 34%)	0.08	0.04	Physical adsorption
7505 (CB 45%)	0.11	0.06	High CB, max effect
8800-3 (CB 34%)	0.08	0.04	Physical adsorption
Organic pigments	0.01~0.05	No change	Radical scavenging justified

3.4 混煉分散度模擬 (P1 — New)
新增 Dispersion Index (0~1)，量化 DCP/AC 在樹脂基質中的分散均勻性。計算基於剪切應力 × 時間 × 填充效應的 sigmoid 函數。
dispersionIndex = 0.3 + 0.68/(1 + exp(-6×(mixingEnergy - 0.5)))
影響：作為 CI 的乘法因子。低分散度 (< 0.6) 觸發 Warning，< 0.75 觸發 Suggestion。典型操作條件下 (18min/55rpm) dispersionIndex ≈ 0.95+，影響微小。極端條件 (< 12min 或 < 30rpm) 會顯著壓低 CI。
3.5 Discharge Temperature Warning 改善
問題：所有 145 筆歷史配方中 29 筆 (20%) 觸發 'Discharge Temperature 接近 AC 有效分解溫度' 警告。經查實際出料溫度均為 125-128°C，錶溫與實際溫度有系統性偏差。
修正方案：保留原始錶溫作為輸入，在結果面板 Discharge Temperature 卡片旁備註 'Est. actual: ~125-128°C'。Warning 分為兩級：tempDiff < -15 保持 [Warning]，-5 ~ -15 降為 [Info] 並顯示估算 AC 損失百分比。
 
4. Feature Enhancements
4.1 Excel v4.0 Import 增強
新增擷取欄位：mixTime (M8/STAR)、密練機型號 (M5)、RPM (M9)、手數 (K5)。一段溫度改為 K14+M14+M15 三模具取平均。Time2 改為只取 K18 加熱時間，不含 M18 冷卻時間。
4.2 History 搜尋功能
History Modal 新增即時搜尋框，支援 model、batch、日期、顏色等關鍵字篩選。與既有的 Category Filter (All/EVA/LDPE/Blend/POE/FR) 聯動，搜尋結果即時更新分頁。
4.3 多檔 BOM Import
Load BOM 的 file input 加入 multiple 屬性，支援同時選取多個 Excel 檔案。v4.0 格式自動偵測：單檔直接匯入表單，多檔解析為 BOM product entry 進入匯入清單。
5. Known Limitations & Future Work
5.1 RE-357 POE 配方 (Pending)
RE-357 系列三筆配方 (Score 53/68/70) 實際上是良品，但模型偏低評分。原因：高 EVA%(70%) + POE + 高色母，POE β-scission 降低 effective crosslinking，但 POE 提供韌性補償。需要在 CI Range 加入 'POE 韌性補償因子'。
5.2 Discharge Temperature 120°C 數據
History 中 145 筆記錄的 dischargeTemp 全部為 120°C，疑似為預設值而非實測。建議未來在生產端記錄實際出料溫度，或加入密練機型號欄位以自動校正。
5.3 原材料批次變異
同一 EVA 牌號的 MFI 波動會影響交聯效率，目前模型無法考量。此為不可控因素，模型只能提供近似分析。
5.4 二次收縮模型
脫模後自由冷卻影響最終密度，高度依賴環境溫度。目前無有效模擬方法，仍在評估中。
6. Change Log
#	Category	Description	Priority	Status
1	Bug Fix	_fillFormFromRecord() 共用函數	P0	✅
2	Bug Fix	formFields scope in Backtest	P0	✅
3	Bug Fix	Comparison panel visibility	P0	✅
4	Bug Fix	Duplicate CI warning	P1	✅
5	Bug Fix	dispIdx scope chain error	P0	✅
6	Bug Fix	BOM v4.0 format detection	P0	✅
7	Model	LDPE-dominant CI Range (EVA<15%)	P0	✅
8	Model	High EVA+Exp CI formula capped	P1	✅
9	Model	Sigmoid timing model	P1	✅
10	Model	CB DCP absorption 0.08→0.04~0.06	P1	✅
11	Model	Dispersion Index → CI modifier	P1	✅
12	Model	Discharge temp warning 2-tier	P2	✅
13	Feature	v4.0 import (mixTime/RPM/multi-mold)	P1	✅
14	Feature	Time2 = heating only (K18)	P1	✅
15	Feature	History search	P2	✅
16	Feature	Multi-file BOM import	P2	✅

