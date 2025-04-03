# 2-Way 머지 게임을 위한 확장 가능한 테이블 구조

  

## 1. 개요

  

이 문서는 2-way 머지 게임을 위한 확장 가능한 테이블 구조를 정의합니다. 게임의 핵심 데이터를 체계적으로 구성하고, 새로운 콘텐츠와 기능을 쉽게 추가할 수 있는 유연한 구조를 제공합니다. 이 테이블 구조는 JSON, CSV, 또는 관계형 데이터베이스 등 다양한 형식으로 구현될 수 있습니다.

  

## 2. 아이템 마스터 테이블

  

### 2.1 구조 정의

  

```typescript

interface ItemMaster {

  id: string;                // 고유 식별자 (예: "SEED_01", "FLOWER_03")

  name: string;              // 아이템 이름

  description: string;       // 아이템 설명

  category: ItemCategory;    // 아이템 카테고리

  type: ItemType;            // 아이템 유형

  level: number;             // 아이템 레벨 (1-10)

  baseValue: number;         // 기본 가치/판매 가격

  rarity: ItemRarity;        // 희귀도

  mergeable: boolean;        // 병합 가능 여부

  splittable: boolean;       // 분할 가능 여부

  maxStack: number;          // 최대 중첩 수량

  energyCost: number;        // 분할 시 에너지 비용

  specialEffects: SpecialEffect[]; // 특수 효과 목록

  visualAsset: string;       // 시각적 에셋 경로

  animationAsset: string;    // 애니메이션 에셋 경로

  soundAsset: string;        // 사운드 에셋 경로

  unlockCondition: UnlockCondition; // 잠금 해제 조건

}

  

enum ItemCategory {

  BASIC,        // 기본 아이템

  RESOURCE,     // 자원 아이템

  TOOL,         // 도구 아이템

  SPECIAL,      // 특수 아이템

  REWARD        // 보상 아이템

}

  

enum ItemType {

  // 기본 아이템 유형

  SEED,         // 씨앗

  PLANT,        // 식물

  FLOWER,       // 꽃

  FRUIT,        // 과일

  STONE,        // 돌

  CRYSTAL,      // 크리스탈

  POTION,       // 물약

  COIN,         // 코인

  // 도구 아이템 유형

  SPLITTER,     // 분할 도구

  CATALYST,     // 촉매 도구

  HAMMER,       // 망치 도구

  ENERGY_BOOST, // 에너지 부스터

  // 특수 아이템 유형

  WILD_CARD,    // 와일드 카드

  TIME_FREEZE,  // 시간 정지

  BOARD_CLEAR,  // 보드 클리어

  // 확장 가능한 추가 유형

  CUSTOM        // 커스텀 유형 (확장용)

}

  

enum ItemRarity {

  COMMON,       // 일반 (흰색)

  UNCOMMON,     // 고급 (초록색)

  RARE,         // 희귀 (파란색)

  EPIC,         // 영웅 (보라색)

  LEGENDARY     // 전설 (주황색)

}

  

interface SpecialEffect {

  type: EffectType;          // 효과 유형

  value: number;             // 효과 값/강도

  duration: number;          // 효과 지속 시간 (턴 또는 초)

  probability: number;       // 발동 확률 (0-1)

  condition: string;         // 발동 조건 (선택적)

}

  

enum EffectType {

  BOOST_MERGE,               // 병합 보상 증가

  BOOST_SPLIT,               // 분할 비용 감소

  GENERATE_ENERGY,           // 에너지 생성

  GENERATE_ITEM,             // 아이템 생성

  CLEAR_OBSTACLE,            // 장애물 제거

  EXTEND_TIME,               // 시간 연장

  ADD_MOVES,                 // 이동 횟수 추가

  TRANSFORM_ITEM,            // 아이템 변환

  AREA_EFFECT                // 영역 효과

}

  

interface UnlockCondition {

  type: UnlockType;          // 잠금 해제 유형

  requirement: any;          // 요구 사항 (레벨, 아이템 등)

  quantity: number;          // 필요 수량

}

  

enum UnlockType {

  LEVEL,                     // 플레이어 레벨

  ITEM_DISCOVERED,           // 아이템 발견

  ACHIEVEMENT,               // 업적 달성

  PURCHASE,                  // 구매

  SPECIAL_EVENT              // 특별 이벤트

}

```

  

