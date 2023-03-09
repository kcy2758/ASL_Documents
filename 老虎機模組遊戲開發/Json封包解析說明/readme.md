# Json 封包解析說明

大綱
* [StartGame 封包](startgame-封包)
* [Spin 封包](spin-封包)
* [NextFever 封包](nextfever-封包)
* [NudgeFeature 封包](nudgefeature-封包)
* [SkillFeature 封包](skillfeature-封包)

## StartGame 封包
* Server 收到遊戲的 StartGame 請求之後，發送給 Client 的封包。
* 簡單介紹 StartGame 封包內主要包含幾種資料：
	* game_state: 遊戲狀態資料
		* current_sg_id: 當前進入遊戲的特殊遊戲ID，若無特殊遊戲則為-1。
		* sg_state: 無特殊遊戲為0，特殊遊戲Recovery為3。
		* current_lines: 線獎總線數。
		* current_bet_id: 當前進入遊戲時的押注段ID。
	* odds: 圖騰獲獎倍率表
		* 不同押注段可以對應不同的獲獎倍率(目前只讀取一組獲獎倍率)。 
	* wheel_blocks: 轉輪資料
		* id: 轉輪區ID。
		* init_wheels: 當前進入遊戲的初始盤面。
		* fake_wheels: 遊戲 Spin 時演出的假轉帶圖騰。
	* extra_info: 遊戲或平台所需的額外資料
		* 例如森林狂歡遊戲內JP的倍率。
		* 例如 AWP 平台需要 NudgeFeature 斷電回復的資料。
* 參考 `MainGameHostEx.SetStartGameData()` 方法，可以簡單了解如何解析 StartGame 封包。
	* 解析遊戲狀態資料：
	```
	startArgs.GameStatusData = ParseGameStatusData(JsonData);
	```
	* 解析圖騰獲獎倍率表：
	```
	startArgs.OddsDic = new Dictionary<int, int[]>();
    JSON oddsJson = JsonData.ToJSON("odds");
    for (int i = 0, count = oddsJson.fields.Count; i < count; i++)
    {
        if (oddsJson.fields.ContainsKey("" + i))
        {
            startArgs.OddsDic.Add(i, oddsJson.ToArray<int>("" + i));
        }
    }
	```
	* 解析轉輪資料：
	```
	startArgs.FakeWheelDataList = new List<WheelDataArgs>();
    JSON[] wheelBlockData = JsonData.ToArray<JSON>("wheel_blocks");
    foreach (JSON wheelblock in wheelBlockData)
    {
        WheelDataArgs wheelArgs = ParseWheelInitData(wheelblock);
        startArgs.FakeWheelDataList.Add(wheelArgs);
    }
	```
	* 解析遊戲或平台所需的額外資料：
	```
	if (JsonData.fields.ContainsKey("extra_info"))
    {
        startArgs.ExtraInfo = JsonData.ToJSON("extra_info");
    }
	```
* 遊戲或平台各自解析額外資料
	* 例如森林狂歡可參考 `G143_GameManager.OnStartGame()` 方法：
	```
    if(startArgs.ExtraInfo.ContainsKey("acorn_big_score_multiple")) {
        acornBigScoreMultiple = startArgs.ExtraInfo.ToInt("acorn_big_score_multiple");
	}
    scoreManager.SetFakeValueData(startArgs.ExtraInfo);
    rewardManager.Init(startArgs.ExtraInfo.ToArray<int>("jp_multiple"));
	```
	* Awp 平台可參考 `Awp_MainGameHost.OnStartGame()` 方法：
	```
	bool _needRecoveryFeature = _startGameArgs.ExtraInfo.ContainsKey("recovery_feature");
	if (_needRecoveryFeature) {
		featureCtrl?.SetRecovery(_startGameArgs.ExtraInfo.ToJSON("recovery_feature"));
	}
	```

## Spin 封包
* Server 收到遊戲的 Spin 請求之後，發送給 Client 的封包。
* 簡單介紹 Spin 封包內主要包含幾種資料：
	* game_state: 遊戲狀態資料
		* current_sg_id: 當前這手會獲得的特殊遊戲ID，若無特殊遊戲則為-1（若有 Nudge 玩法則恆為-1）。
		* sg_state: 當前這手無特殊遊戲為0，若有特殊遊戲為1（若有 Nudge 玩法則恆為0）。
		* current_lines: 線獎總線數。
		* current_bet_id: 當前這手遊戲的押注段ID。
	* wheel_blocks: 轉輪資料
		* id: 轉輪區ID。
		* result_wheels: 當前這手的結果盤面。
		* feature_wheels: 當前這手若獲得 Feature 的資料（若有 Nudge 玩法則必有 NudgeFeature 資料）。
		* pre_win_wheels: 當前這手的 PreWin 演出資料。
	* total_win_amount: 當前這手總贏分（若有 Nudge 玩法則恆為0）
	* win_type: 當前這手贏分類型（若有 Nudge 玩法則恆為0）
		* 0: NoWin
		* 1: NormalWin
		* 2: YouWin
		* 4: bigWin
		* 5: MegaWin
	* extra_info: 遊戲或平台所需的額外資料
		* 例如 AWP 平台需要 NudgeFeature 的相關資料。
