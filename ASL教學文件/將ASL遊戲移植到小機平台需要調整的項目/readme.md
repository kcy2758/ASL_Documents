大綱
* [將 ASL 遊戲移植到小機平台需要調整的項目](#將-asl-遊戲移植到小機平台需要調整的項目)
    * [UnitDefine 資料夾](#unitdefine-資料夾)
    * [Assembly Definition](#assembly-definition)
    * [ASL/AWP 平台相關腳本](#aslawp-平台相關腳本)
    * [MainGameHostEx 相關參考](#maingamehostex-相關參考)
    * [小機專案單機版假封包](#小機專案單機版假封包)
    * [遊戲攝影機與其它](#遊戲攝影機與其它)

# 將 ASL 遊戲移植到小機平台需要調整的項目
此章節以 ASL 範例遊戲移植到老虎機產學專案的情境來說明。

## UnitDefine 資料夾
```
SampleGame -> ASL 範例遊戲資料夾
　└─── UnitDefines -> 存放範例遊戲所定義的 Units 的資料夾
```
UnitDefines 資料夾裡面存放的是 ASL 所使用的操作單元，因此需要整個刪除。

## Assembly Definition
```
SampleGame -> ASL 範例遊戲資料夾
　└─── Game_Sample -> ASL 範例遊戲程式集定義檔
```
根據搬移到的目標專案，需要調整範例遊戲 Assembly 的參考。
例如，範例遊戲的 Assembly 可能需要參考平台的 Assembly 等等。

## ASL/AWP 平台相關腳本
* 遊戲場景初始化
    ```
    SampleGame -> ASL 範例遊戲資料夾
     └─── Scripts -> 存放腳本的資料夾
         ├─── Sample_GameSceneManager.cs
         └─── Sample_GameCtrl.cs
    ```
    Sample_GameSceneManager 腳本會由 ASL 來呼叫遊戲場景初始化，並且該腳本會呼叫Sample_GameCtrl 腳本進行遊戲相關 Unit 的初始化，因此這兩個腳本直接刪除。
* 平台報獎
    ```
    SampleGame -> ASL 範例遊戲資料夾
     └─── Scripts -> 存放腳本的資料夾
         └─── Sample_WinEffectCtrl .cs
    ```
    Sample_WinEffectCtrl 腳本是繼承自 Awp 平台的報獎腳本，移植到小機後會採用小機平台的報獎，因此直接刪除。
* ASL Internal Server
    ```
    SampleGame -> ASL 範例遊戲資料夾
     └─── Scripts -> 存放腳本的資料夾
         └─── Sample_ASLServer.cs
    ```
    Sample_ASLServer 為大機 ASL 專用的 InternalServer，由於小機會有自己的 server client，因此直接刪除。

## MainGameHostEx 相關參考
```
SampleGameAScene -> 範例遊戲場景
　└─── Manager
　　　　　├─── SlotGameCore -> 不用打開
　　　　　├─── InternalServer -> 不用打開
　　　　　└─── MainGameHostEx -> 需要打開
```
將範例遊戲場景中的 MainGameHostEx 遊戲物件打開，並為它設定相關參考如下：

* Creater Root
    ```
    Game145_MagicSweets_MG -> 產學遊戲場景
     └─── UI Root(NGUI) -> 複製此物件包含子物件到 ASL 範例遊戲場景
    ```
    * 先將老虎機產學專案場景中的 UI Root(NGUI) 物件包含子物件整個複製到 ASL 範例遊戲場景。

    ```
    UI Root(NGUI)
     └─── SystemRoot
         └─── PlatformUI -> 掛到 MainGameHostEx 的 Creator Root 欄位
    ```
    * 在 UI Root(NGUI) 底下找到 PlatformUI 物件，將它掛到 MainGameHostEx 的 Creator Root 欄位。

* Slice Input Root
    ```
    Game145_MagicSweets_MG -> 產學遊戲場景
     └─── GameRoot(UGUI)
         └─── MainRoot
             └─── Canvas_Slice -> 複製此物件包含子物件到 ASL 範例遊戲場景
    ```
    * 先將老虎機產學專案場景中的 Canvas_Slice 物件包含子物件整個複製到 ASL 範例遊戲場景。

    ```
    Canvas_Slice
     └─── SliceInputRoot -> 掛到 MainGameHostEx 的 Slice Input Root 欄位
    ```
    * 在 Canvas_Slice 底下找到 SliceInputRoot 物件，將它掛到 MainGameHostEx 的 Slice Input Root 欄位。

* UGUI_Down Bar
    ```
    Game145_MagicSweets_MG -> 產學遊戲場景
     └─── GameRoot(UGUI)
         └─── Canvas_DownBarUI -> 複製此物件到 ASL 範例遊戲場景
    ```
    * 先將老虎機產學專案場景中的 Canvas_DownBarUI 物件複製到 ASL 範例遊戲場景。
    * 將 Canvas_DownBarUI 物件掛到 MainGameHostEx 的 UGUI_Down Bar 欄位。

## 小機專案單機版假封包
```
Game145_MagicSweets_MG -> 產學遊戲場景
　└─── ClientTest -> 複製此物件包含子物件到 ASL 範例遊戲場景
```
產學專案是由場景中的 ClientTest 物件身上所掛的 OfflinePlatform 腳本來驅動，因此我們將該物件複製到 ASL 範例遊戲場景，讓它來驅動 ASL 的範例遊戲。

```
ClientTest
　└─── SettingTool -> 設定此物件身上 UGUI_Wheel Setting Tool 腳本的參考
```
在 ClientTest 底下的 SettingTool 身上掛有 UGUI_WheelSettingTool 腳本，我們要把 ASL 範例遊戲場景中的 WheelBlockController 掛到該腳本的 Wheel Block Ctrl 欄位，並且設定該腳本的圖騰圖片參考。

## 遊戲攝影機與其它
* 由於 ASL 範例遊戲場景中沒有攝影機 (由ASL初始化場景沿用)，因此移植到產學專案後要重新增加一台攝影機，並做對應的調整。
* ASL 範例遊戲搬移到產學專案之後，也需要重新設定所有圖片的圖層 Layer。
* OfflinePlatform 腳本身上的 Start Data 與 Round Data Text Assets 欄位可以設定假資料的文字檔如下圖。
![](https://igshub.i17game.net/rd3_sys_gear/asl/-/raw/Asl_2.0_Demo/Img/ASL%E7%AF%84%E4%BE%8B%E9%81%8A%E6%88%B2%E7%A7%BB%E6%A4%8D%E8%80%81%E8%99%8E%E6%A9%9F%E7%94%A2%E5%AD%B8%E5%B0%88%E6%A1%88%E9%9B%A2%E7%B7%9A%E5%B9%B3%E5%8F%B0%E8%A8%AD%E5%AE%9A.png?inline=false)
