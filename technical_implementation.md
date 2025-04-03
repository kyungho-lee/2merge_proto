# 2-Way 머지 게임 기술 구현 가이드

  

## 1. 개요

  

이 문서는 2-way 머지 게임의 기술적 구현에 대한 상세 가이드를 제공합니다. 게임의 코어 아키텍처를 기반으로 프로토타입 개발을 위한 구체적인 구현 방법, 기술 스택, 최적화 전략 및 확장성 고려사항을 다룹니다.

  

## 2. 기술 스택

  

### 2.1 개발 환경

- **게임 엔진**: Unity 2022.3 LTS

- **프로그래밍 언어**: C#

- **버전 관리**: Git

- **빌드 자동화**: Jenkins 또는 GitHub Actions

- **에셋 관리**: Unity Asset Bundle 시스템

  

### 2.2 서버 기술 (선택적)

- **백엔드**: Node.js 또는 Firebase

- **데이터베이스**: MongoDB 또는 Firebase Realtime Database

- **인증**: Firebase Authentication

- **클라우드 저장소**: Firebase Cloud Storage 또는 AWS S3

- **서버리스 함수**: Firebase Cloud Functions 또는 AWS Lambda

  

### 2.3 프론트엔드 기술

- **UI 프레임워크**: Unity UI 시스템 또는 UI Toolkit

- **애니메이션**: DOTween 또는 Unity Animation System

- **입력 처리**: Unity Input System

- **로컬라이제이션**: Unity Localization Package

  

### 2.4 분석 및 모니터링

- **분석**: Firebase Analytics 또는 Unity Analytics

- **크래시 리포팅**: Firebase Crashlytics

- **성능 모니터링**: Unity Profiler

- **A/B 테스트**: Firebase A/B Testing

  

## 3. 코어 시스템 구현

  

### 3.1 Entity Component System (ECS) 구현

  

#### 3.1.1 Entity 구현

```csharp

public class Entity

{

    private static int nextId = 0;

    public int Id { get; private set; }

    private Dictionary<Type, Component> components = new Dictionary<Type, Component>();

  

    public Entity()

    {

        Id = nextId++;

    }

  

    public T AddComponent<T>(T component) where T : Component

    {

        components[typeof(T)] = component;

        component.Entity = this;

        return component;

    }

  

    public T GetComponent<T>() where T : Component

    {

        if (components.TryGetValue(typeof(T), out Component component))

        {

            return (T)component;

        }

        return null;

    }

  

    public bool HasComponent<T>() where T : Component

    {

        return components.ContainsKey(typeof(T));

    }

  

    public void RemoveComponent<T>() where T : Component

    {

        if (components.ContainsKey(typeof(T)))

        {

            components.Remove(typeof(T));

        }

    }

}

```

  

#### 3.1.2 Component 기본 클래스

```csharp

public abstract class Component

{

    public Entity Entity { get; set; }

}

  

// 위치 컴포넌트 예시

public class PositionComponent : Component

{

    public int X { get; set; }

    public int Y { get; set; }

  

    public PositionComponent(int x, int y)

    {

        X = x;

        Y = y;

    }

}

  

// 아이템 컴포넌트 예시

public class ItemComponent : Component

{

    public string ItemId { get; set; }

    public int Level { get; set; }

    public ItemType Type { get; set; }

    public bool Mergeable { get; set; }

    public bool Splittable { get; set; }

  

    public ItemComponent(string itemId, int level, ItemType type, bool mergeable, bool splittable)

    {

        ItemId = itemId;

        Level = level;

        Type = type;

        Mergeable = mergeable;

        Splittable = splittable;

    }

}

```

  

#### 3.1.3 System 기본 구조

