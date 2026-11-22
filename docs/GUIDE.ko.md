# 預言之子 (Princess Maker: Children of Revelation) 엔딩 조건 정리 (게임 데이터 테이블 기준)

자료 출처: `TableData`(EndingStatus, Ending, Reputation, Quest, Event)와 `C_Flow_*` 이벤트 스크립트이며, 바하 공략의 "엔딩별 파라미터"와 대조했습니다.
최종 업데이트: 2026-11-21

## 읽는 법

- "엔딩 조건"은 곧 `EndingStatus`입니다. **서로 다른 조건 그룹은 "그리고(AND)", 같은 그룹 안은 "또는(OR)"입니다**(게임 판정 코드로 확인됨. 여왕의 전사평가/마법평가만 같은 그룹에 들어 있습니다).
- 평가에는 두 가지 표기가 있습니다. "평가 ≥ N"은 평가 수치 자체이고, "평가 등급 ≥ N"은 등급입니다(1: 0, 2: 50, 3: 150, 4: 250, 5: 350, 6: 500).
- 수업/일 등급: 3 = B, 4 = A, 5 = S (B, A는 포럼 및 `Quest_RAAPresident`의 요구 사항인 "미술, 음악, 무용 A 달성 = 등급 4"와 대조해 확인했고, S는 순서대로 추론한 값입니다).
- "성향"은 곧 선악 수치입니다. ≤ -150이면 혼돈, ≥ 225이면 질서입니다(여왕).
- "선행 이벤트 체인"은 `Flow_*` 스크립트의 발생 조건이며, 엔딩 조건에 `이벤트 완료 Flow_xxx`가 있는 경우에만 필수입니다. **엔딩 조건에 나열되지 않은 이벤트 체인(예: 혼돈의 연금술사)은 엔딩 필수 조건이 아니며** 단지 스토리일 뿐입니다.
- Trigger: 1010 월초, 1017 스케줄 후반 랜덤, 2012 왕성에서 인물 조우, 5001 축제 종료 후, 5002 축제 시작.

## 엔딩이 결정되는 방식 (GameAssembly 디스어셈블 기준: `lxc::mko`/`mau`/`sai`)

**1. 후보 자격.** `EndingType = 2`인 일반 엔딩 55개만 봅니다(`Couple_*`는 `EndingType = 3`이며 이 단계에 포함되지 않습니다). 엔딩이 후보가 되려면 **각 조건 그룹마다 최소 하나의 조건이 성립해야 합니다**(그룹 내 "또는", 그룹 간 "그리고". `Cond_CheckEndingCondition`과 `mko` 두 곳에서 확인됨). 개별 조건의 성립 여부는 다음과 같습니다. `CompareType = 0`(이벤트, 무조건)은 "현재 값 ≥ 1"이고, 나머지는 ≥/≤로 CompareValue와 비교합니다.

**2. 점수 계산.** 각 후보의 점수 = **Weight + 보너스**이며, **점수가 가장 높은 것**만 남기고 **동점은 모두 남깁니다**.

**3. 추첨.** 하나만 남으면 그것을 사용하고, 여럿이면 **무작위로 1개를 뽑습니다**(`sai`). 따라서 동점 엔딩은 무작위로 결정됩니다.

**4. 보너스(`mau`).**
- Weight ≥ 30000인 엔딩: 보너스가 없습니다.
- Weight < 30000인 엔딩: 보너스 = `min(2 × 수치, 8000)`. 수치는 해당 엔딩 조건 안의 모든 "수업/아르바이트 등급" 조건(Type 8)이 가리키는 수업의 플레이어 기록 누적값(기록 필드 `lcq`) 중 **가장 큰 하나**입니다.
- **음유시인(Bard)은 예외입니다**: 보너스 = `min(2 × (Edu_Music + Edu_Literature의 누적값), 8000)`.
- `lcq` 필드명은 난독화되어 있지만, `ld::fno(수업Key, 누적값)`의 디스어셈블로 용도를 확인했습니다. 이 함수는 해당 수업의 `ScheduleStat` 각 등급 행 중 `NeedCount_GradeUp`이 누적값보다 **큰** 행 가운데 가장 작은 행을 현재 등급으로 삼습니다. 즉 `NeedCount_GradeUp`은 **누적 문턱**이고, `lcq`는 그것과 비교되는 누적 횟수입니다(예: 수업은 누적 ≥ 42회면 등급 2, ≥ 96회면 B, ≥ 160회면 A, ≥ 234회면 S). 따라서 **보너스는 실질적으로 2 × 누적 횟수이며 대략 수백, 많아야 천 남짓**이라서 8000의 상한에는 거의 닿지 않습니다.

**5. 이것이 의미하는 바.**
- 난이도가 높은 엔딩(여왕 52000, 마왕 51000, 프린세스 50000, 대장군 등)은 Weight만으로 모든 것을 압도하므로, 조건을 모두 통과하면 그 엔딩이 됩니다.
- 보너스가 작아서 **Weight 구간을 넘지 못합니다**(예: 20000대가 27000대를 따라잡지 못함). 보너스의 역할은 **Weight가 비슷한 엔딩끼리의 순서를 재조정**하는 것입니다. 예컨대 20000, 20080, 20160, 20200, 20240, 20495, 20700, 20850, 20900처럼 수십~수백 차이인 엔딩들은 더 많이 훈련한 쪽이 우선합니다.
- 완전한 동점(예: 대부호, 마법검사, 외교관은 모두 40000이며 보너스 없음)은 무작위로 하나가 뽑힙니다.

| 엔딩 | Weight | 수업/아르바이트 보너스 |
|---|---|---|
| 여왕 (Queen) | 52000 | 없음 |
| 마왕 (DemonKing) | 51000 | 없음 |
| 프린세스 (Princess) | 50000 | 없음 |
| 대장군 (General) | 43000 | 없음 |
| 왕궁마법사 (RoyalWizard) | 42000 | 없음 |
| 왕립예술학회장 (RAAPresident) | 41000 | 없음 |
| 외교관 (Diplomat) | 40000 | 없음 |
| 마법검사 (MagicKnight) | 40000 | 없음 |
| 대부호 (Rich) | 40000 | 없음 |
| 살롱 로비스트 (Sting) | 34000 | 없음 |
| 팜므파탈 (Femme) | 33000 | 없음 |
| 혼돈의 연금술사 (ChaosAlchemist) | 32000 | 없음 |
| 괴도 (MysteriousThief) | 32000 | 없음 |
| 점술가 (FortuneTeller) | 31000 | 없음 |
| 뒷골목의제왕 (TheGodfather) | 31000 | 없음 |
| 음유시인 (Bard) | 27000 | 있음 (약 2 × 누적 횟수) |
| 귀금속 세공사 (Jeweler) | 27000 | 있음 (약 2 × 누적 횟수) |
| 마도 공학자 (MagicEngineer) | 27000 | 있음 (약 2 × 누적 횟수) |
| 조향사 (Perfumer) | 27000 | 있음 (약 2 × 누적 횟수) |
| 마법진 도안사 (SigilDrafter) | 27000 | 있음 (약 2 × 누적 횟수) |
| 검의 무희 (Sworddancer) | 27000 | 있음 (약 2 × 누적 횟수) |
| 아카데미 학장 (AcademyDean) | 23000 | 있음 (약 2 × 누적 횟수) |
| 프리마 발레리나 (Dancer) | 23000 | 있음 (약 2 × 누적 횟수) |
| 프리마돈나 (PrimaDonna) | 23000 | 있음 (약 2 × 누적 횟수) |
| 요리사 (Cook) | 22500 | 있음 (약 2 × 누적 횟수) |
| 마법 사무원 (MagicClerk) | 20900 | 있음 (약 2 × 누적 횟수) |
| 수도정예병단 (CapitalEliteTroop) | 20850 | 없음 |
| 모험가 (Adventurer) | 20700 | 없음 |
| 연금술사 (Alchemist) | 20495 | 있음 (약 2 × 누적 횟수) |
| 무용 교사 (DanceTutor) | 20495 | 있음 (약 2 × 누적 횟수) |
| 예법교사 (Governess) | 20495 | 있음 (약 2 × 누적 횟수) |
| 역사학자 (Historian) | 20495 | 있음 (약 2 × 누적 횟수) |
| 마법 사범 (MagicTutor) | 20495 | 있음 (약 2 × 누적 횟수) |
| 체술사범 (MartialArtsInstructor) | 20495 | 있음 (약 2 × 누적 횟수) |
| 음악 교사 (MusicTutor) | 20495 | 있음 (약 2 × 누적 횟수) |
| 화가 (Painter) | 20495 | 있음 (약 2 × 누적 횟수) |
| 검술 도장 사범 (SwordInstructor) | 20495 | 있음 (약 2 × 누적 횟수) |
| 작가 (Writer) | 20495 | 있음 (약 2 × 누적 횟수) |
| 바텐더 (Bartender) | 20240 | 있음 (약 2 × 누적 횟수) |
| 물류 관리자 (UnloadingManager) | 20240 | 있음 (약 2 × 누적 횟수) |
| 연구조교 (Assistant) | 20200 | 있음 (약 2 × 누적 횟수) |
| 가정교사 (PrivateTutor) | 20200 | 있음 (약 2 × 누적 횟수) |
| 메이드 카페의 슈퍼스타 (Superstar) | 20200 | 있음 (약 2 × 누적 횟수) |
| 대장장이 (Blacksmith) | 20160 | 있음 (약 2 × 누적 횟수) |
| 의류 디자이너 (Designer) | 20160 | 있음 (약 2 × 누적 횟수) |
| 물류 관리인 (Logistics) | 20080 | 있음 (약 2 × 누적 횟수) |
| 병사 (Soldier) | 20080 | 있음 (약 2 × 누적 횟수) |
| 목수 (Carpenter) | 20000 | 있음 (약 2 × 누적 횟수) |
| 마을 치료사 (Doctor) | 20000 | 있음 (약 2 × 누적 횟수) |
| 가사 전문 업체 사장 (Housekeeper) | 20000 | 있음 (약 2 × 누적 횟수) |
| 목장 관리인 (RanchManager) | 20000 | 있음 (약 2 × 누적 횟수) |
| 원예 치료사 (Therapist) | 20000 | 있음 (약 2 × 누적 횟수) |
| 식당 점원 (Waitress) | 20000 | 있음 (약 2 × 누적 횟수) |
| 마을주민 (Villager) | 2 | 없음 |
| 백수 (Jobless) | 1 | 없음 |

## 점술가가 힌트를 주는 방식 (`lxc::mca`/`lzj`/`lyx`)

- 각 엔딩의 진행도 = Σ(각 조건 그룹의 최고 Score × 그 그룹 내 **최선 조건의 진행도**) ÷ Σ 각 그룹의 최고 Score.
- 개별 조건 진행도: 일반 ≥ 조건은 "현재 값 ÷ 목표 값"이며 0~1로 제한됩니다. 등급류(평판, 수업 등급)는 (현재 등급 − 1) ÷ (목표 등급 − 1), 선악 ≤ 조건은 (상한 − 현재) ÷ (상한 − 목표), 예/아니오형(이벤트, 무조건)은 0 또는 1입니다.
- 진행도 ≥ 문턱인 엔딩만 고려합니다. 문턱 = 0.2 + 0.6 × 경과 일수 ÷ 2920(8년), 즉 0.2에서 0.8까지 점차 올라갑니다.
- `Ending_Jobless`(Key 문자열 비교)와 Weight < 1000인 엔딩(Jobless와 Villager)은 제외합니다.
- 정렬: 진행도 내림차순, 진행도가 같으면 Weight + 보너스 내림차순. 최대 **3**개까지 표시합니다.

