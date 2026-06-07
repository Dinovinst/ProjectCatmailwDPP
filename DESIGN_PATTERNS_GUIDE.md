# 📚 Design Patterns Implementation Guide

## 🎯 บทนำ

ไฟล์นี้อธิบายรายละเอียดว่า Design Patterns ใช้อย่างไรในเกม Catmail

---

## 1️⃣ **Observer Pattern** (สัง่เกต/ผู้สังเกต)

### 📍 ที่ไหน?
- `Assets/Script/Patterns/Observer/EventManager.cs`
- `Assets/Script/Patterns/Observer/IObserver.cs`

### ❓ ทำไมใช้?

**ปัญหาเก่า:**
```csharp
// ใน Update() ทุก frame
void Update()
{
    if (playerItemCount changed)
        uiScore.text = playerItemCount.ToString();
    if (playerHealth changed)
        uiHealth.text = playerHealth.ToString();
    if (playerExperience changed)
        uiExp.text = playerExperience.ToString();
    // ... 100+ lines เช็คสิ่งต่าง ๆ
}
```

**ปัญหา:**
- ❌ Performance ไม่ดี - ทำการ check ทุก frame ทั้งที่ไม่เปลี่ยน
- ❌ Tight coupling - UI ต้อง reference Player
- ❌ ยากในการ maintain - ทุกครั้งที่มี new state ต้องเพิ่ม code

**วิธี Observer:**
```csharp
// ใน ItemCollected event
EventManager.OnItemCollected += UIManager.UpdateScore;

// เมื่อจริงๆมี item ถูกเก็บ
EventManager.RaiseItemCollected(coinValue);
// -> UIManager.UpdateScore(coinValue) ถูกเรียกอัตโนมัติ
```

**ประโยชน์:**
- ✅ Performance ดี - update เมื่อมีการเปลี่ยน
- ✅ Loose coupling - UI ไม่รู้จัก GameManager
- ✅ Easy to extend - เพิ่ม listener ใหม่ได้ง่าย

### 💻 ตัวอย่างการใช้:

```csharp
// 1. Event trigger (ใน GameManager.cs)
void OnPlayerCollectItem(Item item)
{
    int value = item.GetValue();
    EventManager.RaiseItemCollected(value);  // 📢 Notify all observers
}

// 2. Subscribe (ใน UIManager.cs)
void Start()
{
    EventManager.OnItemCollected += UpdateScoreUI;
}

// 3. Update UI
void UpdateScoreUI(int value)
{
    score += value;
    scoreText.text = score.ToString();
}
```

---

## 2️⃣ **Factory Pattern** (โรงงาน)

### 📍 ที่ไหน?
- `Assets/Script/Patterns/Factory/ItemFactory.cs`
- `Assets/Script/Patterns/Factory/EnemyFactory.cs`

### ❓ ทำไมใช้?

**ปัญหาเก่า:**
```csharp
// ทั่ว ๆ ไปในหลาย ๆ ที่
if (spawnType == "apple")
{
    GameObject itemObj = Instantiate(applePrefab, pos, Quaternion.identity);
    AppleItem item = itemObj.GetComponent<AppleItem>();
    // ... setup apple
}
else if (spawnType == "coin")
{
    GameObject itemObj = Instantiate(coinPrefab, pos, Quaternion.identity);
    CoinItem item = itemObj.GetComponent<CoinItem>();
    // ... setup coin
}
else if (spawnType == "banana")
{
    // ... 50+ more lines
}
```

**ปัญหา:**
- ❌ ซ้ำ ๆ code (duplicate)
- ❌ ยากจัดการ - เปลี่ยนวิธีสร้าง item ต้องเปลี่ยนหลายที่
- ❌ ง่ายผิด - อาจลืม setup อย่างไรอย่างหนึ่ง

**วิธี Factory:**
```csharp
// ศูนย์กลาง - ที่เดียวสำหรับสร้าง items
IItem item = ItemFactory.Instance.CreateItem(ItemType.Apple, position);
```

**ประโยชน์:**
- ✅ Centralized - ศูนย์กลางการสร้าง
- ✅ Easy to maintain - เปลี่ยน 1 ที่ทำให้ทั้งระบบเปลี่ยน
- ✅ Easy to extend - เพิ่ม item type ใหม่แค่ 2-3 บรรทัด

### 💻 ตัวอย่างการใช้:

