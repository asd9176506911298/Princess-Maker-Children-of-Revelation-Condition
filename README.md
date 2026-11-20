# 預言之子 結局條件整理（依遊戲資料表）使用遊戲版本 v1.1.0 抓取的資料 將資料丟給Ai分析 也有丟巴哈文章給Ai參考

資料來源：`TableData`（EndingStatus、Ending、Reputation、Quest、Event）與 `C_Flow_*` 事件腳本，並對照巴哈攻略「各結局參數」。
最後更新：2026-11-16

## 怎麼讀

- 「結局條件」就是 `EndingStatus`。**不同條件組是「且」，同一組內是「或」**（已由遊戲判定程式確認；只有女王的戰士／魔法評價放在同一組）。
- 評價有兩種寫法：「評價 ≥ N」是評價數值本身；「評價階級 ≥ N」是階級（1：0、2：50、3：150、4：250、5：350、6：500）。
- 課程／工作等級：3 = B、4 = A、5 = S（B、A 由論壇和 `Quest_RAAPresident` 要求「美術、音樂、舞蹈達 A = 等級 4」對上，S 是依序推出）。
- 「性向」就是善惡值：≤ -150 為渾沌，≥ 225 為秩序（女王）。
- 「前置事件鏈」是 `Flow_*` 腳本的發生條件，結局條件裡有 `完成事件 Flow_xxx` 才是必要的。**沒列在結局條件的事件鏈（例如混沌煉金師）不是結局必要條件**，只是劇情。
- Trigger：1010 月初、1017 行程後段隨機、2012 在王城遇人、5001 節慶結束後、5002 節慶開始。

## 結局怎麼決定（來自 GameAssembly 反組譯：`lxc::mko`／`mau`／`sai`）

**1. 候選資格。** 只看 `EndingType = 2` 的 55 個一般結局（`Couple_*` 是 `EndingType = 3`，不在這一輪）。一個結局要成為候選，**每個條件組至少要有一條成立**（組內「或」、組間「且」；已由 `Cond_CheckEndingCondition` 與 `mko` 兩處確認）。單一條件成立與否：`CompareType = 0`（事件、無條件）是「目前值 ≥ 1」，其餘用 ≥／≤ 與 CompareValue 比較。

**2. 計分。** 每個候選的分數 = **Weight ＋ 加成**，只留下**分數最高**的，**同分全部留下**。

**3. 抽籤。** 留下 1 個就用它，多個就**隨機抽 1 個**（`sai`）。所以同分結局是隨機的。

**4. 加成（`mau`）。**
- Weight ≥ 30000 的結局：沒有加成。
- Weight < 30000 的結局：加成 = `min(2 × 數值, 8000)`。數值是該結局條件裡所有「課程／打工階級」條件（Type 8）所指課程在玩家紀錄裡的累積值（紀錄欄位 `lcq`），取**最大的一個**。
- **吟遊詩人（Bard）是特例**：加成 = `min(2 × (Edu_Music ＋ Edu_Literature 的累積值), 8000)`。
- `lcq` 欄位名被混淆，但已用 `ld::fno(課程Key, 累積值)` 的反組譯確認它的用途：該函式在這個課程的 `ScheduleStat` 各階列中，找 `NeedCount_GradeUp` **大於**累積值的列裡最小的那一列，當作目前階級。也就是 `NeedCount_GradeUp` 是**累積門檻**，`lcq` 就是跟它比的累積次數（例如課程：累積 ≥ 42 次為階級 2、≥ 96 次為 B、≥ 160 次為 A、≥ 234 次為 S）。所以**加成實務上是 2 × 累積次數，大約數百，最多一千出頭**，8000 的上限幾乎碰不到。

**5. 對你的意義。**
- 難度高的結局（女王 52000、魔王 51000、公主 50000、大將軍等）靠 Weight 直接壓過一切，條件全過就是它。
- 加成太小，**不足以跨越 Weight 的檔次**（例如 20000 系列追不上 27000 系列），它的作用是在 **Weight 相近的結局之間重新排序**：例如 20000、20080、20160、20200、20240、20495、20700、20850、20900 這些相差幾十到幾百的，誰練得多誰優先。
- 完全同分（例如大富豪、魔法劍士、外交官都是 40000，沒有加成）會隨機抽一個。

| 結局 | Weight | 課程／打工加成 |
|---|---|---|
| 女王（Queen） | 52000 | 無 |
| 魔王（DemonKing） | 51000 | 無 |
| 公主（Princess） | 50000 | 無 |
| 大將軍（General） | 43000 | 無 |
| 王宮魔法師（RoyalWizard） | 42000 | 無 |
| 王立藝術學會會長（RAAPresident） | 41000 | 無 |
| 外交官（Diplomat） | 40000 | 無 |
| 魔法劍士（MagicKnight） | 40000 | 無 |
| 大富豪（Rich） | 40000 | 無 |
| 沙龍說客（Sting） | 34000 | 無 |
| 蛇蠍美人（Femme） | 33000 | 無 |
| 混沌鍊金術師（ChaosAlchemist） | 32000 | 無 |
| 怪盜（MysteriousThief） | 32000 | 無 |
| 占卜師（FortuneTeller） | 31000 | 無 |
| 後巷之王（TheGodfather） | 31000 | 無 |
| 吟遊詩人（Bard） | 27000 | 有（約 2 × 累積次數） |
| 貴金屬工匠（Jeweler） | 27000 | 有（約 2 × 累積次數） |
| 魔導工程師（MagicEngineer） | 27000 | 有（約 2 × 累積次數） |
| 調香師（Perfumer） | 27000 | 有（約 2 × 累積次數） |
| 魔法陣圖案師（SigilDrafter） | 27000 | 有（約 2 × 累積次數） |
| 劍舞姬（Sworddancer） | 27000 | 有（約 2 × 累積次數） |
| 學院院長（AcademyDean） | 23000 | 有（約 2 × 累積次數） |
| 首席芭蕾舞者（Dancer） | 23000 | 有（約 2 × 累積次數） |
| 首席歌姬（PrimaDonna） | 23000 | 有（約 2 × 累積次數） |
| 廚師（Cook） | 22500 | 有（約 2 × 累積次數） |
| 魔法事務員（MagicClerk） | 20900 | 有（約 2 × 累積次數） |
| 首都精銳兵團（CapitalEliteTroop） | 20850 | 無 |
| 冒險家（Adventurer） | 20700 | 無 |
| 鍊金術師（Alchemist） | 20495 | 有（約 2 × 累積次數） |
| 舞蹈教師（DanceTutor） | 20495 | 有（約 2 × 累積次數） |
| 禮法教師（Governess） | 20495 | 有（約 2 × 累積次數） |
| 歷史學者（Historian） | 20495 | 有（約 2 × 累積次數） |
| 魔法教習（MagicTutor） | 20495 | 有（約 2 × 累積次數） |
| 體術教頭（MartialArtsInstructor） | 20495 | 有（約 2 × 累積次數） |
| 音樂教師（MusicTutor） | 20495 | 有（約 2 × 累積次數） |
| 畫家（Painter） | 20495 | 有（約 2 × 累積次數） |
| 劍術道場教頭（SwordInstructor） | 20495 | 有（約 2 × 累積次數） |
| 作家（Writer） | 20495 | 有（約 2 × 累積次數） |
| 調酒師（Bartender） | 20240 | 有（約 2 × 累積次數） |
| 物流主管（UnloadingManager） | 20240 | 有（約 2 × 累積次數） |
| 研究助理（Assistant） | 20200 | 有（約 2 × 累積次數） |
| 家庭教師（PrivateTutor） | 20200 | 有（約 2 × 累積次數） |
| 女僕咖啡廳的超級明星（Superstar） | 20200 | 有（約 2 × 累積次數） |
| 鐵匠（Blacksmith） | 20160 | 有（約 2 × 累積次數） |
| 服裝設計師（Designer） | 20160 | 有（約 2 × 累積次數） |
| 物流管理員（Logistics） | 20080 | 有（約 2 × 累積次數） |
| 士兵（Soldier） | 20080 | 有（約 2 × 累積次數） |
| 木匠（Carpenter） | 20000 | 有（約 2 × 累積次數） |
| 村莊治療師（Doctor） | 20000 | 有（約 2 × 累積次數） |
| 家政服務公司老闆（Housekeeper） | 20000 | 有（約 2 × 累積次數） |
| 牧場管理員（RanchManager） | 20000 | 有（約 2 × 累積次數） |
| 園藝治療師（Therapist） | 20000 | 有（約 2 × 累積次數） |
| 餐廳店員（Waitress） | 20000 | 有（約 2 × 累積次數） |
| 村民（Villager） | 2 | 無 |
| 無業（Jobless） | 1 | 無 |

## 占卜師怎麼提示（`lxc::mca`／`lzj`／`lyx`）

- 每個結局的進度 = Σ（各條件組的最高 Score × 該組內**最佳條件的進度**）÷ Σ 各組的最高 Score。
- 單一條件進度：一般 ≥ 條件是「目前值 ÷ 目標值」，限制在 0 到 1；階級類（聲望、課程階級）是 (目前階級 − 1) ÷ (目標階級 − 1)；善惡 ≤ 條件是 (上限 − 目前) ÷ (上限 − 目標)；是非題（事件、無條件）是 0 或 1。
- 只考慮進度 ≥ 門檻的結局，門檻 = 0.2 ＋ 0.6 × 經過天數 ÷ 2920（8 年），也就是 0.2 漸增到 0.8。
- 剔除 `Ending_Jobless`（用 Key 字串比對）與 Weight < 1000 的結局（Jobless 與 Villager）。
- 排序：進度由高到低，同進度再比 Weight ＋ 加成，由高到低。最多顯示 **3** 個。