## 무용수와 가희의 조건

두 엔딩의 마지막 조건 그룹은 "예술제 우승"(Type 15 무용, 16 음악)입니다. `lxc::hch`의 계산 방식은 다음과 같습니다. 예술제 이력 기록을 순회하면서, 각 항목의 `&`로 구분된 태그가 **`Rank=1`과 `Category=<해당 분류>`를 동시에 포함**해야 1회로 치며, 조건은 ≥ 1입니다. 즉 "해당 분류의 예술제에서 1등을 한 적이 있다"입니다. 또한 `Cond_CheckEndingCondition`이라는 이벤트 노드는 15, 16을 처리하지 않지만, 전체 데이터에서 이 노드를 쓰는 곳은 프린세스 6화뿐이므로 이 두 엔딩에는 영향이 없습니다.

## 연인 엔딩이 결정되는 방식 (`pk::OnTheEnd`/`ShowCard`/`go::glc` 기준)

**연인 엔딩은 EndingStatus 조건과 완전히 무관하며**, "게임 상태의 결혼 상대"라는 숫자 하나만 봅니다. 직업 엔딩과는 **서로 독립된 두 장의 엔딩 카드**입니다. 직업 엔딩은 `lxc::sai`가 결정하고(앞 절), 연인 엔딩은 결혼 상대가 결정하며, 엔딩 화면에는 차례대로 표시됩니다.

**대응 규칙**(결혼 상대 코드 = `EPartnerNPC`, `Act_SetWeddingNPC`로 설정):

| 결혼 상대 코드 | 상대 | 연인 엔딩 | 결혼식 대화 그래프 |
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
| -1(마을 사람)/-3(직장 동료) | 일반 결혼 | `Ending_Couple_Normal` | — |
| -2(독신) | 결혼하지 않음 | 연인 엔딩 없음 | — |

**유일한 예외**: 결혼 상대가 아가레스(DemonMale, 2)이고 **직업 엔딩이 `Ending_Princess`**(프린세스)이면, 연인 엔딩은 `Ending_Couple_DemonMale`이 아니라 **`Ending_Couple_DemonMale_King`**으로 바뀝니다. 코드가 문자열 `Ending_Princess`로 비교하므로(`a::gx`), 프린세스 루트를 끝까지 마친 경우에만 "마왕" 버전을 받습니다. 그 밖의 조합(예: 직업 엔딩이 프린세스가 아닌 경우)은 여전히 `Ending_Couple_DemonMale`입니다.

**그 밖에 확인된 사항**
- 직업 엔딩의 유일한 호출처는 `pk::chs`(엔딩 화면)가 `lxc::sai`를 호출하는 곳이며, 다른 진입점은 없습니다.
- 결혼 상대가 -1이든 -3이든 모두 일반 결혼이며 같은 `Couple_Normal` 카드입니다. -2는 독신으로, "미혼" 업적만 주고 연인 엔딩은 없습니다.
- 업적 "모든 결혼 엔딩"은 지금까지의 클리어 기록 중 결혼 상대 코드 ≥ -4인 서로 다른 상대를 집계하며(-1과 -3은 하나로 합산), 누적 **11명**이 되면 달성됩니다. `go::glb`, `go::glc`가 지금까지 클리어한 엔딩을 컬렉션 목록에 넣는 역할을 합니다.
- 엔딩 화면에는 카드 한 장 `Card_Second_EA`(`a::ha`)가 추가로 들어가는데, 어떤 조건(`hl::ljk()`이 참이고 그 카드가 해금되어 있을 때)에서만 추가되며, 용도는 추적하지 않았습니다.

**엔딩 시점의 최종 판정 (`pk::gde`, 엔딩 화면 `pk::chs` 시작 부분에서 호출되어 결과를 결혼 상대에 다시 기록)**

결혼 상대에는 두 가지 출처가 있습니다. 청혼 이벤트에서 수락할 때 `Act_SetWeddingNPC`로 기록하는 것(코드 ≥ 0인 8명의 종족 파트너), 그리고 엔딩 시점에 `pk::gde`가 완료한 이벤트와 직업 엔딩을 바탕으로 채워 넣는 것입니다. `gde`의 순서는 다음과 같습니다(주소 `0x182638610`).

1. **결혼 상대가 이미 ≥ 0인 경우**(청혼을 수락한 적 있음): 그것을 그대로 사용하고 이후는 모두 건너뜁니다.
2. 그렇지 않으면 순서대로 확인하여 먼저 부합하는 것을 사용합니다.

| 순서 | 조건 (모두 성립) | 결과 |
|---|---|---|
| 1 | 이벤트 `Caim_Episode1` 완료, 그리고 직업 엔딩이 `Ending_Waitress`(식당 점원) | -5 카임 |
| 2 | 이벤트 `Icosa_Episode1` 완료, 그리고 직업 엔딩이 `Ending_MagicTutor`(마법 사범) | -6 아이코사 |
| 3 | 이벤트 `Kimaris_Episode1` 완료, 그리고 직업 엔딩이 `Ending_Soldier`(병사) | -7 키마리스 |
| 4 | 이벤트 `Muzai_Episode1` 완료, 그리고 직업 엔딩이 `Ending_Writer`(작가) | -8 무자이 |
| 5 | 이벤트 `Cube_Episode2` 완료(**직업 엔딩은 보지 않음**) | -4 큐브 |
| 6 | 12가지 기초 능력(체력, 힘, 민첩, 지능, 마력, 정신력, 기품, 예절, 화술, 감수성, 센스, 매력)의 **합계 ≥ 3600** | -2 독신(연인 엔딩 없음) |
| 7 | 직업 엔딩이 `Ending_Jobless` 또는 `Ending_Villager` | -1 일반 결혼(`Couple_Normal`) |
| 8 | 위 어느 것에도 해당하지 않음 | 현재 값 유지. 새 게임의 기본값은 **-3(직장 동료, 역시 `Couple_Normal`)**입니다. 세이브 데이터 클래스 `do`의 생성자 `do::do()`가 이 필드를 -3으로 초기화하는 것을 확인했습니다 |

또한 1~4, 5에는 각각 "현재 값이 이미 그 코드이면 그대로 사용"하는 지름길이 하나씩 더 있으며, 위 결과에는 영향을 주지 않습니다.

**이것이 의미하는 바**
- 카임, 아이코사, 키마리스, 무자이의 연인 엔딩을 얻으려면 **직업 엔딩이 정확히 해당하는 것**(식당 점원, 마법 사범, 병사, 작가)이어야 하며, 각자의 `Episode1` 완료도 필요합니다. 직업 엔딩이 맞지 않으면 바로 다음 항목으로 넘어갑니다.
- 큐브는 `Cube_Episode2`만 완료하면 성립하며 직업은 상관없습니다.
- 이 5명은 독신(합계 ≥ 3600)보다 우선도가 높으므로, 능력치가 매우 높은 딸이라도 해당 이벤트를 완료했다면 여전히 그들의 연인 엔딩을 받습니다.
- 능력 합계 ≥ 3600(평균 300)이고 파트너가 아무도 없을 때만 독신이 됩니다. 능력이 모자란데 직업이 백수/마을주민이면 일반 결혼입니다. 이 3600은 어셈블리에서 읽은 상수이며, 12가지는 `EStatus` 1~12입니다. 게임에서 직접 실측하지는 않았습니다.
- 8명의 종족 파트너(코드 0~7)는 1218년 3월의 청혼 이벤트에서 수락해야 코드가 ≥ 0이 되며, 아래 "각 파트너의 청혼 이벤트"에 조건이 있습니다.

## 바하 공략과의 대조: 대체로 맞지만 수치에 꽤 차이가 있음

**완전히 일치**: 목장 관리인, 연구조교, 음악 교사, 무용 교사, 마법 사범, 역사학자, 마법 사무원(지능·마력·정신력 1500, 마법평가 등급 4 = 250), 프리마 발레리나(무용 A에 무용 대회 우승), 마법검사(힘·마력 600, 두 평가 ≥ 250), 뒷골목의제왕(힘 800), 모험가(용·숲·수인 세 지역 탐험), 여왕의 네 항목 합계(1500/1500/2400/1500), 마왕의 두 항목 합계 2400, 프린세스의 두 항목 합계 2400, 왕립예술학회장의 감수성+센스+매력 2400과 예술평가 350, 왕궁마법사의 지능·마력·정신력 2400과 마법평가 350.

**수치가 다른 경우(데이터 기준)**

| 엔딩 | 바하 표기 | 데이터상 실제 |
|---|---|---|
| 프린세스 | 사교평가 ≥ 450 | **≥ 400** |
| 여왕 | 전사·사교평가 각각 500 | 사교 ≥ 500, **전사 또는 마법 중 하나 ≥ 400** |
| 마왕 | 전사·마법평가 500, 선악 -300 | 엔딩 조건에 **평가가 없고** 능력 합계 2400만 필요, 선악 **≤ -225**(이벤트 체인은 ≤ -150이면 됨) |
| 왕립예술학회장 | 기품+예절+화술 ≥ 1500 | **이 조건 없음**. 다만 **예술제 우승** + 미술·무용·음악 A 달성 필요 |
| 왕궁마법사 | 마법·연금 A | 마법학, 연금술 모두 **S(등급 5)** 필요 |
| 대장군 | 체·힘·민 ≥ 2100, 기품 200, 예절 300 | 체력+힘+민첩 **≥ 2400**, 기품+예절 **≥ 600**, 그리고 **검술 S, 체술 S, 무투회 우승** 필요 |
| 외교관 | 지능 > 750, 세 항목 ≥ 2100, 사교 500 | 지능 **≥ 500**, 기품+예절+화술 **≥ 1800**, 사교 **≥ 350** |
| 살롱 로비스트 | 지능 > 750, 각 700, 사교 500 | 지능 **≥ 500**, 세 항목 합계 **≥ 1600**, 사교 **≥ 350**, 선악 ≤ -150 |
| 팜므파탈 | 바 A, 체력과 화술 > 300 | 주점 **S** 달성, 매력 600, **체력·화술 조건 없음** |
| 뒷골목의제왕 | 전사평가 ≥ 250 | **≥ 350**, 체력 400, 힘 800 |
| 괴도 | 지능 600 | 지능 **≥ 300**, 민첩 600, 센스 600, 이벤트 Flow_MysteriousThief_Ep3 |
| 혼돈의 연금술사 | 마력 > 600, 연금 A, 이벤트 3회 | 연금 A, 지능·마력·정신력 ≥ **1500**, 센스 300. **이벤트는 필수 아님** |
| 점술가 | 지능·마력·정신력·화술 각 500, 마법 A | 정신력, 화술, 감수성 각 **400**, 마법평가 등급 ≥ 4, 선악 ≤ -150. **마법 A는 요구하지 않음** |
| 검의 무희 | 매력·힘 > 600, 무용 A, 검술 A | 검술 A, **무용 B**, 매력 **≥ 500** |
| 예법교사 | 예법 B, 예절·기품 > 300 | 예법 **A**, 예절 400, 기품 400, 감수성 200 |
| 마도 공학자 | 연금 A | 마법 A, 연금 A, **지능·마력·정신력 합계 ≥ 2000** |
| 체술사범 | 힘 300, 정신력 300 | **체력** 300, 정신력 300(체술 B) |
| 바텐더 | 체력, 화술 > 300 | **화술, 기품** 각 300(주점 A) |
| 슈퍼스타 | 감수성·매력 > 400, 사교 두 단계 상승 | 카페 A, 매력 300, 감수성 300, 화술 200, **사교평가 조건 없음** |
| 대부호 | 금전 > 5만 | 금전 ≥ 50000 **그리고** 이벤트 Flow_Rich_Ep4 완료(둘 다 필요, 아래 정정 참조) |

