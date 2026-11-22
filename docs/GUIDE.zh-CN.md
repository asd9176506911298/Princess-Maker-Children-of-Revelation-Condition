# 预言之子 结局条件整理（依游戏资料表）

资料来源：`TableData`（EndingStatus、Ending、Reputation、Quest、Event）与 `C_Flow_*` 事件脚本，并对照巴哈攻略「各结局参数」。
最后更新：2026-11-21

## 怎么读

- 「结局条件」就是 `EndingStatus`。**不同条件组是「且」，同一组内是「或」**（已由游戏判定程式确认；只有女王的战士／魔法评价放在同一组）。
- 评价有两种写法：「评价 ≥ N」是评价数值本身；「评价阶级 ≥ N」是阶级（1：0、2：50、3：150、4：250、5：350、6：500）。
- 课程／工作等级：3 = B、4 = A、5 = S（B、A 由论坛和 `Quest_RAAPresident` 要求「美术、音乐、舞蹈达 A = 等级 4」对上，S 是依序推出）。
- 「性向」就是善恶值：≤ -150 为浑沌，≥ 225 为秩序（女王）。
- 「前置事件链」是 `Flow_*` 脚本的发生条件，结局条件里有 `完成事件 Flow_xxx` 才是必要的。**没列在结局条件的事件链（例如混沌炼金师）不是结局必要条件**，只是剧情。
- Trigger：1010 月初、1017 行程后段随机、2012 在王城遇人、5001 节庆结束后、5002 节庆开始。

## 结局怎么决定（来自 GameAssembly 反组译：`lxc::mko`／`mau`／`sai`）

**1. 候选资格。** 只看 `EndingType = 2` 的 55 个一般结局（`Couple_*` 是 `EndingType = 3`，不在这一轮）。一个结局要成为候选，**每个条件组至少要有一条成立**（组内「或」、组间「且」；已由 `Cond_CheckEndingCondition` 与 `mko` 两处确认）。单一条件成立与否：`CompareType = 0`（事件、无条件）是「目前值 ≥ 1」，其余用 ≥／≤ 与 CompareValue 比较。

**2. 计分。** 每个候选的分数 = **Weight ＋ 加成**，只留下**分数最高**的，**同分全部留下**。

**3. 抽签。** 留下 1 个就用它，多个就**随机抽 1 个**（`sai`）。所以同分结局是随机的。

**4. 加成（`mau`）。**
- Weight ≥ 30000 的结局：没有加成。
- Weight < 30000 的结局：加成 = `min(2 × 数值, 8000)`。数值是该结局条件里所有「课程／打工阶级」条件（Type 8）所指课程在玩家纪录里的累积值（纪录栏位 `lcq`），取**最大的一个**。
- **吟游诗人（Bard）是特例**：加成 = `min(2 × (Edu_Music ＋ Edu_Literature 的累积值), 8000)`。
- `lcq` 栏位名被混淆，但已用 `ld::fno(课程Key, 累积值)` 的反组译确认它的用途：该函式在这个课程的 `ScheduleStat` 各阶列中，找 `NeedCount_GradeUp` **大于**累积值的列里最小的那一列，当作目前阶级。也就是 `NeedCount_GradeUp` 是**累积门槛**，`lcq` 就是跟它比的累积次数（例如课程：累积 ≥ 42 次为阶级 2、≥ 96 次为 B、≥ 160 次为 A、≥ 234 次为 S）。所以**加成实务上是 2 × 累积次数，大约数百，最多一千出头**，8000 的上限几乎碰不到。

**5. 对你的意义。**
- 难度高的结局（女王 52000、魔王 51000、公主 50000、大将军等）靠 Weight 直接压过一切，条件全过就是它。
- 加成太小，**不足以跨越 Weight 的档次**（例如 20000 系列追不上 27000 系列），它的作用是在 **Weight 相近的结局之间重新排序**：例如 20000、20080、20160、20200、20240、20495、20700、20850、20900 这些相差几十到几百的，谁练得多谁优先。
- 完全同分（例如大富豪、魔法剑士、外交官都是 40000，没有加成）会随机抽一个。

| 结局 | Weight | 课程／打工加成 |
|---|---|---|
| 女王（Queen） | 52000 | 无 |
| 魔王（DemonKing） | 51000 | 无 |
| 公主（Princess） | 50000 | 无 |
| 大将军（General） | 43000 | 无 |
| 王宫魔法师（RoyalWizard） | 42000 | 无 |
| 王立艺术学会会长（RAAPresident） | 41000 | 无 |
| 外交官（Diplomat） | 40000 | 无 |
| 魔法剑士（MagicKnight） | 40000 | 无 |
| 大富豪（Rich） | 40000 | 无 |
| 沙龙说客（Sting） | 34000 | 无 |
| 蛇蝎美人（Femme） | 33000 | 无 |
| 混沌炼金术师（ChaosAlchemist） | 32000 | 无 |
| 怪盗（MysteriousThief） | 32000 | 无 |
| 占卜师（FortuneTeller） | 31000 | 无 |
| 后巷之王（TheGodfather） | 31000 | 无 |
| 吟游诗人（Bard） | 27000 | 有（约 2 × 累积次数） |
| 贵金属工匠（Jeweler） | 27000 | 有（约 2 × 累积次数） |
| 魔导工程师（MagicEngineer） | 27000 | 有（约 2 × 累积次数） |
| 调香师（Perfumer） | 27000 | 有（约 2 × 累积次数） |
| 魔法阵图案师（SigilDrafter） | 27000 | 有（约 2 × 累积次数） |
| 剑舞姬（Sworddancer） | 27000 | 有（约 2 × 累积次数） |
| 学院院长（AcademyDean） | 23000 | 有（约 2 × 累积次数） |
| 首席芭蕾舞者（Dancer） | 23000 | 有（约 2 × 累积次数） |
| 首席歌姬（PrimaDonna） | 23000 | 有（约 2 × 累积次数） |
| 厨师（Cook） | 22500 | 有（约 2 × 累积次数） |
| 魔法事务员（MagicClerk） | 20900 | 有（约 2 × 累积次数） |
| 首都精锐兵团（CapitalEliteTroop） | 20850 | 无 |
| 冒险家（Adventurer） | 20700 | 无 |
| 炼金术师（Alchemist） | 20495 | 有（约 2 × 累积次数） |
| 舞蹈教师（DanceTutor） | 20495 | 有（约 2 × 累积次数） |
| 礼法教师（Governess） | 20495 | 有（约 2 × 累积次数） |
| 历史学者（Historian） | 20495 | 有（约 2 × 累积次数） |
| 魔法教习（MagicTutor） | 20495 | 有（约 2 × 累积次数） |
| 体术教头（MartialArtsInstructor） | 20495 | 有（约 2 × 累积次数） |
| 音乐教师（MusicTutor） | 20495 | 有（约 2 × 累积次数） |
| 画家（Painter） | 20495 | 有（约 2 × 累积次数） |
| 剑术道场教头（SwordInstructor） | 20495 | 有（约 2 × 累积次数） |
| 作家（Writer） | 20495 | 有（约 2 × 累积次数） |
| 调酒师（Bartender） | 20240 | 有（约 2 × 累积次数） |
| 物流主管（UnloadingManager） | 20240 | 有（约 2 × 累积次数） |
| 研究助理（Assistant） | 20200 | 有（约 2 × 累积次数） |
| 家庭教师（PrivateTutor） | 20200 | 有（约 2 × 累积次数） |
| 女仆咖啡厅的超级明星（Superstar） | 20200 | 有（约 2 × 累积次数） |
| 铁匠（Blacksmith） | 20160 | 有（约 2 × 累积次数） |
| 服装设计师（Designer） | 20160 | 有（约 2 × 累积次数） |
| 物流管理员（Logistics） | 20080 | 有（约 2 × 累积次数） |
| 士兵（Soldier） | 20080 | 有（约 2 × 累积次数） |
| 木匠（Carpenter） | 20000 | 有（约 2 × 累积次数） |
| 村庄治疗师（Doctor） | 20000 | 有（约 2 × 累积次数） |
| 家政服务公司老板（Housekeeper） | 20000 | 有（约 2 × 累积次数） |
| 牧场管理员（RanchManager） | 20000 | 有（约 2 × 累积次数） |
| 园艺治疗师（Therapist） | 20000 | 有（约 2 × 累积次数） |
| 餐厅店员（Waitress） | 20000 | 有（约 2 × 累积次数） |
| 村民（Villager） | 2 | 无 |
| 无业（Jobless） | 1 | 无 |

## 占卜师怎么提示（`lxc::mca`／`lzj`／`lyx`）

- 每个结局的进度 = Σ（各条件组的最高 Score × 该组内**最佳条件的进度**）÷ Σ 各组的最高 Score。
- 单一条件进度：一般 ≥ 条件是「目前值 ÷ 目标值」，限制在 0 到 1；阶级类（声望、课程阶级）是 (目前阶级 − 1) ÷ (目标阶级 − 1)；善恶 ≤ 条件是 (上限 − 目前) ÷ (上限 − 目标)；是非题（事件、无条件）是 0 或 1。
- 只考虑进度 ≥ 门槛的结局，门槛 = 0.2 ＋ 0.6 × 经过天数 ÷ 2920（8 年），也就是 0.2 渐增到 0.8。
- 剔除 `Ending_Jobless`（用 Key 字串比对）与 Weight < 1000 的结局（Jobless 与 Villager）。
- 排序：进度由高到低，同进度再比 Weight ＋ 加成，由高到低。最多显示 **3** 个。