## 舞者與歌姬的條件

兩者最後一個條件組是「藝術祭勝利」（Type 15 舞蹈、16 音樂）。`lxc::hch` 的算法：走訪藝術祭歷史紀錄，每筆以 `&` 分隔的標籤，必須**同時含 `Rank=1` 與 `Category=<該分類>`**才算一次，條件是 ≥ 1，也就是「該分類的藝術祭拿過第一名」。另外 `Cond_CheckEndingCondition` 這個事件節點沒有處理 15、16，但整套資料只有公主第 6 話用到它，不影響這兩個結局。

## 戀人結局怎麼決定（來自 `pk::OnTheEnd`／`ShowCard`／`go::glc`）

**戀人結局跟 EndingStatus 條件完全無關**，只看「遊戲狀態裡的結婚對象」這一個數字。它和職業結局是**兩張各自獨立的結局卡**：職業結局由 `lxc::sai` 決定（上一節），戀人結局由結婚對象決定，結局畫面會依序顯示。

**對應規則**（結婚對象代碼 = `EPartnerNPC`，由 `Act_SetWeddingNPC` 設定）：

| 結婚對象代碼 | 對象 | 戀人結局 | 婚禮對話圖 |
|---|---|---|---|
| 7 | Dragon_Female | `Ending_Couple_DragonFemale` | `D_DragonFemale_Wedding` |
| 6 | Dragon_Male | `Ending_Couple_DragonMale` | `D_DragonMale_Wedding` |
| 5 | Forest_Female | `Ending_Couple_ForestFemale` | `D_ForestFemale_Wedding` |
| 4 | Forest_Male | `Ending_Couple_ForestMale` | `D_ForestMale_Wedding` |
| 3 | Demon_Female | `Ending_Couple_DemonFemale` | `D_DemonFemale_Wedding` |
| 2 | Demon_Male | `Ending_Couple_DemonMale` | `D_DemonMale_Wedding` |
| 1 | Furry_Female | `Ending_Couple_FurryFemale` | `D_FurryFemale_Wedding` |
| 0 | Furry_Male | `Ending_Couple_FurryMale` | `D_FurryMale_Wedding` |
| -4 | Npc_Cube | `Ending_Couple_Cube` | `D_Cube_Wedding` |
| -5 | Npc_Caim | `Ending_Couple_Caim` | `D_Caim_Wedding` |
| -6 | Npc_Icosa | `Ending_Couple_Icosa` | `D_Icosa_Wedding` |
| -7 | Npc_Kimaris | `Ending_Couple_Kimaris` | `D_Kimaris_Wedding` |
| -8 | Npc_Muzai | `Ending_Couple_Muzai` | `D_Muzai_Wedding` |
| -1（村人）／-3（工作夥伴） | 一般婚姻 | `Ending_Couple_Normal` | — |
| -2（單身） | 沒有結婚 | 無戀人結局 | — |

**唯一的特例**：結婚對象是阿加雷斯（DemonMale, 2），而且**職業結局是 `Ending_Princess`**（公主）時，戀人結局改為 **`Ending_Couple_DemonMale_King`**，而不是 `Ending_Couple_DemonMale`。程式碼用字串 `Ending_Princess` 比對（`a::gx`），所以只有走完公主線才會拿到「魔王」版本。其他組合（例如職業結局不是公主）仍是 `Ending_Couple_DemonMale`。

**其他確認的事**
- 職業結局的唯一呼叫處是 `pk::chs`（結局畫面）呼叫 `lxc::sai`，沒有別的入口。
- 結婚對象為 -1 或 -3 都是一般婚姻、同一張 `Couple_Normal` 卡；-2 是單身，只給「未婚」成就，沒有戀人結局。
- 成就「全部婚姻結局」統計歷次通關中結婚對象代碼 ≥ -4 的不同對象（-1 與 -3 合併算一個），累積到 **11 個**就達成；`go::glb`、`go::glc` 負責把歷次通關的結局收進收藏清單。
- 結局畫面還會多放一張卡 `Card_Second_EA`（`a::ha`），只在某個條件（`hl::ljk()` 為真，且該卡已解鎖）下加入，用途沒追。

**結局時的最終判定（`pk::gde`，在結局畫面 `pk::chs` 開頭呼叫，結果寫回結婚對象）**

結婚對象有兩個來源：求婚事件接受時用 `Act_SetWeddingNPC` 寫入（代碼 ≥ 0 的 8 位種族夥伴），以及結局當下由 `pk::gde` 依已完成事件與職業結局補上。`gde` 的順序如下（地址 `0x182638610`）：

1. **結婚對象已 ≥ 0**（接受過求婚）：直接用它，後面全部略過。
2. 否則依序檢查，先符合先用：

| 順序 | 條件（全部成立） | 結果 |
|---|---|---|
| 1 | 已完成事件 `Caim_Episode1`，且職業結局是 `Ending_Waitress`（餐廳店員） | -5 凱姆 |
| 2 | 已完成事件 `Icosa_Episode1`，且職業結局是 `Ending_MagicTutor`（魔法教習） | -6 艾科薩 |
| 3 | 已完成事件 `Kimaris_Episode1`，且職業結局是 `Ending_Soldier`（士兵） | -7 奇馬里斯 |
| 4 | 已完成事件 `Muzai_Episode1`，且職業結局是 `Ending_Writer`（作家） | -8 穆扎伊 |
| 5 | 已完成事件 `Cube_Episode2`（**不看職業結局**） | -4 吉普 |
| 6 | 12 項基礎能力（體力、力量、敏捷、智力、魔力、精神力、氣質、禮儀、口才、感性、品味、魅力）**總和 ≥ 3600** | -2 單身（沒有戀人結局） |
| 7 | 職業結局是 `Ending_Jobless` 或 `Ending_Villager` | -1 一般婚姻（`Couple_Normal`） |
| 8 | 以上都不符合 | 維持目前值。新遊戲的預設值就是 **-3（工作夥伴，也是 `Couple_Normal`）**：存檔資料類別 `do` 的建構函式 `do::do()` 把這個欄位初始化為 -3，已確認 |

另外 1 到 4、5 各自還有一條「目前值已經是該代碼就沿用」的捷徑，不影響上面的結果。

**這代表**
- 要拿凱姆、艾科薩、奇馬里斯、穆扎伊的戀人結局，**職業結局必須剛好是對應那一個**（餐廳店員、魔法教習、士兵、作家），再加上完成各自的 `Episode1`。職業結局不對，會直接掉到下一條。
- 吉普只要 `Cube_Episode2` 完成就成立，不管職業。
- 這 5 位的優先度比單身（總和 ≥ 3600）高，所以能力很高的女兒只要有完成對應事件，仍然會拿到他們的戀人結局。
- 能力總和 ≥ 3600（平均 300）且沒有任何夥伴時才會是單身；能力沒到而職業是無業／村人時是一般婚姻。這個 3600 是從組語讀到的常數，12 項是 `EStatus` 1 到 12，我沒有在遊戲裡實測。
- 8 位種族夥伴（代碼 0 到 7）必須在 1218 年 3 月求婚事件中接受，才會讓代碼 ≥ 0，下面「各夥伴的求婚事件」有條件。

## 與巴哈攻略對照：基本對得上，但數字有不少出入

**完全對上**：牧場管理員、研究助理、音樂教師、舞蹈教師、魔法師範、歷史學者、魔法事務員（智魔精 1500、魔法評價階級 4 = 250）、首席芭蕾舞者（舞蹈 A 加舞蹈賽冠軍）、魔法劍士（力量、魔力 600，兩評價 ≥ 250）、教父（力量 800）、冒險者（探險龍、森、獸三地區）、女王的四項加總（1500／1500／2400／1500）、魔王的兩項加總 2400、公主的兩項加總 2400、王立藝術學會會長的感受性＋品味＋魅力 2400 與藝術評價 350、王宮魔法師的智魔精 2400 與魔法評價 350。

**數字不同（以資料為準）**

| 結局 | 巴哈寫法 | 資料實際 |
|---|---|---|
| 公主 | 社交評價 ≥ 450 | **≥ 400** |
| 女王 | 戰士、社交評價皆 500 | 社交 ≥ 500，**戰士或魔法擇一 ≥ 400** |
| 魔王 | 戰士、魔法評價 500、善惡 -300 | 結局條件**沒有評價**，只要能力加總 2400；善惡 **≤ -225**（事件鏈只要 ≤ -150） |
| 王立藝術學會會長 | 氣質＋禮儀＋口才 ≥ 1500 | **沒有此條件**；但要**藝術節奪冠**＋藝術、舞蹈、音樂達 A |
| 王宮魔法師 | 魔法、煉金達 A | 魔法學、鍊金術都要 **S（等級 5）** |
| 大將軍 | 體敏力 ≥ 2100、氣質 200、禮儀 300 | 體力＋力量＋敏捷 **≥ 2400**、氣質＋禮儀 **≥ 600**，另要**劍術 S、體術 S、武鬥會奪冠** |
| 外交官 | 智力 > 750、三項 ≥ 2100、社交 500 | 智力 **≥ 500**、氣質＋禮儀＋口才 **≥ 1800**、社交 **≥ 350** |
| 沙龍說客 | 智力 > 750、各 700、社交 500 | 智力 **≥ 500**、三項加總 **≥ 1600**、社交 **≥ 350**、善惡 ≤ -150 |
| 蛇蠍美人 | 酒吧 A、體力與口才 > 300 | 酒館達 **S**、魅力 600，**沒有體力、口才條件** |
| 後巷之王 | 戰士評價 ≥ 250 | **≥ 350**，體力 400、力量 800 |
| 怪盜 | 智力 600 | 智力 **≥ 300**、敏捷 600、品味 600、事件 Flow_MysteriousThief_Ep3 |
| 混沌煉金師 | 魔力 > 600、煉金 A、三次事件 | 煉金 A、智魔精 ≥ **1500**、品味 300；**事件非必要** |
| 占卜師 | 智魔精口才各 500、魔法 A | 精神、口才、感受性各 **400**、魔法評價階級 ≥ 4、善惡 ≤ -150；**不要求魔法 A** |
| 劍舞姬 | 魅力、力量 > 600，舞蹈 A、劍術 A | 劍術 A、**舞蹈 B**、魅力 **≥ 500** |
| 禮儀教師 | 禮法 B、禮儀氣質 > 300 | 禮法 **A**、禮儀 400、氣質 400、感受性 200 |
| 魔導工程師 | 煉金 A | 魔法 A、煉金 A、**智魔精加總 ≥ 2000** |
| 體術教頭 | 力量 300、精神力 300 | **體力** 300、精神 300（體術 B） |
| 調酒師 | 體力、口才 > 300 | **口才、氣質** 各 300（酒館 A） |
| 超級巨星 | 感性、魅力 > 400、社交升兩階 | 咖啡廳 A、魅力 300、感受性 300、口才 200，**沒有社交評價條件** |
| 大富豪 | 金錢 > 5 萬 | 金錢 ≥ 50000 **且**完成事件 Flow_Rich_Ep4（兩者都要，見下文更正） |

