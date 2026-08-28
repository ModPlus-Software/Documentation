# Система конфигураций ModPlus для WPF-плагинов

Для использования необходимо подключить nuget-пакет ModPlusAPI версии не ниже 35.0 (если в пакете модуля версия ниже)!

```<PackageReference Include="ModPlusAPI" Version="35.0.0" ExcludeAssets="runtime" />```

---

Система конфигураций ModPlus предоставляет разработчикам удобный механизм хранения нескольких наборов настроек, их переключения и управления ими через готовый UI-компонент. Она включает:

- базовый класс конфигурации `BaseConfiguration`;
- базовый контекст `BaseConfigurationsContext<TConfiguration>`;
- UI-контрол `ConfigurationsControl`.

Ниже описано, как использовать эти элементы в собственных плагинах.

---

## 1. BaseConfiguration — базовый класс конфигурации

Каждая конфигурация должна наследоваться от `BaseConfiguration`.  
Он предоставляет три свойства:

* `Id` (`Guid`)
Уникальный идентификатор конфигурации.

* `Name` (`string`)
Имя конфигурации, отображаемое в UI.

* `Description` (`string`)
Описание конфигурации (используется только если контекст создан с поддержкой описаний).

Все свойства реализуют уведомления об изменениях, поэтому их можно безопасно привязывать к WPF‑UI.

### Пример наследования

```csharp
public class MyConfiguration : BaseConfiguration, ICloneable<MyConfiguration>
{
    public int Offset { get; set; }
    public string Category { get; set; }

    public MyConfiguration Clone() => new()
    {
        Id = Id,
        Name = Name,
        Description = Description,
        Offset = Offset,
        Category = Category
    };
}
```

---

## 2. BaseConfigurationsContext<TConfiguration> — управление конфигурациями

`BaseConfigurationsContext<TConfiguration>` содержит всю логику работы с конфигурациями:

- загрузку конфигураций;
- сохранение при закрытии окна;
- переключение между конфигурациями;
- отслеживание изменений свойств;
- интеграцию с пользовательским файлом настроек и Настройками организации;
- предоставление команд, которые автоматически используются `ConfigurationsControl`.

Тип `TConfiguration` должен наследоваться от `BaseConfiguration` и реализовывать `ICloneable<TConfiguration>`.

---

### 2.1 Основные свойства контекста

* `Configurations`
Коллекция всех конфигураций пользователя.

*  `Configuration`
Текущая выбранная конфигурация.

*  `IsConfigurationSelected`
Указывает, выбрана ли конфигурация.

*  `ParentWindow`
Родительское окно, через которое выводятся диалоги и уведомления.

---

### 2.2. Публичные утилиты для работы с настройками

Контекст предоставляет два важных публичных свойства, которые могут использоваться **не только для конфигураций**, но и для любых пользовательских настроек плагина.

---

*  `UserConfigFileUtils`
Утилита для доступа к локальному пользовательскому файлу настроек.

Её можно использовать для:

- хранения простых параметров (последняя открытая конфигурация, координаты окна и т.д.);
- загрузки параметров между запусками плагина.

Контекст сам использует её для сохранения имени выбранной конфигурации.

---

*  `StorageFile`
Сервис работы с файлом настроек, который учитывает **Настройки организации**.

Он автоматически определяет, из какого файла читать данные:

- локального пользовательского файла,
- либо файла Настроек организации (если включены).

Может использоваться разработчиком для сохранения/загрузки дополнительных данных, которые должны подчиняться организационной политике.
Подробнее о работе с настройками в статье [Работа с пользовательскими настройками](user-settings.md)

---

### 2.3 События контекста

*  `OnConfigurationChanging`
Вызывается перед сменой конфигурации.

*  `OnConfigurationPropertyChanged`
Вызывается при изменении любого свойства текущей конфигурации.

---

### 2.4 Методы, которые должен реализовать разработчик

*  `OnLoaded()`
Вызывается после загрузки конфигураций.  
Подходит для инициализации логики плагина.

*  `OnClosed()`
Вызывается перед сохранением.  
Используется для финализации параметров.

---

### 2.5 Пример собственного контекста

```csharp
public class MyContext : BaseConfigurationsContext<MyConfiguration>
{
    public MyContext(IModPlusPluginGeneral connector, ModPlusWindow parentWindow)
        : base(connector,
               parentWindow,
               configurationsWithDescription: true,
               askNameOnConfigurationCreation: true)
    {
        OnConfigurationChanging += (s, oldConfig) =>
        {
            ParentWindow.ShowNotification("Switching from: " + oldConfig.Name);
        };
    }

    public override void OnLoaded()
    {
        // Например: читаем доп. настройки через UserConfigFileUtils
        ParentWindow.ShowNotification("Configurations loaded");
    }

    public override void OnClosed()
    {
        // Например: сохраняем что-то ещё через StorageFile или UserConfigFileUtils
        ParentWindow.ShowNotification("Configurations saved");
    }
}
```

---

## 3. ConfigurationsControl — UI‑контрол для управления конфигурациями

Готовый WPF‑контрол, отображающий:

- заголовок (опционально),
- выпадающий список конфигураций,
- кнопки управления (создание, копирование, удаление, импорт/экспорт).

Контрол автоматически привязывается к свойствам и командам контекста.

### Основные свойства

*  `Orientation`
Расположение элементов (`Horizontal` или `Vertical`).

*  `ShowHeader`
Показывать или скрывать заголовок.

### Пример использования

```xml
<mp:ConfigurationsControl
    Orientation="Horizontal"
    ShowHeader="True" />
```

---

## 4. Полная интеграция в окно плагина

### XAML

```xml
<mp:ConfigurationsControl Orientation="Horizontal"
                          ShowHeader="True" />
```

### Code-behind

```csharp
public partial class MyWindow : ModPlusWindow
{
    public MyWindow()
    {
        InitializeComponent();

        var connector = new MyPluginConnector();
        DataContext = new MyContext(connector, this);
    }
}
```

После этого:

- конфигурации автоматически загрузятся при открытии окна;
- UI предоставит полный набор функций для пользователя;
- данные сохранятся при закрытии.

---

## 5. Когда использовать систему конфигураций

Используйте её, если вашему плагину необходимо:

- хранить несколько профилей настроек;
- давать пользователю возможность управлять этими профилями;
- учитывать Настройки организации;
- автоматически сохранять выбор пользователя;
- использовать готовый, единообразный UI.
