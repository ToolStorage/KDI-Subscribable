# Kylin Subscribable Property

Unity 6 전용 반응형 프로퍼티 시스템. 값 변경 관찰, UI 바인딩, 상태 동기화에 사용한다. 외부 라이브러리(UniRx, R3) 없이 독립적으로 동작한다.

```
com.kylin.subscribable v1.0.0 | Unity 6000.0+ | MIT License
```

---

## 설치

### Scoped Registry (권장)

`Packages/manifest.json`에 추가:

```json
{
  "scopedRegistries": [
    {
      "name": "Kylin",
      "url": "https://registry.npmjs.org",
      "scopes": ["com.kylin"]
    }
  ],
  "dependencies": {
    "com.kylin.subscribable": "1.0.0"
  }
}
```

### Git URL

```
https://github.com/ToolStorage/KDI-Subscribable.git
```

---

## 주요 기능

### SubscribableProperty\<T\>

값 변경을 관찰할 수 있는 반응형 프로퍼티.

```csharp
var health = new SubscribableProperty<int>(100);

health.Subscribe(hp => Debug.Log($"HP: {hp}"), invokeInitial: true);
health.Value = 80; // "HP: 80" 출력
```

### LINQ 변환

```csharp
health
    .Select(hp => hp / 100f)        // int → float
    .Where(ratio => ratio <= 0.3f)  // 30% 이하만
    .Subscribe(ratio => ShowWarning())
    .AddTo(_cd);
```

### SubscribableCollection\<T\>

리스트의 변경(추가/삭제/교체/이동/초기화)을 관찰.

```csharp
var items = new SubscribableCollection<string>();

items.Subscribe(change =>
{
    switch (change.Type)
    {
        case CollectionChangeType.Add:
            Debug.Log($"추가: {change.NewValue}");
            break;
        case CollectionChangeType.Remove:
            Debug.Log($"삭제: {change.OldValue}");
            break;
    }
});

items.Add("Sword");  // "추가: Sword"
```

### SubscribableDictionary\<TKey, TValue\>

```csharp
var stats = new SubscribableDictionary<string, int>();

stats.SubscribeAdd((key, val) => Debug.Log($"스탯 추가: {key}={val}"));
stats.SubscribeReplace((key, oldVal, newVal) => Debug.Log($"변경: {key} {oldVal}→{newVal}"));

stats["ATK"] = 50;  // "스탯 추가: ATK=50"
stats["ATK"] = 75;  // "변경: ATK 50→75"
```

### SubscribableCommand

조건부 실행이 가능한 커맨드 패턴.

```csharp
var canAttack = new SubscribableProperty<bool>(true);
var attackCmd = new SubscribableCommand(canAttack, () => PerformAttack());

attackCmd.Execute();           // canAttack이 true이므로 실행됨
canAttack.Value = false;
attackCmd.Execute();           // 실행 안 됨
```

### CompositeDisposable

구독 수명 관리.

```csharp
var cd = new CompositeDisposable();

health.Subscribe(hp => UpdateUI(hp)).AddTo(cd);
items.SubscribeCount(count => UpdateCount(count)).AddTo(cd);

cd.Dispose(); // 모든 구독 한 번에 해제
```

### Reaction (트랜잭션)

여러 값 변경을 하나의 트랜잭션으로 묶어 구독자 호출을 최종 값으로 1회만 발생.

```csharp
using (Reaction.Begin())
{
    health.Value = 90;
    health.Value = 80;
    health.Value = 70;
} // 여기서 구독자에게 70으로 1회만 통지
```

---

## 디버그 도구

### Closure Profiler (에디터 전용)

구독 시 생성되는 클로저의 메모리 캡처를 분석하는 에디터 윈도우.

### SubscribableProperty Debugger (에디터 전용)

활성/해제된 구독 히스토리를 실시간 모니터링.