## 舞者与歌姬的条件

两者最后一个条件组是「艺术祭胜利」（Type 15 舞蹈、16 音乐）。`lxc::hch` 的算法：走访艺术祭历史纪录，每笔以 `&` 分隔的标签，必须**同时含 `Rank=1` 与 `Category=<该分类>`**才算一次，条件是 ≥ 1，也就是「该分类的艺术祭拿过第一名」。另外 `Cond_CheckEndingCondition` 这个事件节点没有处理 15、16，但整套资料只有公主第 6 话用到它，不影响这两个结局。

## 恋人结局怎么决定（来自 `pk::OnTheEnd`／`ShowCard`／`go::glc`）

**恋人结局跟 EndingStatus 条件完全无关**，只看「游戏状态里的结婚对象」这一个数字。它和职业结局是**两张各自独立的结局卡**：职业结局由 `lxc::sai` 决定（上一节），恋人结局由结婚对象决定，结局画面会依序显示。

**对应规则**（结婚对象代码 = `EPartnerNPC`，由 `Act_SetWeddingNPC` 设定）：

| 结婚对象代码 | 对象 | 恋人结局 | 婚礼对话图 |
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
| -1（村人）／-3（工作伙伴） | 一般婚姻 | `Ending_Couple_Normal` | — |
| -2（单身） | 没有结婚 | 无恋人结局 | — |

**唯一的特例**：结婚对象是阿加雷斯（DemonMale, 2），而且**职业结局是 `Ending_Princess`**（公主）时，恋人结局改为 **`Ending_Couple_DemonMale_King`**，而不是 `Ending_Couple_DemonMale`。程式码用字串 `Ending_Princess` 比对（`a::gx`），所以只有走完公主线才会拿到「魔王」版本。其他组合（例如职业结局不是公主）仍是 `Ending_Couple_DemonMale`。

**其他确认的事**
- 职业结局的唯一呼叫处是 `pk::chs`（结局画面）呼叫 `lxc::sai`，没有别的入口。
- 结婚对象为 -1 或 -3 都是一般婚姻、同一张 `Couple_Normal` 卡；-2 是单身，只给「未婚」成就，没有恋人结局。
- 成就「全部婚姻结局」统计历次通关中结婚对象代码 ≥ -4 的不同对象（-1 与 -3 合并算一个），累积到 **11 个**就达成；`go::glb`、`go::glc` 负责把历次通关的结局收进收藏清单。
- 结局画面还会多放一张卡 `Card_Second_EA`（`a::ha`），只在某个条件（`hl::ljk()` 为真，且该卡已解锁）下加入，用途没追。

**结局时的最终判定（`pk::gde`，在结局画面 `pk::chs` 开头呼叫，结果写回结婚对象）**

结婚对象有两个来源：求婚事件接受时用 `Act_SetWeddingNPC` 写入（代码 ≥ 0 的 8 位种族伙伴），以及结局当下由 `pk::gde` 依已完成事件与职业结局补上。`gde` 的顺序如下（地址 `0x182638610`）：

1. **结婚对象已 ≥ 0**（接受过求婚）：直接用它，后面全部略过。
2. 否则依序检查，先符合先用：

| 顺序 | 条件（全部成立） | 结果 |
|---|---|---|
| 1 | 已完成事件 `Caim_Episode1`，且职业结局是 `Ending_Waitress`（餐厅店员） | -5 凯姆 |
| 2 | 已完成事件 `Icosa_Episode1`，且职业结局是 `Ending_MagicTutor`（魔法教习） | -6 艾科萨 |
| 3 | 已完成事件 `Kimaris_Episode1`，且职业结局是 `Ending_Soldier`（士兵） | -7 奇马里斯 |
| 4 | 已完成事件 `Muzai_Episode1`，且职业结局是 `Ending_Writer`（作家） | -8 穆扎伊 |
| 5 | 已完成事件 `Cube_Episode2`（**不看职业结局**） | -4 吉普 |
| 6 | 12 项基础能力（体力、力量、敏捷、智力、魔力、精神力、气质、礼仪、口才、感性、品味、魅力）**总和 ≥ 3600** | -2 单身（没有恋人结局） |
| 7 | 职业结局是 `Ending_Jobless` 或 `Ending_Villager` | -1 一般婚姻（`Couple_Normal`） |
| 8 | 以上都不符合 | 维持目前值。新游戏的预设值就是 **-3（工作伙伴，也是 `Couple_Normal`）**：存档资料类别 `do` 的建构函式 `do::do()` 把这个栏位初始化为 -3，已确认 |

另外 1 到 4、5 各自还有一条「目前值已经是该代码就沿用」的捷径，不影响上面的结果。

**这代表**
- 要拿凯姆、艾科萨、奇马里斯、穆扎伊的恋人结局，**职业结局必须刚好是对应那一个**（餐厅店员、魔法教习、士兵、作家），再加上完成各自的 `Episode1`。职业结局不对，会直接掉到下一条。
- 吉普只要 `Cube_Episode2` 完成就成立，不管职业。
- 这 5 位的优先度比单身（总和 ≥ 3600）高，所以能力很高的女儿只要有完成对应事件，仍然会拿到他们的恋人结局。
- 能力总和 ≥ 3600（平均 300）且没有任何伙伴时才会是单身；能力没到而职业是无业／村人时是一般婚姻。这个 3600 是从组语读到的常数，12 项是 `EStatus` 1 到 12，我没有在游戏里实测。
- 8 位种族伙伴（代码 0 到 7）必须在 1218 年 3 月求婚事件中接受，才会让代码 ≥ 0，下面「各伙伴的求婚事件」有条件。

## 与巴哈攻略对照：基本对得上，但数字有不少出入

**完全对上**：牧场管理员、研究助理、音乐教师、舞蹈教师、魔法师范、历史学者、魔法事务员（智魔精 1500、魔法评价阶级 4 = 250）、首席芭蕾舞者（舞蹈 A 加舞蹈赛冠军）、魔法剑士（力量、魔力 600，两评价 ≥ 250）、教父（力量 800）、冒险者（探险龙、森、兽三地区）、女王的四项加总（1500／1500／2400／1500）、魔王的两项加总 2400、公主的两项加总 2400、王立艺术学会会长的感受性＋品味＋魅力 2400 与艺术评价 350、王宫魔法师的智魔精 2400 与魔法评价 350。

**数字不同（以资料为准）**

| 结局 | 巴哈写法 | 资料实际 |
|---|---|---|
| 公主 | 社交评价 ≥ 450 | **≥ 400** |
| 女王 | 战士、社交评价皆 500 | 社交 ≥ 500，**战士或魔法择一 ≥ 400** |
| 魔王 | 战士、魔法评价 500、善恶 -300 | 结局条件**没有评价**，只要能力加总 2400；善恶 **≤ -225**（事件链只要 ≤ -150） |
| 王立艺术学会会长 | 气质＋礼仪＋口才 ≥ 1500 | **没有此条件**；但要**艺术节夺冠**＋艺术、舞蹈、音乐达 A |
| 王宫魔法师 | 魔法、炼金达 A | 魔法学、炼金术都要 **S（等级 5）** |
| 大将军 | 体敏力 ≥ 2100、气质 200、礼仪 300 | 体力＋力量＋敏捷 **≥ 2400**、气质＋礼仪 **≥ 600**，另要**剑术 S、体术 S、武斗会夺冠** |
| 外交官 | 智力 > 750、三项 ≥ 2100、社交 500 | 智力 **≥ 500**、气质＋礼仪＋口才 **≥ 1800**、社交 **≥ 350** |
| 沙龙说客 | 智力 > 750、各 700、社交 500 | 智力 **≥ 500**、三项加总 **≥ 1600**、社交 **≥ 350**、善恶 ≤ -150 |
| 蛇蝎美人 | 酒吧 A、体力与口才 > 300 | 酒馆达 **S**、魅力 600，**没有体力、口才条件** |
| 后巷之王 | 战士评价 ≥ 250 | **≥ 350**，体力 400、力量 800 |
| 怪盗 | 智力 600 | 智力 **≥ 300**、敏捷 600、品味 600、事件 Flow_MysteriousThief_Ep3 |
| 混沌炼金师 | 魔力 > 600、炼金 A、三次事件 | 炼金 A、智魔精 ≥ **1500**、品味 300；**事件非必要** |
| 占卜师 | 智魔精口才各 500、魔法 A | 精神、口才、感受性各 **400**、魔法评价阶级 ≥ 4、善恶 ≤ -150；**不要求魔法 A** |
| 剑舞姬 | 魅力、力量 > 600，舞蹈 A、剑术 A | 剑术 A、**舞蹈 B**、魅力 **≥ 500** |
| 礼仪教师 | 礼法 B、礼仪气质 > 300 | 礼法 **A**、礼仪 400、气质 400、感受性 200 |
| 魔导工程师 | 炼金 A | 魔法 A、炼金 A、**智魔精加总 ≥ 2000** |
| 体术教头 | 力量 300、精神力 300 | **体力** 300、精神 300（体术 B） |
| 调酒师 | 体力、口才 > 300 | **口才、气质** 各 300（酒馆 A） |
| 超级巨星 | 感性、魅力 > 400、社交升两阶 | 咖啡厅 A、魅力 300、感受性 300、口才 200，**没有社交评价条件** |
| 大富豪 | 金钱 > 5 万 | 金钱 ≥ 50000 **且**完成事件 Flow_Rich_Ep4（两者都要，见下文更正） |

