# Конфигурации плагина

Для использования необходимо подключить nuget-пакет ModPlusAPI версии не ниже 35.0 (если в пакете модуля версия ниже)!

```<PackageReference Include="ModPlusAPI" Version="35.0.0" ExcludeAssets="runtime" />```

!!! warning "Для `BaseConfigurationsContextWithoutOrgSettings` нужен ModPlusAPI 42.0"
    Контекст без учёта Настроек организации появился в ModPlusAPI 42.0. Если в плагине используется именно он, версия пакета должна быть не ниже 42.0:

    ```<PackageReference Include="ModPlusAPI" Version="42.0.0" ExcludeAssets="runtime" />```

---

## Что это и зачем

**Конфигурация** — это именованный набор настроек плагина. Пользователь может создать несколько конфигураций (например, «Фасады», «Планы этажей», «Для заказчика А») и быстро переключаться между ними, не перенастраивая окно каждый раз заново.

В окне плагина это выглядит как выпадающий список конфигураций и ряд кнопок рядом с ним: создать, переименовать, дублировать, экспортировать, импортировать, удалить. Всё это уже реализовано в ModPlusAPI — разработчику остаётся описать, *какие* настройки входят в конфигурацию, и разместить готовый контрол в окне.

Систему конфигураций стоит использовать, если плагину нужно:

- хранить несколько наборов настроек и переключаться между ними;
- давать пользователю создавать, копировать, удалять и передавать коллегам (экспорт/импорт) эти наборы;
- запоминать, какой набор был выбран в прошлый раз;
- выглядеть единообразно с другими плагинами ModPlus.

Если у плагина один набор настроек без переключения, конфигурации не нужны — достаточно [пользовательских настроек](user-settings.md).

---

## Из чего состоит

| Элемент | Что это | Кто пишет |
|---|---|---|
| `BaseConfiguration` | Базовый класс одной конфигурации: `Id`, `Name`, `Description` | Разработчик наследует свой класс и добавляет в него настройки |
| `BaseConfigurationsContext<TConfiguration>` | Контекст (ViewModel) окна: загрузка, сохранение, переключение, команды для кнопок. **Учитывает Настройки организации** | Разработчик наследует свой контекст |
| `BaseConfigurationsContextWithoutOrgSettings<TConfiguration>` | То же самое, но **без учёта Настроек организации** (ModPlusAPI 42.0+) | Разработчик наследует свой контекст |
| `ConfigurationsControl` | Готовый UI: заголовок, список конфигураций, кнопки | Просто размещается в XAML окна |

Пространства имён: `ModPlusAPI.Models` (`BaseConfiguration`), `ModPlusAPI.Abstractions` (`ICloneable<T>`), `ModPlusAPI.Contexts` (контексты), `ModPlusAPI.Windows.Controls` (`ConfigurationsControl`). Сборка: `ModPlusAPI`.

---

## Какой контекст выбрать

Оба контекста имеют одинаковый набор свойств, событий, команд и методов и работают с одним и тем же `ConfigurationsControl`. Отличаются они только тем, **откуда читаются и куда сохраняются конфигурации**.

| | `BaseConfigurationsContext` | `BaseConfigurationsContextWithoutOrgSettings` |
|---|---|---|
| Минимальная версия ModPlusAPI | 35.0 | **42.0** |
| Настройки организации | Учитываются | Не учитываются |
| Где хранятся конфигурации | В пользовательском файле или в файле организации (если он задан администратором) | Всегда в пользовательском файле |
| Тип свойства `StorageFile` | `OrgSettingsService` | `UserSettingsService` |
| Блокировка файла на время работы | Да, для файла организации | Нет |
| Запрет редактирования | Возможен, если у пользователя нет прав на файл организации | Нет |
| Переключение между файлами конфигураций | Возможно, если администратор задал несколько файлов | Нет |

**Как выбрать:**

- Конфигурации плагина имеет смысл централизованно раздавать сотрудникам организации (стандарты оформления, шаблоны, правила именования) → `BaseConfigurationsContext`.
- Конфигурации — личное дело каждого пользователя, и Настройки организации для этого плагина не нужны → `BaseConfigurationsContextWithoutOrgSettings`.