**바하에는 없고 데이터에는 있는 엔딩**: 요리사, 가정교사, 디자이너, 병사, 식당 점원, 대장장이, 물류(과일창고), 하역 관리자(포럼의 "물류 관리자"는 사실 부둣가이며 하역 관리자에 해당), 수도정예병단, 아카데미 학장, 조향사, 귀금속 세공사, 음유시인, 마법진 도안사, 프리마돈나, 작가, 화가, 검술 도장 사범, 체술사범, 집사(가사 전문 업체 사장), 마을주민.

## 정정

앞서 대부호를 "금전 5만 **또는** 이벤트"라고 말했는데 **그것은 틀렸습니다**. 두 조건은 서로 다른 조건 그룹에 있으므로 일관된 규칙에 따라 **둘 다 필요**합니다. 먼저 Flow_Rich 이벤트 체인을 완료해야 하며(Ep1은 금전 ≥ 20000부터 시작, Ep2는 먼저 `Event_Rich_Invest`가 있어야 하고 3개월 간격, Ep3은 4개월 간격, Ep4는 8개월 간격), 엔딩 시점에 금전도 ≥ 50000이어야 합니다.

## 퀘스트 조건 (Quest 테이블)

- `Quest_RoyalWizard1`: 마법으로 마수 20마리 토벌. `Quest_RoyalWizard2`: 마법으로 무투회 우승.
- `Quest_RAAPresident`: 미술, 음악, 무용 세 과목 모두 A(등급 4) 달성.
- `Quest_Adventurer1`: 어둠의 마법사 토벌. `Quest_Adventurer2`: 사막 약탈자 토벌.
- `Quest_Queen1`: 연회에서 대리섭정 방문. `Quest_Queen2`: 마왕성에서 대리섭정 방문.

## 선택이 필요한 핵심 이벤트 (D_Flow 대화 그래프 기준)

| 이벤트 | 골라야 할 선택지 | 잘못 골랐을 때의 결과 |
|---|---|---|
| Flow_DemonKing_Ep5 (1218년 2월, 단 한 번, MaxCount 1) | "알겠어요. 함께 가요." → `Event_DemonKing_Allow` 설정 | "……아직 모르겠어."를 고르면 문이 다시 열리지 않아 마왕 엔딩이 끊김 |
| Flow_Rich_Ep1 | "투자한다."(5000 골드 소비) → `Event_Rich_Invest` 설정 | "투자하지 않는다"를 고르면 Ep2 조건이 성립하지 않아 대부호 체인이 끊김 |
| Flow_MysteriousThief_Ep1 | "……정말 재미있네. 내가 숨겨 줄게."(선악 -10) | "병사에게 넘긴다"를 고르면 200 골드를 받지만 `Event_MysteriousThief_Prison`이 설정되고, Ep2 조건은 이 플래그가 **없을 것**을 요구하므로 괴도 체인이 끊김 |
| Flow_Adventurer_Ep1 | "수락한다." → `Quest_Adventurer1` 설정 | "거절한다"를 고르면 Ep2에 필요한 퀘스트가 없음 |
| Flow_TheGodfather_Ep1/Ep2 | 혼돈 쪽으로 가야 함. Ep1은 "계속 저들에게 화풀이한다"(-20), Ep2는 "제안을 받아들인다"(-30) | "경비에게 넘긴다"는 각각 +20이며 이벤트 체인에는 영향이 없지만 선악을 질서 쪽으로 밀어서, 뒷골목의제왕은 선악 ≤ -150을 요구하므로 불리해짐 |

## 프린세스 루트: 아가레스(DemonMale) 청혼 조건 (Partner 이벤트 그래프 기준)

호감 등급(`Cond_CompareFavorGrade`): 3 호감(20–29), 4 애정(30–39), 5 특별(40–49), 6 소울메이트(50).

| 이벤트 | 조건 |
|---|---|
| Episode1 | DemonMale_FirstEpisode 완료, Ep1 미완료. 게임 연도 ≤ 1213. 1213년 중에는 바로 발생 가능하고, 그 이전에는 FirstEpisode 이후 랜덤 0–2개월 뒤 |
| Episode2 | DemonMale_InterEpi2 완료, Ep2 미완료. 연도 ≤ 1213, 마찬가지로 1213년 중에는 발생 가능하고 그 이전에는 랜덤 0–2개월 |
| Episode3 | Ep2 완료. **수업 누적 ≥ 2회**. 연도 ≤ 1213. 이후는 위와 동일 |
| Episode4 | Ep3 완료. Karen 나이 ≥ 14 |
| Episode5 | Ep4 완료. 나이 ≥ 15. 호감 등급 ≥ 3(호감) |
| Episode6 | Ep5 완료. 나이 ≥ 16. 호감 등급 ≥ 3 |
| Episode7 | Ep6 완료. 나이 ≥ 17. 호감 등급 ≥ **4(애정)** |
| Episode8 | Ep7 완료(다른 조건 없음) |
| **Proposal(청혼)** | **1218년 3월 월초**(Trigger 1010). Ep8 완료. 그리고 **`Event_DemonKing_Allow` 미설정** |

청혼 이벤트에서 "청혼을 받아들인다"를 골라야 `Act_SetWeddingNPC(DemonMale)`가 실행되고 **`DemonMale_Proposal_Allow`**가 설정됩니다. "청혼을 거절한다"는 어떤 플래그도 남기지 않으며, 청혼 이벤트는 한 번뿐입니다(MaxCount 1). 이벤트에 진입할 때 **이미 결혼 상대가 있으면**(`Cond_HasWeddingPartner`) 대화는 바로 건너뛰고 플래그도 설정되지 않습니다.

따라서:
- **프린세스 엔딩에 필요한 플래그는 1218년 3월에 아가레스의 청혼을 수락해야 얻어집니다.**
- **마왕과 프린세스는 상호 배타적입니다**: 마왕 5화의 "함께 가요"(`Event_DemonKing_Allow`)를 고르면 청혼이 발생하지 않습니다.
- 호감 등급은 Ep5~Ep7에서 3, 3, 4만 요구하며, **데이터상 소울메이트(등급 6)까지는 요구하지 않습니다**. 바하의 "소울메이트까지 육성"은 안전책일 뿐입니다. Ep8과 청혼 자체에는 호감 문턱이 없지만, Ep8의 이벤트 내용은 확인하지 않았으므로 숨은 제한이 있을 수 있습니다.
- 다른 파트너의 청혼 이벤트도 분석을 마쳤습니다(다음 절 참조). 마찬가지로 1218년 3월 월초이며, 다른 사람의 청혼을 먼저 수락했다면 아가레스의 청혼은 "이미 결혼 상대가 있음"으로 건너뜁니다. 유일한 예외는 숲의 일족 여성으로, 그녀의 청혼은 이미 상대가 있는지 확인하지 않습니다.

## 각 파트너의 청혼 이벤트와 스토리 체인 (Partner 이벤트 그래프 기준)

**청혼 이벤트 공통 규칙**(FurryMale/FurryFemale/ForestMale/ForestFemale/DragonMale/DragonFemale/DemonFemale/DemonMale, 총 8명):
- 발생 시기가 완전히 동일합니다. **1218년 3월 월초**(Trigger 1010)이며, **`Event_DemonKing_Allow` 미설정**이어야 하고, **해당 파트너의 마지막 스토리 이벤트가 완료**되어 있어야 합니다(아래 표의 마지막 행 참조).
- 8개의 청혼 이벤트는 Priority가 모두 1이고 MaxCount도 모두 1입니다. 각자 한 번만 청혼하므로 거절하면 기회를 소진합니다.
- 대화 흐름은 같습니다. 이벤트에 진입하면 먼저 `Cond_HasWeddingPartner`를 확인하며, **이미 결혼 상대가 있으면 바로 건너뜁니다**. 없으면 "수락/거절" 두 선택지가 나오고, 수락해야만 `Act_SetWeddingNPC(해당 파트너)`가 실행됩니다.
- **ForestFemale(숲의 일족 여성)의 대화 그래프에는 `Cond_HasWeddingPartner` 확인이 없으므로**, 이미 다른 사람의 청혼을 수락했더라도 그녀의 청혼은 여전히 나타나며, 수락하면 결혼 상대를 **덮어씁니다**(그래프 구조상 그렇게 보이며 실측하지는 않았습니다).
- **아가레스(DemonMale)의 청혼만 플래그 `DemonMale_Proposal_Allow`를 설정하므로** 프린세스 엔딩으로 이어지는 것은 그뿐입니다. 나머지 7명의 청혼을 수락하면 각자의 연인 엔딩만 받습니다.
- 아가레스가 아닌 7명의 파트너는 모두 **청혼에 특정 호감 등급이 필요하지 않습니다**. 호감은 스토리 체인 안에서만 나오며(아래 표 참조), 청혼 자체는 "스토리 체인의 마지막 화가 완료되었는가"만 봅니다.

**같은 달에 여러 파트너가 조건을 충족할 때**: 8개 이벤트의 Trigger는 모두 1010, Priority는 모두 1, MaxCount는 1입니다. `EventManager::sch`의 로직에 따르면 같은 트리거 지점에서는 "가장 작은 양수 우선도" 그룹만 골라 대기열에 넣고 같은 그룹은 전부 넣으므로, 8명의 청혼 이벤트가 함께 줄을 섭니다(3월에는 다른 Priority 1의 1010 이벤트, 예컨대 Flow_Queen_Ep6, Flow_Rich_Ep1~4, Flow_DemonKing_Ep2도 있으며, Priority 0 이벤트가 그 달에 성립하면 먼저 뽑혀 나갑니다). 대기 순서는 Event 테이블의 행 순서를 따르는 것으로 **추정**됩니다. ForestFemale → ForestMale → DragonFemale → DragonMale → FurryMale → FurryFemale → DemonFemale → DemonMale 순입니다(이 부분은 추정이며 `jq::fca`의 정렬은 검증하지 않았습니다). 모든 대화가 먼저 `Cond_HasWeddingPartner`를 확인하고 "수락"해야만 상대가 설정되므로, 보통은 앞순서에 있으면서 수락한 사람이 결혼 상대가 되고 뒤의 청혼은 건너뜁니다(ForestFemale의 대화에는 이 확인이 없으니 주의가 필요합니다). 안전한 방법은 역시 같은 해에 한 명만 육성하는 것입니다.
### 수인족 남성

