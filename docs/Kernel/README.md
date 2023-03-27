
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
### Config
Контейнер для файлов конфигураций.

### Economic
Сервис обработки и хранения внутриигровых валют.

### Inventory
Сервис для работы с инвентарями.

### LevelsManager
Сервис переключения игровых уровней.

### UI
Сервис для работы с игровым графических интерфейсом.

### VFXFactory
Сервис для оптимизированного создания графических эффектов из преднастроенных пулов.