```csharp
// การสร้าง items
void SpawnItems()
{
    for (int i = 0; i < 5; i++)
    {
        Vector3 pos = Random.insideUnitCircle * 10;
        IItem item = ItemFactory.Instance.CreateItem(ItemType.Apple, pos);
    }
}

// สร้าง enemies
void SpawnWave()
{
    EnemyFactory.Instance.SpawnEnemyWave(
        EnemyType.Goblin, 
        count: 5, 
        centerPos: Vector3.zero,
        spawnRadius: 5f
    );
}
```

---

## 3️⃣ **State Pattern** (รูปแบบสถานะ)

### 📍 ที่ไหน?
- `Assets/Script/Patterns/State/PlayerStateMachine.cs`

### ❓ ทำไมใช้?

**ปัญหาเก่า:**
```csharp
void Update()
{
    if (currentState == "idle")
    {
        if (Input.GetKey(KeyCode.W))
        {
            currentState = "running";
            // setup running...
        }
        else if (Input.GetKeyDown(KeyCode.Space))
        {
            currentState = "jumping";
            // setup jumping...
        }
        // ... more states
    }
    else if (currentState == "running")
    {
        if (!Input.GetKey(KeyCode.W))
        {
            currentState = "idle";
            // cleanup running...
        }
        // ... 100+ lines of if-else
    }
    else if (currentState == "jumping")
    {
        // ... more logic
    }
    // ... spaghetti code!
}
```

**ปัญหา:**
- ❌ Spaghetti code - nested if-else ยุ่งเหยิง
- ❌ Hard to debug - ไม่รู้ว่าอยู่ state ไหน
- ❌ Hard to add state - ต้องเพิ่ม logic ทุกที่

**วิธี State Pattern:**
```csharp
// แต่ละ state เป็นตัวเอง
public class IdleState : IState
{
    public void Update()
    {
        if (Input.GetKey(KeyCode.W))
            stateMachine.TransitionTo(new RunningState());
    }
}

public class RunningState : IState
{
    public void Update()
    {
        if (!Input.GetKey(KeyCode.W))
            stateMachine.TransitionTo(new IdleState());
    }
}

// เปลี่ยน state ได้ง่าย
stateMachine.TransitionTo(runningState);
```

**ประโยชน์:**
- ✅ Clean code - ไม่มี nested if-else
- ✅ Easy to debug - print state name
- ✅ Easy to extend - เพิ่ม state ใหม่เป็น class ใหม่
- ✅ Single Responsibility - แต่ละ state มี 1 job

### 💻 ตัวอย่างการใช้:

```csharp
// ใน PlayerStateMachine
void Awake()
{
    idleState = new IdleState(this, rb, animator);
    runningState = new RunningState(this, rb, animator);
    jumpingState = new JumpingState(this, rb, animator);
    
    TransitionTo(idleState);  // Start with Idle
}

// ใน Player.cs
void OnCollisionEnter2D(Collision2D collision)
{
    if (collision.gameObject.tag == "Enemy")
    {
        stateMachine.TransitionTo(stateMachine.TakingDamageState);
    }
}
```

---

## 📊 เปรียบเทียบ Before vs After

| ด้าน | ก่อนใช้ Pattern | หลังใช้ Pattern |
|-----|---------------|-------------------|
| **Code Complexity** | เยอะ (100+ lines) | น้อย (10-20 lines) |
| **Coupling** | Tight | Loose |
| **Maintainability** | ยาก | ง่าย |
| **Extensibility** | ยาก (ต้องแก้เก่า) | ง่าย (เพิ่ม class ใหม่) |
| **Debugging** | ยาก (trace if-else) | ง่าย (รู้ state ปัจจุบัน) |
| **Performance** | ไม่มีประสิทธิภาพ | ดี |

---

## 🎓 เรียนรู้เพิ่มเติม

### Observer Pattern
- **When to use:** Event-driven systems, UI updates, loosely coupled communication
- **Benefits:** Loose coupling, real-time updates, easy to extend
- **Drawback:** Complex debugging if too many observers

### Factory Pattern
- **When to use:** Complex object creation, many types, customizable creation
- **Benefits:** Centralized logic, easy to maintain, easy to extend
- **Drawback:** More code at first

### State Pattern
- **When to use:** Objects with multiple states, complex state transitions
- **Benefits:** Clean code, easy debugging, easy to extend
- **Drawback:** More classes (but cleaner overall)

---

## 🚀 ขั้นตอนการศึกษา

1. **อ่าน** code ใน `Assets/Script/Patterns/`
2. **เข้าใจ** comments ที่อธิบาย
3. **รัน** เกมและ trace execution
4. **ปรับเปลี่ยน** code เพื่อเห็นผลกระทบ
5. **เพิ่มฟีเจอร์** ใหม่ใช้ patterns เดิม

---

**Happy Learning! 🎉**