**巴哈沒有、資料有的結局**：廚師、家教、設計師、士兵、女服務生、鐵匠、物流（水果倉庫）、卸貨主管（論壇的「物流主管」其實是碼頭，對應卸貨主管）、首都精銳部隊、學院院長、調香師、珠寶匠、吟遊詩人、魔紋繪師、首席歌姬、作家、畫家、劍術教師、體術教頭、管家、村民。

## 更正

先前我說大富豪是「金錢 5 萬**或**事件」，**那是錯的**。兩條在不同條件組，依一致規則是**兩者都要**：先完成 Flow_Rich 事件鏈（Ep1 金錢 ≥ 20000 起，Ep2 要先有 `Event_Rich_Invest` 且隔 3 個月，Ep3 隔 4 個月，Ep4 隔 8 個月），結局時金錢也要 ≥ 50000。

## 任務條件（Quest 表）

- `Quest_RoyalWizard1`：用魔法討伐魔獸 20 隻。`Quest_RoyalWizard2`：用魔法在武鬥會奪冠。
- `Quest_RAAPresident`：美術、音樂、舞蹈三門都達 A（等級 4）。
- `Quest_Adventurer1`：討伐黑暗魔法師。`Quest_Adventurer2`：討伐沙漠掠奪者。
- `Quest_Queen1`：在宴會上拜訪代理攝政。`Quest_Queen2`：在魔王城拜訪代理攝政。

## 需要做選擇的關鍵事件（來自 D_Flow 對話圖）

| 事件 | 要選的 | 選錯的後果 |
|---|---|---|
| Flow_DemonKing_Ep5（1218 年 2 月，只有一次，MaxCount 1） | 「我明白了。一起走吧。」→ 設定 `Event_DemonKing_Allow` | 選「……我還不知道。」門不會再開，魔王結局斷線 |
| Flow_Rich_Ep1 | 「投資。」（花 5000 金）→ 設定 `Event_Rich_Invest` | 選「不投資」則 Ep2 條件不成立，大富豪鏈斷線 |
| Flow_MysteriousThief_Ep1 | 「……真有意思。我幫你藏起來。」（善惡 -10） | 選「交給士兵」得 200 金，但設定 `Event_MysteriousThief_Prison`，Ep2 條件要求**不可**有此旗標，怪盜鏈斷線 |
| Flow_Adventurer_Ep1 | 「接受。」→ 設定 `Quest_Adventurer1` | 選「拒絕」就沒有 Ep2 需要的任務 |
| Flow_TheGodfather_Ep1／Ep2 | 要往渾沌走：Ep1 選「繼續拿他們出氣」（-20）；Ep2 選「接受提議」（-30） | 選「交給守衛」各 +20，不影響事件鏈，但會把善惡往秩序推，教父要求善惡 ≤ -150 |

## 公主線：阿加雷斯（DemonMale）求婚條件（來自 Partner 事件圖）

好感階級（`Cond_CompareFavorGrade`）：3 好感（20–29）、4 愛慕（30–39）、5 特別（40–49）、6 靈魂伴侶（50）。

| 事件 | 條件 |
|---|---|
| Episode1 | 已完成 DemonMale_FirstEpisode、尚未完成 Ep1；遊戲年 ≤ 1213。1213 年內直接可觸發，之前則要在 FirstEpisode 後隨機 0–2 個月 |
| Episode2 | 已完成 DemonMale_InterEpi2、尚未完成 Ep2；年 ≤ 1213，同樣 1213 年內可觸發，之前隨機 0–2 個月 |
| Episode3 | 已完成 Ep2；**課程累計 ≥ 2 次**；年 ≤ 1213；之後同上 |
| Episode4 | 已完成 Ep3；Karen 年齡 ≥ 14 |
| Episode5 | 已完成 Ep4；年齡 ≥ 15；好感階級 ≥ 3（好感） |
| Episode6 | 已完成 Ep5；年齡 ≥ 16；好感階級 ≥ 3 |
| Episode7 | 已完成 Ep6；年齡 ≥ 17；好感階級 ≥ **4（愛慕）** |
| Episode8 | 已完成 Ep7（沒有其他條件） |
| **Proposal（求婚）** | **1218 年 3 月月初**（Trigger 1010）；已完成 Ep8；且**未設定 `Event_DemonKing_Allow`** |

求婚事件裡選「接受求婚」才會執行 `Act_SetWeddingNPC(DemonMale)` 並設定 **`DemonMale_Proposal_Allow`**；選「拒絕求婚」沒有任何旗標，求婚事件只有一次（MaxCount 1）。若進入事件時**已經有結婚對象**（`Cond_HasWeddingPartner`），對話直接略過，也不會設定旗標。

所以：
- **公主結局需要的旗標，來自 1218 年 3 月接受阿加雷斯的求婚。**
- **魔王與公主互斥**：選了魔王第 5 話的「一起走吧」（`Event_DemonKing_Allow`），求婚就不會發生。
- 好感階級在 Ep5 到 Ep7 才要求 3、3、4，**資料沒有要求到靈魂伴侶（階級 6）**；巴哈寫的「培養至靈魂伴侶」只是保險。Ep8 與求婚本身沒有好感門檻，不過 Ep8 的事件內容我沒有檢查，可能有隱含限制。
- 其他夥伴的求婚事件已解析（見下一節）：同樣是 1218 年 3 月月初，若先接受了其他人的求婚，阿加雷斯的求婚會因為「已有結婚對象」而略過；唯一例外是森族女，她的求婚不檢查是否已有對象。

## 各夥伴的求婚事件與劇情鏈（來自 Partner 事件圖）

**求婚事件共通規則**（FurryMale／FurryFemale／ForestMale／ForestFemale／DragonMale／DragonFemale／DemonFemale／DemonMale 共 8 位）：
- 發生時間完全相同：**1218 年 3 月月初**（Trigger 1010），要求**未設定 `Event_DemonKing_Allow`**，且**該夥伴最後一個劇情事件已完成**（見下表最後一列）。
- 8 個求婚事件 Priority 都是 1、MaxCount 都是 1：每位只會提一次，選拒絕就用掉了。
- 對話流程相同：進入事件先檢查 `Cond_HasWeddingPartner`，**已有結婚對象就直接略過**；否則出現「接受／拒絕」兩個選項，選接受才執行 `Act_SetWeddingNPC(該夥伴)`。
- **ForestFemale（森族女）的對話圖裡沒有 `Cond_HasWeddingPartner` 檢查**，所以就算已經先接受了別人的求婚，她的求婚仍會出現，接受會**覆蓋**結婚對象（從圖的結構看，沒有實測）。
- **只有阿加雷斯（DemonMale）的求婚會設定旗標 `DemonMale_Proposal_Allow`**，所以只有他能通往公主結局；其他 7 位接受求婚只會得到各自的戀人結局。
- 7 位非阿加雷斯的夥伴都**不需要特定好感階級才能求婚**，好感只出現在劇情鏈中（見下表），求婚本身只看「劇情鏈最後一話已完成」。

**同一個月有多位夥伴符合時**：8 個事件 Trigger 都是 1010、Priority 都是 1、MaxCount 1。依 `EventManager::sch` 的邏輯，同一觸發點只會挑「最小的正數優先度」那一組放進佇列，同組全部排入，所以 8 位求婚事件會一起排隊（3 月同時還有其他 Priority 1 的 1010 事件，例如 Flow_Queen_Ep6、Flow_Rich_Ep1~4、Flow_DemonKing_Ep2，若 Priority 0 的事件當月成立則會先被挑走）。排隊順序**推測**依 Event 表列順序：ForestFemale → ForestMale → DragonFemale → DragonMale → FurryMale → FurryFemale → DemonFemale → DemonMale（這一點是推測，未驗證 `jq::fca` 的排序）。每個對話都先檢查 `Cond_HasWeddingPartner`，只有「接受」才會設定對象，所以通常是排在前面且你接受的那位成為對象，後面的會被略過（ForestFemale 的對話缺這個檢查，需注意）。保險做法仍是同一年只養成一位。

### 獸族男