**巴哈没有、资料有的结局**：厨师、家教、设计师、士兵、女服务生、铁匠、物流（水果仓库）、卸货主管（论坛的「物流主管」其实是码头，对应卸货主管）、首都精锐部队、学院院长、调香师、珠宝匠、吟游诗人、魔纹绘师、首席歌姬、作家、画家、剑术教师、体术教头、管家、村民。

## 更正

先前我说大富豪是「金钱 5 万**或**事件」，**那是错的**。两条在不同条件组，依一致规则是**两者都要**：先完成 Flow_Rich 事件链（Ep1 金钱 ≥ 20000 起，Ep2 要先有 `Event_Rich_Invest` 且隔 3 个月，Ep3 隔 4 个月，Ep4 隔 8 个月），结局时金钱也要 ≥ 50000。

## 任务条件（Quest 表）

- `Quest_RoyalWizard1`：用魔法讨伐魔兽 20 只。`Quest_RoyalWizard2`：用魔法在武斗会夺冠。
- `Quest_RAAPresident`：美术、音乐、舞蹈三门都达 A（等级 4）。
- `Quest_Adventurer1`：讨伐黑暗魔法师。`Quest_Adventurer2`：讨伐沙漠掠夺者。
- `Quest_Queen1`：在宴会上拜访代理摄政。`Quest_Queen2`：在魔王城拜访代理摄政。

## 需要做选择的关键事件（来自 D_Flow 对话图）

| 事件 | 要选的 | 选错的后果 |
|---|---|---|
| Flow_DemonKing_Ep5（1218 年 2 月，只有一次，MaxCount 1） | 「我明白了。一起走吧。」→ 设定 `Event_DemonKing_Allow` | 选「……我还不知道。」门不会再开，魔王结局断线 |
| Flow_Rich_Ep1 | 「投资。」（花 5000 金）→ 设定 `Event_Rich_Invest` | 选「不投资」则 Ep2 条件不成立，大富豪链断线 |
| Flow_MysteriousThief_Ep1 | 「……真有意思。我帮你藏起来。」（善恶 -10） | 选「交给士兵」得 200 金，但设定 `Event_MysteriousThief_Prison`，Ep2 条件要求**不可**有此旗标，怪盗链断线 |
| Flow_Adventurer_Ep1 | 「接受。」→ 设定 `Quest_Adventurer1` | 选「拒绝」就没有 Ep2 需要的任务 |
| Flow_TheGodfather_Ep1／Ep2 | 要往浑沌走：Ep1 选「继续拿他们出气」（-20）；Ep2 选「接受提议」（-30） | 选「交给守卫」各 +20，不影响事件链，但会把善恶往秩序推，教父要求善恶 ≤ -150 |

## 公主线：阿加雷斯（DemonMale）求婚条件（来自 Partner 事件图）

好感阶级（`Cond_CompareFavorGrade`）：3 好感（20–29）、4 爱慕（30–39）、5 特别（40–49）、6 灵魂伴侣（50）。

| 事件 | 条件 |
|---|---|
| Episode1 | 已完成 DemonMale_FirstEpisode、尚未完成 Ep1；游戏年 ≤ 1213。1213 年内直接可触发，之前则要在 FirstEpisode 后随机 0–2 个月 |
| Episode2 | 已完成 DemonMale_InterEpi2、尚未完成 Ep2；年 ≤ 1213，同样 1213 年内可触发，之前随机 0–2 个月 |
| Episode3 | 已完成 Ep2；**课程累计 ≥ 2 次**；年 ≤ 1213；之后同上 |
| Episode4 | 已完成 Ep3；Karen 年龄 ≥ 14 |
| Episode5 | 已完成 Ep4；年龄 ≥ 15；好感阶级 ≥ 3（好感） |
| Episode6 | 已完成 Ep5；年龄 ≥ 16；好感阶级 ≥ 3 |
| Episode7 | 已完成 Ep6；年龄 ≥ 17；好感阶级 ≥ **4（爱慕）** |
| Episode8 | 已完成 Ep7（没有其他条件） |
| **Proposal（求婚）** | **1218 年 3 月月初**（Trigger 1010）；已完成 Ep8；且**未设定 `Event_DemonKing_Allow`** |

求婚事件里选「接受求婚」才会执行 `Act_SetWeddingNPC(DemonMale)` 并设定 **`DemonMale_Proposal_Allow`**；选「拒绝求婚」没有任何旗标，求婚事件只有一次（MaxCount 1）。若进入事件时**已经有结婚对象**（`Cond_HasWeddingPartner`），对话直接略过，也不会设定旗标。

所以：
- **公主结局需要的旗标，来自 1218 年 3 月接受阿加雷斯的求婚。**
- **魔王与公主互斥**：选了魔王第 5 话的「一起走吧」（`Event_DemonKing_Allow`），求婚就不会发生。
- 好感阶级在 Ep5 到 Ep7 才要求 3、3、4，**资料没有要求到灵魂伴侣（阶级 6）**；巴哈写的「培养至灵魂伴侣」只是保险。Ep8 与求婚本身没有好感门槛，不过 Ep8 的事件内容我没有检查，可能有隐含限制。
- 其他伙伴的求婚事件已解析（见下一节）：同样是 1218 年 3 月月初，若先接受了其他人的求婚，阿加雷斯的求婚会因为「已有结婚对象」而略过；唯一例外是森族女，她的求婚不检查是否已有对象。

## 各伙伴的求婚事件与剧情链（来自 Partner 事件图）

**求婚事件共通规则**（FurryMale／FurryFemale／ForestMale／ForestFemale／DragonMale／DragonFemale／DemonFemale／DemonMale 共 8 位）：
- 发生时间完全相同：**1218 年 3 月月初**（Trigger 1010），要求**未设定 `Event_DemonKing_Allow`**，且**该伙伴最后一个剧情事件已完成**（见下表最后一列）。
- 8 个求婚事件 Priority 都是 1、MaxCount 都是 1：每位只会提一次，选拒绝就用掉了。
- 对话流程相同：进入事件先检查 `Cond_HasWeddingPartner`，**已有结婚对象就直接略过**；否则出现「接受／拒绝」两个选项，选接受才执行 `Act_SetWeddingNPC(该伙伴)`。
- **ForestFemale（森族女）的对话图里没有 `Cond_HasWeddingPartner` 检查**，所以就算已经先接受了别人的求婚，她的求婚仍会出现，接受会**覆盖**结婚对象（从图的结构看，没有实测）。
- **只有阿加雷斯（DemonMale）的求婚会设定旗标 `DemonMale_Proposal_Allow`**，所以只有他能通往公主结局；其他 7 位接受求婚只会得到各自的恋人结局。
- 7 位非阿加雷斯的伙伴都**不需要特定好感阶级才能求婚**，好感只出现在剧情链中（见下表），求婚本身只看「剧情链最后一话已完成」。

**同一个月有多位伙伴符合时**：8 个事件 Trigger 都是 1010、Priority 都是 1、MaxCount 1。依 `EventManager::sch` 的逻辑，同一触发点只会挑「最小的正数优先度」那一组放进伫列，同组全部排入，所以 8 位求婚事件会一起排队（3 月同时还有其他 Priority 1 的 1010 事件，例如 Flow_Queen_Ep6、Flow_Rich_Ep1~4、Flow_DemonKing_Ep2，若 Priority 0 的事件当月成立则会先被挑走）。排队顺序**推测**依 Event 表列顺序：ForestFemale → ForestMale → DragonFemale → DragonMale → FurryMale → FurryFemale → DemonFemale → DemonMale（这一点是推测，未验证 `jq::fca` 的排序）。每个对话都先检查 `Cond_HasWeddingPartner`，只有「接受」才会设定对象，所以通常是排在前面且你接受的那位成为对象，后面的会被略过（ForestFemale 的对话缺这个检查，需注意）。保险做法仍是同一年只养成一位。

### 兽族男

| 事件 | 触发／地点 | 条件 |
|---|---|---|
| Episode1 | 行程开始／208 | 女儿年龄 ≤ 13 且 FurryMale 好感阶级 ≥ 2 |
| Episode2 | 3301／1205 | 完成对话 FurryMale_Episode1 |
| Episode3 | 行程开始／204 | 完成对话 FurryMale_Episode2 且 FurryMale 好感阶级 ≥ 3 |
| Episode4 | 3304／1204 | 完成对话 FurryMale_InterEpi |
| Episode5 | 3301／1205 | 完成对话 FurryMale_Ready |
| Episode6 | 月初／0 | 完成对话 FurryMale_Episode5 且 距 FurryMale_Episode5 完成当月月初 ≥ 0年2月0日 |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成伙伴行程 FurryMale_Episode6 |

### 兽族女

| 事件 | 触发／地点 | 条件 |
|---|---|---|
| Episode1 | 行程结束前／304 | FurryFemale 好感阶级 ≥ 2 |
| Episode2 | 行程结束前／304 | 完成对话 FurryFemale_Episode1 |
| Episode3 | 行程结束前／304 | FurryFemale 好感阶级 ≥ 3 |
| Episode4 | 3301／1205 | 完成对话 FurryFemale_Episode3 且 距 FurryFemale_Episode3 完成当月月初 ≥ 0年2月0日 |
| Episode5 | 行程开始／304 | 完成对话 FurryFemale_Bag 且 FurryFemale 好感阶级 ≥ 4 |
| Episode6 | 3301／1205 | 已完成事件 FurryFemale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成伙伴行程 FurryFemale_Episode6 |