!!! tip "Смена контекста не теряет данные пользователей"
    Если Настройки организации не заданы, `BaseConfigurationsContext` хранит конфигурации в том же файле и том же узле, что и `BaseConfigurationsContextWithoutOrgSettings`: `<папка пользовательских настроек ModPlus>\UserData\<Имя плагина>.xml`, узел `Configurations`. Поэтому при переходе плагина с одного контекста на другой ранее созданные пользователем конфигурации сохраняются.

---

## Быстрый старт

Ниже — минимальный рабочий пример из четырёх шагов. Пример использует `BaseConfigurationsContext`; для контекста без Настроек организации меняется только базовый класс в шаге 2.

### Шаг 1. Класс конфигурации

Класс наследуется от `BaseConfiguration`, реализует `ICloneable<T>` и содержит настройки плагина в виде публичных свойств.

```csharp
namespace MyPlugin.Models;

using ModPlusAPI.Abstractions;
using ModPlusAPI.Models;

/// <summary>
/// Конфигурация плагина
/// </summary>
public class MyConfiguration : BaseConfiguration, ICloneable<MyConfiguration>
{
    private int _startNumber = 1;
    private string _prefix;

    /// <summary>
    /// Начальный номер
    /// </summary>
    public int StartNumber
    {
        get => _startNumber;
        set
        {
            if (_startNumber == value)
                return;
            _startNumber = value;
            OnPropertyChanged();
        }
    }

    /// <summary>
    /// Префикс
    /// </summary>
    public string Prefix
    {
        get => _prefix;
        set
        {
            if (_prefix == value)
                return;
            _prefix = value;
            OnPropertyChanged();
        }
    }

    /// <inheritdoc />
    public MyConfiguration Clone() => new()
    {
        Id = Id,
        Name = Name,
        Description = Description,
        StartNumber = StartNumber,
        Prefix = Prefix
    };
}
```

Требования к классу конфигурации:

- **Публичный конструктор без параметров** — он нужен и для создания новой конфигурации, и для XML-сериализации.
- **Публичные свойства с `get; set;`** — сохраняются только они. Поля, свойства только для чтения и свойства с приватным сеттером в файл не попадут.
- **`OnPropertyChanged()` в сеттерах** — без этого UI не узнает об изменениях, а событие контекста `OnConfigurationPropertyChanged` не сработает.
- **`Clone()` копирует все настройки** — метод используется кнопкой «Дублировать». Забытое в `Clone()` свойство у копии получит значение по умолчанию.

!!! note ""
    `BaseConfiguration` уже содержит `Id`, `Name` и `Description` с уведомлениями об изменениях — объявлять их повторно не нужно.

### Шаг 2. Контекст окна

Контекст наследуется от одного из базовых контекстов и обязан реализовать два метода — `OnLoaded()` и `OnClosed()`. Если никакой дополнительной логики не нужно, их тело может быть пустым.

```csharp
namespace MyPlugin.ViewModels;

using ModPlusAPI.Abstractions;
using ModPlusAPI.Contexts;
using ModPlusStyle.Controls;
using Models;

/// <summary>
/// Контекст главного окна
/// </summary>
public class MainContext : BaseConfigurationsContext<MyConfiguration>
{
    /// <summary>
    /// Initializes a new instance of the <see cref="MainContext"/> class.
    /// </summary>
    /// <param name="connector">Коннектор плагина</param>
    /// <param name="parentWindow">Окно плагина</param>
    public MainContext(IModPlusPluginGeneral connector, ModPlusWindow parentWindow)
        : base(connector, parentWindow)
    {
    }

    /// <inheritdoc />
    public override void OnLoaded()
    {
        // Конфигурации уже загружены, текущая выбрана
    }

    /// <inheritdoc />
    public override void OnClosed()
    {
        // Вызывается перед сохранением конфигураций
    }
}
```

Для контекста без Настроек организации (ModPlusAPI 42.0+) меняется только базовый класс:

```csharp
public class MainContext : BaseConfigurationsContextWithoutOrgSettings<MyConfiguration>
```

### Шаг 3. Разметка окна

В окне нужно сделать три вещи:

1. **Привязать события окна к командам контекста.** `Loaded` → `OnLoadedCommand` (загрузка конфигураций), `Closed` → `OnClosedCommand` (сохранение). Сам контекст окно не отслеживает: без этих привязок конфигурации не загрузятся и не сохранятся.
2. **Разместить `ConfigurationsControl`.** Он берёт `DataContext` окна и сам привязывается к нужным свойствам и командам.
3. **Привязать элементы настроек к `Configuration.<Свойство>`** — то есть к свойствам текущей выбранной конфигурации.