| 이벤트 | 트리거／장소 | 조건 |
|---|---|---|
| Episode1 | 스케줄 시작／208 | 딸의 나이 ≤ 13 그리고 FurryMale 호감 등급 ≥ 2 |
| Episode2 | 3301／1205 | 대화 FurryMale_Episode1 완료 |
| Episode3 | 스케줄 시작／204 | 대화 FurryMale_Episode2 완료 그리고 FurryMale 호감 등급 ≥ 3 |
| Episode4 | 3304／1204 | 대화 FurryMale_InterEpi 완료 |
| Episode5 | 3301／1205 | 대화 FurryMale_Ready 완료 |
| Episode6 | 월초／0 | 대화 FurryMale_Episode5 완료 그리고 FurryMale_Episode5 완료 당월 월초로부터 경과 ≥ 0년 2월 0일 |
| **청혼** | 월초／— | NOT 이벤트 Event_DemonKing_Allow 완료 그리고 연도 = 1218 그리고 월 = 3 그리고 파트너 이벤트 FurryMale_Episode6 완료 |

### 수인족 여성

| 이벤트 | 트리거／장소 | 조건 |
|---|---|---|
| Episode1 | 스케줄 종료 전／304 | FurryFemale 호감 등급 ≥ 2 |
| Episode2 | 스케줄 종료 전／304 | 대화 FurryFemale_Episode1 완료 |
| Episode3 | 스케줄 종료 전／304 | FurryFemale 호감 등급 ≥ 3 |
| Episode4 | 3301／1205 | 대화 FurryFemale_Episode3 완료 그리고 FurryFemale_Episode3 완료 당월 월초로부터 경과 ≥ 0년 2월 0일 |
| Episode5 | 스케줄 시작／304 | 대화 FurryFemale_Bag 완료 그리고 FurryFemale 호감 등급 ≥ 4 |
| Episode6 | 3301／1205 | 이벤트 FurryFemale_InterEpi 완료 |
| **청혼** | 월초／— | NOT 이벤트 Event_DemonKing_Allow 완료 그리고 연도 = 1218 그리고 월 = 3 그리고 파트너 이벤트 FurryFemale_Episode6 완료 |

### 숲의 일족 남성

| 이벤트 | 트리거／장소 | 조건 |
|---|---|---|
| Episode1 | 스케줄 시작／203 | 이벤트 ForestMale_FirstEpisode 완료 그리고 ForestMale 호감 등급 ≥ 2 |
| Episode2 | 스케줄 종료 전／304 | 대화 ForestMale_Episode1 완료 그리고 ForestMale 호감 등급 ≥ 2 |
| Episode3 | 스케줄 시작／306 | 대화 ForestMale_Episode2 완료 그리고 ForestMale 호감 등급 ≥ 3 |
| Episode4 | 방문(2005)／503 | 대화 ForestMale_Episode3 완료 그리고 딸의 나이 ≥ 14 그리고 ForestMale 호감 등급 ≥ 3 |
| Episode5 | 3202／1303 | 대화 ForestMale_Episode4 완료 그리고 ForestMale 호감 등급 ≥ 4 |
| Episode6 | 3202／1303 | 파트너 이벤트 ForestMale_InterEpi 완료 |
| **청혼** | 월초／— | NOT 이벤트 Event_DemonKing_Allow 완료 그리고 연도 = 1218 그리고 월 = 3 그리고 파트너 이벤트 ForestMale_Episode6 완료 |

### 숲의 일족 여성

| 이벤트 | 트리거／장소 | 조건 |
|---|---|---|
| Episode1 | 스케줄 시작／206 | 딸의 나이 < 14 그리고 대화 ForestFemale_FirstEpisode 완료 그리고 ForestFemale 호감 등급 ≥ 2 |
| Episode2 | 스케줄 시작／206 | 대화 ForestFemale_Episode1 완료 |
| Episode3 | 스케줄 종료 전／308 | 대화 ForestFemale_Episode2 완료 그리고 ForestFemale 호감 등급 ≥ 3 |
| Episode4 | 스케줄 시작／205 | 대화 ForestFemale_Episode3 완료 그리고 딸의 나이 ≥ 14 |
| Episode5 | 스케줄 시작／206 | 대화 ForestFemale_Episode4 완료 그리고 ForestFemale 호감 등급 ≥ 4 |
| Episode6 | 3005／1301 | 대화 ForestFemale_InterEpi 완료 |
| **청혼** | 월초／— | NOT 이벤트 Event_DemonKing_Allow 완료 그리고 연도 = 1218 그리고 월 = 3 그리고 파트너 이벤트 ForestFemale_Episode6 완료 |

### 용족 남성

| 이벤트 | 트리거／장소 | 조건 |
|---|---|---|
| Episode1 | 3103／1105 | 대화 DragonMale_FirstEpisode 완료 그리고 DragonMale 호감 등급 ≥ 2 |
| Episode2 | 3101／1103 | 대화 DragonMale_Episode1 완료 |
| Episode3 | 스케줄 시작／302 | 대화 DragonMale_Episode2 완료 그리고 DragonMale_Episode2 완료 당월 월초로부터 경과 > 0년 3월 0일 |
| Episode4 | 3103／1105 | 대화 DragonMale_Episode3 완료 그리고 딸의 나이 ≥ 14 |
| Episode5 | 스케줄 시작／213 | 대화 DragonMale_Episode4 완료 그리고 DragonMale 호감 등급 ≥ 4 |
| Episode6 | 3101／1103 | 대화 DragonMale_InterEpi 완료 |
| **청혼** | 월초／— | NOT 이벤트 Event_DemonKing_Allow 완료 그리고 연도 = 1218 그리고 월 = 3 그리고 파트너 이벤트 DragonMale_Episode6 완료 |

### 용족 여성

| 이벤트 | 트리거／장소 | 조건 |
|---|---|---|
| Episode1 | 스케줄 시작／305 | 대화 DragonFemale_FirstEpisode 완료 그리고 DragonFemale 호감 등급 ≥ 2 |
| Episode2 | 스케줄 시작／305 | 대화 DragonFemale_Episode1 완료 |
| Episode3 | 스케줄 시작／307 | 대화 DragonFemale_Episode2 완료 그리고 Episode2 완료 다음 달 월초부터 마법학과 예법 수업 횟수 합계 ≥ 2 (날짜 오프셋 파라미터는 1년 0월 0일이며, 정확한 구간은 확인되지 않음) |
| Episode4 | 3103／1105 | 대화 DragonFemale_Episode3 완료 |
| Episode5 | 스케줄 시작／305 | 대화 DragonFemale_Episode4 완료 그리고 DragonFemale 호감 등급 ≥ 4 |
| Episode6 | 3101／1103 | 대화 DragonFemale_InterEpi 완료 |
| **청혼** | 월초／— | NOT 이벤트 Event_DemonKing_Allow 완료 그리고 연도 = 1218 그리고 월 = 3 그리고 파트너 이벤트 DragonFemale_Episode6 완료 |

### 마족 여성(DemonFemale)

| 이벤트 | 트리거／장소 | 조건 |
|---|---|---|
| Episode1 | 스케줄 시작／303 | 대화 DemonFemale_FirstEpisode 완료 그리고 DemonFemale 호감 등급 ≥ 2 |
| Episode2 | 스케줄 시작／212 | 메이드카페 누적 횟수 ≥ 1 그리고 대화 DemonFemale_Episode1 완료 그리고 이벤트 DemonFemale_Encounter_Dance 완료 |
| Episode3 | 왕성 인물 조우／710 | 대화 DemonFemale_Episode2 완료 그리고 DemonFemale 호감 등급 ≥ 3 |
| Episode4 | 스케줄 시작／309 | 대화 DemonFemale_Episode3 완료 그리고 DemonFemale 호감 등급 ≥ 3 |
| Episode5 | 왕성 인물 조우／710 | 대화 DemonFemale_Episode4 완료 그리고 DemonFemale 호감 등급 ≥ 4 그리고 이벤트 DemonFemale_DemonEvent 완료 |
| Episode6 | 왕성 인물 조우／710 | 대화 DemonFemale_InterEpi 완료 |
| **청혼** | 월초／— | NOT 이벤트 Event_DemonKing_Allow 완료 그리고 연도 = 1218 그리고 월 = 3 그리고 파트너 이벤트 DemonFemale_Episode6 완료 |

### 큐브와 기타 NPC(Cube, Caim, Icosa, Kimaris, Muzai)

이 5명에게는 청혼 이벤트도, "수락" 대화 그래프도 없습니다. 그래프에는 스토리(`Episode1`, 큐브는 1, 2)와 결혼식 대화만 있으며, 결혼 상대가 되는 것은 엔딩 시 `pk::gde`가 "어떤 이벤트를 완료했는가"와 "직업 엔딩"으로 결정합니다(위의 "엔딩 시점의 최종 판정" 참조). 이들의 스토리 이벤트 조건은 다음과 같습니다.

| 이벤트 | 트리거 | 조건 |
|---|---|---|
| Cube_Episode1 | 1002(장소 201 집) | 큐브와의 관계 ≥ 70 |
| Cube_Episode2 | 1002(장소 201 집) | 이벤트 Cube_Episode1 완료 그리고 큐브와의 관계 ≥ 120 |
| Caim_Episode1(카임, 식당 점장) | 1017 스케줄 후반, 우선도 30 | 이벤트 Job_Restaurant_Ep3 완료 그리고 이번 달 스케줄이 식당 |
| Icosa_Episode1(아이코사, 마법 아카데미) | 1017, 우선도 30 | 이벤트 Edu_Magic_Ep3 완료 그리고 이번 달 스케줄이 마법학 |
| Kimaris_Episode1(키마리스, 검문소) | 1017, 우선도 30 | 이벤트 Job_Checkpoint_Ep3 완료 그리고 이번 달 스케줄이 검문소 |
| Muzai_Episode1(무자이, 문학 교사) | 1017, 우선도 30 | 이벤트 Edu_Literature_Ep3 완료 그리고 이번 달 스케줄이 문학 |