| 事件 | 觸發／地點 | 條件 |
|---|---|---|
| Episode1 | 行程開始／208 | 女兒年齡 ≤ 13 且 FurryMale 好感階級 ≥ 2 |
| Episode2 | 3301／1205 | 完成對話 FurryMale_Episode1 |
| Episode3 | 行程開始／204 | 完成對話 FurryMale_Episode2 且 FurryMale 好感階級 ≥ 3 |
| Episode4 | 3304／1204 | 完成對話 FurryMale_InterEpi |
| Episode5 | 3301／1205 | 完成對話 FurryMale_Ready |
| Episode6 | 月初／0 | 完成對話 FurryMale_Episode5 且 距 FurryMale_Episode5 完成當月月初 ≥ 0年2月0日 |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成夥伴行程 FurryMale_Episode6 |

### 獸族女

| 事件 | 觸發／地點 | 條件 |
|---|---|---|
| Episode1 | 行程結束前／304 | FurryFemale 好感階級 ≥ 2 |
| Episode2 | 行程結束前／304 | 完成對話 FurryFemale_Episode1 |
| Episode3 | 行程結束前／304 | FurryFemale 好感階級 ≥ 3 |
| Episode4 | 3301／1205 | 完成對話 FurryFemale_Episode3 且 距 FurryFemale_Episode3 完成當月月初 ≥ 0年2月0日 |
| Episode5 | 行程開始／304 | 完成對話 FurryFemale_Bag 且 FurryFemale 好感階級 ≥ 4 |
| Episode6 | 3301／1205 | 已完成事件 FurryFemale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成夥伴行程 FurryFemale_Episode6 |

### 森族男

| 事件 | 觸發／地點 | 條件 |
|---|---|---|
| Episode1 | 行程開始／203 | 已完成事件 ForestMale_FirstEpisode 且 ForestMale 好感階級 ≥ 2 |
| Episode2 | 行程結束前／304 | 完成對話 ForestMale_Episode1 且 ForestMale 好感階級 ≥ 2 |
| Episode3 | 行程開始／306 | 完成對話 ForestMale_Episode2 且 ForestMale 好感階級 ≥ 3 |
| Episode4 | 拜訪(2005)／503 | 完成對話 ForestMale_Episode3 且 女兒年齡 ≥ 14 且 ForestMale 好感階級 ≥ 3 |
| Episode5 | 3202／1303 | 完成對話 ForestMale_Episode4 且 ForestMale 好感階級 ≥ 4 |
| Episode6 | 3202／1303 | 已完成夥伴行程 ForestMale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成夥伴行程 ForestMale_Episode6 |

### 森族女

| 事件 | 觸發／地點 | 條件 |
|---|---|---|
| Episode1 | 行程開始／206 | 女兒年齡 < 14 且 完成對話 ForestFemale_FirstEpisode 且 ForestFemale 好感階級 ≥ 2 |
| Episode2 | 行程開始／206 | 完成對話 ForestFemale_Episode1 |
| Episode3 | 行程結束前／308 | 完成對話 ForestFemale_Episode2 且 ForestFemale 好感階級 ≥ 3 |
| Episode4 | 行程開始／205 | 完成對話 ForestFemale_Episode3 且 女兒年齡 ≥ 14 |
| Episode5 | 行程開始／206 | 完成對話 ForestFemale_Episode4 且 ForestFemale 好感階級 ≥ 4 |
| Episode6 | 3005／1301 | 完成對話 ForestFemale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成夥伴行程 ForestFemale_Episode6 |

### 龍族男

| 事件 | 觸發／地點 | 條件 |
|---|---|---|
| Episode1 | 3103／1105 | 完成對話 DragonMale_FirstEpisode 且 DragonMale 好感階級 ≥ 2 |
| Episode2 | 3101／1103 | 完成對話 DragonMale_Episode1 |
| Episode3 | 行程開始／302 | 完成對話 DragonMale_Episode2 且 距 DragonMale_Episode2 完成當月月初 > 0年3月0日 |
| Episode4 | 3103／1105 | 完成對話 DragonMale_Episode3 且 女兒年齡 ≥ 14 |
| Episode5 | 行程開始／213 | 完成對話 DragonMale_Episode4 且 DragonMale 好感階級 ≥ 4 |
| Episode6 | 3101／1103 | 完成對話 DragonMale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成夥伴行程 DragonMale_Episode6 |

### 龍族女

| 事件 | 觸發／地點 | 條件 |
|---|---|---|
| Episode1 | 行程開始／305 | 完成對話 DragonFemale_FirstEpisode 且 DragonFemale 好感階級 ≥ 2 |
| Episode2 | 行程開始／305 | 完成對話 DragonFemale_Episode1 |
| Episode3 | 行程開始／307 | 已完成對話 DragonFemale_Episode2 且 自 Episode2 完成次月月初起，魔法學與禮法的課程次數合計 ≥ 2（日期偏移參數為 1 年 0 月 0 日，確切區間未確認） |
| Episode4 | 3103／1105 | 完成對話 DragonFemale_Episode3 |
| Episode5 | 行程開始／305 | 完成對話 DragonFemale_Episode4 且 DragonFemale 好感階級 ≥ 4 |
| Episode6 | 3101／1103 | 完成對話 DragonFemale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成夥伴行程 DragonFemale_Episode6 |

### 魔族女（DemonFemale）

| 事件 | 觸發／地點 | 條件 |
|---|---|---|
| Episode1 | 行程開始／303 | 完成對話 DemonFemale_FirstEpisode 且 DemonFemale 好感階級 ≥ 2 |
| Episode2 | 行程開始／212 | 女僕咖啡廳 累計次數 ≥ 1 且 完成對話 DemonFemale_Episode1 且 已完成事件 DemonFemale_Encounter_Dance |
| Episode3 | 王城遇人／710 | 完成對話 DemonFemale_Episode2 且 DemonFemale 好感階級 ≥ 3 |
| Episode4 | 行程開始／309 | 完成對話 DemonFemale_Episode3 且 DemonFemale 好感階級 ≥ 3 |
| Episode5 | 王城遇人／710 | 完成對話 DemonFemale_Episode4 且 DemonFemale 好感階級 ≥ 4 且 已完成事件 DemonFemale_DemonEvent |
| Episode6 | 王城遇人／710 | 完成對話 DemonFemale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成夥伴行程 DemonFemale_Episode6 |

### 方塊與其他 NPC（Cube、Caim、Icosa、Kimaris、Muzai）

這 5 位沒有求婚事件，也沒有「接受」的對話圖。圖裡只有劇情（`Episode1`，方塊有 1、2）與婚禮對話；成為結婚對象是結局時由 `pk::gde` 依「完成哪個事件」加「職業結局」決定（見上面「結局時的最終判定」）。他們的劇情事件條件如下：

| 事件 | 觸發 | 條件 |
|---|---|---|
| Cube_Episode1 | 1002（地點 201 家） | 與吉普的關係 ≥ 70 |
| Cube_Episode2 | 1002（地點 201 家） | 已完成事件 Cube_Episode1 且 與吉普的關係 ≥ 120 |
| Caim_Episode1（凱姆，餐廳店長） | 1017 行程後段、優先度 30 | 已完成事件 Job_Restaurant_Ep3 且 本月行程為 餐廳 |
| Icosa_Episode1（艾科薩，魔法學院） | 1017、優先度 30 | 已完成事件 Edu_Magic_Ep3 且 本月行程為 魔法學 |
| Kimaris_Episode1（奇馬里斯，檢問處） | 1017、優先度 30 | 已完成事件 Job_Checkpoint_Ep3 且 本月行程為 檢問處 |
| Muzai_Episode1（穆扎伊，文學老師） | 1017、優先度 30 | 已完成事件 Edu_Literature_Ep3 且 本月行程為 文學 |

四位老闆／老師的 Episode1 都只能觸發一次（MaxCount 1），而且要在**那個行程進行的那個月**才會出現，所以要先把對應的 `_Ep3` 打完，並且之後再排一次同樣的行程。吉普的「與吉普的關係」是 Status 21（範圍 -100~500）。整個資料庫與全部流程圖中，只有 `E_Job_House_Ep1/2/3`（家事打工的三段事件）會加它：每段不論選哪個選項都 +1，合計只有 3，達不到 70／120，所以主要增加來源不在資料或圖裡，應該在 C# 程式（例如家事類行程每次執行時結算）。程式端已確認：`hl::sim(EStatus,int)` 是通用的狀態加減（上下限取 Status 表的 Min／Max，Cube 為 -100~500），沒有針對 Cube 的特殊處理；`hl::dfs` 是「設成指定值」，呼叫者多為除錯或介面。資料端：`Schedule` 表的 `Job_House` 在 `NeedStatCritical` 欄使用 StatType 21（Cube，範圍 10~80），應是「大成功判定用的能力值」（推測），不是增加好感度的來源；其他表沒有以代號 21 引用 Cube。**來源已找到（程式碼確認）**：`ue::cez::MoveNext`（行程執行流程）在行程開始時，如果該行程的地點是 201（0xC9）就執行 `hl::sim(RelationshipCube, 結束日 - 開始日 + 1)`。資料表裡地點為 201 的行程只有 `Job_House`（家事打工），所以**每做一次家事打工，與吉普的關係就增加「該次行程天數」**，在行程開始時就結算，與成功、失敗、大成功無關。（一次行程的天數由 `mbg::sfv` 決定：每月 3 個行程格，上旬、中旬各 10 天，下旬為「當月天數 − 20」天，也就是 30 天的月份 10 天、31 天 11 天、2 月 8～9 天。所以一次家事打工通常 +10：累積 70 需要 7 次，累積 120 需要 12 次；三段事件各 +1 合計只有 +3，省不下任何一次。也就是每個月三格都排家事打工，約 3 個月多（7 格）達 70、約 4 個月（12 格）達 120。）另外，`E_Job_House_Ep1/2/3` 三段事件每段再 +1。其他途徑沒有：`ue::ulx`（行程結算只套用 Stats 欄，Job_House 為 3／11／6，大成功加倍）、送禮（`Partner_Npc_Cube` 的 FavorStatus 為 0）、道具／茶／飲食、每月結算 `uk::cdv`（只動體力、心智、疲勞、金錢）都不會加。