* 參考 `MainGameHostEx.SetSpinData()` 方法，可以簡單了解如何解析 Spin 封包。
	* 解析遊戲狀態資料：
	```
	spinArgs.GameStatusData = ParseGameStatusData(JsonData);
	```
	* 解析轉輪資料：
	```
	spinArgs.WheelResultArgsList = new List<WheelBlockResultArgs>();
    JSON[] wheelBlockData = JsonData.ToArray<JSON>("wheel_blocks");
    for(int i = 0; i < wheelBlockData.Length; i++)
    {
        spinArgs.WheelResultArgsList.Add(ParseWheelResultData(wheelBlockData[i]));
    }
	```
	* 解析贏分資料：
	```
	if (JsonData.fields.ContainsKey("total_win_amount"))
    {
        spinArgs.TotalWin = JsonData.ToInt64("total_win_amount");
    }
    if (JsonData.fields.ContainsKey("win_type"))
    {
        int winType = JsonData.ToInt("win_type");
        spinArgs.ThisWinType = (WinType)winType;
    }
	```
	* 解析遊戲或平台所需的額外資料：
	```
	if (JsonData.fields.ContainsKey("extra_info"))
    {
        spinArgs.ExtraInfo = JsonData.ToJSON("extra_info");
    }
	```
* 遊戲或平台各自解析額外資料
	* Awp 平台可參考 `NudgeFeature.PlayFeature()` 方法：
	```
	JSON _value = (JSON)DataArg.Value;
	nudgeResult = _value.ToArray<long>("nudge_result");
	ShowNudgeItem(_value.ToList<int>("nudge_best"));
	```

## NextFever 封包
* Server 收到遊戲的 NextFever 請求之後，發送給 Client 的封包。
* NextFever 封包的長相又會分為以下三類(可參考[老虎機模組開發文件](https://docs.arcade.igs.com.tw/oo/r/602127552357608211) _8.SpecialGame規格開發_ 的 _封包結構_ 章節)：
	* Init: 特殊遊戲初始化封包。
	* Process: 每一手特殊遊戲的封包。
	* End: 特殊遊戲最後一手封包。
* 簡單介紹 NextFever 封包內主要包含幾種資料：
	* sg_id: 特殊遊戲ID
	* sg_state: 特殊遊戲狀態
		* 1: 特殊遊戲初始化。
		* 2: 特殊遊戲玩一手。
		* 3: 特殊遊戲 Recovery。
		* 4: 特殊遊戲最後一手。
	* sg_map: 特殊遊戲資料
		* Init 封包會包含初始化資料，例如特殊遊戲的假轉帶。
		* Process 封包會包含玩一手的資料，例如轉輪結果。
		* End 封包會包含最後一手的資料與結束時的演出資料，例如是否獲得最終 bonus 大獎。
	* total_win_amount: 森林狂歡只在最後一手有贏分
* 參考 `G143_FreeGame.ReceiveInitData()` 與 `G143_FreeGame.ReceiveProcessData()` 方法，可以簡單了解如何解析 NextFever 封包。
	* 解析 Init 封包：
	```
	ParseFakeWheelData(sgMapJson);
	bool isUnlock = sgMapJson.ToBoolean("unlock");
	int[] totalSpinTimesAry = sgMapJson.ToArray<int>("spins");
	```
	* 解析 Process 封包：
	```
	int blockID = sgMapJson.ToInt16("id");
	currentSpinTimes = sgMapJson.ToInt("current_spins");
	ParseResultData(sgMapJson.ToJSON("result"));
	G143_GameManager.Instance.scoreManager.SetResultData(blockID, SlotBaseFunc.Analysis2DAry_long(sgMapJson["current_scores"]));
	ParseAcornData(sgMapJson.ToArray<JSON>("current_feature_sym"));
	JSON[] ary = sgMapJson.ToArray<JSON>("next_prewin_frames");
	```
	* 解析 End 封包：
	```
	acornWinValue = sgMapJson.ToInt64("win");
	totalWinValue = jsonData.ToInt64("total_win_amount");
	ParseFinalData(sgMapJson.ToArray<JSON>("final"));
	```

## NudgeFeature 封包
* Client 在 Nudge 玩法主動發送給 Server 的封包，此封包為 Awp 平台的 Nudge 玩法擴充規格。
* 簡單介紹 NudgeFeature 封包內主要包含幾種資料：
	* nudge_pos: 玩家選擇的 nudge 位置
	* nudge_result: 玩家選擇 nudge 後的贏分
* 參考 `NudgeFeature.OnClickNudgeBtn()` 方法呼叫 SlotAPI。
* 參考 `GameSystem.NudgeCmd()` 方法組成 NudgeFeature 封包。

## SkillFeature 封包
* Client 在 Skill 玩法主動發送給 Server 的封包，此封包為 Awp 平台的 Skill 玩法擴充規格。
* 簡單介紹 SkillFeature 封包內主要包含幾種資料：
	* skill_ratio: 玩家完成 skill 遊戲後的該手贏分比例
* 參考 `SkillFeature.TestFinishSkill()` 方法呼叫 SlotAPI。
* 參考 `GameSystem.SkillCmd()` 方法組成 SkillFeature 封包。
