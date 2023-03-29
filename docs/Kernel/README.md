## Базовые понятия
**Kernel** - статический класс, точка входа в функционал GLG SDK.

**EntityData** - скрипт, который следует добавлять на все значимые предметы. Например оружие, персонажи, скины и прочее.
В нем хранится тип предмета (ItemKind), хозяин предмета (Owner), а также уникальный ключ (UniversalKey), который пригодится в следующих ситуациях:
- сохранение предмета в инвентаре
- сортировка предметов
- взаимодействие с предметами
- создание предметов из ресурсов
- получение конфигураций, модификаторов, рецептов, множителей и прочего
Иными словами этот ключ является уникальным идентификатором предмета.

**Управляемый скрипт** - класс, зарегистрированный в Kernel и реализующий один 
или несколько интерфейсов: IManaged, IManagedFixed, IManagedLate.
Позволяет заменить стандартные сообщения юнити Update, FixedUpdate и ManagedUpdate, что повышает производительность.

**Сервис. Базовый сервис и кастомный сервис** - система (скрипт, класс, набор), зарегистрированая в Kernel и выполняющая
определенный набор функционала. Kernel в данном случае выполняет функцию сервис-локатора.
Базовые сервисы уже реализованы в SDK и доступны через свойства в Kernel. Кастомные сервисы можно добавлять, удалять и получать в Kernel.

## Kernel
Точка входа в бóльшую часть функционала GLG.
Регистрирует управляемые скрипты **IManaged**, **IManagedFixed**, **IManagedLate**, а также управляемые сервисы **IManagedService**

### Базовые сервисы доступны через свойства:

|  Сервис |  Описание |
| ------------ | ------------ |
| `Kernel.Config;` | Контейнер файлов конфигураций  |
| `Kernel.Economic;`  |  Система управления финансами игрока |
|`Kernel.Ads;`|[WIP] Обёртка рекламных агрегаторов. В РАЗРАБОТКЕ|
|`Kernel.UI;`|Система управления графическим интерфейсом|
|`Kernel.LevelsManager;`|Переключатель игровых уровней (сцен)|
|`Kernel.VFXFactory`|Спавнер графических эффектов|
|`Kernel.Analytics`|Обёртка сервисов аналитики|
|`Kernel.Inventory`|Система инфвентарей|

И через метод получения сервиса
```csharp
ServiceType service = Kernel.Get<ServiceType>();
```

### Объект для запуска независимых корутин
```csharp
Kernel.CoroutinesObject.StartCoroutine(myCoroutine);
```

### Работа с кастомными сервисами
```csharp
ServiceType myCustomService;

Kernel.RegisterService(myCustomService); // регистрация сервиса в системе

ServiceType service = Kernel.Get<ServiceType>(); // получение сервиса из системы

Kernel.UnregisterService(service); // удаление сервиса из системы
```
### Работа с управляемыми скриптами

```csharp
void Awake
{
	Kernel.RegisterManaged(this); // регистрирует скрипт в системе
}

void OnDestroy()
{
	Kernel.UnregisterManaged(this); // удаляет скрипт из системы
}
```

## Базовые сервисы

### Ads [WIP]
Обёртка поставщиков рекламы сейчас в процессе разработки.

### Analytics
Обёртка сервисов аналитики. Дает возможность отправлять сообщения сразу во все сервисы аналитики.

На данный момент доступны 2 адаптера: Facebook, AppMetrica. 
Чтобы начать их использовать, необходимо сначала импортировать все необходимые ресурсы от поставщиков аналитики,
затем в Scripting Define Symbols добавить AppMetrica и Facebook.

Для отправки сообщений используются следующие методы:
```csharp
Kernel.Analytics.Track("eventId"); // Отправка события без параметров
Kernel.Analytics.Track("eventId", "paramName", "paramValue"); // Отправка события с одним параметром
Kernel.Analytics.Track("eventId", ("paramName1","paramValue1"), ("paramName2","paramValue2")); // Отправка события с массивом параметров
Kernel.Analytics.Track("eventId", new Dictionary<string, object> {...}); // Отправка события со словарем параметров
```