**「一般婚姻」與「單身」**：`D_Village_Wedding`、`D_Job_Wedding`、`D_None_Wedding` 對應 -1、-3、-2。沒有事件設定它們，是 `pk::gde` 在結局時依上面第 6 到 8 條給值。

## 還缺的部分

1. `Event_DemonKing_Allow`、`DemonMale_Proposal_Allow` 這兩個旗標是對話選項設定的，上面已確認。
2. `Score` 欄是占卜師進度的加權，`Weight` 是最終判定的優先權，已由程式碼確認。
3. 課程／打工累積值 `lcq` 已確認是跟 `NeedCount_GradeUp` 比較的累積次數；`yaw` 與 `lcr` 兩個欄位的意義（`yaw` 用於 Type 9 課程次數）未細查。
4. 同一個月多位夥伴同時符合求婚時的先後順序未確認；「與吉普的關係」來源與天數都已確認（家事打工每次 +10，下旬依當月天數 +8～11）。結婚對象預設值 -3 已確認是 `do::do()` 建構函式寫入；另外 `ei::hbo`（會顯示「求婚相關資料已初始化」的韓文提示，像是除錯或重置工具）與 `du::cga`／`du::lzk`（把結局條件直接套到存檔的除錯功能）也會寫它，正常遊玩不會走到。
5. Flow_Vacance（度假事件）沒有解析。
6. 王城遇人事件的 Place（703 到 708）對應的場景名稱，在程式碼裡。

---

# 各結局完整條件（依優先權由高到低）

## 女王（Queen）  優先權 Weight 52000
**結局條件（全部都要滿足）**
- 社交評價 ≥ 500
- 下列任一：戰士評價 ≥ 400 或 魔法評價 ≥ 400
- 體力＋力量＋敏捷 ≥ 1500
- 智力＋魔力＋精神力 ≥ 1500
- 氣質＋禮儀＋口才 ≥ 2400
- 感性＋品味＋魅力 ≥ 1500
- 性向 ≥ 225

**前置事件鏈**
- `Flow_Queen_Ep1`：與 Castle_RegentDeputy 見面次數 ≥ 3；口才 ≥ 200
- `Flow_Queen_Ep2`：已完成事件 Flow_Queen_Ep1；距 Flow_Queen_Ep1 完成當月月初 ≥ 0年2月0日
- `Flow_Queen_Ep3`：已完成事件 Flow_Queen_Ep2；戰士三項(體力+力量+敏捷)總和 ≥ 840；戰士評價 ≥ 245
- `Flow_Queen_Ep3`：已完成事件 Flow_Queen_Ep2；戰士三項(體力+力量+敏捷)總和 ≥ 840；【不可】戰士評價 ≥ 245；魔法三項(智力+魔力+精神)總和 ≥ 840；戰士評價 ≥ 245
- `Flow_Queen_Ep3`：已完成事件 Flow_Queen_Ep2；【不可】戰士三項(體力+力量+敏捷)總和 ≥ 840；魔法三項(智力+魔力+精神)總和 ≥ 840；戰士評價 ≥ 245
- `Flow_Queen_Ep4`：已完成事件 Flow_Queen_Ep3；距 Flow_Queen_Ep3 完成當月月初 ≥ 0年2月0日
- `Flow_Queen_Ep5`：已完成事件 Flow_Queen_Ep4
- `Flow_Queen_Ep6`：已完成事件 Flow_Queen_Ep5；戰士評價 ≥ 400；社交評價 ≥ 500；戰士三項(體力+力量+敏捷)總和 ≥ 1500；魔法三項(智力+魔力+精神)總和 ≥ 1500；社交三項(氣質+禮儀+口才)總和 ≥ 2400；藝術三項(感受性+品味+魅力)總和 ≥ 1500；性向 ≥ 225
- `Flow_Queen_Ep6`：已完成事件 Flow_Queen_Ep5；【不可】戰士評價 ≥ 400；魔法評價 ≥ 400；社交評價 ≥ 500；戰士三項(體力+力量+敏捷)總和 ≥ 1500；魔法三項(智力+魔力+精神)總和 ≥ 1500；社交三項(氣質+禮儀+口才)總和 ≥ 2400；藝術三項(感受性+品味+魅力)總和 ≥ 1500；性向 ≥ 225
- `Flow_Queen_Ep7`：已完成事件 Flow_Queen_Ep6

## 魔王（DemonKing）  優先權 Weight 51000
**結局條件（全部都要滿足）**
- 完成事件 Event_DemonKing_Allow
- 性向 ≤ -225
- 體力＋力量＋敏捷 ≥ 2400
- 智力＋魔力＋精神力 ≥ 2400

**前置事件鏈**
- `Flow_DemonKing_Ep1`：性向 ≤ -150；戰士三項(體力+力量+敏捷)總和 ≥ 800；魔法三項(智力+魔力+精神)總和 ≥ 800
- `Flow_DemonKing_Ep2`：已完成事件 Flow_DemonKing_Ep1；性向 ≤ -150；戰士三項(體力+力量+敏捷)總和 ≥ 1200；魔法三項(智力+魔力+精神)總和 ≥ 1200
- `Flow_DemonKing_Ep3`：已完成事件 Flow_DemonKing_Ep2
- `Flow_DemonKing_Ep4`：已完成事件 Flow_DemonKing_Ep3；性向 ≤ -150；戰士三項(體力+力量+敏捷)總和 ≥ 2000；魔法三項(智力+魔力+精神)總和 ≥ 2000
- `Flow_DemonKing_Ep5`：已完成事件 Flow_DemonKing_Ep4；性向 ≤ -225；戰士三項(體力+力量+敏捷)總和 ≥ 2400；魔法三項(智力+魔力+精神)總和 ≥ 2400；年份 = 1218；月份 = 2

## 公主（Princess）  優先權 Weight 50000
**結局條件（全部都要滿足）**
- 完成事件 DemonMale_Proposal_Allow
- 社交評價 ≥ 400
- 藝術評價 ≥ 400
- 氣質＋禮儀＋口才 ≥ 2400
- 感性＋品味＋魅力 ≥ 2400
- 性向 ≥ -149

**前置事件鏈**
- `Flow_Princess_Ep1`：與 Castle_Consort 見面次數 ≥ 3；圖變數 KarenRank = 1；完成對話 DemonMale_Episode3
- `Flow_Princess_Ep2`：完成對話 Flow_Princess_Ep1；魅力 ≥ 400
- `Flow_Princess_Ep3`：完成對話 Flow_Princess_Ep2；社交三項(氣質+禮儀+口才)總和 ≥ 2000
- `Flow_Princess_Ep4`：完成對話 Flow_Princess_Ep3
- `Flow_Princess_Ep5`：完成對話 Flow_Princess_Ep4
- `Flow_Princess_Ep6`：完成對話 Flow_Princess_Ep5；目前滿足結局條件 Ending_Princess
- `Flow_Princess_Ep7`：完成對話 Flow_Princess_Ep6

## 大將軍（General）  優先權 Weight 43000
**結局條件（全部都要滿足）**
- 武鬥會奪冠
- 劍術達 S（等級 5）
- 體術達 S（等級 5）
- 戰士評價 ≥ 350
- 體力＋力量＋敏捷 ≥ 2400
- 氣質＋禮儀 ≥ 600
- 完成事件 Flow_General_Ep3
- 性向 ≥ -149

**前置事件鏈**
- `Flow_General_Ep0`：與 Castle_RegentDeputy 見面次數 > 0；Festival_Battle 歷史符合 [Rank=1&Weapon=Physical]；已完成事件 Flow_CapitalEliteTroop_Ep2；性向 ≥ -149
- `Flow_General_Ep1`：已完成事件 Flow_General_Ep0
- `Flow_General_Ep2`：已完成事件 Flow_General_Ep1；戰士評價階級 ≥ 3；體術等級 ≥ 5；劍術等級 ≥ 5；性向 ≥ -149
- `Flow_General_Ep3`：已完成事件 Flow_General_Ep2

## 王宮魔法師（RoyalWizard）  優先權 Weight 42000
**結局條件（全部都要滿足）**
- 鍊金術達 S（等級 5）
- 魔法學達 S（等級 5）
- 完成事件 Flow_RoyalWizard_Ep3
- 魔法評價 ≥ 350
- 智力＋魔力＋精神力 ≥ 2400
- 性向 ≥ -149

**前置事件鏈**
- `Flow_RoyalWizard_Ep1`：與 Castle_Archmage 見面次數 ≥ 1；魔法評價階級 > 4；鍊金術等級 ≥ 3；魔法學等級 ≥ 4
- `Flow_RoyalWizard_Ep2`：已完成事件 Flow_RoyalWizard_Ep1；任務條件達成 Quest_RoyalWizard1
- `Flow_RoyalWizard_Ep3`：已完成事件 Flow_RoyalWizard_Ep2；任務條件達成 Quest_RoyalWizard2

## 王立藝術學會會長（RAAPresident）  優先權 Weight 41000
**結局條件（全部都要滿足）**
- 美術達 A（等級 4）
- 舞蹈達 A（等級 4）
- 音樂達 A（等級 4）
- 藝術評價 ≥ 350
- 感性＋品味＋魅力 ≥ 2400
- 藝術節奪冠
- 性向 ≥ -149

