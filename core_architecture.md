# 머지 게임 코어 아키텍처 설계

  

## 1. 개요

  

이 문서는 2-way 머지 게임의 코어 아키텍처를 설계하고 확장 가능한 테이블 구조를 제시합니다. 머지 게임의 핵심 메커니즘은 동일한 아이템 두 개를 합쳐 상위 레벨의 아이템을 생성하는 것입니다. 이 아키텍처는 Entity Component System(ECS) 패턴을 기반으로 설계되었으며, 확장성과 유지보수성을 고려했습니다.

  

## 2. 아키텍처 개요

  

### 2.1 핵심 아키텍처 다이어그램

  

```

+-------------------+     +-------------------+     +-------------------+

|    Game Manager   |     |    Board Manager  |     |   Item Factory    |

+-------------------+     +-------------------+     +-------------------+

         |                        |                         |

         v                        v                         v

+-------------------+     +-------------------+     +-------------------+

|  Entity Manager   |<--->|  Component System |<--->|   Event System    |

+-------------------+     +-------------------+     +-------------------+

         |                        |                         |

         v                        v                         v

+-------------------+     +-------------------+     +-------------------+

|   Render System   |     |   Physics System  |     |   Input System    |

+-------------------+     +-------------------+     +-------------------+

```

  

### 2.2 주요 시스템 구성요소

  

#### 2.2.1 Entity Component System (ECS)

  

ECS 패턴을 사용하여 게임 객체와 그 동작을 모듈화합니다:

  

- **Entity**: 게임 내 모든 객체(아이템, 보드 셀 등)의 기본 단위

- **Component**: 데이터만 포함하는 순수 데이터 컨테이너(위치, 아이템 레벨, 병합 가능 여부 등)

- **System**: 특정 컴포넌트를 가진 엔티티들을 처리하는 로직(병합 시스템, 생성 시스템 등)

  

#### 2.2.2 핵심 매니저 클래스

  

- **Game Manager**: 게임 상태 관리 및 전체 게임 흐름 제어

- **Board Manager**: 게임 보드 상태 및 셀 관리

- **Item Factory**: 다양한 아이템 생성 및 관리

- **Entity Manager**: 모든 게임 엔티티의 생성, 추적 및 소멸 관리

  

#### 2.2.3 시스템 클래스

  

- **Merge System**: 아이템 병합 로직 처리

- **Generation System**: 새 아이템 생성 로직

- **Progression System**: 게임 진행 및 레벨업 관리

- **Event System**: 게임 내 이벤트 처리(병합, 생성, 특수 효과 등)

  

## 3. 데이터 구조

  

### 3.1 아이템 데이터 구조

  

```typescript

interface ItemData {

  id: string;           // 아이템 고유 식별자

  level: number;        // 아이템 레벨 (1부터 시작)

  type: ItemType;       // 아이템 유형 (열거형)

  mergeable: boolean;   // 병합 가능 여부

  value: number;        // 아이템 가치/점수

  special: boolean;     // 특수 아이템 여부

  effects: Effect[];    // 아이템 효과 목록

}

  

enum ItemType {

  BASIC,

  RESOURCE,

  TOOL,

  SPECIAL,

  REWARD

}

  

interface Effect {

  type: EffectType;     // 효과 유형

  value: number;        // 효과 값/강도

  duration: number;     // 효과 지속 시간 (해당되는 경우)

}

```

  

### 3.2 보드 데이터 구조

  

```typescript

interface Board {

  width: number;        // 보드 너비

  height: number;       // 보드 높이

  cells: Cell[][];      // 2차원 셀 배열

}

  

interface Cell {

  x: number;            // X 좌표

  y: number;            // Y 좌표

  item: ItemData | null; // 셀에 있는 아이템 (없으면 null)

  locked: boolean;      // 셀 잠금 여부

  special: boolean;     // 특수 셀 여부

}

```

  

### 3.3 확장 가능한 테이블 구조

  

머지 게임의 핵심은 아이템 병합 규칙과 진행 테이블입니다. 아래는 확장 가능한 테이블 구조입니다:

  

#### 3.3.1 아이템 병합 테이블

  

```typescript

interface MergeRule {

  sourceType: ItemType;     // 소스 아이템 유형

  sourceLevel: number;      // 소스 아이템 레벨

  resultType: ItemType;     // 결과 아이템 유형

  resultLevel: number;      // 결과 아이템 레벨

  specialCondition?: any;   // 특수 조건 (선택적)

}

  

// 예시 테이블 구조

const mergeRules: MergeRule[] = [

  { sourceType: ItemType.BASIC, sourceLevel: 1, resultType: ItemType.BASIC, resultLevel: 2 },

  { sourceType: ItemType.BASIC, sourceLevel: 2, resultType: ItemType.BASIC, resultLevel: 3 },

  // 특수 병합 규칙

  {

    sourceType: ItemType.TOOL,

    sourceLevel: 2,

    resultType: ItemType.SPECIAL,

    resultLevel: 1,

    specialCondition: { requiredItem: "KEY" }

  },

  // ...

];

```

  