Для некоторых поставщиков аналитики (например AppMetrika) иногда требуется принудительная отправка сообщений:
```csharp
Kernel.Analytics.Flush();
```
### Config
Контейнер для файлов конфигураций.

### Economic
Сервис обработки и хранения внутриигровых валют.

### Inventory
Сервис для работы с инвентарями. 
Система позволяет работать с несколькими инвентарями, в кажом из которых содержатся предметы определенного типа. 
Типы предмета содержатся в перечислении EntityKind. 
После добавления нового типа предметов необходимо прописать его в методе EntityData.GetKinds() по аналогии с остальными типами.

При сохранении обрабатываются только измененные инвентари, что гарантирует низкие затраты ресурсов даже при наличии множества типов предметов.


#### Загрузка и сохранение
 предметов инвентаря
```csharp
// Идентификатор набора инвентарей.
string inventoryIdentifier = "mainInventory_";

// Загружает набор инвентарей с указанным идентификатором
Kernel.Inventory.Load(inventoryIdentifier); 

// Сохраняет измененные инвентари
Kernel.Inventory.Save(); 
```
#### Получение
 предметов инвентаря
```csharp
// Тип инвентаря.
EntityKind entityKind = EntityKind.Weapon; 
// Универсальный ключ искомого предмета.
string universalKey = "grenade"; 
// Данные искомого предмета.
EntityData entityData = someWeapon.GetComponent<EntityData>(); 

// Получает количество указанных предметов.
Kernel.Inventory.GetItems(entityKind, universalKey); 

// Получает количество указанных предметов.
Kernel.Inventory.GetItems(entityData); 

// Получает весь инвентарь указанного типа.
Kernel.Inventory.GetInventory(entityKind); 
```
#### Добавление
 предметов в инвентарь
```csharp
// Тип инвентаря.
EntityKind entityKind = EntityKind.Weapon; 
// Универсальный ключ искомого предмета.
string universalKey = "grenade"; 
// Данные искомого предмета.
EntityData entityData = someWeapon.GetComponent<EntityData>(); 

// Добавляет указанное количество предметов в инвентарь.
Kernel.Inventory.AddItems(entityKind, universalKey, value); 

// Добавляет указанное количество предметов в инвентарь.
Kernel.Inventory.AddItems(entityData, value); 
```
#### Установка количества
 предметов в инвентаре
```csharp
// Тип инвентаря.
EntityKind entityKind = EntityKind.Weapon; 
// Универсальный ключ искомого предмета.
string universalKey = "grenade"; 
// Данные искомого предмета.
EntityData entityData = someWeapon.GetComponent<EntityData>(); 

Kernel.Inventory.SetItems(entityKind, universalKey, value); // 
Kernel.Inventory.SetItems(entityData, value); // 
```
#### Удаление
 предметов из инвентаря
```csharp
// Тип инвентаря.
EntityKind entityKind = EntityKind.Weapon; 
// Универсальный ключ искомого предмета.
string universalKey = "grenade"; 
// Данные искомого предмета.
EntityData entityData = someWeapon.GetComponent<EntityData>(); 

// Удаляет указанное количество предметов из инвентаря.
Kernel.Inventory.RemoveItems(EntityKind entityKind, string universalKey, int value);
// Удаляет указанное количество предметов из инвентаря.
Kernel.Inventory.RemoveItems(EntityData entityData, int value);
```
#### Подписка на изменение
 предметов в инвентаре
```csharp
// Тип инвентаря.
EntityKind entityKind = EntityKind.Weapon;
// Метод будет выполняться при изменении инвентаря
void InventoryChangedHandler()
{
	// do smth
}
// Подписывается на событие изменения определенного типа предметов в инвентаре.
Kernel.Inventory.SubscribeToInventoryChanges(EntityKind entityKind, System.Action<string, int> callback);
// Отписывается от события изменения определенного типа предметов в инвентаре.
Kernel.Inventory.UnsubscribeFromInventoryChanges(EntityKind entityKind, System.Action<string, int> callback);
```

### LevelsManager
Сервис переключения игровых уровней.

### UI
Сервис для работы с игровым графических интерфейсом.

### VFXFactory
Сервис для оптимизированного создания графических эффектов из преднастроенных пулов.