### 2.2 예시 데이터

  

```json

[

  {

    "id": "SEED_01",

    "name": "기본 씨앗",

    "description": "가장 기본적인 씨앗입니다. 두 개를 합치면 새싹이 됩니다.",

    "category": "BASIC",

    "type": "SEED",

    "level": 1,

    "baseValue": 10,

    "rarity": "COMMON",

    "mergeable": true,

    "splittable": false,

    "maxStack": 99,

    "energyCost": 0,

    "specialEffects": [],

    "visualAsset": "assets/items/seed_01.png",

    "animationAsset": "assets/animations/seed_01.json",

    "soundAsset": "assets/sounds/seed_merge.mp3",

    "unlockCondition": {

      "type": "LEVEL",

      "requirement": 1,

      "quantity": 1

    }

  },

  {

    "id": "PLANT_02",

    "name": "작은 새싹",

    "description": "씨앗에서 자란 작은 새싹입니다. 두 개를 합치면 꽃이 됩니다.",

    "category": "BASIC",

    "type": "PLANT",

    "level": 2,

    "baseValue": 25,

    "rarity": "COMMON",

    "mergeable": true,

    "splittable": true,

    "maxStack": 99,

    "energyCost": 5,

    "specialEffects": [],

    "visualAsset": "assets/items/plant_02.png",

    "animationAsset": "assets/animations/plant_02.json",

    "soundAsset": "assets/sounds/plant_merge.mp3",

    "unlockCondition": {

      "type": "ITEM_DISCOVERED",

      "requirement": "SEED_01",

      "quantity": 1

    }

  },

  {

    "id": "SPLITTER_01",

    "name": "기본 분할 도구",

    "description": "아이템을 하위 레벨로 분할할 수 있는 도구입니다.",

    "category": "TOOL",

    "type": "SPLITTER",

    "level": 1,

    "baseValue": 100,

    "rarity": "UNCOMMON",

    "mergeable": false,

    "splittable": false,

    "maxStack": 10,

    "energyCost": 0,

    "specialEffects": [

      {

        "type": "BOOST_SPLIT",

        "value": 0.2,

        "duration": 1,

        "probability": 1,

        "condition": ""

      }

    ],

    "visualAsset": "assets/items/splitter_01.png",

    "animationAsset": "assets/animations/splitter_01.json",

    "soundAsset": "assets/sounds/splitter_use.mp3",

    "unlockCondition": {

      "type": "LEVEL",

      "requirement": 5,

      "quantity": 1

    }

  }

]

```

  

## 3. 병합 규칙 테이블

  

### 3.1 구조 정의

  

```typescript

interface MergeRule {

  id: string;                // 규칙 고유 식별자

  sourceItemId1: string;     // 소스 아이템 1 ID

  sourceItemId2: string;     // 소스 아이템 2 ID (일반적으로 sourceItemId1과 동일)

  resultItemId: string;      // 결과 아이템 ID

  resultQuantity: number;    // 결과 아이템 수량 (기본값: 1)

  energyGain: number;        // 병합 시 획득 에너지

  experienceGain: number;    // 병합 시 획득 경험치

  coinGain: number;          // 병합 시 획득 코인

  specialCondition: string;  // 특수 조건 (선택적)

  bonusItemId: string;       // 보너스 아이템 ID (선택적)

  bonusProbability: number;  // 보너스 아이템 생성 확률 (0-1)

  unlockLevel: number;       // 규칙 잠금 해제 레벨

}

```

  

### 3.2 예시 데이터

  

```json

[

  {

    "id": "MERGE_SEED_01",

    "sourceItemId1": "SEED_01",

    "sourceItemId2": "SEED_01",

    "resultItemId": "PLANT_02",

    "resultQuantity": 1,

    "energyGain": 2,

    "experienceGain": 5,

    "coinGain": 5,

    "specialCondition": "",

    "bonusItemId": "",

    "bonusProbability": 0,

    "unlockLevel": 1

  },

  {

    "id": "MERGE_PLANT_02",

    "sourceItemId1": "PLANT_02",

    "sourceItemId2": "PLANT_02",

    "resultItemId": "FLOWER_03",

    "resultQuantity": 1,

    "energyGain": 5,

    "experienceGain": 10,

    "coinGain": 15,

    "specialCondition": "",

    "bonusItemId": "SEED_01",

    "bonusProbability": 0.2,

    "unlockLevel": 2

  },

  {

    "id": "MERGE_SPECIAL_01",

    "sourceItemId1": "SEED_01",

    "sourceItemId2": "WATER_01",

    "resultItemId": "SPECIAL_PLANT_01",

    "resultQuantity": 1,

    "energyGain": 10,

    "experienceGain": 20,

    "coinGain": 25,

    "specialCondition": "RAINY_DAY",

    "bonusItemId": "",

    "bonusProbability": 0,

    "unlockLevel": 10

  }

]

```

  