**前置事件鏈**
- `Flow_RAAPresident_Ep1`：與 Castle_Consort 見面次數 > 2；【不可】完成任務 Quest_RAAPresident；Festival_Art 歷史符合 [Rank=1]
- `Flow_RAAPresident_Ep2`：完成任務 Quest_RAAPresident

## 大富豪（Rich）  優先權 Weight 40000
**結局條件（全部都要滿足）**
- 持有金錢 ≥ 50000
- 完成事件 Flow_Rich_Ep4

**前置事件鏈**
- `Flow_Rich_Ep1`：持有金錢 ≥ 20000
- `Flow_Rich_Ep2`：已完成事件 Event_Rich_Invest；距 Flow_Rich_Ep1 完成當月月初 ≥ 0年3月0日
- `Flow_Rich_Ep3`：已完成事件 Flow_Rich_Ep2；距 Flow_Rich_Ep2 完成當月月初 ≥ 0年4月0日
- `Flow_Rich_Ep4`：已完成事件 Flow_Rich_Ep3；距 Flow_Rich_Ep3 完成當月月初 ≥ 0年8月0日

## 魔法劍士（MagicKnight）  優先權 Weight 40000
**結局條件（全部都要滿足）**
- 戰士評價階級 ≥ 4（評價 ≥ 250）
- 魔法評價階級 ≥ 4（評價 ≥ 250）
- 完成事件 Flow_MagicKnight_Ep1
- 魔力 ≥ 600
- 力量 ≥ 600

**前置事件鏈**
- `Flow_MagicKnight_Ep1`：戰士評價 ≥ 2；魔法學等級 ≥ 3；符合 [Rank!=1&Weapon=Physical]
- `Flow_MagicKnight_Ep2`：已完成事件 Flow_MagicKnight_Ep1；符合 [Rank=1]

## 外交官（Diplomat）  優先權 Weight 40000
**結局條件（全部都要滿足）**
- 完成事件 Flow_Diplomat_Ep3
- 社交評價 ≥ 350
- 氣質＋禮儀＋口才 ≥ 1800
- 智力 ≥ 500
- 性向 ≥ -149

**前置事件鏈**
- `Flow_Diplomat_Ep1`：社交三項(氣質+禮儀+口才)總和 ≥ 1200；社交評價階級 ≥ 3；智力 ≥ 100；性向 ≥ -149
- `Flow_Diplomat_Ep2`：已完成事件 Flow_Diplomat_Ep1
- `Flow_Diplomat_Ep3`：已完成事件 Flow_Diplomat_Ep2；社交三項(氣質+禮儀+口才)總和 ≥ 1500；智力 ≥ 300

## 沙龍說客（Sting）  優先權 Weight 34000
**結局條件（全部都要滿足）**
- 性向 ≤ -150
- 社交評價 ≥ 350
- 氣質＋禮儀＋口才 ≥ 1600
- 智力 ≥ 500

**前置事件鏈**
- `Flow_Sting_Ep1`：社交評價階級 ≥ 3
- `Flow_Sting_Ep2`：已完成事件 Flow_Sting_Ep1；性向 ≤ -150
- `Flow_Sting_Ep3`：已完成事件 Flow_Sting_Ep2；智力 > 400；性向 ≤ -150

## 蛇蠍美人（Femme）  優先權 Weight 33000
**結局條件（全部都要滿足）**
- 性向 ≤ -150
- 酒館達 S（等級 5）
- 魅力 ≥ 600
- 魅力 ≥ 400
- 魅力 ≥ 600

**前置事件鏈**
- `Flow_Femme_Ep1`：酒館 累計次數 ≥ 10；性向 ≤ -90；魅力 > 100
- `Flow_Femme_Ep2`：已完成事件 Flow_Femme_Ep1；酒館等級 ≥ 4；魅力 > 300；性向 ≤ -120
- `Flow_Femme_Ep3`：已完成事件 Flow_Femme_Ep2；酒館 累計次數 ≥ 25；魅力 > 500；性向 ≤ -150

## 混沌鍊金術師（ChaosAlchemist）  優先權 Weight 32000
**結局條件（全部都要滿足）**
- 性向 ≤ -150
- 鍊金術達 A（等級 4）
- 智力＋魔力＋精神力 ≥ 1500
- 品味 ≥ 300

**前置事件鏈**
- `Flow_ChaosAlchemist_Ep1`：鍊金術等級 ≥ 3
- `Flow_ChaosAlchemist_Ep2`：已完成事件 Flow_ChaosAlchemist_Ep1；鍊金術等級 ≥ 4；性向 ≤ -10
- `Flow_ChaosAlchemist_Ep3`：已完成事件 Flow_ChaosAlchemist_Ep2；性向 ≤ -90

## 怪盜（MysteriousThief）  優先權 Weight 32000
**結局條件（全部都要滿足）**
- 性向 ≤ -150
- 智力 ≥ 300
- 敏捷 ≥ 600
- 品味 ≥ 600
- 完成事件 Flow_MysteriousThief_Ep3

**前置事件鏈**
- `Flow_MysteriousThief_Ep1`：敏捷 ≥ 400
- `Flow_MysteriousThief_Ep2`：【不可】已完成事件 Event_MysteriousThief_Prison；已完成事件 Flow_MysteriousThief_Ep1；品味 ≥ 400；性向 ≤ -150
- `Flow_MysteriousThief_Ep3`：已完成事件 Flow_MysteriousThief_Ep2；智力 ≥ 200；性向 ≤ -150

## 占卜師（FortuneTeller）  優先權 Weight 31000
**結局條件（全部都要滿足）**
- 魔法評價階級 ≥ 4（評價 ≥ 250）
- 性向 ≤ -150
- 精神力 ≥ 400
- 口才 ≥ 400
- 感性 ≥ 400

**前置事件鏈**
- `Flow_FortuneTeller_Ep1`：月份 ≠ 4；口才 > 200；感性 > 200；精神力 > 200；性向 ≤ -150
- `Flow_FortuneTeller_Ep2`：月份 ≠ 4；口才 > 300；精神力 > 300；感性 > 300；性向 ≤ -150

## 後巷之王（TheGodfather）  優先權 Weight 31000
**結局條件（全部都要滿足）**
- 性向 ≤ -150
- 力量 ≥ 800
- 體力 ≥ 400
- 戰士評價 ≥ 350

**前置事件鏈**
- `Flow_TheGodfather_Ep1`：已完成事件 Flow_UnloadingManager_Ep1；戰士評價階級 ≥ 2
- `Flow_TheGodfather_Ep2`：已完成事件 Flow_TheGodfather_Ep1；【不可】已完成事件 Flow_TheGodfather2_Ep2；戰士評價階級 ≥ 3

## 魔導工程師（MagicEngineer）  優先權 Weight 27000
**結局條件（全部都要滿足）**
- 魔法學達 A（等級 4）
- 鍊金術達 A（等級 4）
- 智力＋魔力＋精神力 ≥ 2000

**前置事件鏈**
- `Flow_MagicEngineer_Ep1`：本月行程為 鍊金術；魔法學 累計次數 ≥ 13；鍊金術 累計次數 ≥ 13；魔法三項(智力+魔力+精神)總和 ≥ 1500
- `Flow_MagicEngineer_Ep2`：本月行程為 鍊金術；魔法學 累計次數 ≥ 18；鍊金術 累計次數 ≥ 18；魔法三項(智力+魔力+精神)總和 ≥ 1800

## 劍舞姬（Sworddancer）  優先權 Weight 27000
**結局條件（全部都要滿足）**
- 劍術達 A（等級 4）
- 舞蹈達 B（等級 3）
- 魅力 ≥ 500

**前置事件鏈**
- `Flow_Sworddancer_Ep1`：本月行程為 劍術；劍術 累計次數 ≥ 12；舞蹈 累計次數 ≥ 8；敏捷 ≥ 150；已完成事件 StatCheck_Sword_Ep1
- `Flow_Sworddancer_Ep2`：本月行程為 舞蹈；劍術 累計次數 ≥ 16；舞蹈 累計次數 ≥ 10；魅力 ≥ 450；已完成事件 Flow_Sworddancer_Ep1

## 調香師（Perfumer）  優先權 Weight 27000
**結局條件（全部都要滿足）**
- 植物園達 A（等級 4）
- 鍊金術達 B（等級 3）
- 氣質 ≥ 500

**前置事件鏈**
- `Flow_Perfumer_Ep1`：本月行程為 植物園；植物園 累計次數 ≥ 15；鍊金術 累計次數 ≥ 8；感性 ≥ 150；已完成事件 StatCheck_Garden_Ep1
- `Flow_Perfumer_Ep2`：本月行程為 鍊金術；植物園 累計次數 ≥ 25；鍊金術 累計次數 ≥ 10；氣質 ≥ 450；已完成事件 Flow_Perfumer_Ep1

## 貴金屬工匠（Jeweler）  優先權 Weight 27000
**結局條件（全部都要滿足）**
- 鐵匠鋪達 A（等級 4）
- 美術達 B（等級 3）
- 藝術評價 ≥ 100
- 氣質 ≥ 500

**前置事件鏈**
- `Flow_Jeweler_Ep1`：本月行程為 鐵匠鋪；鐵匠鋪 累計次數 ≥ 12；美術 累計次數 ≥ 8；體力 ≥ 150
- `Flow_Jeweler_Ep2`：本月行程為 鐵匠鋪；鐵匠鋪 累計次數 ≥ 16；美術 累計次數 ≥ 10；藝術評價 ≥ 100；氣質 ≥ 450；已完成事件 Flow_Jeweler_Ep1

## 吟遊詩人（Bard）  優先權 Weight 27000
**結局條件（全部都要滿足）**
- 音樂達 A（等級 4）
- 文學達 B（等級 3）
- 完成事件 Flow_Bard_Ep3
- 口才 ≥ 500