```csharp

public abstract class System

{

    protected EntityManager entityManager;

  

    public System(EntityManager entityManager)

    {

        this.entityManager = entityManager;

    }

  

    public abstract void Update(float deltaTime);

}

  

// 병합 시스템 예시

public class MergeSystem : System

{

    private MergeRuleManager mergeRuleManager;

  

    public MergeSystem(EntityManager entityManager, MergeRuleManager mergeRuleManager)

        : base(entityManager)

    {

        this.mergeRuleManager = mergeRuleManager;

    }

  

    public override void Update(float deltaTime)

    {

        // 병합 가능한 아이템 쌍 찾기

        // 병합 규칙 적용

        // 병합 이벤트 발생

    }

  

    public bool TryMerge(Entity item1, Entity item2)

    {

        var itemComp1 = item1.GetComponent<ItemComponent>();

        var itemComp2 = item2.GetComponent<ItemComponent>();

  

        if (itemComp1 == null || itemComp2 == null || !itemComp1.Mergeable || !itemComp2.Mergeable)

            return false;

  

        // 병합 규칙 확인

        MergeRule rule = mergeRuleManager.GetMergeRule(itemComp1.ItemId, itemComp2.ItemId);

        if (rule == null)

            return false;

  

        // 병합 실행

        Entity resultItem = CreateResultItem(rule);

        // 원본 아이템 제거

        entityManager.RemoveEntity(item1);

        entityManager.RemoveEntity(item2);

  

        // 병합 이벤트 발생

        EventManager.Instance.TriggerEvent(new MergeEvent(item1, item2, resultItem, rule));

  

        return true;

    }

  

    private Entity CreateResultItem(MergeRule rule)

    {

        // 결과 아이템 생성 로직

        // ...

        return null; // 실제 구현에서는 새 아이템 반환

    }

}

```

  

### 3.2 보드 시스템 구현

  

#### 3.2.1 보드 클래스

```csharp

public class Board

{

    private Cell[,] cells;

    public int Width { get; private set; }

    public int Height { get; private set; }

  

    public Board(int width, int height)

    {

        Width = width;

        Height = height;

        cells = new Cell[width, height];

  

        // 셀 초기화

        for (int x = 0; x < width; x++)

        {

            for (int y = 0; y < height; y++)

            {

                cells[x, y] = new Cell(x, y);

            }

        }

    }

  

    public Cell GetCell(int x, int y)

    {

        if (x < 0 || x >= Width || y < 0 || y >= Height)

            return null;

        return cells[x, y];

    }

  

    public bool IsValidPosition(int x, int y)

    {

        return x >= 0 && x < Width && y >= 0 && y < Height;

    }

  

    public bool IsCellEmpty(int x, int y)

    {

        Cell cell = GetCell(x, y);

        return cell != null && cell.IsEmpty;

    }

  

    public bool PlaceItem(Entity item, int x, int y)

    {

        Cell cell = GetCell(x, y);

        if (cell == null || !cell.IsEmpty)

            return false;

  

        cell.Item = item;

        // 아이템 위치 업데이트

        var posComp = item.GetComponent<PositionComponent>();

        if (posComp != null)

        {

            posComp.X = x;

            posComp.Y = y;

        }

        return true;

    }

  

    public Entity RemoveItem(int x, int y)

    {

        Cell cell = GetCell(x, y);

        if (cell == null || cell.IsEmpty)

            return null;

  

        Entity item = cell.Item;

        cell.Item = null;

        return item;

    }

  

    public List<Cell> GetAdjacentCells(int x, int y)

    {

        List<Cell> adjacentCells = new List<Cell>();

        // 상하좌우 인접 셀 확인

        int[] dx = { 0, 1, 0, -1 };

        int[] dy = { -1, 0, 1, 0 };

        for (int i = 0; i < 4; i++)

        {

            int newX = x + dx[i];

            int newY = y + dy[i];

            Cell cell = GetCell(newX, newY);

            if (cell != null)

            {

                adjacentCells.Add(cell);

            }

        }

        return adjacentCells;

    }

}

```

  

#### 3.2.2 셀 클래스

```csharp

public class Cell

{

    public int X { get; private set; }

    public int Y { get; private set; }

    public Entity Item { get; set; }

    public bool IsLocked { get; set; }

    public CellType Type { get; set; }

  

    public bool IsEmpty => Item == null;

  

    public Cell(int x, int y, CellType type = CellType.Normal)

    {

        X = x;

        Y = y;

        Item = null;

        IsLocked = false;

        Type = type;

    }

}

  

public enum CellType

{

    Normal,

    Generator,

    Locked,

    Boost,

    Sink,

    Special

}

```

  

### 3.3 아이템 시스템 구현

  

#### 3.3.1 아이템 팩토리

```csharp

public class ItemFactory

{

    private ItemDataManager itemDataManager;

    private EntityManager entityManager;

  

    public ItemFactory(ItemDataManager itemDataManager, EntityManager entityManager)

    {

        this.itemDataManager = itemDataManager;

        this.entityManager = entityManager;

    }

  

    public Entity CreateItem(string itemId, int x, int y)

    {

        ItemData itemData = itemDataManager.GetItemData(itemId);

        if (itemData == null)

            return null;

  

        Entity item = entityManager.CreateEntity();

        // 기본 컴포넌트 추가

        item.AddComponent(new PositionComponent(x, y));

        item.AddComponent(new ItemComponent(

            itemData.Id,

            itemData.Level,

            itemData.Type,

            itemData.Mergeable,

            itemData.Splittable

        ));

        // 시각적 컴포넌트 추가

        item.AddComponent(new VisualComponent(itemData.VisualAsset));

        // 특수 효과 컴포넌트 추가 (있는 경우)

        if (itemData.SpecialEffects.Count > 0)

        {

            item.AddComponent(new EffectsComponent(itemData.SpecialEffects));

        }

        return item;

    }

}

```

  

