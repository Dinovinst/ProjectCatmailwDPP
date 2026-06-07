# Project Catmail - Game Prototype with Design Patterns

## คำอธิบาย

**Project Catmail** เป็น 2D game prototype ที่พัฒนาบน **Unity 2D** 

### เป้าหมายของเกม
- ตามหาความรงจำของแมวน้อย
- ส่งพัสดุให้ถูกบ้าน

---

## Design Patterns ที่ใช้

### 1. **Observer Pattern** 
**ที่ไหน:** Event system สำหรับ UI updates

**ทำไมใช้:**
- เมื่อ Player ส่งพัสดุ ตัว UI ต้องอัพเดตอัตโนมัติ
- Observer ทำให้ Game Manager "push" ข้อมูลไปให้ UI อัตโนมัติ

**ตัวอย่างการใช้:**
```csharp
// Event สำหรับเมื่อเก็บ item
public static event System.Action<int> OnItemCollected;

// UI subscribe เข้ามา
UIManager.OnItemCollected += UpdateItemCount;

// เมื่อเก็บ item ก็ trigger event
OnItemCollected?.Invoke(itemCount);
```

**ประโยชน์:**
- Loose coupling - UI ไม่ต้อง reference GameManager  
- ง่ายการเพิ่ม listener ใหม่  
- ไม่ต้องมา modify GameManager ทุกครั้งที่มี listener ใหม่

---

### 2. **Factory Pattern** 
**ที่ไหน:** ItemFactory และ EnemyFactory

**ทำไมใช้:**
- สร้าง Item (apple, banana, coin) ทุกประเภท
- สร้าง Enemy ทุกประเภท
- เก่าเป็นวิธี `Instantiate(prefab)` ทั่วๆ ไปข้างหลังงาน
- Factory ศูนย์กลาง logic การสร้าง objects

**ตัวอย่างการใช้:**
```csharp
// ItemFactory ควบคุมวิธีการสร้าง item
public class ItemFactory
{
    public Item CreateItem(ItemType type, Vector3 position)
    {
        switch(type)
        {
            case ItemType.Apple:
                return CreateApple(position);
            case ItemType.Coin:
                return CreateCoin(position);
            default:
                return null;
        }
    }
}

// เรียกใช้ง่าย ๆ
Item newItem = factory.CreateItem(ItemType.Apple, spawnPos);
```

**ประโยชน์:**
- ศูนย์กลางการสร้าง objects  
- เปลี่ยนวิธีการสร้าง item ได้ยังไงหลายที่ต้องเปลี่ยน  
- ง่ายการเพิ่ม item type ใหม่

---

### 3. **State Pattern** 
**ที่ไหน:** PlayerController และ Enemy AI states

**ทำไมใช้:**
- Player มี states: Idle, Running, Jumping, Taking Damage
- Enemy มี states: Patrolling, Chasing, Attacking
- เก่าเป็น "if else" ยาวๆ มัน messy ยาก maintain
- State Pattern แยกแต่ละ state เป็น class ตัวเอง

**ตัวอย่างการใช้:**
```csharp
public interface IState
{
    void Enter();
    void Update();
    void Exit();
}

// IdleState
public class IdleState : IState
{
    public void Enter() { /* animation idle */ }
    public void Update() { /* check input */ }
    public void Exit() { /* cleanup */ }
}

// Transition states
stateMachine.TransitionTo(new RunningState());
```

**ประโยชน์:**
- ลบ nested if else  
- แต่ละ state เป็นตัวเอง ง่ายจัดการ  
- ง่ายการเพิ่ม state ใหม่ 

---

## โครงสร้างโปรเจกต์

```
ProjectCatmail/
├── Assets/
│   ├── Script/
│   │   ├── Patterns/
│   │   │   ├── Observer/
│   │   │   │   ├── EventManager.cs
│   │   │   │   └── IObserver.cs
│   │   │   ├── Factory/
│   │   │   │   ├── ItemFactory.cs
│   │   │   │   └── EnemyFactory.cs
│   │   │   └── State/
│   │   │       ├── IState.cs
│   │   │       ├── PlayerStateMachine.cs
│   │   │       └── States/
│   │   ├── Managers/
│   │   │   ├── GameManager.cs
│   │   │   └── UIManager.cs
│   │   ├── Entities/
│   │   │   ├── Player.cs
│   │   │   ├── Item.cs
│   │   │   └── Enemy.cs
│   │   └── Utils/
│   ├── Scenes/
│   ├── Sprites/
│   └── Prefab/
├── README.md
└── .gitignore
```

---

## วิธีการเล่น Game

### ควบคุมการเล่น:
- **Arrow Keys / WASD**: เดินไปมา
- **Space**: กระโดด
- **อัตโนมัติ**: เก็บ item เมื่อสัมผัส

---

## Design Patterns Reference

| Pattern | ไฟล์ | ประเภท | ประโยชน์ |
|---------|------|--------|---------|
| **Observer** | EventManager.cs | Behavioral | Event-driven, Loose coupling |
| **Factory** | ItemFactory.cs, EnemyFactory.cs | Creational | Centralized object creation |
| **State** | PlayerStateMachine.cs | Behavioral | State management, Clean code |

---

## Commit History

ทุก commit จะแสดงวิธีการปรับปรุงโค้ดและการใช้ Design Pattern:

```
Implement Observer Pattern for event system
   - จำหน่ายระบบ event การเก็บ item
   - UI subscribe เข้ามารับ updates
   
Implement Factory Pattern for item creation
   - Centralize logic สร้าง item ต่าง ๆ
   - ง่ายการเพิ่ม item type ใหม่

Implement State Pattern for player control
   - แยก Idle, Running, Jumping state
   - เอาเอา nested if-else ออกไป
```

---

## วิธีการเรียนรู้ Patterns

1. **ศึกษา commit ต่างๆ** เพื่อเข้าใจว่า pattern ไหนใช้ที่ไหน
2. **ดู code comments** ที่อธิบายรายละเอียด
3. **Modify code** และทดลองเพิ่ม features ใหม่
4. **เห็นความแตกต่าง** ระหว่าง old code vs pattern-based code