### 森族男

| 事件 | 触发／地点 | 条件 |
|---|---|---|
| Episode1 | 行程开始／203 | 已完成事件 ForestMale_FirstEpisode 且 ForestMale 好感阶级 ≥ 2 |
| Episode2 | 行程结束前／304 | 完成对话 ForestMale_Episode1 且 ForestMale 好感阶级 ≥ 2 |
| Episode3 | 行程开始／306 | 完成对话 ForestMale_Episode2 且 ForestMale 好感阶级 ≥ 3 |
| Episode4 | 拜访(2005)／503 | 完成对话 ForestMale_Episode3 且 女儿年龄 ≥ 14 且 ForestMale 好感阶级 ≥ 3 |
| Episode5 | 3202／1303 | 完成对话 ForestMale_Episode4 且 ForestMale 好感阶级 ≥ 4 |
| Episode6 | 3202／1303 | 已完成伙伴行程 ForestMale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成伙伴行程 ForestMale_Episode6 |

### 森族女

| 事件 | 触发／地点 | 条件 |
|---|---|---|
| Episode1 | 行程开始／206 | 女儿年龄 < 14 且 完成对话 ForestFemale_FirstEpisode 且 ForestFemale 好感阶级 ≥ 2 |
| Episode2 | 行程开始／206 | 完成对话 ForestFemale_Episode1 |
| Episode3 | 行程结束前／308 | 完成对话 ForestFemale_Episode2 且 ForestFemale 好感阶级 ≥ 3 |
| Episode4 | 行程开始／205 | 完成对话 ForestFemale_Episode3 且 女儿年龄 ≥ 14 |
| Episode5 | 行程开始／206 | 完成对话 ForestFemale_Episode4 且 ForestFemale 好感阶级 ≥ 4 |
| Episode6 | 3005／1301 | 完成对话 ForestFemale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成伙伴行程 ForestFemale_Episode6 |

### 龙族男

| 事件 | 触发／地点 | 条件 |
|---|---|---|
| Episode1 | 3103／1105 | 完成对话 DragonMale_FirstEpisode 且 DragonMale 好感阶级 ≥ 2 |
| Episode2 | 3101／1103 | 完成对话 DragonMale_Episode1 |
| Episode3 | 行程开始／302 | 完成对话 DragonMale_Episode2 且 距 DragonMale_Episode2 完成当月月初 > 0年3月0日 |
| Episode4 | 3103／1105 | 完成对话 DragonMale_Episode3 且 女儿年龄 ≥ 14 |
| Episode5 | 行程开始／213 | 完成对话 DragonMale_Episode4 且 DragonMale 好感阶级 ≥ 4 |
| Episode6 | 3101／1103 | 完成对话 DragonMale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成伙伴行程 DragonMale_Episode6 |

### 龙族女

| 事件 | 触发／地点 | 条件 |
|---|---|---|
| Episode1 | 行程开始／305 | 完成对话 DragonFemale_FirstEpisode 且 DragonFemale 好感阶级 ≥ 2 |
| Episode2 | 行程开始／305 | 完成对话 DragonFemale_Episode1 |
| Episode3 | 行程开始／307 | 已完成对话 DragonFemale_Episode2 且 自 Episode2 完成次月月初起，魔法学与礼法的课程次数合计 ≥ 2（日期偏移参数为 1 年 0 月 0 日，确切区间未确认） |
| Episode4 | 3103／1105 | 完成对话 DragonFemale_Episode3 |
| Episode5 | 行程开始／305 | 完成对话 DragonFemale_Episode4 且 DragonFemale 好感阶级 ≥ 4 |
| Episode6 | 3101／1103 | 完成对话 DragonFemale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成伙伴行程 DragonFemale_Episode6 |

### 魔族女（DemonFemale）

| 事件 | 触发／地点 | 条件 |
|---|---|---|
| Episode1 | 行程开始／303 | 完成对话 DemonFemale_FirstEpisode 且 DemonFemale 好感阶级 ≥ 2 |
| Episode2 | 行程开始／212 | 女仆咖啡厅 累计次数 ≥ 1 且 完成对话 DemonFemale_Episode1 且 已完成事件 DemonFemale_Encounter_Dance |
| Episode3 | 王城遇人／710 | 完成对话 DemonFemale_Episode2 且 DemonFemale 好感阶级 ≥ 3 |
| Episode4 | 行程开始／309 | 完成对话 DemonFemale_Episode3 且 DemonFemale 好感阶级 ≥ 3 |
| Episode5 | 王城遇人／710 | 完成对话 DemonFemale_Episode4 且 DemonFemale 好感阶级 ≥ 4 且 已完成事件 DemonFemale_DemonEvent |
| Episode6 | 王城遇人／710 | 完成对话 DemonFemale_InterEpi |
| **求婚** | 月初／— | 非 已完成事件 Event_DemonKing_Allow 且 年份 = 1218 且 月份 = 3 且 已完成伙伴行程 DemonFemale_Episode6 |

### 方块与其他 NPC（Cube、Caim、Icosa、Kimaris、Muzai）

这 5 位没有求婚事件，也没有「接受」的对话图。图里只有剧情（`Episode1`，方块有 1、2）与婚礼对话；成为结婚对象是结局时由 `pk::gde` 依「完成哪个事件」加「职业结局」决定（见上面「结局时的最终判定」）。他们的剧情事件条件如下：

| 事件 | 触发 | 条件 |
|---|---|---|
| Cube_Episode1 | 1002（地点 201 家） | 与吉普的关系 ≥ 70 |
| Cube_Episode2 | 1002（地点 201 家） | 已完成事件 Cube_Episode1 且 与吉普的关系 ≥ 120 |
| Caim_Episode1（凯姆，餐厅店长） | 1017 行程后段、优先度 30 | 已完成事件 Job_Restaurant_Ep3 且 本月行程为 餐厅 |
| Icosa_Episode1（艾科萨，魔法学院） | 1017、优先度 30 | 已完成事件 Edu_Magic_Ep3 且 本月行程为 魔法学 |
| Kimaris_Episode1（奇马里斯，检问处） | 1017、优先度 30 | 已完成事件 Job_Checkpoint_Ep3 且 本月行程为 检问处 |
| Muzai_Episode1（穆扎伊，文学老师） | 1017、优先度 30 | 已完成事件 Edu_Literature_Ep3 且 本月行程为 文学 |

四位老板／老师的 Episode1 都只能触发一次（MaxCount 1），而且要在**那个行程进行的那个月**才会出现，所以要先把对应的 `_Ep3` 打完，并且之后再排一次同样的行程。吉普的「与吉普的关系」是 Status 21（范围 -100~500）。整个资料库与全部流程图中，只有 `E_Job_House_Ep1/2/3`（家事打工的三段事件）会加它：每段不论选哪个选项都 +1，合计只有 3，达不到 70／120，所以主要增加来源不在资料或图里，应该在 C# 程式（例如家事类行程每次执行时结算）。程式端已确认：`hl::sim(EStatus,int)` 是通用的状态加减（上下限取 Status 表的 Min／Max，Cube 为 -100~500），没有针对 Cube 的特殊处理；`hl::dfs` 是「设成指定值」，呼叫者多为除错或介面。资料端：`Schedule` 表的 `Job_House` 在 `NeedStatCritical` 栏使用 StatType 21（Cube，范围 10~80），应是「大成功判定用的能力值」（推测），不是增加好感度的来源；其他表没有以代号 21 引用 Cube。**来源已找到（程式码确认）**：`ue::cez::MoveNext`（行程执行流程）在行程开始时，如果该行程的地点是 201（0xC9）就执行 `hl::sim(RelationshipCube, 结束日 - 开始日 + 1)`。资料表里地点为 201 的行程只有 `Job_House`（家事打工），所以**每做一次家事打工，与吉普的关系就增加「该次行程天数」**，在行程开始时就结算，与成功、失败、大成功无关。（一次行程的天数由 `mbg::sfv` 决定：每月 3 个行程格，上旬、中旬各 10 天，下旬为「当月天数 − 20」天，也就是 30 天的月份 10 天、31 天 11 天、2 月 8～9 天。所以一次家事打工通常 +10：累积 70 需要 7 次，累积 120 需要 12 次；三段事件各 +1 合计只有 +3，省不下任何一次。也就是每个月三格都排家事打工，约 3 个月多（7 格）达 70、约 4 个月（12 格）达 120。）另外，`E_Job_House_Ep1/2/3` 三段事件每段再 +1。其他途径没有：`ue::ulx`（行程结算只套用 Stats 栏，Job_House 为 3／11／6，大成功加倍）、送礼（`Partner_Npc_Cube` 的 FavorStatus 为 0）、道具／茶／饮食、每月结算 `uk::cdv`（只动体力、心智、疲劳、金钱）都不会加。

**「一般婚姻」与「单身」**：`D_Village_Wedding`、`D_Job_Wedding`、`D_None_Wedding` 对应 -1、-3、-2。没有事件设定它们，是 `pk::gde` 在结局时依上面第 6 到 8 条给值。

## 还缺的部分