## 4. 분할 규칙 테이블

  

### 4.1 구조 정의

  

```typescript

interface SplitRule {

  id: string;                // 규칙 고유 식별자

  sourceItemId: string;      // 소스 아이템 ID

  resultItemId1: string;     // 결과 아이템 1 ID

  resultItemId2: string;     // 결과 아이템 2 ID (일반적으로 resultItemId1과 동일)

  resultQuantity1: number;   // 결과 아이템 1 수량

  resultQuantity2: number;   // 결과 아이템 2 수량

  energyCost: number;        // 분할 시 소모 에너지

  requiredToolId: string;    // 필요한 도구 ID (선택적)

  specialCondition: string;  // 특수 조건 (선택적)

  bonusItemId: string;       // 보너스 아이템 ID (선택적)

  bonusProbability: number;  // 보너스 아이템 생성 확률 (0-1)

  unlockLevel: number;       // 규칙 잠금 해제 레벨

}

```

  

### 4.2 예시 데이터

  

```json

[

  {

    "id": "SPLIT_PLANT_02",

    "sourceItemId": "PLANT_02",

    "resultItemId1": "SEED_01",

    "resultItemId2": "SEED_01",

    "resultQuantity1": 1,

    "resultQuantity2": 1,

    "energyCost": 5,

    "requiredToolId": "",

    "specialCondition": "",

    "bonusItemId": "",

    "bonusProbability": 0,

    "unlockLevel": 5

  },

  {

    "id": "SPLIT_FLOWER_03",

    "sourceItemId": "FLOWER_03",

    "resultItemId1": "PLANT_02",

    "resultItemId2": "PLANT_02",

    "resultQuantity1": 1,

    "resultQuantity2": 1,

    "energyCost": 10,

    "requiredToolId": "SPLITTER_01",

    "specialCondition": "",

    "bonusItemId": "SEED_01",

    "bonusProbability": 0.1,

    "unlockLevel": 8

  },

  {

    "id": "SPLIT_SPECIAL_01",

    "sourceItemId": "CRYSTAL_05",

    "resultItemId1": "GEM_04",

    "resultItemId2": "DUST_02",

    "resultQuantity1": 1,

    "resultQuantity2": 3,

    "energyCost": 20,

    "requiredToolId": "SPLITTER_02",

    "specialCondition": "FULL_MOON",

    "bonusItemId": "MAGIC_ESSENCE_01",

    "bonusProbability": 0.3,

    "unlockLevel": 15

  }

]

```

  

## 5. 레벨 정의 테이블

  

### 5.1 구조 정의

  