```xml
<mps:ModPlusWindow x:Class="MyPlugin.Views.MainWindow"
                   xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                   xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                   xmlns:b="https://modplus.org/xaml/behaviors"
                   xmlns:mps="https://modplus.org/winfx/xaml/controls"
                   xmlns:apiControls="clr-namespace:ModPlusAPI.Windows.Controls;assembly=ModPlusAPI"
                   xmlns:apiConverters="clr-namespace:ModPlusAPI.Converters;assembly=ModPlusAPI">
    <mps:ModPlusWindow.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="pack://application:,,,/ModPlusStyle;component/Styles/Controls.xaml" />
            </ResourceDictionary.MergedDictionaries>
            <XmlDataProvider x:Key="LangCommon" XPath="ModPlus/Common" />
            <apiConverters:BooleanToVisibilityConverter x:Key="TrueToVisibleFalseToCollapsed"
                                                        FalseVisibility="Collapsed"
                                                        TrueVisibility="Visible" />
            <apiConverters:BooleanToVisibilityConverter x:Key="TrueToCollapsedFalseToVisible"
                                                        FalseVisibility="Visible"
                                                        TrueVisibility="Collapsed" />
        </ResourceDictionary>
    </mps:ModPlusWindow.Resources>

    <!-- 1. Загрузка и сохранение конфигураций -->
    <b:Interaction.Triggers>
        <b:EventTrigger EventName="Loaded">
            <b:InvokeCommandAction Command="{Binding Path=OnLoadedCommand}" />
        </b:EventTrigger>
        <b:EventTrigger EventName="Closed">
            <b:InvokeCommandAction Command="{Binding Path=OnClosedCommand}" />
        </b:EventTrigger>
    </b:Interaction.Triggers>

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>

        <!-- 2. Список конфигураций и кнопки управления -->
        <apiControls:ConfigurationsControl Grid.Row="0"
                                           Orientation="Horizontal"
                                           ShowHeader="True" />

        <!-- 3. Настройки текущей конфигурации: видны, только если конфигурация выбрана -->
        <StackPanel Grid.Row="1"
                    Visibility="{Binding Path=IsConfigurationSelected, Converter={StaticResource TrueToVisibleFalseToCollapsed}}">
            <TextBox Text="{Binding Path=Configuration.Prefix, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
            <mps:NumericBox Value="{Binding Path=Configuration.StartNumber, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
        </StackPanel>

        <!-- Подсказка, если конфигураций ещё нет -->
        <TextBlock Grid.Row="1"
                   HorizontalAlignment="Center"
                   VerticalAlignment="Center"
                   TextWrapping="Wrap"
                   Text="{Binding Source={StaticResource LangCommon}, XPath=configurationCreateOrImport}"
                   Visibility="{Binding Path=IsConfigurationSelected, Converter={StaticResource TrueToCollapsedFalseToVisible}}" />
    </Grid>
</mps:ModPlusWindow>
```

!!! tip "Первый запуск"
    При первом запуске у пользователя нет ни одной конфигурации, и `Configuration` равен `null`. Поэтому блок настроек скрывается по `IsConfigurationSelected`, а вместо него показывается подсказка «создайте или импортируйте конфигурацию» (строка `configurationCreateOrImport` из общих языковых ресурсов).

### Шаг 4. Установка DataContext

```csharp
namespace MyPlugin.Views;

using ViewModels;

/// <summary>
/// Главное окно плагина
/// </summary>
public partial class MainWindow
{
    /// <summary>
    /// Initializes a new instance of the <see cref="MainWindow"/> class.
    /// </summary>
    public MainWindow()
    {
        InitializeComponent();
        DataContext = new MainContext(ModPlusConnector.Instance, this);
    }
}
```

Готово: при открытии окна конфигурации загрузятся и выберется та, с которой пользователь работал в прошлый раз, а при закрытии всё сохранится автоматически.

---

## Как это работает

### Открытие окна

1. Окно вызывает `OnLoadedCommand` (событие `Loaded`).
2. Контекст читает конфигурации из хранилища и сортирует их по имени.
3. Выбирается конфигурация, имя которой было сохранено при прошлом закрытии окна. Если такой нет — первая в списке. Если конфигураций нет — ничего не выбирается.
4. Вызывается ваш `OnLoaded()`.