1. `Event_DemonKing_Allow`、`DemonMale_Proposal_Allow` 这两个旗标是对话选项设定的，上面已确认。
2. `Score` 栏是占卜师进度的加权，`Weight` 是最终判定的优先权，已由程式码确认。
3. 课程／打工累积值 `lcq` 已确认是跟 `NeedCount_GradeUp` 比较的累积次数；`yaw` 与 `lcr` 两个栏位的意义（`yaw` 用于 Type 9 课程次数）未细查。
4. 同一个月多位伙伴同时符合求婚时的先后顺序未确认；「与吉普的关系」来源与天数都已确认（家事打工每次 +10，下旬依当月天数 +8～11）。结婚对象预设值 -3 已确认是 `do::do()` 建构函式写入；另外 `ei::hbo`（会显示「求婚相关资料已初始化」的韩文提示，像是除错或重置工具）与 `du::cga`／`du::lzk`（把结局条件直接套到存档的除错功能）也会写它，正常游玩不会走到。
5. Flow_Vacance（度假事件）没有解析。
6. 王城遇人事件的 Place（703 到 708）对应的场景名称，在程式码里。

---

# 各结局完整条件（依优先权由高到低）

## 女王（Queen）  优先权 Weight 52000
**结局条件（全部都要满足）**
- 社交评价 ≥ 500
- 下列任一：战士评价 ≥ 400 或 魔法评价 ≥ 400
- 体力＋力量＋敏捷 ≥ 1500
- 智力＋魔力＋精神力 ≥ 1500
- 气质＋礼仪＋口才 ≥ 2400
- 感性＋品味＋魅力 ≥ 1500
- 性向 ≥ 225

**前置事件链**
- `Flow_Queen_Ep1`：与 Castle_RegentDeputy 见面次数 ≥ 3；口才 ≥ 200
- `Flow_Queen_Ep2`：已完成事件 Flow_Queen_Ep1；距 Flow_Queen_Ep1 完成当月月初 ≥ 0年2月0日
- `Flow_Queen_Ep3`：已完成事件 Flow_Queen_Ep2；战士三项(体力+力量+敏捷)总和 ≥ 840；战士评价 ≥ 245
- `Flow_Queen_Ep3`：已完成事件 Flow_Queen_Ep2；战士三项(体力+力量+敏捷)总和 ≥ 840；【不可】战士评价 ≥ 245；魔法三项(智力+魔力+精神)总和 ≥ 840；战士评价 ≥ 245
- `Flow_Queen_Ep3`：已完成事件 Flow_Queen_Ep2；【不可】战士三项(体力+力量+敏捷)总和 ≥ 840；魔法三项(智力+魔力+精神)总和 ≥ 840；战士评价 ≥ 245
- `Flow_Queen_Ep4`：已完成事件 Flow_Queen_Ep3；距 Flow_Queen_Ep3 完成当月月初 ≥ 0年2月0日
- `Flow_Queen_Ep5`：已完成事件 Flow_Queen_Ep4
- `Flow_Queen_Ep6`：已完成事件 Flow_Queen_Ep5；战士评价 ≥ 400；社交评价 ≥ 500；战士三项(体力+力量+敏捷)总和 ≥ 1500；魔法三项(智力+魔力+精神)总和 ≥ 1500；社交三项(气质+礼仪+口才)总和 ≥ 2400；艺术三项(感受性+品味+魅力)总和 ≥ 1500；性向 ≥ 225
- `Flow_Queen_Ep6`：已完成事件 Flow_Queen_Ep5；【不可】战士评价 ≥ 400；魔法评价 ≥ 400；社交评价 ≥ 500；战士三项(体力+力量+敏捷)总和 ≥ 1500；魔法三项(智力+魔力+精神)总和 ≥ 1500；社交三项(气质+礼仪+口才)总和 ≥ 2400；艺术三项(感受性+品味+魅力)总和 ≥ 1500；性向 ≥ 225
- `Flow_Queen_Ep7`：已完成事件 Flow_Queen_Ep6

## 魔王（DemonKing）  优先权 Weight 51000
**结局条件（全部都要满足）**
- 完成事件 Event_DemonKing_Allow
- 性向 ≤ -225
- 体力＋力量＋敏捷 ≥ 2400
- 智力＋魔力＋精神力 ≥ 2400

**前置事件链**
- `Flow_DemonKing_Ep1`：性向 ≤ -150；战士三项(体力+力量+敏捷)总和 ≥ 800；魔法三项(智力+魔力+精神)总和 ≥ 800
- `Flow_DemonKing_Ep2`：已完成事件 Flow_DemonKing_Ep1；性向 ≤ -150；战士三项(体力+力量+敏捷)总和 ≥ 1200；魔法三项(智力+魔力+精神)总和 ≥ 1200
- `Flow_DemonKing_Ep3`：已完成事件 Flow_DemonKing_Ep2
- `Flow_DemonKing_Ep4`：已完成事件 Flow_DemonKing_Ep3；性向 ≤ -150；战士三项(体力+力量+敏捷)总和 ≥ 2000；魔法三项(智力+魔力+精神)总和 ≥ 2000
- `Flow_DemonKing_Ep5`：已完成事件 Flow_DemonKing_Ep4；性向 ≤ -225；战士三项(体力+力量+敏捷)总和 ≥ 2400；魔法三项(智力+魔力+精神)总和 ≥ 2400；年份 = 1218；月份 = 2

## 公主（Princess）  优先权 Weight 50000
**结局条件（全部都要满足）**
- 完成事件 DemonMale_Proposal_Allow
- 社交评价 ≥ 400
- 艺术评价 ≥ 400
- 气质＋礼仪＋口才 ≥ 2400
- 感性＋品味＋魅力 ≥ 2400
- 性向 ≥ -149

**前置事件链**
- `Flow_Princess_Ep1`：与 Castle_Consort 见面次数 ≥ 3；图变数 KarenRank = 1；完成对话 DemonMale_Episode3
- `Flow_Princess_Ep2`：完成对话 Flow_Princess_Ep1；魅力 ≥ 400
- `Flow_Princess_Ep3`：完成对话 Flow_Princess_Ep2；社交三项(气质+礼仪+口才)总和 ≥ 2000
- `Flow_Princess_Ep4`：完成对话 Flow_Princess_Ep3
- `Flow_Princess_Ep5`：完成对话 Flow_Princess_Ep4
- `Flow_Princess_Ep6`：完成对话 Flow_Princess_Ep5；目前满足结局条件 Ending_Princess
- `Flow_Princess_Ep7`：完成对话 Flow_Princess_Ep6

## 大将军（General）  优先权 Weight 43000
**结局条件（全部都要满足）**
- 武斗会夺冠
- 剑术达 S（等级 5）
- 体术达 S（等级 5）
- 战士评价 ≥ 350
- 体力＋力量＋敏捷 ≥ 2400
- 气质＋礼仪 ≥ 600
- 完成事件 Flow_General_Ep3
- 性向 ≥ -149

**前置事件链**
- `Flow_General_Ep0`：与 Castle_RegentDeputy 见面次数 > 0；Festival_Battle 历史符合 [Rank=1&Weapon=Physical]；已完成事件 Flow_CapitalEliteTroop_Ep2；性向 ≥ -149
- `Flow_General_Ep1`：已完成事件 Flow_General_Ep0
- `Flow_General_Ep2`：已完成事件 Flow_General_Ep1；战士评价阶级 ≥ 3；体术等级 ≥ 5；剑术等级 ≥ 5；性向 ≥ -149
- `Flow_General_Ep3`：已完成事件 Flow_General_Ep2

## 王宫魔法师（RoyalWizard）  优先权 Weight 42000
**结局条件（全部都要满足）**
- 炼金术达 S（等级 5）
- 魔法学达 S（等级 5）
- 完成事件 Flow_RoyalWizard_Ep3
- 魔法评价 ≥ 350
- 智力＋魔力＋精神力 ≥ 2400
- 性向 ≥ -149

**前置事件链**
- `Flow_RoyalWizard_Ep1`：与 Castle_Archmage 见面次数 ≥ 1；魔法评价阶级 > 4；炼金术等级 ≥ 3；魔法学等级 ≥ 4
- `Flow_RoyalWizard_Ep2`：已完成事件 Flow_RoyalWizard_Ep1；任务条件达成 Quest_RoyalWizard1
- `Flow_RoyalWizard_Ep3`：已完成事件 Flow_RoyalWizard_Ep2；任务条件达成 Quest_RoyalWizard2

## 王立艺术学会会长（RAAPresident）  优先权 Weight 41000
**结局条件（全部都要满足）**
- 美术达 A（等级 4）
- 舞蹈达 A（等级 4）
- 音乐达 A（等级 4）
- 艺术评价 ≥ 350
- 感性＋品味＋魅力 ≥ 2400
- 艺术节夺冠
- 性向 ≥ -149

**前置事件链**
- `Flow_RAAPresident_Ep1`：与 Castle_Consort 见面次数 > 2；【不可】完成任务 Quest_RAAPresident；Festival_Art 历史符合 [Rank=1]
- `Flow_RAAPresident_Ep2`：完成任务 Quest_RAAPresident

## 大富豪（Rich）  优先权 Weight 40000
**结局条件（全部都要满足）**
- 持有金钱 ≥ 50000
- 完成事件 Flow_Rich_Ep4