**前置事件鏈**
- `Flow_Bard_Ep1`：本月行程為 文學；文學 累計次數 ≥ 8；音樂 累計次數 ≥ 10；感性 ≥ 150
- `Flow_Bard_Ep2`：本月行程為 音樂；音樂 累計次數 ≥ 16；文學 累計次數 ≥ 10；口才 ≥ 450；已完成事件 Flow_Bard_Ep1
- `Flow_Bard_Ep3`：口才 ≥ 450；已完成事件 Flow_Bard_Ep2

## 魔法陣圖案師（SigilDrafter）  優先權 Weight 27000
**結局條件（全部都要滿足）**
- 魔法學達 A（等級 4）
- 美術達 B（等級 3）
- 藝術評價 ≥ 100
- 感性 ≥ 500

**前置事件鏈**
- `Flow_SigilDrafter_Ep1`：本月行程為 魔法學；魔法學 累計次數 ≥ 12；美術 累計次數 ≥ 8；魔力 ≥ 150
- `Flow_SigilDrafter_Ep2`：本月行程為 美術；魔法學 累計次數 ≥ 16；美術 累計次數 ≥ 10；藝術評價 ≥ 100；感性 ≥ 450；已完成事件 Flow_SigilDrafter_Ep1

## 首席芭蕾舞者（Dancer）  優先權 Weight 23000
**結局條件（全部都要滿足）**
- 舞蹈達 A（等級 4）
- 魅力 ≥ 300
- 品味 ≥ 300
- 氣質 ≥ 300
- 藝術節舞蹈賽奪冠

**前置事件鏈**
- `Flow_Dancer_Ep1`：本月行程為 舞蹈；舞蹈 累計次數 ≥ 22；氣質 ≥ 100
- `Flow_Dancer_Ep2`：本月行程為 舞蹈；舞蹈 累計次數 ≥ 27；氣質 ≥ 200

## 首席歌姬（PrimaDonna）  優先權 Weight 23000
**結局條件（全部都要滿足）**
- 音樂達 A（等級 4）
- 感性 ≥ 300
- 魅力 ≥ 300
- 口才 ≥ 300
- 藝術節音樂賽奪冠

**前置事件鏈**
- `Flow_PrimaDonna_Ep1`：本月行程為 音樂；音樂 累計次數 ≥ 22；口才 ≥ 100
- `Flow_PrimaDonna_Ep2`：本月行程為 音樂；音樂 累計次數 ≥ 27；口才 ≥ 200

## 學院院長（AcademyDean）  優先權 Weight 23000
**結局條件（全部都要滿足）**
- 學院達 A（等級 4）
- 智力 ≥ 500
- 口才 ≥ 300

**前置事件鏈**
- `Flow_AcademyDean_Ep1`：本月行程為 學院；學院 累計次數 ≥ 23；智力 ≥ 200；品味 ≥ 150
- `Flow_AcademyDean_Ep2`：本月行程為 學院；學院 累計次數 ≥ 28；智力 ≥ 300；已完成事件 Flow_AcademyDean_Ep1
- `Flow_AcademyDean_Ep3`：本月行程為 學院；智力 ≥ 450；品味 ≥ 200；口才 ≥ 270；已完成事件 Flow_AcademyDean_Ep2

## 廚師（Cook）  優先權 Weight 22500
**結局條件（全部都要滿足）**
- 餐廳達 S（等級 5）
- 廚藝祭奪冠
- 品味 ≥ 500
- 魔力 ≥ 200

**前置事件鏈**
- `Flow_Cook_Ep1`：本月行程為 餐廳；餐廳 累計次數 ≥ 38
- `Flow_Cook_Ep2`：本月行程為 餐廳；餐廳 累計次數 ≥ 45；Festival_Cook 優勝次數 ≥ 1

## 魔法事務員（MagicClerk）  優先權 Weight 20900
**結局條件（全部都要滿足）**
- 魔法評價階級 ≥ 4（評價 ≥ 250）
- 魔法學達 A（等級 4）
- 智力＋魔力＋精神力 ≥ 1500
- 完成事件 Flow_MagicClerk_Ep2

**前置事件鏈**
- `Flow_MagicClerk_Ep1`：本月行程為 魔法學；魔法學 累計次數 ≥ 10；Castle_MagicClerk 拜訪次數 ≥ 3
- `Flow_MagicClerk_Ep2`：已完成事件 Flow_MagicClerk_Ep1；魔法學 累計次數 ≥ 10；Castle_MagicClerk 拜訪次數 ≥ 3

## 首都精銳兵團（CapitalEliteTroop）  優先權 Weight 20850
**結局條件（全部都要滿足）**
- 戰士評價階級 ≥ 4（評價 ≥ 250）
- 性向 ≥ 30
- 體力＋力量＋敏捷 ≥ 1500

**前置事件鏈**
- `Flow_CapitalEliteTroop_Ep1`：本月行程為 自由活動；劍術等級 ≥ 3
- `Flow_CapitalEliteTroop_Ep2`：已完成事件 Flow_CapitalEliteTroop_Ep1；力量 > 400

## 冒險家（Adventurer）  優先權 Weight 20700
**結局條件（全部都要滿足）**
- 戰士評價階級 ≥ 4（評價 ≥ 250）
- 體力 ≥ 400
- 力量 ≥ 400
- 探險過龍族地區
- 探險過森族地區
- 探險過獸族地區

**前置事件鏈**
- `Flow_Adventurer_Ep1`：擊殺怪物數(全部) > 10
- `Flow_Adventurer_Ep2`：已完成事件 Flow_Adventurer_Ep1；任務條件達成 Quest_Adventurer1
- `Flow_Adventurer_Ep3`：已完成事件 Flow_Adventurer_Ep2；戰士評價 > 3
- `Flow_Adventurer_Ep4`：已完成事件 Flow_Adventurer_Ep3；任務條件達成 Quest_Adventurer2

## 劍術道場教頭（SwordInstructor）  優先權 Weight 20495
**結局條件（全部都要滿足）**
- 劍術達 B（等級 3）
- 敏捷 ≥ 300
- 體力 ≥ 300

**前置事件鏈**
- `Flow_SwordInstructor_Ep1`：本月行程為 劍術；劍術 累計次數 ≥ 12
- `Flow_SwordInstructor_Ep2`：本月行程為 劍術；劍術 累計次數 ≥ 20；敏捷 ≥ 270；體力 ≥ 270

## 作家（Writer）  優先權 Weight 20495
**結局條件（全部都要滿足）**
- 文學達 B（等級 3）
- 口才 ≥ 300
- 感性 ≥ 300

**前置事件鏈**
- `Flow_Writer_Ep1`：本月行程為 文學；文學 累計次數 ≥ 12
- `Flow_Writer_Ep2`：本月行程為 文學；文學 累計次數 ≥ 20；口才 ≥ 270；感性 ≥ 270

## 畫家（Painter）  優先權 Weight 20495
**結局條件（全部都要滿足）**
- 美術達 B（等級 3）
- 品味 ≥ 300
- 感性 ≥ 300

**前置事件鏈**
- `Flow_Painter_Ep1`：本月行程為 美術；美術 累計次數 ≥ 12
- `Flow_Painter_Ep2`：本月行程為 美術；美術 累計次數 ≥ 20；品味 ≥ 270；感性 ≥ 270

## 舞蹈教師（DanceTutor）  優先權 Weight 20495
**結局條件（全部都要滿足）**
- 舞蹈達 B（等級 3）
- 魅力 ≥ 300
- 品味 ≥ 300

**前置事件鏈**
- `Flow_DanceTutor_Ep1`：本月行程為 舞蹈；舞蹈 累計次數 ≥ 12
- `Flow_DanceTutor_Ep2`：本月行程為 舞蹈；舞蹈 累計次數 ≥ 20；魅力 ≥ 270；品味 ≥ 270

## 音樂教師（MusicTutor）  優先權 Weight 20495
**結局條件（全部都要滿足）**
- 音樂達 B（等級 3）
- 感性 ≥ 300
- 魅力 ≥ 300

**前置事件鏈**
- `Flow_MusicTutor_Ep1`：本月行程為 音樂；音樂 累計次數 ≥ 12
- `Flow_MusicTutor_Ep2`：本月行程為 音樂；音樂 累計次數 ≥ 20；感性 ≥ 270；魅力 ≥ 270

## 歷史學者（Historian）  優先權 Weight 20495
**結局條件（全部都要滿足）**
- 歷史達 B（等級 3）
- 智力 ≥ 300
- 氣質 ≥ 300

**前置事件鏈**
- `Flow_Historian_Ep1`：本月行程為 歷史；歷史 累計次數 ≥ 12
- `Flow_Historian_Ep2`：本月行程為 歷史；歷史 累計次數 ≥ 20；智力 ≥ 270；氣質 ≥ 270

## 魔法教習（MagicTutor）  優先權 Weight 20495
**結局條件（全部都要滿足）**
- 魔法學達 B（等級 3）
- 魔力 ≥ 300
- 智力 ≥ 300

**前置事件鏈**
- `Flow_MagicTutor_Ep1`：本月行程為 魔法學；魔法學 累計次數 ≥ 12
- `Flow_MagicTutor_Ep2`：本月行程為 魔法學；魔法學 累計次數 ≥ 20；智力 ≥ 270；魔力 ≥ 270

## 禮法教師（Governess）  優先權 Weight 20495
**結局條件（全部都要滿足）**
- 禮法達 A（等級 4）
- 禮儀 ≥ 400
- 氣質 ≥ 400
- 感性 ≥ 200

**前置事件鏈**
- `Flow_Governess_Ep1`：本月行程為 禮法；禮法 累計次數 ≥ 12
- `Flow_Governess_Ep2`：本月行程為 禮法；禮法 累計次數 ≥ 20；禮儀 ≥ 350；氣質 ≥ 350