네 명의 사장／교사의 Episode1은 모두 한 번만 발생하며(MaxCount 1), 반드시 **그 스케줄을 진행하는 그 달**에 나타납니다. 따라서 먼저 해당 `_Ep3`를 끝낸 뒤, 이후 같은 스케줄을 다시 한 번 짜야 합니다. 큐브의 "큐브와의 관계"는 Status 21(범위 -100~500)입니다. 전체 데이터베이스와 모든 플로우 그래프를 통틀어 이 값을 올리는 것은 `E_Job_House_Ep1/2/3`(집안일 아르바이트의 3단계 이벤트)뿐이며, 각 단계에서 어떤 선택지를 골라도 +1이라 합계가 3밖에 되지 않아 70／120에 도달할 수 없습니다. 따라서 주된 증가 경로는 데이터나 그래프에 없고 C# 코드(예: 집안일 계열 스케줄을 실행할 때마다 정산)에 있는 것으로 보입니다. 코드 측에서 확인된 내용은 다음과 같습니다. `hl::sim(EStatus,int)`는 범용 상태 증감 함수이며(상·하한은 Status 테이블의 Min／Max를 따르고, Cube는 -100~500) 큐브에 대한 특수 처리는 없습니다. `hl::dfs`는 "지정한 값으로 설정"하는 함수이고 호출자는 대부분 디버그 또는 인터페이스입니다. 데이터 측에서는 `Schedule` 테이블의 `Job_House`가 `NeedStatCritical` 열에서 StatType 21(Cube, 범위 10~80)을 사용하는데, 이는 "대성공 판정에 쓰이는 능력치"로 추정되며 호감도를 올리는 경로가 아닙니다. 그 밖의 테이블에서 코드 21로 Cube를 참조하는 곳은 없습니다. **경로를 찾았습니다(코드로 확인)**: `ue::cez::MoveNext`(스케줄 실행 흐름)가 스케줄 시작 시 그 스케줄의 장소가 201(0xC9)이면 `hl::sim(RelationshipCube, 종료일 - 시작일 + 1)`을 실행합니다. 데이터 테이블에서 장소가 201인 스케줄은 `Job_House`(집안일 아르바이트)뿐이므로, **집안일 아르바이트를 한 번 할 때마다 큐브와의 관계가 "그 스케줄의 일수"만큼 증가**하며, 스케줄 시작 시점에 정산되어 성공, 실패, 대성공과 무관합니다. (한 스케줄의 일수는 `mbg::sfv`가 정합니다. 한 달에 스케줄 칸이 3개이고 상순, 중순이 각 10일, 하순은 "그 달의 일수 − 20"일이라서 30일인 달은 10일, 31일인 달은 11일, 2월은 8~9일입니다. 따라서 집안일 아르바이트 한 번은 보통 +10이며, 누적 70에는 7회, 누적 120에는 12회가 필요합니다. 세 단계 이벤트가 각 +1로 합계 +3뿐이라 줄여 주는 횟수는 한 번도 없습니다. 즉 매달 세 칸 모두 집안일 아르바이트를 넣으면 약 3개월 남짓(7칸)에 70, 약 4개월(12칸)에 120에 도달합니다.) 또한 `E_Job_House_Ep1/2/3` 세 단계 이벤트가 각 단계마다 +1을 더합니다. 다른 경로는 없습니다. `ue::ulx`(스케줄 정산은 Stats 열만 적용하며 Job_House는 3／11／6이고 대성공 시 두 배), 선물(`Partner_Npc_Cube`의 FavorStatus는 0), 아이템／차／식사, 월 정산 `uk::cdv`(체력, 정신, 피로, 금전만 변동)는 모두 이 값을 올리지 않습니다.

**"일반 결혼"과 "독신"**: `D_Village_Wedding`, `D_Job_Wedding`, `D_None_Wedding`은 각각 -1, -3, -2에 대응합니다. 이 값들을 설정하는 이벤트는 없으며, 엔딩 시 `pk::gde`가 위의 6번부터 8번 항목에 따라 값을 부여합니다.

## 아직 부족한 부분

1. `Event_DemonKing_Allow`, `DemonMale_Proposal_Allow` 두 플래그는 대화 선택지가 설정하는 것으로 위에서 확인했습니다.
2. `Score` 열은 점술가 진행도의 가중치이고, `Weight`는 최종 판정의 우선순위입니다. 코드로 확인했습니다.
3. 수업／아르바이트 누적값 `lcq`는 `NeedCount_GradeUp`과 비교되는 누적 횟수로 확인했습니다. `yaw`와 `lcr` 두 필드의 의미(`yaw`는 Type 9 수업 횟수에 사용)는 자세히 조사하지 않았습니다.
4. 같은 달에 여러 파트너가 동시에 청혼 조건을 충족했을 때의 순서는 확인하지 못했습니다. "큐브와의 관계"의 출처와 일수는 모두 확인했습니다(집안일 아르바이트 1회당 +10, 하순은 그 달의 일수에 따라 +8~11). 결혼 상대 기본값 -3은 `do::do()` 생성자가 기록하는 것으로 확인했습니다. 그 밖에 `ei::hbo`("청혼 관련 데이터가 초기화되었습니다"라는 한국어 안내를 표시하므로 디버그 또는 리셋 도구로 보임)와 `du::cga`／`du::lzk`(엔딩 조건을 세이브에 직접 적용하는 디버그 기능)도 이 값을 기록하지만, 정상 플레이에서는 거치지 않습니다.
5. Flow_Vacance(바캉스 이벤트)는 분석하지 않았습니다.
6. 왕성 인물 조우 이벤트의 Place(703~708)에 대응하는 장면 이름은 코드 안에 있습니다.

---

# 엔딩별 전체 조건 (우선순위가 높은 순서)

## 여왕(Queen)  우선순위 Weight 52000
**엔딩 조건(모두 충족해야 함)**
- 사교평가 ≥ 500
- 다음 중 하나: 전사평가 ≥ 400 또는 마법평가 ≥ 400
- 체력＋힘＋민첩 ≥ 1500
- 지능＋마력＋정신력 ≥ 1500
- 기품＋예절＋화술 ≥ 2400
- 감수성＋센스＋매력 ≥ 1500
- 성향 ≥ 225

**선행 이벤트 체인**
- `Flow_Queen_Ep1`: Castle_RegentDeputy와 만난 횟수 ≥ 3; 화술 ≥ 200
- `Flow_Queen_Ep2`: 이벤트 Flow_Queen_Ep1 완료; Flow_Queen_Ep1 완료 당월 월초로부터 경과 ≥ 0년 2월 0일
- `Flow_Queen_Ep3`: 이벤트 Flow_Queen_Ep2 완료; 전사 3항목(체력+힘+민첩) 합계 ≥ 840; 전사평가 ≥ 245
- `Flow_Queen_Ep3`: 이벤트 Flow_Queen_Ep2 완료; 전사 3항목(체력+힘+민첩) 합계 ≥ 840; 【불가】전사평가 ≥ 245; 마법 3항목(지능+마력+정신) 합계 ≥ 840; 전사평가 ≥ 245
- `Flow_Queen_Ep3`: 이벤트 Flow_Queen_Ep2 완료; 【불가】전사 3항목(체력+힘+민첩) 합계 ≥ 840; 마법 3항목(지능+마력+정신) 합계 ≥ 840; 전사평가 ≥ 245
- `Flow_Queen_Ep4`: 이벤트 Flow_Queen_Ep3 완료; Flow_Queen_Ep3 완료 당월 월초로부터 경과 ≥ 0년 2월 0일
- `Flow_Queen_Ep5`: 이벤트 Flow_Queen_Ep4 완료
- `Flow_Queen_Ep6`: 이벤트 Flow_Queen_Ep5 완료; 전사평가 ≥ 400; 사교평가 ≥ 500; 전사 3항목(체력+힘+민첩) 합계 ≥ 1500; 마법 3항목(지능+마력+정신) 합계 ≥ 1500; 사교 3항목(기품+예절+화술) 합계 ≥ 2400; 예술 3항목(감수성+센스+매력) 합계 ≥ 1500; 성향 ≥ 225
- `Flow_Queen_Ep6`: 이벤트 Flow_Queen_Ep5 완료; 【불가】전사평가 ≥ 400; 마법평가 ≥ 400; 사교평가 ≥ 500; 전사 3항목(체력+힘+민첩) 합계 ≥ 1500; 마법 3항목(지능+마력+정신) 합계 ≥ 1500; 사교 3항목(기품+예절+화술) 합계 ≥ 2400; 예술 3항목(감수성+센스+매력) 합계 ≥ 1500; 성향 ≥ 225
- `Flow_Queen_Ep7`: 이벤트 Flow_Queen_Ep6 완료

## 마왕(DemonKing)  우선순위 Weight 51000
**엔딩 조건(모두 충족해야 함)**
- 이벤트 Event_DemonKing_Allow 완료
- 성향 ≤ -225
- 체력＋힘＋민첩 ≥ 2400
- 지능＋마력＋정신력 ≥ 2400

**선행 이벤트 체인**
- `Flow_DemonKing_Ep1`: 성향 ≤ -150; 전사 3항목(체력+힘+민첩) 합계 ≥ 800; 마법 3항목(지능+마력+정신) 합계 ≥ 800
- `Flow_DemonKing_Ep2`: 이벤트 Flow_DemonKing_Ep1 완료; 성향 ≤ -150; 전사 3항목(체력+힘+민첩) 합계 ≥ 1200; 마법 3항목(지능+마력+정신) 합계 ≥ 1200
- `Flow_DemonKing_Ep3`: 이벤트 Flow_DemonKing_Ep2 완료
- `Flow_DemonKing_Ep4`: 이벤트 Flow_DemonKing_Ep3 완료; 성향 ≤ -150; 전사 3항목(체력+힘+민첩) 합계 ≥ 2000; 마법 3항목(지능+마력+정신) 합계 ≥ 2000
- `Flow_DemonKing_Ep5`: 이벤트 Flow_DemonKing_Ep4 완료; 성향 ≤ -225; 전사 3항목(체력+힘+민첩) 합계 ≥ 2400; 마법 3항목(지능+마력+정신) 합계 ≥ 2400; 연도 = 1218; 월 = 2

## 프린세스(Princess)  우선순위 Weight 50000
**엔딩 조건(모두 충족해야 함)**
- 이벤트 DemonMale_Proposal_Allow 완료
- 사교평가 ≥ 400
- 예술평가 ≥ 400
- 기품＋예절＋화술 ≥ 2400
- 감수성＋센스＋매력 ≥ 2400
- 성향 ≥ -149

**선행 이벤트 체인**
- `Flow_Princess_Ep1`: Castle_Consort와 만난 횟수 ≥ 3; 그래프 변수 KarenRank = 1; 대화 DemonMale_Episode3 완료
- `Flow_Princess_Ep2`: 대화 Flow_Princess_Ep1 완료; 매력 ≥ 400
- `Flow_Princess_Ep3`: 대화 Flow_Princess_Ep2 완료; 사교 3항목(기품+예절+화술) 합계 ≥ 2000
- `Flow_Princess_Ep4`: 대화 Flow_Princess_Ep3 완료
- `Flow_Princess_Ep5`: 대화 Flow_Princess_Ep4 완료
- `Flow_Princess_Ep6`: 대화 Flow_Princess_Ep5 완료; 현재 엔딩 조건 Ending_Princess 충족
- `Flow_Princess_Ep7`: 대화 Flow_Princess_Ep6 완료

## 대장군(General)  우선순위 Weight 43000
**엔딩 조건(모두 충족해야 함)**
- 무투회 우승
- 검술 S 달성(등급 5)
- 체술 S 달성(등급 5)
- 전사평가 ≥ 350
- 체력＋힘＋민첩 ≥ 2400
- 기품＋예절 ≥ 600
- 이벤트 Flow_General_Ep3 완료
- 성향 ≥ -149

**선행 이벤트 체인**
- `Flow_General_Ep0`: Castle_RegentDeputy와 만난 횟수 > 0; Festival_Battle 기록이 [Rank=1&Weapon=Physical]에 해당; 이벤트 Flow_CapitalEliteTroop_Ep2 완료; 성향 ≥ -149
- `Flow_General_Ep1`: 이벤트 Flow_General_Ep0 완료
- `Flow_General_Ep2`: 이벤트 Flow_General_Ep1 완료; 전사평가 등급 ≥ 3; 체술 등급 ≥ 5; 검술 등급 ≥ 5; 성향 ≥ -149
- `Flow_General_Ep3`: 이벤트 Flow_General_Ep2 완료