**前置事件链**
- `Flow_Rich_Ep1`：持有金钱 ≥ 20000
- `Flow_Rich_Ep2`：已完成事件 Event_Rich_Invest；距 Flow_Rich_Ep1 完成当月月初 ≥ 0年3月0日
- `Flow_Rich_Ep3`：已完成事件 Flow_Rich_Ep2；距 Flow_Rich_Ep2 完成当月月初 ≥ 0年4月0日
- `Flow_Rich_Ep4`：已完成事件 Flow_Rich_Ep3；距 Flow_Rich_Ep3 完成当月月初 ≥ 0年8月0日

## 魔法剑士（MagicKnight）  优先权 Weight 40000
**结局条件（全部都要满足）**
- 战士评价阶级 ≥ 4（评价 ≥ 250）
- 魔法评价阶级 ≥ 4（评价 ≥ 250）
- 完成事件 Flow_MagicKnight_Ep1
- 魔力 ≥ 600
- 力量 ≥ 600

**前置事件链**
- `Flow_MagicKnight_Ep1`：战士评价 ≥ 2；魔法学等级 ≥ 3；符合 [Rank!=1&Weapon=Physical]
- `Flow_MagicKnight_Ep2`：已完成事件 Flow_MagicKnight_Ep1；符合 [Rank=1]

## 外交官（Diplomat）  优先权 Weight 40000
**结局条件（全部都要满足）**
- 完成事件 Flow_Diplomat_Ep3
- 社交评价 ≥ 350
- 气质＋礼仪＋口才 ≥ 1800
- 智力 ≥ 500
- 性向 ≥ -149

**前置事件链**
- `Flow_Diplomat_Ep1`：社交三项(气质+礼仪+口才)总和 ≥ 1200；社交评价阶级 ≥ 3；智力 ≥ 100；性向 ≥ -149
- `Flow_Diplomat_Ep2`：已完成事件 Flow_Diplomat_Ep1
- `Flow_Diplomat_Ep3`：已完成事件 Flow_Diplomat_Ep2；社交三项(气质+礼仪+口才)总和 ≥ 1500；智力 ≥ 300

## 沙龙说客（Sting）  优先权 Weight 34000
**结局条件（全部都要满足）**
- 性向 ≤ -150
- 社交评价 ≥ 350
- 气质＋礼仪＋口才 ≥ 1600
- 智力 ≥ 500

**前置事件链**
- `Flow_Sting_Ep1`：社交评价阶级 ≥ 3
- `Flow_Sting_Ep2`：已完成事件 Flow_Sting_Ep1；性向 ≤ -150
- `Flow_Sting_Ep3`：已完成事件 Flow_Sting_Ep2；智力 > 400；性向 ≤ -150

## 蛇蝎美人（Femme）  优先权 Weight 33000
**结局条件（全部都要满足）**
- 性向 ≤ -150
- 酒馆达 S（等级 5）
- 魅力 ≥ 600
- 魅力 ≥ 400
- 魅力 ≥ 600

**前置事件链**
- `Flow_Femme_Ep1`：酒馆 累计次数 ≥ 10；性向 ≤ -90；魅力 > 100
- `Flow_Femme_Ep2`：已完成事件 Flow_Femme_Ep1；酒馆等级 ≥ 4；魅力 > 300；性向 ≤ -120
- `Flow_Femme_Ep3`：已完成事件 Flow_Femme_Ep2；酒馆 累计次数 ≥ 25；魅力 > 500；性向 ≤ -150

## 混沌炼金术师（ChaosAlchemist）  优先权 Weight 32000
**结局条件（全部都要满足）**
- 性向 ≤ -150
- 炼金术达 A（等级 4）
- 智力＋魔力＋精神力 ≥ 1500
- 品味 ≥ 300

**前置事件链**
- `Flow_ChaosAlchemist_Ep1`：炼金术等级 ≥ 3
- `Flow_ChaosAlchemist_Ep2`：已完成事件 Flow_ChaosAlchemist_Ep1；炼金术等级 ≥ 4；性向 ≤ -10
- `Flow_ChaosAlchemist_Ep3`：已完成事件 Flow_ChaosAlchemist_Ep2；性向 ≤ -90

## 怪盗（MysteriousThief）  优先权 Weight 32000
**结局条件（全部都要满足）**
- 性向 ≤ -150
- 智力 ≥ 300
- 敏捷 ≥ 600
- 品味 ≥ 600
- 完成事件 Flow_MysteriousThief_Ep3

**前置事件链**
- `Flow_MysteriousThief_Ep1`：敏捷 ≥ 400
- `Flow_MysteriousThief_Ep2`：【不可】已完成事件 Event_MysteriousThief_Prison；已完成事件 Flow_MysteriousThief_Ep1；品味 ≥ 400；性向 ≤ -150
- `Flow_MysteriousThief_Ep3`：已完成事件 Flow_MysteriousThief_Ep2；智力 ≥ 200；性向 ≤ -150

## 占卜师（FortuneTeller）  优先权 Weight 31000
**结局条件（全部都要满足）**
- 魔法评价阶级 ≥ 4（评价 ≥ 250）
- 性向 ≤ -150
- 精神力 ≥ 400
- 口才 ≥ 400
- 感性 ≥ 400

**前置事件链**
- `Flow_FortuneTeller_Ep1`：月份 ≠ 4；口才 > 200；感性 > 200；精神力 > 200；性向 ≤ -150
- `Flow_FortuneTeller_Ep2`：月份 ≠ 4；口才 > 300；精神力 > 300；感性 > 300；性向 ≤ -150

## 后巷之王（TheGodfather）  优先权 Weight 31000
**结局条件（全部都要满足）**
- 性向 ≤ -150
- 力量 ≥ 800
- 体力 ≥ 400
- 战士评价 ≥ 350

**前置事件链**
- `Flow_TheGodfather_Ep1`：已完成事件 Flow_UnloadingManager_Ep1；战士评价阶级 ≥ 2
- `Flow_TheGodfather_Ep2`：已完成事件 Flow_TheGodfather_Ep1；【不可】已完成事件 Flow_TheGodfather2_Ep2；战士评价阶级 ≥ 3

## 魔导工程师（MagicEngineer）  优先权 Weight 27000
**结局条件（全部都要满足）**
- 魔法学达 A（等级 4）
- 炼金术达 A（等级 4）
- 智力＋魔力＋精神力 ≥ 2000

**前置事件链**
- `Flow_MagicEngineer_Ep1`：本月行程为 炼金术；魔法学 累计次数 ≥ 13；炼金术 累计次数 ≥ 13；魔法三项(智力+魔力+精神)总和 ≥ 1500
- `Flow_MagicEngineer_Ep2`：本月行程为 炼金术；魔法学 累计次数 ≥ 18；炼金术 累计次数 ≥ 18；魔法三项(智力+魔力+精神)总和 ≥ 1800

## 剑舞姬（Sworddancer）  优先权 Weight 27000
**结局条件（全部都要满足）**
- 剑术达 A（等级 4）
- 舞蹈达 B（等级 3）
- 魅力 ≥ 500

**前置事件链**
- `Flow_Sworddancer_Ep1`：本月行程为 剑术；剑术 累计次数 ≥ 12；舞蹈 累计次数 ≥ 8；敏捷 ≥ 150；已完成事件 StatCheck_Sword_Ep1
- `Flow_Sworddancer_Ep2`：本月行程为 舞蹈；剑术 累计次数 ≥ 16；舞蹈 累计次数 ≥ 10；魅力 ≥ 450；已完成事件 Flow_Sworddancer_Ep1

## 调香师（Perfumer）  优先权 Weight 27000
**结局条件（全部都要满足）**
- 植物园达 A（等级 4）
- 炼金术达 B（等级 3）
- 气质 ≥ 500

**前置事件链**
- `Flow_Perfumer_Ep1`：本月行程为 植物园；植物园 累计次数 ≥ 15；炼金术 累计次数 ≥ 8；感性 ≥ 150；已完成事件 StatCheck_Garden_Ep1
- `Flow_Perfumer_Ep2`：本月行程为 炼金术；植物园 累计次数 ≥ 25；炼金术 累计次数 ≥ 10；气质 ≥ 450；已完成事件 Flow_Perfumer_Ep1

## 贵金属工匠（Jeweler）  优先权 Weight 27000
**结局条件（全部都要满足）**
- 铁匠铺达 A（等级 4）
- 美术达 B（等级 3）
- 艺术评价 ≥ 100
- 气质 ≥ 500

**前置事件链**
- `Flow_Jeweler_Ep1`：本月行程为 铁匠铺；铁匠铺 累计次数 ≥ 12；美术 累计次数 ≥ 8；体力 ≥ 150
- `Flow_Jeweler_Ep2`：本月行程为 铁匠铺；铁匠铺 累计次数 ≥ 16；美术 累计次数 ≥ 10；艺术评价 ≥ 100；气质 ≥ 450；已完成事件 Flow_Jeweler_Ep1

## 吟游诗人（Bard）  优先权 Weight 27000
**结局条件（全部都要满足）**
- 音乐达 A（等级 4）
- 文学达 B（等级 3）
- 完成事件 Flow_Bard_Ep3
- 口才 ≥ 500

**前置事件链**
- `Flow_Bard_Ep1`：本月行程为 文学；文学 累计次数 ≥ 8；音乐 累计次数 ≥ 10；感性 ≥ 150
- `Flow_Bard_Ep2`：本月行程为 音乐；音乐 累计次数 ≥ 16；文学 累计次数 ≥ 10；口才 ≥ 450；已完成事件 Flow_Bard_Ep1
- `Flow_Bard_Ep3`：口才 ≥ 450；已完成事件 Flow_Bard_Ep2