#### 3.3.2 아이템 진행 테이블

  

```typescript

interface ProgressionTable {

  level: number;            // 게임 레벨

  availableItems: ItemType[]; // 사용 가능한 아이템 유형

  spawnRates: SpawnRate[];  // 아이템 생성 확률

  objectives: Objective[];  // 레벨 목표

}

  

interface SpawnRate {

  itemType: ItemType;       // 아이템 유형

  itemLevel: number;        // 아이템 레벨

  rate: number;             // 생성 확률 (0-1)

}

  

interface Objective {

  type: ObjectiveType;      // 목표 유형

  target: any;              // 목표 대상

  count: number;            // 목표 수량

}

  

// 예시 테이블

const progressionTable: ProgressionTable[] = [

  {

    level: 1,

    availableItems: [ItemType.BASIC],

    spawnRates: [

      { itemType: ItemType.BASIC, itemLevel: 1, rate: 0.8 },

      { itemType: ItemType.BASIC, itemLevel: 2, rate: 0.2 }

    ],

    objectives: [

      { type: ObjectiveType.CREATE, target: { type: ItemType.BASIC, level: 5 }, count: 1 }

    ]

  },

  // 추가 레벨...

];

```

  

## 4. 핵심 시스템 흐름

  

### 4.1 병합 시스템 흐름

  

1. 사용자가 두 아이템을 선택하여 병합 시도

2. 병합 시스템이 두 아이템의 병합 가능 여부 확인

   - 동일한 유형과 레벨인지 확인

   - 특수 병합 규칙 확인

3. 병합 가능하면 병합 규칙 테이블에서 결과 아이템 결정

4. 원본 아이템 제거 및 결과 아이템 생성

5. 병합 이벤트 발생 및 관련 시스템에 통지

6. UI 업데이트 및 효과 표시

  

### 4.2 아이템 생성 시스템 흐름

  

1. 게임 상태에 따라 새 아이템 생성 필요 여부 결정

2. 현재 게임 레벨의 진행 테이블에서 생성 가능한 아이템 및 확률 확인

3. 확률에 따라 아이템 유형 및 레벨 결정

4. 보드에 빈 셀 확인 및 아이템 배치

5. 생성 이벤트 발생 및 관련 시스템에 통지

  

## 5. 확장성 고려사항

  

### 5.1 새로운 아이템 유형 추가

  

새로운 아이템 유형을 추가하려면:

1. `ItemType` 열거형에 새 유형 추가

2. 병합 규칙 테이블에 새 유형 관련 규칙 추가

3. 진행 테이블에 새 유형 생성 확률 추가

4. 필요한 경우 새 유형에 대한 특수 컴포넌트 구현

  

### 5.2 새로운 게임 메커니즘 추가

  

새로운 게임 메커니즘을 추가하려면:

1. 관련 컴포넌트 정의

2. 새 시스템 클래스 구현

3. 이벤트 시스템에 새 이벤트 유형 등록

4. 기존 시스템과의 상호작용 정의

  

### 5.3 다중 보드 지원

  

여러 보드를 지원하려면:

1. 보드 관리자를 확장하여 여러 보드 인스턴스 관리

2. 보드 간 아이템 이동 메커니즘 구현

3. 보드별 규칙 및 제약 조건 정의

  

## 6. 성능 최적화 전략

  

1. **객체 풀링**: 자주 생성/제거되는 아이템에 객체 풀링 적용

2. **지연 처리**: 대량의 업데이트가 필요한 경우 프레임에 분산 처리

3. **공간 분할**: 대규모 보드의 경우 공간 분할 기법 적용

4. **데이터 지역성**: 관련 데이터를 메모리에 인접하게 배치하여 캐시 효율성 향상

5. **이벤트 배치 처리**: 유사한 이벤트를 배치로 처리하여 오버헤드 감소

  

## 7. 결론

  

이 코어 아키텍처는 2-way 머지 게임의 기본 구조를 제공하며, Entity Component System 패턴을 활용하여 확장성과 유지보수성을 높였습니다. 테이블 기반 설계를 통해 게임 규칙과 진행을 쉽게 조정할 수 있으며, 새로운 기능과 아이템을 추가하기 용이합니다. 이 아키텍처를 기반으로 다양한 테마와 메커니즘을 가진 머지 게임을 개발할 수 있습니다.