```typescript

interface LevelDefinition {

  id: string;                // 레벨 고유 식별자

  name: string;              // 레벨 이름

  description: string;       // 레벨 설명

  areaId: string;            // 영역 ID

  difficulty: number;        // 난이도 (1-10)

  boardWidth: number;        // 보드 너비

  boardHeight: number;       // 보드 높이

  moveLimit: number;         // 이동 제한 (0 = 무제한)

  timeLimit: number;         // 시간 제한 (초, 0 = 무제한)

  energyCost: number;        // 플레이 시 소모 에너지

  objectives: Objective[];   // 목표 목록

  rewards: Reward[];         // 보상 목록

  initialItems: InitialItem[]; // 초기 아이템 목록

  specialCells: SpecialCell[]; // 특수 셀 목록

  obstacles: Obstacle[];     // 장애물 목록

  specialRules: SpecialRule[]; // 특수 규칙 목록

  unlockCondition: UnlockCondition; // 잠금 해제 조건

}

  

interface Objective {

  type: ObjectiveType;       // 목표 유형

  targetId: string;          // 대상 ID (아이템, 장애물 등)

  quantity: number;          // 필요 수량

  optional: boolean;         // 선택적 목표 여부

}

  

enum ObjectiveType {

  CREATE_ITEM,               // 아이템 생성

  COLLECT_ITEM,              // 아이템 수집

  REMOVE_OBSTACLE,           // 장애물 제거

  MERGE_COUNT,               // 병합 횟수

  SPLIT_COUNT,               // 분할 횟수

  REACH_SCORE,               // 점수 달성

  SPECIAL_PATTERN            // 특수 패턴 완성

}

  

interface Reward {

  type: RewardType;          // 보상 유형

  itemId: string;            // 아이템 ID

  quantity: number;          // 수량

  probability: number;       // 확률 (0-1, 1 = 100%)

  condition: string;         // 조건 (선택적)

}

  

enum RewardType {

  ITEM,                      // 아이템

  COIN,                      // 코인

  GEM,                       // 보석

  ENERGY,                    // 에너지

  EXPERIENCE,                // 경험치

  SPECIAL                    // 특수 보상

}

  

interface InitialItem {

  itemId: string;            // 아이템 ID

  x: number;                 // X 좌표

  y: number;                 // Y 좌표

  quantity: number;          // 수량

}

  

interface SpecialCell {

  type: CellType;            // 셀 유형

  x: number;                 // X 좌표

  y: number;                 // Y 좌표

  effect: string;            // 효과 (선택적)

}

  

enum CellType {

  GENERATOR,                 // 생성기 셀

  LOCKED,                    // 잠긴 셀

  BOOST,                     // 부스트 셀

  SINK,                      // 싱크 셀 (아이템 제거)

  SPECIAL                    // 특수 셀

}

  

interface Obstacle {

  type: ObstacleType;        // 장애물 유형

  x: number;                 // X 좌표

  y: number;                 // Y 좌표

  health: number;            // 체력 (제거에 필요한 액션 수)

  effect: string;            // 효과 (선택적)

}

  

enum ObstacleType {

  ROCK,                      // 바위

  ICE,                       // 얼음

  LOCK,                      // 자물쇠

  VINE,                      // 덩굴

  PORTAL,                    // 포털

  CUSTOM                     // 커스텀 장애물

}

  

interface SpecialRule {

  type: RuleType;            // 규칙 유형

  value: any;                // 규칙 값

  description: string;       // 규칙 설명

}

  

enum RuleType {

  NO_MERGE,                  // 병합 불가

  NO_SPLIT,                  // 분할 불가

  ITEM_LIMIT,                // 아이템 제한

  TIME_PRESSURE,             // 시간 압박 (시간이 지날수록 난이도 증가)

  SPECIAL_CONDITION          // 특수 조건

}

```

  

### 5.2 예시 데이터

  