## 왕궁마법사(RoyalWizard)  우선순위 Weight 42000
**엔딩 조건(모두 충족해야 함)**
- 연금술 S 달성(등급 5)
- 마법학 S 달성(등급 5)
- 이벤트 Flow_RoyalWizard_Ep3 완료
- 마법평가 ≥ 350
- 지능＋마력＋정신력 ≥ 2400
- 성향 ≥ -149

**선행 이벤트 체인**
- `Flow_RoyalWizard_Ep1`: Castle_Archmage와 만난 횟수 ≥ 1; 마법평가 등급 > 4; 연금술 등급 ≥ 3; 마법학 등급 ≥ 4
- `Flow_RoyalWizard_Ep2`: 이벤트 Flow_RoyalWizard_Ep1 완료; 퀘스트 조건 Quest_RoyalWizard1 달성
- `Flow_RoyalWizard_Ep3`: 이벤트 Flow_RoyalWizard_Ep2 완료; 퀘스트 조건 Quest_RoyalWizard2 달성

## 왕립예술학회장(RAAPresident)  우선순위 Weight 41000
**엔딩 조건(모두 충족해야 함)**
- 미술 A 달성(등급 4)
- 무용 A 달성(등급 4)
- 음악 A 달성(등급 4)
- 예술평가 ≥ 350
- 감수성＋센스＋매력 ≥ 2400
- 예술제 우승
- 성향 ≥ -149

**선행 이벤트 체인**
- `Flow_RAAPresident_Ep1`: Castle_Consort와 만난 횟수 > 2; 【불가】퀘스트 Quest_RAAPresident 완료; Festival_Art 기록이 [Rank=1]에 해당
- `Flow_RAAPresident_Ep2`: 퀘스트 Quest_RAAPresident 완료

## 대부호(Rich)  우선순위 Weight 40000
**엔딩 조건(모두 충족해야 함)**
- 소지금 ≥ 50000
- 이벤트 Flow_Rich_Ep4 완료

**선행 이벤트 체인**
- `Flow_Rich_Ep1`: 소지금 ≥ 20000
- `Flow_Rich_Ep2`: 이벤트 Event_Rich_Invest 완료; Flow_Rich_Ep1 완료 당월 월초로부터 경과 ≥ 0년 3월 0일
- `Flow_Rich_Ep3`: 이벤트 Flow_Rich_Ep2 완료; Flow_Rich_Ep2 완료 당월 월초로부터 경과 ≥ 0년 4월 0일
- `Flow_Rich_Ep4`: 이벤트 Flow_Rich_Ep3 완료; Flow_Rich_Ep3 완료 당월 월초로부터 경과 ≥ 0년 8월 0일

## 마법검사(MagicKnight)  우선순위 Weight 40000
**엔딩 조건(모두 충족해야 함)**
- 전사평가 등급 ≥ 4(평가 ≥ 250)
- 마법평가 등급 ≥ 4(평가 ≥ 250)
- 이벤트 Flow_MagicKnight_Ep1 완료
- 마력 ≥ 600
- 힘 ≥ 600

**선행 이벤트 체인**
- `Flow_MagicKnight_Ep1`: 전사평가 ≥ 2; 마법학 등급 ≥ 3; [Rank!=1&Weapon=Physical]에 해당
- `Flow_MagicKnight_Ep2`: 이벤트 Flow_MagicKnight_Ep1 완료; [Rank=1]에 해당

## 외교관(Diplomat)  우선순위 Weight 40000
**엔딩 조건(모두 충족해야 함)**
- 이벤트 Flow_Diplomat_Ep3 완료
- 사교평가 ≥ 350
- 기품＋예절＋화술 ≥ 1800
- 지능 ≥ 500
- 성향 ≥ -149

**선행 이벤트 체인**
- `Flow_Diplomat_Ep1`: 사교 3항목(기품+예절+화술) 합계 ≥ 1200; 사교평가 등급 ≥ 3; 지능 ≥ 100; 성향 ≥ -149
- `Flow_Diplomat_Ep2`: 이벤트 Flow_Diplomat_Ep1 완료
- `Flow_Diplomat_Ep3`: 이벤트 Flow_Diplomat_Ep2 완료; 사교 3항목(기품+예절+화술) 합계 ≥ 1500; 지능 ≥ 300

## 살롱 로비스트(Sting)  우선순위 Weight 34000
**엔딩 조건(모두 충족해야 함)**
- 성향 ≤ -150
- 사교평가 ≥ 350
- 기품＋예절＋화술 ≥ 1600
- 지능 ≥ 500

**선행 이벤트 체인**
- `Flow_Sting_Ep1`: 사교평가 등급 ≥ 3
- `Flow_Sting_Ep2`: 이벤트 Flow_Sting_Ep1 완료; 성향 ≤ -150
- `Flow_Sting_Ep3`: 이벤트 Flow_Sting_Ep2 완료; 지능 > 400; 성향 ≤ -150

## 팜므파탈(Femme)  우선순위 Weight 33000
**엔딩 조건(모두 충족해야 함)**
- 성향 ≤ -150
- 주점 S 달성(등급 5)
- 매력 ≥ 600
- 매력 ≥ 400
- 매력 ≥ 600

**선행 이벤트 체인**
- `Flow_Femme_Ep1`: 주점 누적 횟수 ≥ 10; 성향 ≤ -90; 매력 > 100
- `Flow_Femme_Ep2`: 이벤트 Flow_Femme_Ep1 완료; 주점 등급 ≥ 4; 매력 > 300; 성향 ≤ -120
- `Flow_Femme_Ep3`: 이벤트 Flow_Femme_Ep2 완료; 주점 누적 횟수 ≥ 25; 매력 > 500; 성향 ≤ -150

## 혼돈의 연금술사(ChaosAlchemist)  우선순위 Weight 32000
**엔딩 조건(모두 충족해야 함)**
- 성향 ≤ -150
- 연금술 A 달성(등급 4)
- 지능＋마력＋정신력 ≥ 1500
- 센스 ≥ 300

**선행 이벤트 체인**
- `Flow_ChaosAlchemist_Ep1`: 연금술 등급 ≥ 3
- `Flow_ChaosAlchemist_Ep2`: 이벤트 Flow_ChaosAlchemist_Ep1 완료; 연금술 등급 ≥ 4; 성향 ≤ -10
- `Flow_ChaosAlchemist_Ep3`: 이벤트 Flow_ChaosAlchemist_Ep2 완료; 성향 ≤ -90

## 괴도(MysteriousThief)  우선순위 Weight 32000
**엔딩 조건(모두 충족해야 함)**
- 성향 ≤ -150
- 지능 ≥ 300
- 민첩 ≥ 600
- 센스 ≥ 600
- 이벤트 Flow_MysteriousThief_Ep3 완료

**선행 이벤트 체인**
- `Flow_MysteriousThief_Ep1`: 민첩 ≥ 400
- `Flow_MysteriousThief_Ep2`: 【불가】이벤트 Event_MysteriousThief_Prison 완료; 이벤트 Flow_MysteriousThief_Ep1 완료; 센스 ≥ 400; 성향 ≤ -150
- `Flow_MysteriousThief_Ep3`: 이벤트 Flow_MysteriousThief_Ep2 완료; 지능 ≥ 200; 성향 ≤ -150

## 점술가(FortuneTeller)  우선순위 Weight 31000
**엔딩 조건(모두 충족해야 함)**
- 마법평가 등급 ≥ 4(평가 ≥ 250)
- 성향 ≤ -150
- 정신력 ≥ 400
- 화술 ≥ 400
- 감수성 ≥ 400

**선행 이벤트 체인**
- `Flow_FortuneTeller_Ep1`: 월 ≠ 4; 화술 > 200; 감수성 > 200; 정신력 > 200; 성향 ≤ -150
- `Flow_FortuneTeller_Ep2`: 월 ≠ 4; 화술 > 300; 정신력 > 300; 감수성 > 300; 성향 ≤ -150

## 뒷골목의제왕(TheGodfather)  우선순위 Weight 31000
**엔딩 조건(모두 충족해야 함)**
- 성향 ≤ -150
- 힘 ≥ 800
- 체력 ≥ 400
- 전사평가 ≥ 350

**선행 이벤트 체인**
- `Flow_TheGodfather_Ep1`: 이벤트 Flow_UnloadingManager_Ep1 완료; 전사평가 등급 ≥ 2
- `Flow_TheGodfather_Ep2`: 이벤트 Flow_TheGodfather_Ep1 완료; 【불가】이벤트 Flow_TheGodfather2_Ep2 완료; 전사평가 등급 ≥ 3

## 마도 공학자(MagicEngineer)  우선순위 Weight 27000
**엔딩 조건(모두 충족해야 함)**
- 마법학 A 달성(등급 4)
- 연금술 A 달성(등급 4)
- 지능＋마력＋정신력 ≥ 2000

**선행 이벤트 체인**
- `Flow_MagicEngineer_Ep1`: 이번 달 스케줄이 연금술; 마법학 누적 횟수 ≥ 13; 연금술 누적 횟수 ≥ 13; 마법 3항목(지능+마력+정신) 합계 ≥ 1500
- `Flow_MagicEngineer_Ep2`: 이번 달 스케줄이 연금술; 마법학 누적 횟수 ≥ 18; 연금술 누적 횟수 ≥ 18; 마법 3항목(지능+마력+정신) 합계 ≥ 1800

## 검의 무희(Sworddancer)  우선순위 Weight 27000
**엔딩 조건(모두 충족해야 함)**
- 검술 A 달성(등급 4)
- 무용 B 달성(등급 3)
- 매력 ≥ 500

**선행 이벤트 체인**
- `Flow_Sworddancer_Ep1`: 이번 달 스케줄이 검술; 검술 누적 횟수 ≥ 12; 무용 누적 횟수 ≥ 8; 민첩 ≥ 150; 이벤트 StatCheck_Sword_Ep1 완료
- `Flow_Sworddancer_Ep2`: 이번 달 스케줄이 무용; 검술 누적 횟수 ≥ 16; 무용 누적 횟수 ≥ 10; 매력 ≥ 450; 이벤트 Flow_Sworddancer_Ep1 완료

## 조향사(Perfumer)  우선순위 Weight 27000
**엔딩 조건(모두 충족해야 함)**
- 식물원 A 달성(등급 4)
- 연금술 B 달성(등급 3)
- 기품 ≥ 500

**선행 이벤트 체인**
- `Flow_Perfumer_Ep1`: 이번 달 스케줄이 식물원; 식물원 누적 횟수 ≥ 15; 연금술 누적 횟수 ≥ 8; 감수성 ≥ 150; 이벤트 StatCheck_Garden_Ep1 완료
- `Flow_Perfumer_Ep2`: 이번 달 스케줄이 연금술; 식물원 누적 횟수 ≥ 25; 연금술 누적 횟수 ≥ 10; 기품 ≥ 450; 이벤트 Flow_Perfumer_Ep1 완료