## 魔法阵图案师（SigilDrafter）  优先权 Weight 27000
**结局条件（全部都要满足）**
- 魔法学达 A（等级 4）
- 美术达 B（等级 3）
- 艺术评价 ≥ 100
- 感性 ≥ 500

**前置事件链**
- `Flow_SigilDrafter_Ep1`：本月行程为 魔法学；魔法学 累计次数 ≥ 12；美术 累计次数 ≥ 8；魔力 ≥ 150
- `Flow_SigilDrafter_Ep2`：本月行程为 美术；魔法学 累计次数 ≥ 16；美术 累计次数 ≥ 10；艺术评价 ≥ 100；感性 ≥ 450；已完成事件 Flow_SigilDrafter_Ep1

## 首席芭蕾舞者（Dancer）  优先权 Weight 23000
**结局条件（全部都要满足）**
- 舞蹈达 A（等级 4）
- 魅力 ≥ 300
- 品味 ≥ 300
- 气质 ≥ 300
- 艺术节舞蹈赛夺冠

**前置事件链**
- `Flow_Dancer_Ep1`：本月行程为 舞蹈；舞蹈 累计次数 ≥ 22；气质 ≥ 100
- `Flow_Dancer_Ep2`：本月行程为 舞蹈；舞蹈 累计次数 ≥ 27；气质 ≥ 200

## 首席歌姬（PrimaDonna）  优先权 Weight 23000
**结局条件（全部都要满足）**
- 音乐达 A（等级 4）
- 感性 ≥ 300
- 魅力 ≥ 300
- 口才 ≥ 300
- 艺术节音乐赛夺冠

**前置事件链**
- `Flow_PrimaDonna_Ep1`：本月行程为 音乐；音乐 累计次数 ≥ 22；口才 ≥ 100
- `Flow_PrimaDonna_Ep2`：本月行程为 音乐；音乐 累计次数 ≥ 27；口才 ≥ 200

## 学院院长（AcademyDean）  优先权 Weight 23000
**结局条件（全部都要满足）**
- 学院达 A（等级 4）
- 智力 ≥ 500
- 口才 ≥ 300

**前置事件链**
- `Flow_AcademyDean_Ep1`：本月行程为 学院；学院 累计次数 ≥ 23；智力 ≥ 200；品味 ≥ 150
- `Flow_AcademyDean_Ep2`：本月行程为 学院；学院 累计次数 ≥ 28；智力 ≥ 300；已完成事件 Flow_AcademyDean_Ep1
- `Flow_AcademyDean_Ep3`：本月行程为 学院；智力 ≥ 450；品味 ≥ 200；口才 ≥ 270；已完成事件 Flow_AcademyDean_Ep2

## 厨师（Cook）  优先权 Weight 22500
**结局条件（全部都要满足）**
- 餐厅达 S（等级 5）
- 厨艺祭夺冠
- 品味 ≥ 500
- 魔力 ≥ 200

**前置事件链**
- `Flow_Cook_Ep1`：本月行程为 餐厅；餐厅 累计次数 ≥ 38
- `Flow_Cook_Ep2`：本月行程为 餐厅；餐厅 累计次数 ≥ 45；Festival_Cook 优胜次数 ≥ 1

## 魔法事务员（MagicClerk）  优先权 Weight 20900
**结局条件（全部都要满足）**
- 魔法评价阶级 ≥ 4（评价 ≥ 250）
- 魔法学达 A（等级 4）
- 智力＋魔力＋精神力 ≥ 1500
- 完成事件 Flow_MagicClerk_Ep2

**前置事件链**
- `Flow_MagicClerk_Ep1`：本月行程为 魔法学；魔法学 累计次数 ≥ 10；Castle_MagicClerk 拜访次数 ≥ 3
- `Flow_MagicClerk_Ep2`：已完成事件 Flow_MagicClerk_Ep1；魔法学 累计次数 ≥ 10；Castle_MagicClerk 拜访次数 ≥ 3

## 首都精锐兵团（CapitalEliteTroop）  优先权 Weight 20850
**结局条件（全部都要满足）**
- 战士评价阶级 ≥ 4（评价 ≥ 250）
- 性向 ≥ 30
- 体力＋力量＋敏捷 ≥ 1500

**前置事件链**
- `Flow_CapitalEliteTroop_Ep1`：本月行程为 自由活动；剑术等级 ≥ 3
- `Flow_CapitalEliteTroop_Ep2`：已完成事件 Flow_CapitalEliteTroop_Ep1；力量 > 400

## 冒险家（Adventurer）  优先权 Weight 20700
**结局条件（全部都要满足）**
- 战士评价阶级 ≥ 4（评价 ≥ 250）
- 体力 ≥ 400
- 力量 ≥ 400
- 探险过龙族地区
- 探险过森族地区
- 探险过兽族地区

**前置事件链**
- `Flow_Adventurer_Ep1`：击杀怪物数(全部) > 10
- `Flow_Adventurer_Ep2`：已完成事件 Flow_Adventurer_Ep1；任务条件达成 Quest_Adventurer1
- `Flow_Adventurer_Ep3`：已完成事件 Flow_Adventurer_Ep2；战士评价 > 3
- `Flow_Adventurer_Ep4`：已完成事件 Flow_Adventurer_Ep3；任务条件达成 Quest_Adventurer2

## 剑术道场教头（SwordInstructor）  优先权 Weight 20495
**结局条件（全部都要满足）**
- 剑术达 B（等级 3）
- 敏捷 ≥ 300
- 体力 ≥ 300

**前置事件链**
- `Flow_SwordInstructor_Ep1`：本月行程为 剑术；剑术 累计次数 ≥ 12
- `Flow_SwordInstructor_Ep2`：本月行程为 剑术；剑术 累计次数 ≥ 20；敏捷 ≥ 270；体力 ≥ 270

## 作家（Writer）  优先权 Weight 20495
**结局条件（全部都要满足）**
- 文学达 B（等级 3）
- 口才 ≥ 300
- 感性 ≥ 300

**前置事件链**
- `Flow_Writer_Ep1`：本月行程为 文学；文学 累计次数 ≥ 12
- `Flow_Writer_Ep2`：本月行程为 文学；文学 累计次数 ≥ 20；口才 ≥ 270；感性 ≥ 270

## 画家（Painter）  优先权 Weight 20495
**结局条件（全部都要满足）**
- 美术达 B（等级 3）
- 品味 ≥ 300
- 感性 ≥ 300

**前置事件链**
- `Flow_Painter_Ep1`：本月行程为 美术；美术 累计次数 ≥ 12
- `Flow_Painter_Ep2`：本月行程为 美术；美术 累计次数 ≥ 20；品味 ≥ 270；感性 ≥ 270

## 舞蹈教师（DanceTutor）  优先权 Weight 20495
**结局条件（全部都要满足）**
- 舞蹈达 B（等级 3）
- 魅力 ≥ 300
- 品味 ≥ 300

**前置事件链**
- `Flow_DanceTutor_Ep1`：本月行程为 舞蹈；舞蹈 累计次数 ≥ 12
- `Flow_DanceTutor_Ep2`：本月行程为 舞蹈；舞蹈 累计次数 ≥ 20；魅力 ≥ 270；品味 ≥ 270

## 音乐教师（MusicTutor）  优先权 Weight 20495
**结局条件（全部都要满足）**
- 音乐达 B（等级 3）
- 感性 ≥ 300
- 魅力 ≥ 300

**前置事件链**
- `Flow_MusicTutor_Ep1`：本月行程为 音乐；音乐 累计次数 ≥ 12
- `Flow_MusicTutor_Ep2`：本月行程为 音乐；音乐 累计次数 ≥ 20；感性 ≥ 270；魅力 ≥ 270

## 历史学者（Historian）  优先权 Weight 20495
**结局条件（全部都要满足）**
- 历史达 B（等级 3）
- 智力 ≥ 300
- 气质 ≥ 300

**前置事件链**
- `Flow_Historian_Ep1`：本月行程为 历史；历史 累计次数 ≥ 12
- `Flow_Historian_Ep2`：本月行程为 历史；历史 累计次数 ≥ 20；智力 ≥ 270；气质 ≥ 270

## 魔法教习（MagicTutor）  优先权 Weight 20495
**结局条件（全部都要满足）**
- 魔法学达 B（等级 3）
- 魔力 ≥ 300
- 智力 ≥ 300

**前置事件链**
- `Flow_MagicTutor_Ep1`：本月行程为 魔法学；魔法学 累计次数 ≥ 12
- `Flow_MagicTutor_Ep2`：本月行程为 魔法学；魔法学 累计次数 ≥ 20；智力 ≥ 270；魔力 ≥ 270

## 礼法教师（Governess）  优先权 Weight 20495
**结局条件（全部都要满足）**
- 礼法达 A（等级 4）
- 礼仪 ≥ 400
- 气质 ≥ 400
- 感性 ≥ 200

**前置事件链**
- `Flow_Governess_Ep1`：本月行程为 礼法；礼法 累计次数 ≥ 12
- `Flow_Governess_Ep2`：本月行程为 礼法；礼法 累计次数 ≥ 20；礼仪 ≥ 350；气质 ≥ 350