```json

[

  {

    "id": "LEVEL_1_1",

    "name": "첫 번째 도전",

    "description": "머지 마스터의 세계에 오신 것을 환영합니다! 기본적인 병합을 배워봅시다.",

    "areaId": "EMERALD_FOREST",

    "difficulty": 1,

    "boardWidth": 5,

    "boardHeight": 5,

    "moveLimit": 15,

    "timeLimit": 0,

    "energyCost": 5,

    "objectives": [

      {

        "type": "CREATE_ITEM",

        "targetId": "PLANT_02",

        "quantity": 2,

        "optional": false

      }

    ],

    "rewards": [

      {

        "type": "COIN",

        "itemId": "",

        "quantity": 50,

        "probability": 1,

        "condition": ""

      },

      {

        "type": "ITEM",

        "itemId": "SEED_01",

        "quantity": 2,

        "probability": 1,

        "condition": ""

      }

    ],

    "initialItems": [

      {

        "itemId": "SEED_01",

        "x": 1,

        "y": 2,

        "quantity": 1

      },

      {

        "itemId": "SEED_01",

        "x": 3,

        "y": 2,

        "quantity": 1

      },

      {

        "itemId": "SEED_01",

        "x": 2,

        "y": 3,

        "quantity": 1

      }

    ],

    "specialCells": [

      {

        "type": "GENERATOR",

        "x": 0,

        "y": 0,

        "effect": "SEED_01"

      }

    ],

    "obstacles": [],

    "specialRules": [],

    "unlockCondition": {

      "type": "LEVEL",

      "requirement": 1,

      "quantity": 1

    }

  },

  {

    "id": "LEVEL_2_5",

    "name": "분할의 비밀",

    "description": "이제 아이템을 분할하는 방법을 배워봅시다!",

    "areaId": "EMERALD_FOREST",

    "difficulty": 3,

    "boardWidth": 5,

    "boardHeight": 5,

    "moveLimit": 20,

    "timeLimit": 0,

    "energyCost": 8,

    "objectives": [

      {

        "type": "SPLIT_COUNT",

        "targetId": "",

        "quantity": 2,

        "optional": false

      },

      {

        "type": "CREATE_ITEM",

        "targetId": "FLOWER_03",

        "quantity": 1,

        "optional": false

      }

    ],

    "rewards": [

      {

        "type": "COIN",

        "itemId": "",

        "quantity": 100,

        "probability": 1,

        "condition": ""

      },

      {

        "type": "ITEM",

        "itemId": "SPLITTER_01",

        "quantity": 1,

        "probability": 1,

        "condition": ""

      }

    ],

    "initialItems": [

      {

        "itemId": "PLANT_02",

        "x": 1,

        "y": 1,

        "quantity": 1

      },

      {

        "itemId": "PLANT_02",

        "x": 3,

        "y": 3,

        "quantity": 1

      },

      {

        "itemId": "SPLITTER_01",

        "x": 2,

        "y": 2,

        "quantity": 1

      }

    ],

    "specialCells": [],

    "obstacles": [

      {

        "type": "ROCK",

        "x": 0,

        "y": 0,

        "health": 2,

        "effect": ""

      },

      {

        "type": "ROCK",

        "x": 4,

        "y": 4,

        "health": 2,

        "effect": ""

      }

    ],

    "specialRules": [],

    "unlockCondition": {

      "type": "LEVEL",

      "requirement": 5,

      "quantity": 1

    }

  }

]

```

  

## 6. 플레이어 진행 테이블

  

### 6.1 구조 정의

  

```typescript

interface PlayerProgression {

  level: number;             // 플레이어 레벨

  experienceRequired: number; // 다음 레벨로 진행하는 데 필요한 경험치

  maxEnergy: number;         // 최대 에너지

  energyRechargeRate: number; // 에너지 충전 속도 (분당)

  unlockedFeatures: string[]; // 잠금 해제된 기능 목록

  unlockedItems: string[];   // 잠금 해제된 아이템 목록

  rewards: Reward[];         // 레벨업 보상

}

```

  

### 6.2 예시 데이터

  

```json

[

  {

    "level": 1,

    "experienceRequired": 0,

    "maxEnergy": 50,

    "energyRechargeRate": 1,

    "unlockedFeatures": ["BASIC_MERGE"],

    "unlockedItems": ["SEED_01"],

    "rewards": [

      {

        "type": "COIN",

        "itemId": "",

        "quantity": 100,

        "probability": 1,

        "condition": ""

      }

    ]

  },

  {

    "level": 2,

    "experienceRequired": 100,

    "maxEnergy": 55,

    "energyRechargeRate": 1,

    "unlockedFeatures": ["ITEM_SHOP"],

    "unlockedItems": ["PLANT_02"],

    "rewards": [

      {

        "type": "COIN",

        "itemId": "",

        "quantity": 150,

        "probability": 1,

        "condition": ""

      },

      {

        "type": "ITEM",

        "itemId": "SEED_01",

        "quantity": 3,

        "probability": 1,

        "condition": ""

      }

    ]

  },

  {

    "level": 5,

    "experienceRequired": 500,

    "maxEnergy": 70,

    "energyRechargeRate": 1.2,

    "unlockedFeatures": ["SPLIT_SYSTEM", "DAILY_CHALLENGES"],

    "unlockedItems": ["SPLITTER_01", "FLOWER_03"],

    "rewards": [

      {

        "type": "COIN",

        "itemId": "",

        "quantity": 300,

        "probability": 1,

        "condition": ""

      },

      {

        "type": "ITEM",

        "itemId": "SPLITTER_01",

        "quantity": 1,

        "proba

(Content truncated due to size limit. Use line ranges to read in chunks)