## 귀금속 세공사(Jeweler)  우선순위 Weight 27000
**엔딩 조건(모두 충족해야 함)**
- 대장간 A 달성(등급 4)
- 미술 B 달성(등급 3)
- 예술평가 ≥ 100
- 기품 ≥ 500

**선행 이벤트 체인**
- `Flow_Jeweler_Ep1`: 이번 달 스케줄이 대장간; 대장간 누적 횟수 ≥ 12; 미술 누적 횟수 ≥ 8; 체력 ≥ 150
- `Flow_Jeweler_Ep2`: 이번 달 스케줄이 대장간; 대장간 누적 횟수 ≥ 16; 미술 누적 횟수 ≥ 10; 예술평가 ≥ 100; 기품 ≥ 450; 이벤트 Flow_Jeweler_Ep1 완료

## 음유시인(Bard)  우선순위 Weight 27000
**엔딩 조건(모두 충족해야 함)**
- 음악 A 달성(등급 4)
- 문학 B 달성(등급 3)
- 이벤트 Flow_Bard_Ep3 완료
- 화술 ≥ 500

**선행 이벤트 체인**
- `Flow_Bard_Ep1`: 이번 달 스케줄이 문학; 문학 누적 횟수 ≥ 8; 음악 누적 횟수 ≥ 10; 감수성 ≥ 150
- `Flow_Bard_Ep2`: 이번 달 스케줄이 음악; 음악 누적 횟수 ≥ 16; 문학 누적 횟수 ≥ 10; 화술 ≥ 450; 이벤트 Flow_Bard_Ep1 완료
- `Flow_Bard_Ep3`: 화술 ≥ 450; 이벤트 Flow_Bard_Ep2 완료

## 마법진 도안사(SigilDrafter)  우선순위 Weight 27000
**엔딩 조건(모두 충족해야 함)**
- 마법학 A 달성(등급 4)
- 미술 B 달성(등급 3)
- 예술평가 ≥ 100
- 감수성 ≥ 500

**선행 이벤트 체인**
- `Flow_SigilDrafter_Ep1`: 이번 달 스케줄이 마법학; 마법학 누적 횟수 ≥ 12; 미술 누적 횟수 ≥ 8; 마력 ≥ 150
- `Flow_SigilDrafter_Ep2`: 이번 달 스케줄이 미술; 마법학 누적 횟수 ≥ 16; 미술 누적 횟수 ≥ 10; 예술평가 ≥ 100; 감수성 ≥ 450; 이벤트 Flow_SigilDrafter_Ep1 완료

## 프리마 발레리나(Dancer)  우선순위 Weight 23000
**엔딩 조건(모두 충족해야 함)**
- 무용 A 달성(등급 4)
- 매력 ≥ 300
- 센스 ≥ 300
- 기품 ≥ 300
- 예술제 무용 대회 우승

**선행 이벤트 체인**
- `Flow_Dancer_Ep1`: 이번 달 스케줄이 무용; 무용 누적 횟수 ≥ 22; 기품 ≥ 100
- `Flow_Dancer_Ep2`: 이번 달 스케줄이 무용; 무용 누적 횟수 ≥ 27; 기품 ≥ 200

## 프리마돈나(PrimaDonna)  우선순위 Weight 23000
**엔딩 조건(모두 충족해야 함)**
- 음악 A 달성(등급 4)
- 감수성 ≥ 300
- 매력 ≥ 300
- 화술 ≥ 300
- 예술제 음악 대회 우승

**선행 이벤트 체인**
- `Flow_PrimaDonna_Ep1`: 이번 달 스케줄이 음악; 음악 누적 횟수 ≥ 22; 화술 ≥ 100
- `Flow_PrimaDonna_Ep2`: 이번 달 스케줄이 음악; 음악 누적 횟수 ≥ 27; 화술 ≥ 200
## 아카데미 학장 (AcademyDean)  우선순위 Weight 23000
**엔딩 조건(모두 충족해야 함)**
- 아카데미 A 달성(등급 4)
- 지능 ≥ 500
- 화술 ≥ 300

**선행 이벤트 체인**
- `Flow_AcademyDean_Ep1`: 이번 달 일정이 아카데미; 아카데미 누적 횟수 ≥ 23; 지능 ≥ 200; 센스 ≥ 150
- `Flow_AcademyDean_Ep2`: 이번 달 일정이 아카데미; 아카데미 누적 횟수 ≥ 28; 지능 ≥ 300; 이벤트 완료 Flow_AcademyDean_Ep1
- `Flow_AcademyDean_Ep3`: 이번 달 일정이 아카데미; 지능 ≥ 450; 센스 ≥ 200; 화술 ≥ 270; 이벤트 완료 Flow_AcademyDean_Ep2

## 요리사 (Cook)  우선순위 Weight 22500
**엔딩 조건(모두 충족해야 함)**
- 식당 S 달성(등급 5)
- 요리 축제 우승
- 센스 ≥ 500
- 마력 ≥ 200

**선행 이벤트 체인**
- `Flow_Cook_Ep1`: 이번 달 일정이 식당; 식당 누적 횟수 ≥ 38
- `Flow_Cook_Ep2`: 이번 달 일정이 식당; 식당 누적 횟수 ≥ 45; Festival_Cook 우승 횟수 ≥ 1

## 마법 사무원 (MagicClerk)  우선순위 Weight 20900
**엔딩 조건(모두 충족해야 함)**
- 마법평가 등급 ≥ 4(평가 ≥ 250)
- 마법학 A 달성(등급 4)
- 지능+마력+정신력 ≥ 1500
- 이벤트 완료 Flow_MagicClerk_Ep2

**선행 이벤트 체인**
- `Flow_MagicClerk_Ep1`: 이번 달 일정이 마법학; 마법학 누적 횟수 ≥ 10; Castle_MagicClerk 방문 횟수 ≥ 3
- `Flow_MagicClerk_Ep2`: 이벤트 완료 Flow_MagicClerk_Ep1; 마법학 누적 횟수 ≥ 10; Castle_MagicClerk 방문 횟수 ≥ 3

## 수도정예병단 (CapitalEliteTroop)  우선순위 Weight 20850
**엔딩 조건(모두 충족해야 함)**
- 전사평가 등급 ≥ 4(평가 ≥ 250)
- 성향 ≥ 30
- 체력+힘+민첩 ≥ 1500

**선행 이벤트 체인**
- `Flow_CapitalEliteTroop_Ep1`: 이번 달 일정이 자유활동; 검술 등급 ≥ 3
- `Flow_CapitalEliteTroop_Ep2`: 이벤트 완료 Flow_CapitalEliteTroop_Ep1; 힘 > 400

## 모험가 (Adventurer)  우선순위 Weight 20700
**엔딩 조건(모두 충족해야 함)**
- 전사평가 등급 ≥ 4(평가 ≥ 250)
- 체력 ≥ 400
- 힘 ≥ 400
- 용족 지역 탐험 경험 있음
- 숲의 일족 지역 탐험 경험 있음
- 수인족 지역 탐험 경험 있음

**선행 이벤트 체인**
- `Flow_Adventurer_Ep1`: 몬스터 처치 수(전체) > 10
- `Flow_Adventurer_Ep2`: 이벤트 완료 Flow_Adventurer_Ep1; 퀘스트 조건 달성 Quest_Adventurer1
- `Flow_Adventurer_Ep3`: 이벤트 완료 Flow_Adventurer_Ep2; 전사평가 > 3
- `Flow_Adventurer_Ep4`: 이벤트 완료 Flow_Adventurer_Ep3; 퀘스트 조건 달성 Quest_Adventurer2

## 검술 도장 사범 (SwordInstructor)  우선순위 Weight 20495
**엔딩 조건(모두 충족해야 함)**
- 검술 B 달성(등급 3)
- 민첩 ≥ 300
- 체력 ≥ 300

**선행 이벤트 체인**
- `Flow_SwordInstructor_Ep1`: 이번 달 일정이 검술; 검술 누적 횟수 ≥ 12
- `Flow_SwordInstructor_Ep2`: 이번 달 일정이 검술; 검술 누적 횟수 ≥ 20; 민첩 ≥ 270; 체력 ≥ 270

## 작가 (Writer)  우선순위 Weight 20495
**엔딩 조건(모두 충족해야 함)**
- 문학 B 달성(등급 3)
- 화술 ≥ 300
- 감수성 ≥ 300

**선행 이벤트 체인**
- `Flow_Writer_Ep1`: 이번 달 일정이 문학; 문학 누적 횟수 ≥ 12
- `Flow_Writer_Ep2`: 이번 달 일정이 문학; 문학 누적 횟수 ≥ 20; 화술 ≥ 270; 감수성 ≥ 270

## 화가 (Painter)  우선순위 Weight 20495
**엔딩 조건(모두 충족해야 함)**
- 미술 B 달성(등급 3)
- 센스 ≥ 300
- 감수성 ≥ 300

**선행 이벤트 체인**
- `Flow_Painter_Ep1`: 이번 달 일정이 미술; 미술 누적 횟수 ≥ 12
- `Flow_Painter_Ep2`: 이번 달 일정이 미술; 미술 누적 횟수 ≥ 20; 센스 ≥ 270; 감수성 ≥ 270

## 무용 교사 (DanceTutor)  우선순위 Weight 20495
**엔딩 조건(모두 충족해야 함)**
- 무용 B 달성(등급 3)
- 매력 ≥ 300
- 센스 ≥ 300

**선행 이벤트 체인**
- `Flow_DanceTutor_Ep1`: 이번 달 일정이 무용; 무용 누적 횟수 ≥ 12
- `Flow_DanceTutor_Ep2`: 이번 달 일정이 무용; 무용 누적 횟수 ≥ 20; 매력 ≥ 270; 센스 ≥ 270

## 음악 교사 (MusicTutor)  우선순위 Weight 20495
**엔딩 조건(모두 충족해야 함)**
- 음악 B 달성(등급 3)
- 감수성 ≥ 300
- 매력 ≥ 300

**선행 이벤트 체인**
- `Flow_MusicTutor_Ep1`: 이번 달 일정이 음악; 음악 누적 횟수 ≥ 12
- `Flow_MusicTutor_Ep2`: 이번 달 일정이 음악; 음악 누적 횟수 ≥ 20; 감수성 ≥ 270; 매력 ≥ 270

## 역사학자 (Historian)  우선순위 Weight 20495
**엔딩 조건(모두 충족해야 함)**
- 역사 B 달성(등급 3)
- 지능 ≥ 300
- 기품 ≥ 300

**선행 이벤트 체인**
- `Flow_Historian_Ep1`: 이번 달 일정이 역사; 역사 누적 횟수 ≥ 12
- `Flow_Historian_Ep2`: 이번 달 일정이 역사; 역사 누적 횟수 ≥ 20; 지능 ≥ 270; 기품 ≥ 270

## 마법 사범 (MagicTutor)  우선순위 Weight 20495
**엔딩 조건(모두 충족해야 함)**
- 마법학 B 달성(등급 3)
- 마력 ≥ 300
- 지능 ≥ 300

**선행 이벤트 체인**
- `Flow_MagicTutor_Ep1`: 이번 달 일정이 마법학; 마법학 누적 횟수 ≥ 12
- `Flow_MagicTutor_Ep2`: 이번 달 일정이 마법학; 마법학 누적 횟수 ≥ 20; 지능 ≥ 270; 마력 ≥ 270