#### 3.3.2 아이템 데이터 관리자

```csharp

public class ItemDataManager

{

    private Dictionary<string, ItemData> itemDataMap = new Dictionary<string, ItemData>();

  

    public void LoadItemData(string jsonFilePath)

    {

        string json = File.ReadAllText(jsonFilePath);

        List<ItemData> itemDataList = JsonUtility.FromJson<List<ItemData>>(json);

        foreach (var itemData in itemDataList)

        {

            itemDataMap[itemData.Id] = itemData;

        }

    }

  

    public ItemData GetItemData(string itemId)

    {

        if (itemDataMap.TryGetValue(itemId, out ItemData itemData))

        {

            return itemData;

        }

        return null;

    }

  

    public List<ItemData> GetItemsByType(ItemType type)

    {

        return itemDataMap.Values.Where(item => item.Type == type).ToList();

    }

  

    public List<ItemData> GetItemsByLevel(int level)

    {

        return itemDataMap.Values.Where(item => item.Level == level).ToList();

    }

}

```

  

### 3.4 병합 및 분할 시스템 구현

  

#### 3.4.1 병합 규칙 관리자

```csharp

public class MergeRuleManager

{

    private List<MergeRule> mergeRules = new List<MergeRule>();

  

    public void LoadMergeRules(string jsonFilePath)

    {

        string json = File.ReadAllText(jsonFilePath);

        mergeRules = JsonUtility.FromJson<List<MergeRule>>(json);

    }

  

    public MergeRule GetMergeRule(string sourceItemId1, string sourceItemId2)

    {

        return mergeRules.FirstOrDefault(rule =>

            (rule.SourceItemId1 == sourceItemId1 && rule.SourceItemId2 == sourceItemId2) ||

            (rule.SourceItemId1 == sourceItemId2 && rule.SourceItemId2 == sourceItemId1));

    }

  

    public List<MergeRule> GetRulesByResultItem(string resultItemId)

    {

        return mergeRules.Where(rule => rule.ResultItemId == resultItemId).ToList();

    }

}

```

  

#### 3.4.2 분할 시스템

```csharp

public class SplitSystem : System

{

    private SplitRuleManager splitRuleManager;

    private ItemFactory itemFactory;

    private Board board;

  

    public SplitSystem(EntityManager entityManager, SplitRuleManager splitRuleManager,

                      ItemFactory itemFactory, Board board)

        : base(entityManager)

    {

        this.splitRuleManager = splitRuleManager;

        this.itemFactory = itemFactory;

        this.board = board;

    }

  

    public override void Update(float deltaTime)

    {

        // 분할 관련 로직 업데이트

    }

  

    public bool TrySplit(Entity item, Entity tool = null)

    {

        var itemComp = item.GetComponent<ItemComponent>();

        var posComp = item.GetComponent<PositionComponent>();

        if (itemComp == null || posComp == null || !itemComp.Splittable)

            return false;

  

        // 분할 규칙 확인

        SplitRule rule = splitRuleManager.GetSplitRule(itemComp.ItemId);

        if (rule == null)

            return false;

  

        // 도구 요구사항 확인

        if (!string.IsNullOrEmpty(rule.RequiredToolId) &&

            (tool == null || tool.GetComponent<ItemComponent>()?.ItemId != rule.RequiredToolId))

            return false;

  

        // 에너지 비용 확인

        if (!PlayerManager.Instance.TrySpendEnergy(rule.EnergyCost))

            return false;

  

        // 빈 공간 확인

        List<Cell> emptyCells = FindEmptyCellsNear(posComp.X, posComp.Y);

        if (emptyCells.Count < 1) // 최소 1개의 빈 셀 필요

            return false;

  

        // 분할 실행

        Entity resultItem1 = itemFactory.CreateItem(rule.ResultItemId1, posComp.X, posComp.Y);

        board.PlaceItem(resultItem1, posComp.X, posComp.Y);

  

        // 두 번째 아이템은 인접한 빈 셀에 배치

        Cell emptyCell = emptyCells[0];

        Entity resultItem2 = itemFactory.CreateItem(rule.ResultItemId2, emptyCell.X, emptyCell.Y);

        board.PlaceItem(resultItem2, emptyCell.X, emptyCell.Y);

  

        // 원본 아이템 제거

        entityManager.RemoveEntity(item);

  

        // 분할 이벤트 발생

        EventManager.Instance.TriggerEvent(new SplitEvent(item, resultItem1, resultItem2, rule));

  

        return true;

    }

  

    private List<Cell> FindEmptyCellsNear(int x, int y)

    {

        return board.GetAdjacentCells(x, y).Where(cell => cell.IsEmpty && !cell.IsLocked).ToList();

    }

}

```

  

