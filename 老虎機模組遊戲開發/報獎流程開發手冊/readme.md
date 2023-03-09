# 報獎流程開發手冊

大綱
* [報獎流程介紹](#報獎流程介紹)
* [報獎類型](#報獎類型)

## 報獎流程介紹
* 報獎流程會由 MainGameHost 的 ShowAward 狀態進入 AwardController。
* AwardController 內的報獎流程又細分如下：

| 流程                     | 說明 |
| ------------------------ | ------------- |
| Start                    | 發送開始報獎的事件。 |
| ShowBingoFrame           | 演出線獎框。 |
| ShowWin                  | 發送報獎事件、開始報獎演出與滾分。 |
| CheckBingoFrameAndWinEnd | 檢查線獎框與報獎演出是否結束。 |
| PlayBingoAlarmSound      | 播放大獎鈴聲。 |
| ShowSpecialSymbolWin     | 特殊圖騰中獎演出。 |
| End                      | 發送報獎結束事件，讓流程回到 MainGameHost。 |

![](./報獎.gif)

## 報獎類型
* MegaWin: 贏分倍率(win/bet) >= 50
* BigWin: 贏分倍率(win/bet) >= 20
* YouWin: 贏分倍率(win/bet) >= 10
* 不報獎: 贏分倍率(win/bet) < 10