## 體術教頭（MartialArtsInstructor）  優先權 Weight 20495
**結局條件（全部都要滿足）**
- 體術達 B（等級 3）
- 精神力 ≥ 300
- 體力 ≥ 300

**前置事件鏈**
- `Flow_MartialArtsInstructor_Ep1`：本月行程為 體術；體術 累計次數 ≥ 12；精神力 ≥ 100；已完成事件 StatCheck_MartialArts_Ep1
- `Flow_MartialArtsInstructor_Ep2`：本月行程為 體術；體術 累計次數 ≥ 20；精神力 ≥ 270；體力 ≥ 270；已完成事件 Flow_MartialArtsInstructor_Ep1

## 鍊金術師（Alchemist）  優先權 Weight 20495
**結局條件（全部都要滿足）**
- 鍊金術達 B（等級 3）
- 智力 ≥ 300
- 魔力 ≥ 300

**前置事件鏈**
- `Flow_Alchemist_Ep1`：本月行程為 鍊金術；鍊金術 累計次數 ≥ 12；智力 ≥ 100
- `Flow_Alchemist_Ep2`：本月行程為 鍊金術；鍊金術 累計次數 ≥ 20；智力 ≥ 270；魔力 ≥ 270；已完成事件 Flow_Alchemist_Ep1

## 物流主管（UnloadingManager）  優先權 Weight 20240
**結局條件（全部都要滿足）**
- 碼頭達 A（等級 4）
- 體力 ≥ 300
- 精神力 ≥ 300

**前置事件鏈**
- `Flow_UnloadingManager_Ep1`：本月行程為 碼頭；碼頭 累計次數 ≥ 10
- `Flow_UnloadingManager_Ep2`：本月行程為 碼頭；碼頭等級 ≥ 4；體力 ≥ 270；精神力 ≥ 270

## 調酒師（Bartender）  優先權 Weight 20240
**結局條件（全部都要滿足）**
- 酒館達 A（等級 4）
- 口才 ≥ 300
- 氣質 ≥ 300

**前置事件鏈**
- `Flow_Bartender_Ep1`：本月行程為 酒館；酒館 累計次數 ≥ 10
- `Flow_Bartender_Ep2`：本月行程為 酒館；酒館等級 ≥ 4；口才 ≥ 270；氣質 ≥ 270

## 家庭教師（PrivateTutor）  優先權 Weight 20200
**結局條件（全部都要滿足）**
- 家庭教師達 A（等級 4）
- 智力 ≥ 300
- 禮儀 ≥ 300

**前置事件鏈**
- `Flow_PrivateTutor_Ep1`：本月行程為 家庭教師；家庭教師 累計次數 ≥ 10
- `Flow_PrivateTutor_Ep2`：本月行程為 家庭教師；家庭教師等級 ≥ 4；智力 ≥ 270；禮儀 ≥ 270

## 女僕咖啡廳的超級明星（Superstar）  優先權 Weight 20200
**結局條件（全部都要滿足）**
- 女僕咖啡廳達 A（等級 4）
- 魅力 ≥ 300
- 感性 ≥ 300
- 口才 ≥ 200

**前置事件鏈**
- `Flow_Superstar_Ep1`：本月行程為 女僕咖啡廳；女僕咖啡廳 累計次數 ≥ 10；魅力 ≥ 300；感性 ≥ 150
- `Flow_Superstar_Ep2`：本月行程為 女僕咖啡廳；女僕咖啡廳等級 ≥ 4；魅力 ≥ 500；感性 ≥ 300；口才 ≥ 150

## 研究助理（Assistant）  優先權 Weight 20200
**結局條件（全部都要滿足）**
- 學院達 B（等級 3）
- 智力 ≥ 300
- 品味 ≥ 300

**前置事件鏈**
- `Flow_Assistant_Ep1`：本月行程為 學院；學院 累計次數 ≥ 10
- `Flow_Assistant_Ep2`：本月行程為 學院；學院 累計次數 ≥ 19；智力 ≥ 270；品味 ≥ 270

## 服裝設計師（Designer）  優先權 Weight 20160
**結局條件（全部都要滿足）**
- 服飾店達 A（等級 4）
- 氣質 ≥ 300
- 魅力 ≥ 300

**前置事件鏈**
- `Flow_Designer_Ep1`：本月行程為 服飾店；服飾店 累計次數 ≥ 11
- `Flow_Designer_Ep2`：本月行程為 服飾店；服飾店 累計次數 ≥ 24；氣質 ≥ 280；魅力 ≥ 280

## 鐵匠（Blacksmith）  優先權 Weight 20160
**結局條件（全部都要滿足）**
- 鐵匠鋪達 A（等級 4）
- 體力 ≥ 300
- 魔力 ≥ 300

**前置事件鏈**
- `Flow_Blacksmith_Ep1`：本月行程為 鐵匠鋪；鐵匠鋪 累計次數 ≥ 12
- `Flow_Blacksmith_Ep2`：本月行程為 鐵匠鋪；鐵匠鋪 累計次數 ≥ 24；體力 ≥ 270；魔力 ≥ 270

## 士兵（Soldier）  優先權 Weight 20080
**結局條件（全部都要滿足）**
- 檢問處達 A（等級 4）
- 禮儀 ≥ 300
- 體力 ≥ 300

**前置事件鏈**
- `Flow_Soldier_Ep1`：本月行程為 檢問處；檢問處 累計次數 ≥ 11
- `Flow_Soldier_Ep2`：本月行程為 檢問處；檢問處等級 ≥ 4；禮儀 ≥ 270；體力 ≥ 270

## 物流管理員（Logistics）  優先權 Weight 20080
**結局條件（全部都要滿足）**
- 水果倉庫達 A（等級 4）
- 力量 ≥ 300
- 精神力 ≥ 300

**前置事件鏈**
- `Flow_Logistics_Ep1`：本月行程為 水果倉庫；水果倉庫 累計次數 ≥ 13
- `Flow_Logistics_Ep2`：本月行程為 水果倉庫；水果倉庫等級 ≥ 4；力量 ≥ 270；精神力 ≥ 270

## 家政服務公司老闆（Housekeeper）  優先權 Weight 20000
**結局條件（全部都要滿足）**
- 家務達 A（等級 4）
- 智力 ≥ 250

**前置事件鏈**
- `Flow_Housekeeper_Ep1`：本月行程為 家務；家務 累計次數 ≥ 18
- `Flow_Housekeeper_Ep2`：本月行程為 家務；家務 累計次數 ≥ 38

## 村莊治療師（Doctor）  優先權 Weight 20000
**結局條件（全部都要滿足）**
- 診療所達 B（等級 3）
- 精神力 ≥ 300
- 體力 ≥ 300

**前置事件鏈**
- `Flow_Doctor_Ep1`：本月行程為 診療所；診療所 累計次數 ≥ 18
- `Flow_Doctor_Ep2`：本月行程為 診療所；診療所 累計次數 ≥ 35；體力 ≥ 270；精神力 ≥ 270

## 木匠（Carpenter）  優先權 Weight 20000
**結局條件（全部都要滿足）**
- 木工坊達 B（等級 3）
- 力量 ≥ 300
- 品味 ≥ 300

**前置事件鏈**
- `Flow_Carpenter_Ep1`：本月行程為 木工坊；木工坊 累計次數 ≥ 18
- `Flow_Carpenter_Ep2`：本月行程為 木工坊；木工坊 累計次數 ≥ 35；力量 ≥ 270；品味 ≥ 270

## 餐廳店員（Waitress）  優先權 Weight 20000
**結局條件（全部都要滿足）**
- 餐廳達 B（等級 3）
- 品味 ≥ 300
- 口才 ≥ 300

**前置事件鏈**
- `Flow_Waitress_Ep1`：本月行程為 餐廳；餐廳 累計次數 ≥ 18
- `Flow_Waitress_Ep2`：本月行程為 餐廳；餐廳 累計次數 ≥ 30

## 牧場管理員（RanchManager）  優先權 Weight 20000
**結局條件（全部都要滿足）**
- 牧場達 B（等級 3）
- 體力 ≥ 300
- 感性 ≥ 300

**前置事件鏈**
- `Flow_RanchManager_Ep1`：本月行程為 牧場；牧場 累計次數 ≥ 18
- `Flow_RanchManager_Ep2`：本月行程為 牧場；牧場 累計次數 ≥ 35

## 園藝治療師（Therapist）  優先權 Weight 20000
**結局條件（全部都要滿足）**
- 植物園達 B（等級 3）
- 感性 ≥ 300
- 魔力 ≥ 300

**前置事件鏈**
- `Flow_Therapist_Ep1`：本月行程為 植物園；植物園 累計次數 ≥ 18
- `Flow_Therapist_Ep2`：本月行程為 植物園；植物園 累計次數 ≥ 35；感性 ≥ 270；魔力 ≥ 270

## 村民（Villager）  優先權 Weight 2
**結局條件（全部都要滿足）**
- 禮儀 ≥ 100

**前置事件鏈**
- `Flow_Villager_Ep1`：本月行程為 家務；家務 累計次數 ≥ 15
- `Flow_Villager_Ep2`：本月行程為 家務；家務 累計次數 ≥ 29

## 無業（Jobless）  優先權 Weight 1
**結局條件（全部都要滿足）**
- （無條件）

**前置事件鏈**
- `Flow_Jobless_Ep1`：本月行程為 在家休息；年份 ≤ 1210；Rest 次數 ≥ 15
- `Flow_Jobless_Ep2`：本月行程為 在家休息；年份 ≤ 1211；Rest 次數 ≥ 30