### 3.5 이벤트 시스템 구현

  

#### 3.5.1 이벤트 기본 클래스

```csharp

public abstract class GameEvent

{

    public string EventType { get; protected set; }

    public float Timestamp { get; protected set; }

  

    protected GameEvent()

    {

        Timestamp = Time.time;

    }

}

  

public class MergeEvent : GameEvent

{

    public Entity SourceItem1 { get; private set; }

    public Entity SourceItem2 { get; private set; }

    public Entity ResultItem { get; private set; }

    public MergeRule Rule { get; private set; }

  

    public MergeEvent(Entity sourceItem1, Entity sourceItem2, Entity resultItem, MergeRule rule)

    {

        EventType = "Merge";

        SourceItem1 = sourceItem1;

        SourceItem2 = sourceItem2;

        ResultItem = resultItem;

        Rule = rule;

    }

}

  

public class SplitEvent : GameEvent

{

    public Entity SourceItem { get; private set; }

    public Entity ResultItem1 { get; private set; }

    public Entity ResultItem2 { get; private set; }

    public SplitRule Rule { get; private set; }

  

    public SplitEvent(Entity sourceItem, Entity resultItem1, Entity resultItem2, SplitRule rule)

    {

        EventType = "Split";

        SourceItem = sourceItem;

        ResultItem1 = resultItem1;

        ResultItem2 = resultItem2;

        Rule = rule;

    }

}

```

  

#### 3.5.2 이벤트 관리자

```csharp

public class EventManager

{

    private static EventManager instance;

    public static EventManager Instance

    {

        get

        {

            if (instance == null)

                instance = new EventManager();

            return instance;

        }

    }

  

    private Dictionary<string, List<Action<GameEvent>>> eventListeners =

        new Dictionary<string, List<Action<GameEvent>>>();

  

    public void AddListener(string eventType, Action<GameEvent> listener)

    {

        if (!eventListeners.ContainsKey(eventType))

        {

            eventListeners[eventType] = new List<Action<GameEvent>>();

        }

        eventListeners[eventType].Add(listener);

    }

  

    public void RemoveListener(string eventType, Action<GameEvent> listener)

    {

        if (eventListeners.ContainsKey(eventType))

        {

            eventListeners[eventType].Remove(listener);

        }

    }

  

    public void TriggerEvent(GameEvent gameEvent)

    {

        if (eventListeners.ContainsKey(gameEvent.EventType))

        {

            foreach (var listener in eventListeners[gameEvent.EventType])

            {

                listener.Invoke(gameEvent);

            }

        }

    }

}

```

  

## 4. 사용자 인터페이스 구현

  

### 4.1 UI 관리자

```csharp

public class UIManager : MonoBehaviour

{

    [SerializeField] private GameObject boardUI;

    [SerializeField] private GameObject itemPrefab;

    [SerializeField] private GameObject objectivesPanel;

    [SerializeField] private GameObject inventoryPanel;

    [SerializeField] private Text energyText;

    [SerializeField] private Text scoreText;

    [SerializeField] private Text movesText;

  

    private Dictionary<int, GameObject> itemUIObjects = new Dictionary<int, GameObject>();

    private Board board;

    private LevelManager levelManager;

  

    public void Initialize(Board board, LevelManager levelManager)

    {

        this.board = board;

        this.levelManager = levelManager;

        // 이벤트 리스너 등록

        EventManager.Instance.AddListener("Merge", OnMergeEvent);

        EventManager.Instance.AddListener("Split", OnSplitEvent);

        // 초기 UI 설정

        UpdateEnergyDisplay();

        UpdateScoreDisplay();

        UpdateMovesDisplay();

        UpdateObjectivesDisplay();

    }

  

    pu

(Content truncated due to size limit. Use line ranges to read in chunks)