## 体术教头（MartialArtsInstructor）  优先权 Weight 20495
**结局条件（全部都要满足）**
- 体术达 B（等级 3）
- 精神力 ≥ 300
- 体力 ≥ 300

**前置事件链**
- `Flow_MartialArtsInstructor_Ep1`：本月行程为 体术；体术 累计次数 ≥ 12；精神力 ≥ 100；已完成事件 StatCheck_MartialArts_Ep1
- `Flow_MartialArtsInstructor_Ep2`：本月行程为 体术；体术 累计次数 ≥ 20；精神力 ≥ 270；体力 ≥ 270；已完成事件 Flow_MartialArtsInstructor_Ep1

## 炼金术师（Alchemist）  优先权 Weight 20495
**结局条件（全部都要满足）**
- 炼金术达 B（等级 3）
- 智力 ≥ 300
- 魔力 ≥ 300

**前置事件链**
- `Flow_Alchemist_Ep1`：本月行程为 炼金术；炼金术 累计次数 ≥ 12；智力 ≥ 100
- `Flow_Alchemist_Ep2`：本月行程为 炼金术；炼金术 累计次数 ≥ 20；智力 ≥ 270；魔力 ≥ 270；已完成事件 Flow_Alchemist_Ep1

## 物流主管（UnloadingManager）  优先权 Weight 20240
**结局条件（全部都要满足）**
- 码头达 A（等级 4）
- 体力 ≥ 300
- 精神力 ≥ 300

**前置事件链**
- `Flow_UnloadingManager_Ep1`：本月行程为 码头；码头 累计次数 ≥ 10
- `Flow_UnloadingManager_Ep2`：本月行程为 码头；码头等级 ≥ 4；体力 ≥ 270；精神力 ≥ 270

## 调酒师（Bartender）  优先权 Weight 20240
**结局条件（全部都要满足）**
- 酒馆达 A（等级 4）
- 口才 ≥ 300
- 气质 ≥ 300

**前置事件链**
- `Flow_Bartender_Ep1`：本月行程为 酒馆；酒馆 累计次数 ≥ 10
- `Flow_Bartender_Ep2`：本月行程为 酒馆；酒馆等级 ≥ 4；口才 ≥ 270；气质 ≥ 270

## 家庭教师（PrivateTutor）  优先权 Weight 20200
**结局条件（全部都要满足）**
- 家庭教师达 A（等级 4）
- 智力 ≥ 300
- 礼仪 ≥ 300

**前置事件链**
- `Flow_PrivateTutor_Ep1`：本月行程为 家庭教师；家庭教师 累计次数 ≥ 10
- `Flow_PrivateTutor_Ep2`：本月行程为 家庭教师；家庭教师等级 ≥ 4；智力 ≥ 270；礼仪 ≥ 270

## 女仆咖啡厅的超级明星（Superstar）  优先权 Weight 20200
**结局条件（全部都要满足）**
- 女仆咖啡厅达 A（等级 4）
- 魅力 ≥ 300
- 感性 ≥ 300
- 口才 ≥ 200

**前置事件链**
- `Flow_Superstar_Ep1`：本月行程为 女仆咖啡厅；女仆咖啡厅 累计次数 ≥ 10；魅力 ≥ 300；感性 ≥ 150
- `Flow_Superstar_Ep2`：本月行程为 女仆咖啡厅；女仆咖啡厅等级 ≥ 4；魅力 ≥ 500；感性 ≥ 300；口才 ≥ 150

## 研究助理（Assistant）  优先权 Weight 20200
**结局条件（全部都要满足）**
- 学院达 B（等级 3）
- 智力 ≥ 300
- 品味 ≥ 300

**前置事件链**
- `Flow_Assistant_Ep1`：本月行程为 学院；学院 累计次数 ≥ 10
- `Flow_Assistant_Ep2`：本月行程为 学院；学院 累计次数 ≥ 19；智力 ≥ 270；品味 ≥ 270

## 服装设计师（Designer）  优先权 Weight 20160
**结局条件（全部都要满足）**
- 服装店达 A（等级 4）
- 气质 ≥ 300
- 魅力 ≥ 300

**前置事件链**
- `Flow_Designer_Ep1`：本月行程为 服装店；服装店 累计次数 ≥ 11
- `Flow_Designer_Ep2`：本月行程为 服装店；服装店 累计次数 ≥ 24；气质 ≥ 280；魅力 ≥ 280

## 铁匠（Blacksmith）  优先权 Weight 20160
**结局条件（全部都要满足）**
- 铁匠铺达 A（等级 4）
- 体力 ≥ 300
- 魔力 ≥ 300

**前置事件链**
- `Flow_Blacksmith_Ep1`：本月行程为 铁匠铺；铁匠铺 累计次数 ≥ 12
- `Flow_Blacksmith_Ep2`：本月行程为 铁匠铺；铁匠铺 累计次数 ≥ 24；体力 ≥ 270；魔力 ≥ 270

## 士兵（Soldier）  优先权 Weight 20080
**结局条件（全部都要满足）**
- 检问处达 A（等级 4）
- 礼仪 ≥ 300
- 体力 ≥ 300

**前置事件链**
- `Flow_Soldier_Ep1`：本月行程为 检问处；检问处 累计次数 ≥ 11
- `Flow_Soldier_Ep2`：本月行程为 检问处；检问处等级 ≥ 4；礼仪 ≥ 270；体力 ≥ 270

## 物流管理员（Logistics）  优先权 Weight 20080
**结局条件（全部都要满足）**
- 水果仓库达 A（等级 4）
- 力量 ≥ 300
- 精神力 ≥ 300

**前置事件链**
- `Flow_Logistics_Ep1`：本月行程为 水果仓库；水果仓库 累计次数 ≥ 13
- `Flow_Logistics_Ep2`：本月行程为 水果仓库；水果仓库等级 ≥ 4；力量 ≥ 270；精神力 ≥ 270

## 家政服务公司老板（Housekeeper）  优先权 Weight 20000
**结局条件（全部都要满足）**
- 家务达 A（等级 4）
- 智力 ≥ 250

**前置事件链**
- `Flow_Housekeeper_Ep1`：本月行程为 家务；家务 累计次数 ≥ 18
- `Flow_Housekeeper_Ep2`：本月行程为 家务；家务 累计次数 ≥ 38

## 村庄治疗师（Doctor）  优先权 Weight 20000
**结局条件（全部都要满足）**
- 诊疗所达 B（等级 3）
- 精神力 ≥ 300
- 体力 ≥ 300

**前置事件链**
- `Flow_Doctor_Ep1`：本月行程为 诊疗所；诊疗所 累计次数 ≥ 18
- `Flow_Doctor_Ep2`：本月行程为 诊疗所；诊疗所 累计次数 ≥ 35；体力 ≥ 270；精神力 ≥ 270

## 木匠（Carpenter）  优先权 Weight 20000
**结局条件（全部都要满足）**
- 木工坊达 B（等级 3）
- 力量 ≥ 300
- 品味 ≥ 300

**前置事件链**
- `Flow_Carpenter_Ep1`：本月行程为 木工坊；木工坊 累计次数 ≥ 18
- `Flow_Carpenter_Ep2`：本月行程为 木工坊；木工坊 累计次数 ≥ 35；力量 ≥ 270；品味 ≥ 270

## 餐厅店员（Waitress）  优先权 Weight 20000
**结局条件（全部都要满足）**
- 餐厅达 B（等级 3）
- 品味 ≥ 300
- 口才 ≥ 300

**前置事件链**
- `Flow_Waitress_Ep1`：本月行程为 餐厅；餐厅 累计次数 ≥ 18
- `Flow_Waitress_Ep2`：本月行程为 餐厅；餐厅 累计次数 ≥ 30

## 牧场管理员（RanchManager）  优先权 Weight 20000
**结局条件（全部都要满足）**
- 牧场达 B（等级 3）
- 体力 ≥ 300
- 感性 ≥ 300

**前置事件链**
- `Flow_RanchManager_Ep1`：本月行程为 牧场；牧场 累计次数 ≥ 18
- `Flow_RanchManager_Ep2`：本月行程为 牧场；牧场 累计次数 ≥ 35

## 园艺治疗师（Therapist）  优先权 Weight 20000
**结局条件（全部都要满足）**
- 植物园达 B（等级 3）
- 感性 ≥ 300
- 魔力 ≥ 300

**前置事件链**
- `Flow_Therapist_Ep1`：本月行程为 植物园；植物园 累计次数 ≥ 18
- `Flow_Therapist_Ep2`：本月行程为 植物园；植物园 累计次数 ≥ 35；感性 ≥ 270；魔力 ≥ 270

## 村民（Villager）  优先权 Weight 2
**结局条件（全部都要满足）**
- 礼仪 ≥ 100

**前置事件链**
- `Flow_Villager_Ep1`：本月行程为 家务；家务 累计次数 ≥ 15
- `Flow_Villager_Ep2`：本月行程为 家务；家务 累计次数 ≥ 29

## 无业（Jobless）  优先权 Weight 1
**结局条件（全部都要满足）**
- （无条件）

**前置事件链**
- `Flow_Jobless_Ep1`：本月行程为 在家休息；年份 ≤ 1210；Rest 次数 ≥ 15
- `Flow_Jobless_Ep2`：本月行程为 在家休息；年份 ≤ 1211；Rest 次数 ≥ 30