## 예법교사 (Governess)  우선순위 Weight 20495
**엔딩 조건(모두 충족해야 함)**
- 예법 A 달성(등급 4)
- 예절 ≥ 400
- 기품 ≥ 400
- 감수성 ≥ 200

**선행 이벤트 체인**
- `Flow_Governess_Ep1`: 이번 달 일정이 예법; 예법 누적 횟수 ≥ 12
- `Flow_Governess_Ep2`: 이번 달 일정이 예법; 예법 누적 횟수 ≥ 20; 예절 ≥ 350; 기품 ≥ 350

## 체술사범 (MartialArtsInstructor)  우선순위 Weight 20495
**엔딩 조건(모두 충족해야 함)**
- 체술 B 달성(등급 3)
- 정신력 ≥ 300
- 체력 ≥ 300

**선행 이벤트 체인**
- `Flow_MartialArtsInstructor_Ep1`: 이번 달 일정이 체술; 체술 누적 횟수 ≥ 12; 정신력 ≥ 100; 이벤트 완료 StatCheck_MartialArts_Ep1
- `Flow_MartialArtsInstructor_Ep2`: 이번 달 일정이 체술; 체술 누적 횟수 ≥ 20; 정신력 ≥ 270; 체력 ≥ 270; 이벤트 완료 Flow_MartialArtsInstructor_Ep1

## 연금술사 (Alchemist)  우선순위 Weight 20495
**엔딩 조건(모두 충족해야 함)**
- 연금술 B 달성(등급 3)
- 지능 ≥ 300
- 마력 ≥ 300

**선행 이벤트 체인**
- `Flow_Alchemist_Ep1`: 이번 달 일정이 연금술; 연금술 누적 횟수 ≥ 12; 지능 ≥ 100
- `Flow_Alchemist_Ep2`: 이번 달 일정이 연금술; 연금술 누적 횟수 ≥ 20; 지능 ≥ 270; 마력 ≥ 270; 이벤트 완료 Flow_Alchemist_Ep1

## 물류 관리자 (UnloadingManager)  우선순위 Weight 20240
**엔딩 조건(모두 충족해야 함)**
- 부둣가 A 달성(등급 4)
- 체력 ≥ 300
- 정신력 ≥ 300

**선행 이벤트 체인**
- `Flow_UnloadingManager_Ep1`: 이번 달 일정이 부둣가; 부둣가 누적 횟수 ≥ 10
- `Flow_UnloadingManager_Ep2`: 이번 달 일정이 부둣가; 부둣가 등급 ≥ 4; 체력 ≥ 270; 정신력 ≥ 270

## 바텐더 (Bartender)  우선순위 Weight 20240
**엔딩 조건(모두 충족해야 함)**
- 주점 A 달성(등급 4)
- 화술 ≥ 300
- 기품 ≥ 300

**선행 이벤트 체인**
- `Flow_Bartender_Ep1`: 이번 달 일정이 주점; 주점 누적 횟수 ≥ 10
- `Flow_Bartender_Ep2`: 이번 달 일정이 주점; 주점 등급 ≥ 4; 화술 ≥ 270; 기품 ≥ 270

## 가정교사 (PrivateTutor)  우선순위 Weight 20200
**엔딩 조건(모두 충족해야 함)**
- 가정교사 A 달성(등급 4)
- 지능 ≥ 300
- 예절 ≥ 300

**선행 이벤트 체인**
- `Flow_PrivateTutor_Ep1`: 이번 달 일정이 가정교사; 가정교사 누적 횟수 ≥ 10
- `Flow_PrivateTutor_Ep2`: 이번 달 일정이 가정교사; 가정교사 등급 ≥ 4; 지능 ≥ 270; 예절 ≥ 270

## 메이드 카페의 슈퍼스타 (Superstar)  우선순위 Weight 20200
**엔딩 조건(모두 충족해야 함)**
- 메이드카페 A 달성(등급 4)
- 매력 ≥ 300
- 감수성 ≥ 300
- 화술 ≥ 200

**선행 이벤트 체인**
- `Flow_Superstar_Ep1`: 이번 달 일정이 메이드카페; 메이드카페 누적 횟수 ≥ 10; 매력 ≥ 300; 감수성 ≥ 150
- `Flow_Superstar_Ep2`: 이번 달 일정이 메이드카페; 메이드카페 등급 ≥ 4; 매력 ≥ 500; 감수성 ≥ 300; 화술 ≥ 150

## 연구조교 (Assistant)  우선순위 Weight 20200
**엔딩 조건(모두 충족해야 함)**
- 아카데미 B 달성(등급 3)
- 지능 ≥ 300
- 센스 ≥ 300

**선행 이벤트 체인**
- `Flow_Assistant_Ep1`: 이번 달 일정이 아카데미; 아카데미 누적 횟수 ≥ 10
- `Flow_Assistant_Ep2`: 이번 달 일정이 아카데미; 아카데미 누적 횟수 ≥ 19; 지능 ≥ 270; 센스 ≥ 270

## 의류 디자이너 (Designer)  우선순위 Weight 20160
**엔딩 조건(모두 충족해야 함)**
- 의상점 A 달성(등급 4)
- 기품 ≥ 300
- 매력 ≥ 300

**선행 이벤트 체인**
- `Flow_Designer_Ep1`: 이번 달 일정이 의상점; 의상점 누적 횟수 ≥ 11
- `Flow_Designer_Ep2`: 이번 달 일정이 의상점; 의상점 누적 횟수 ≥ 24; 기품 ≥ 280; 매력 ≥ 280

## 대장장이 (Blacksmith)  우선순위 Weight 20160
**엔딩 조건(모두 충족해야 함)**
- 대장간 A 달성(등급 4)
- 체력 ≥ 300
- 마력 ≥ 300

**선행 이벤트 체인**
- `Flow_Blacksmith_Ep1`: 이번 달 일정이 대장간; 대장간 누적 횟수 ≥ 12
- `Flow_Blacksmith_Ep2`: 이번 달 일정이 대장간; 대장간 누적 횟수 ≥ 24; 체력 ≥ 270; 마력 ≥ 270

## 병사 (Soldier)  우선순위 Weight 20080
**엔딩 조건(모두 충족해야 함)**
- 검문소 A 달성(등급 4)
- 예절 ≥ 300
- 체력 ≥ 300

**선행 이벤트 체인**
- `Flow_Soldier_Ep1`: 이번 달 일정이 검문소; 검문소 누적 횟수 ≥ 11
- `Flow_Soldier_Ep2`: 이번 달 일정이 검문소; 검문소 등급 ≥ 4; 예절 ≥ 270; 체력 ≥ 270

## 물류 관리인 (Logistics)  우선순위 Weight 20080
**엔딩 조건(모두 충족해야 함)**
- 과일창고 A 달성(등급 4)
- 힘 ≥ 300
- 정신력 ≥ 300

**선행 이벤트 체인**
- `Flow_Logistics_Ep1`: 이번 달 일정이 과일창고; 과일창고 누적 횟수 ≥ 13
- `Flow_Logistics_Ep2`: 이번 달 일정이 과일창고; 과일창고 등급 ≥ 4; 힘 ≥ 270; 정신력 ≥ 270

## 가사 전문 업체 사장 (Housekeeper)  우선순위 Weight 20000
**엔딩 조건(모두 충족해야 함)**
- 집안일 A 달성(등급 4)
- 지능 ≥ 250

**선행 이벤트 체인**
- `Flow_Housekeeper_Ep1`: 이번 달 일정이 집안일; 집안일 누적 횟수 ≥ 18
- `Flow_Housekeeper_Ep2`: 이번 달 일정이 집안일; 집안일 누적 횟수 ≥ 38

## 마을 치료사 (Doctor)  우선순위 Weight 20000
**엔딩 조건(모두 충족해야 함)**
- 치료소 B 달성(등급 3)
- 정신력 ≥ 300
- 체력 ≥ 300

**선행 이벤트 체인**
- `Flow_Doctor_Ep1`: 이번 달 일정이 치료소; 치료소 누적 횟수 ≥ 18
- `Flow_Doctor_Ep2`: 이번 달 일정이 치료소; 치료소 누적 횟수 ≥ 35; 체력 ≥ 270; 정신력 ≥ 270

## 목수 (Carpenter)  우선순위 Weight 20000
**엔딩 조건(모두 충족해야 함)**
- 목재공방 B 달성(등급 3)
- 힘 ≥ 300
- 센스 ≥ 300

**선행 이벤트 체인**
- `Flow_Carpenter_Ep1`: 이번 달 일정이 목재공방; 목재공방 누적 횟수 ≥ 18
- `Flow_Carpenter_Ep2`: 이번 달 일정이 목재공방; 목재공방 누적 횟수 ≥ 35; 힘 ≥ 270; 센스 ≥ 270

## 식당 점원 (Waitress)  우선순위 Weight 20000
**엔딩 조건(모두 충족해야 함)**
- 식당 B 달성(등급 3)
- 센스 ≥ 300
- 화술 ≥ 300

**선행 이벤트 체인**
- `Flow_Waitress_Ep1`: 이번 달 일정이 식당; 식당 누적 횟수 ≥ 18
- `Flow_Waitress_Ep2`: 이번 달 일정이 식당; 식당 누적 횟수 ≥ 30

## 목장 관리인 (RanchManager)  우선순위 Weight 20000
**엔딩 조건(모두 충족해야 함)**
- 목장 B 달성(등급 3)
- 체력 ≥ 300
- 감수성 ≥ 300

**선행 이벤트 체인**
- `Flow_RanchManager_Ep1`: 이번 달 일정이 목장; 목장 누적 횟수 ≥ 18
- `Flow_RanchManager_Ep2`: 이번 달 일정이 목장; 목장 누적 횟수 ≥ 35

## 원예 치료사 (Therapist)  우선순위 Weight 20000
**엔딩 조건(모두 충족해야 함)**
- 식물원 B 달성(등급 3)
- 감수성 ≥ 300
- 마력 ≥ 300

**선행 이벤트 체인**
- `Flow_Therapist_Ep1`: 이번 달 일정이 식물원; 식물원 누적 횟수 ≥ 18
- `Flow_Therapist_Ep2`: 이번 달 일정이 식물원; 식물원 누적 횟수 ≥ 35; 감수성 ≥ 270; 마력 ≥ 270

## 마을주민 (Villager)  우선순위 Weight 2
**엔딩 조건(모두 충족해야 함)**
- 예절 ≥ 100

**선행 이벤트 체인**
- `Flow_Villager_Ep1`: 이번 달 일정이 집안일; 집안일 누적 횟수 ≥ 15
- `Flow_Villager_Ep2`: 이번 달 일정이 집안일; 집안일 누적 횟수 ≥ 29

## 백수 (Jobless)  우선순위 Weight 1
**엔딩 조건(모두 충족해야 함)**
- (조건 없음)

**선행 이벤트 체인**
- `Flow_Jobless_Ep1`: 이번 달 일정이 집에서 휴식; 연도 ≤ 1210; Rest 횟수 ≥ 15
- `Flow_Jobless_Ep2`: 이번 달 일정이 집에서 휴식; 연도 ≤ 1211; Rest 횟수 ≥ 30