Если на шаге 2 или 4 произошла ошибка, пользователь увидит сообщение, а окно закроется **без сохранения** — так файл с конфигурациями не будет перезаписан неполными данными.

### Закрытие окна

1. Окно вызывает `OnClosedCommand` (событие `Closed`).
2. Вызывается ваш `OnClosed()` — в нём можно дописать что-то в текущую конфигурацию или сохранить дополнительные данные.
3. Имя выбранной конфигурации сохраняется в общий файл `mpConfig.mpcf` через `UserConfigFileUtils`.
4. Список конфигураций сохраняется в хранилище (`SaveConfigurations()`).

### Переключение конфигурации

Когда пользователь выбирает другую конфигурацию в списке, сначала срабатывает событие `OnConfigurationChanging` (в аргументе — **предыдущая** конфигурация), затем меняется свойство `Configuration`. Если до этого ни одна конфигурация не была выбрана, событие не срабатывает.

### Имена конфигураций

- Список всегда отсортирован по имени.
- Имена уникальны: при совпадении к имени добавляется номер — `Фасады (1)`, `Фасады (2)`.
- При дублировании и импорте конфигурация получает имя исходной с номером.
- Если создание без запроса имени (`askNameOnConfigurationCreation: false`), новые конфигурации называются «Конфигурация 1», «Конфигурация 2» и т.д.

!!! note ""
    Выбранная конфигурация запоминается **по имени**, поэтому после переименования она всё равно будет выбрана при следующем открытии — имя сохраняется в момент закрытия окна.

---

## Где хранятся данные

| Что | `BaseConfigurationsContext` | `BaseConfigurationsContextWithoutOrgSettings` |
|---|---|---|
| Список конфигураций | Пользовательский файл `UserData\<Имя плагина>.xml` или файл организации, заданный администратором | Всегда пользовательский файл `UserData\<Имя плагина>.xml` |
| Имя выбранной конфигурации | `mpConfig.mpcf` | `mpConfig.mpcf` |
| Экспортированная конфигурация | Отдельный XML-файл, путь выбирает пользователь | То же |

Имя выбранной конфигурации всегда хранится в личном `mpConfig.mpcf`, даже при работе с файлом организации: этот файл может быть открыт только на чтение, а выбор у каждого пользователя свой.

Подробнее о файлах пользовательских настроек — в статье [Работа с пользовательскими настройками](user-settings.md).

---

## Справочник API

### Конструктор контекста

Оба контекста имеют одинаковый конструктор:

```csharp
protected BaseConfigurationsContext(
    IModPlusPluginGeneral connector,
    ModPlusWindow parentWindow,
    bool configurationsWithDescription = false,
    bool askNameOnConfigurationCreation = true)
```

| Параметр | Назначение |
|---|---|
| `connector` | Коннектор плагина. Его `Name` задаёт имя файла хранения и узла в `mpConfig.mpcf` |
| `parentWindow` | Окно плагина. Через него показываются диалоги ввода имени и сообщения |
| `configurationsWithDescription` | `true` — у конфигураций есть описание: оно показывается в выпадающем списке, а диалог создания/переименования запрашивает имя **и** описание |
| `askNameOnConfigurationCreation` | `true` — при создании конфигурации запрашивается имя. `false` — конфигурация создаётся сразу с автоматическим именем |

### Свойства

| Свойство | Тип | Описание |
|---|---|---|
| `Configurations` | `ObservableCollection<TConfiguration>` | Все конфигурации |
| `Configuration` | `TConfiguration` | Текущая выбранная конфигурация, может быть `null` |
| `IsConfigurationSelected` | `bool` | Выбрана ли конфигурация. Удобно для `Visibility`/`IsEnabled` |
| `HasConfigurations` | `bool` | Есть ли хотя бы одна конфигурация |
| `ConfigurationsWithDescription` | `bool` | Значение одноимённого параметра конструктора |
| `ParentWindow` | `ModPlusWindow` | Окно плагина |
| `UserConfigFileUtils` | `UserConfigFileUtils` | Доступ к личному файлу `mpConfig.mpcf` в узле плагина |
| `StorageFile` | `OrgSettingsService` / `UserSettingsService` | Сервис файла, в котором хранятся конфигурации (тип зависит от контекста) |

`UserConfigFileUtils` и `StorageFile` можно использовать не только для конфигураций, но и для любых других настроек плагина, например в `OnLoaded()`/`OnClosed()`:

- `UserConfigFileUtils` — для простых значений, которые не входят в конфигурацию: состояние окна, последний выбранный элемент и т.п.
- `StorageFile` — для дополнительных данных, которые должны храниться рядом с конфигурациями. В `BaseConfigurationsContext` это `OrgSettingsService`: методы `Save(data, name)`/`Load<T>(name)`, данные подчиняются Настройкам организации. В `BaseConfigurationsContextWithoutOrgSettings` это `UserSettingsService`: методы `Set(data, name)`/`Get<T>(name)`, данные всегда в пользовательском файле.

!!! warning ""
    Не используйте в `StorageFile` имя узла `Configurations` — под этим именем хранится сам список конфигураций.

### События

| Событие | Когда срабатывает |
|---|---|
| `OnConfigurationChanging` | Перед сменой выбранной конфигурации. Аргумент — предыдущая конфигурация |
| `OnConfigurationPropertyChanged` | При изменении любого свойства текущей конфигурации. `sender` — конфигурация, `e.PropertyName` — имя свойства |

### Абстрактные методы

| Метод | Когда вызывается | Для чего |
|---|---|---|
| `OnLoaded()` | После загрузки конфигураций и выбора текущей | Инициализация: загрузка дополнительных данных, подписки на события |
| `OnClosed()` | При закрытии окна, **до** сохранения конфигураций | Финализация: дописать данные в конфигурацию, сохранить дополнительные настройки. Не вызывается, если при загрузке произошла ошибка |

### Публичные методы

| Метод | Описание |
|---|---|
| `AddNewConfiguration(string baseName, string description)` | Добавляет конфигурацию, делает её текущей и сортирует список. Пустое `baseName` — автоматическое имя «Конфигурация N» |
| `RenameConfiguration(string baseName, string description)` | Переименовывает текущую конфигурацию с учётом уникальности имени |
| `SaveConfigurations()` | Сохраняет список конфигураций немедленно, не дожидаясь закрытия окна |

### Команды

Команды уже привязаны в `ConfigurationsControl`, но их можно использовать и в собственной разметке — например, если нужен нестандартный UI.

| Команда | Действие |
|---|---|
| `OnLoadedCommand` | Загрузка конфигураций — привязывается к `Loaded` окна |
| `OnClosedCommand` | Сохранение конфигураций — привязывается к `Closed` окна |
| `AddNewConfigurationCommand` | Создать конфигурацию |
| `RenameConfigurationCommand` | Переименовать текущую |
| `CloneConfigurationCommand` | Дублировать текущую |
| `ExportCurrentConfigurationCommand` | Экспортировать текущую в XML-файл |
| `ImportConfigurationCommand` | Импортировать конфигурацию из XML-файла. Файл без подходящих данных отклоняется с сообщением |
| `RemoveCurrentConfigurationCommand` | Удалить текущую |

---

## ConfigurationsControl

| Свойство | Значение по умолчанию | Описание |
|---|---|---|
| `Orientation` | `Horizontal` | `Horizontal` — заголовок, список и кнопки в одну строку. `Vertical` — друг под другом, ширина контрола фиксирована (компактный вариант для боковых панелей) |
| `ShowHeader` | `True` | Показывать заголовок «Конфигурация:» |

```xml
<apiControls:ConfigurationsControl Orientation="Vertical"
                                   ShowHeader="False" />
```

При работе с `BaseConfigurationsContext` кнопки создания, переименования, дублирования, импорта и удаления автоматически блокируются, если у пользователя нет прав на редактирование файла организации.

---

## Частые ошибки

**Конфигурации не загружаются или не сохраняются.**
Не привязаны события окна `Loaded`/`Closed` к `OnLoadedCommand`/`OnClosedCommand` (шаг 3).

**Изменения в полях не отражаются или не сохраняются.**
В сеттерах свойств конфигурации нет `OnPropertyChanged()`, либо свойство не публичное или без публичного сеттера.

**У дубликата часть настроек сброшена.**
Новое свойство не добавлено в `Clone()`.

**Сборка падает с ошибкой «тип `BaseConfigurationsContextWithoutOrgSettings` не найден».**
В проекте подключён ModPlusAPI ниже 42.0 — обновите версию пакета.

**Файл с конфигурациями вдруг стал общим для всех пользователей.**
Для плагина заданы Настройки организации, а используется `BaseConfigurationsContext`. Если плагину это не нужно — используйте `BaseConfigurationsContextWithoutOrgSettings`.
