# Конфигурации плагина

Для использования необходимо подключить nuget-пакет ModPlusAPI версии не ниже 44.0 (если в пакете модуля версия ниже)!

```<PackageReference Include="ModPlusAPI" Version="44.0.0" ExcludeAssets="runtime" />```

!!! warning "`BaseConfigurationsContext` и `BaseConfigurationsContextWithoutOrgSettings` устарели"
    В ModPlusAPI появились новые контексты `ConfigurationsContext` и `ConfigurationsContextWithoutOrgSettings`. Старые контексты помечены атрибутом `[Obsolete]` и оставлены только для совместимости: в них есть ошибки, которые нельзя исправить без изменения поведения уже написанных плагинов. В новых плагинах используйте только новые контексты, а существующие переводите по разделу [Переход со старых контекстов](#переход-со-старых-контекстов).

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
| `ConfigurationsContext<TConfiguration>` | Контекст (ViewModel) окна: загрузка, сохранение, переключение, команды для кнопок. **Учитывает Настройки организации** | Разработчик наследует свой контекст |
| `ConfigurationsContextWithoutOrgSettings<TConfiguration>` | То же самое, но **без учёта Настроек организации** | Разработчик наследует свой контекст |
| `ConfigurationsContextCore<TConfiguration>` | Общая основа двух контекстов выше. Напрямую наследоваться от неё нельзя | — |
| `ConfigurationsControl` | Готовый UI: заголовок, список конфигураций, кнопки | Просто размещается в XAML окна |

Пространства имён: `ModPlusAPI.Models` (`BaseConfiguration`), `ModPlusAPI.Abstractions` (`ICloneable<T>`), `ModPlusAPI.Contexts` (контексты), `ModPlusAPI.Windows.Controls` (`ConfigurationsControl`). Сборка: `ModPlusAPI`.

---

## Какой контекст выбрать

Оба контекста имеют одинаковый набор свойств, событий, команд и методов, одинаковый порядок вызова методов и работают с одним и тем же `ConfigurationsControl`. Отличаются они только тем, **откуда читаются и куда сохраняются конфигурации**.

| | `ConfigurationsContext` | `ConfigurationsContextWithoutOrgSettings` |
|---|---|---|
| Настройки организации | Учитываются | Не учитываются |
| Где хранятся конфигурации | В пользовательском файле или в файле организации (если он задан администратором) | Всегда в пользовательском файле |
| Тип свойства `StorageFile` | `OrgSettingsService` | `UserSettingsService` |
| Блокировка файла на время работы | Да, для файла организации. Снимается после закрытия окна | Нет |
| Запрет редактирования | Возможен, если у пользователя нет прав на файл организации | Нет |
| Переключение между файлами конфигураций | Возможно, если администратор задал несколько файлов | Нет |
| Выбранная конфигурация запоминается | Отдельно для каждого файла | Одна на плагин |

**Как выбрать:**

- Конфигурации плагина имеет смысл централизованно раздавать сотрудникам организации (стандарты оформления, шаблоны, правила именования) → `ConfigurationsContext`.
- Конфигурации — личное дело каждого пользователя, и Настройки организации для этого плагина не нужны → `ConfigurationsContextWithoutOrgSettings`.

!!! tip "Смена контекста не теряет данные пользователей"
    Если Настройки организации не заданы, `ConfigurationsContext` хранит конфигурации в том же файле и том же узле, что и `ConfigurationsContextWithoutOrgSettings`: `<папка пользовательских настроек ModPlus>\UserData\<Имя плагина>.xml`, узел `Configurations`. Те же файл и узел использовали и устаревшие контексты. Поэтому при переходе плагина с одного контекста на другой — в том числе со старых на новые — ранее созданные пользователем конфигурации сохраняются, миграция не нужна.

---

## Быстрый старт

Ниже — минимальный рабочий пример из четырёх шагов. Пример использует `ConfigurationsContext`; для контекста без Настроек организации меняется только базовый класс в шаге 2.

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

Контекст наследуется от одного из базовых контекстов и обязан реализовать пять методов. Если никакой дополнительной логики не нужно, их тело может быть пустым. Когда и зачем вызывается каждый метод — в разделе [Как это работает](#как-это-работает).

```csharp
namespace MyPlugin.ViewModels;

using ModPlusAPI.Abstractions;
using ModPlusAPI.Contexts;
using ModPlusStyle.Controls;
using Models;

/// <summary>
/// Контекст главного окна
/// </summary>
public class MainContext : ConfigurationsContext<MyConfiguration>
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
        // Один раз при открытии окна, ДО загрузки конфигураций:
        // подписка на внешние события, сбор данных документа
    }

    /// <inheritdoc />
    public override void PrepareConfiguration(MyConfiguration configuration)
    {
        // Для каждой конфигурации, попадающей в список:
        // миграция старого формата, заполнение данных из документа
    }

    /// <inheritdoc />
    public override void OnConfigurationsLoaded()
    {
        // После каждой загрузки конфигураций, текущая уже выбрана:
        // конфигурация по умолчанию, если список пуст; обновление UI
    }

    /// <inheritdoc />
    public override void OnConfigurationsSaving()
    {
        // Перед каждым сохранением конфигураций:
        // перенос в конфигурации состояния UI, не связанного привязками
    }

    /// <inheritdoc />
    public override void OnClosed()
    {
        // Один раз при закрытии окна, ПОСЛЕ сохранения конфигураций:
        // отписка от событий, освобождение ресурсов
    }
}
```

Для контекста без Настроек организации меняется только базовый класс:

```csharp
public class MainContext : ConfigurationsContextWithoutOrgSettings<MyConfiguration>
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
    При первом запуске у пользователя нет ни одной конфигурации, и `Configuration` равен `null`. Поэтому блок настроек скрывается по `IsConfigurationSelected`, а вместо него показывается подсказка «создайте или импортируйте конфигурацию» (строка `configurationCreateOrImport` из общих языковых ресурсов). Другой вариант — создать конфигурацию по умолчанию в `OnConfigurationsLoaded()`, см. [пример ниже](#конфигурация-по-умолчанию).

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

Жизненный цикл контекста разделён на две независимые пары методов:

- **`OnLoaded()` / `OnClosed()` — жизнь окна.** Вызываются ровно по одному разу: при открытии и при закрытии окна. Предназначены для подписки на внешние события (приложения, документа) и освобождения ресурсов.
- **`OnConfigurationsLoaded()` / `OnConfigurationsSaving()` — жизнь набора конфигураций.** Вызываются при каждой загрузке и каждом сохранении конфигураций, в том числе при смене пользователем файла конфигураций в `ConfigurationsContext`.

Отдельно стоит `PrepareConfiguration(configuration)` — он вызывается для каждой конфигурации, попадающей в список.

Базовый класс сам выполняет только загрузку, выбор и сохранение конфигураций и сохранение имени выбранной конфигурации. Всё остальное — подписки, освобождение ресурсов, перенос состояния UI в конфигурации — делает плагин в своих реализациях этих методов.

### Открытие окна

1. Окно вызывает `OnLoadedCommand` (событие `Loaded`).
2. Вызывается ваш `OnLoaded()`. **Конфигурации ещё не загружены:** список `Configurations` пуст, `Configuration` равен `null`.
3. Контекст читает конфигурации из хранилища, сортирует их по имени и добавляет в список. Для каждой вызывается ваш `PrepareConfiguration()`.
4. Выбирается конфигурация, имя которой было сохранено в прошлый раз. Если такой нет — первая в списке. Если конфигураций нет — ничего не выбирается.
5. Вызывается ваш `OnConfigurationsLoaded()`. Если после него конфигурации есть, а выбранной нет (например, вы создали конфигурации, не выбрав ни одну), выбирается первая.

`OnLoadedCommand` выполняется один раз: повторное событие `Loaded` (например, при повторном включении окна в визуальное дерево) игнорируется, `OnLoaded()` гарантированно вызывается один раз, а конфигурации из источника заново не загружаются. Если данные, подготовленные в `PrepareConfiguration()`, зависят от документа, при смене документа плагин обновляет их сам.

Сохранять конфигурации явно через `SaveConfigurations()` можно уже в `OnConfigurationsLoaded()` — например, сразу после переноса старых настроек. Автоматическое сохранение при закрытии окна и смене файла включается только после успешного выполнения `OnConfigurationsLoaded()`.

### Закрытие окна

1. Окно вызывает `OnClosedCommand` (событие `Closed`).
2. Вызывается ваш `OnConfigurationsSaving()` — перенесите в конфигурации всё, что не связано с ними привязками.
3. Имя выбранной конфигурации сохраняется в личный файл `mpConfig.mpcf` через `UserConfigFileUtils`.
4. Список конфигураций сохраняется в хранилище (`SaveConfigurations()`).
5. Вызывается ваш `OnClosed()`. **Конфигурации к этому моменту уже сохранены**: изменения, внесённые в них здесь, в файл не попадут.
6. `ConfigurationsContext` освобождает `StorageFile` — снимается блокировка файла организации.

Каждый шаг выполняется отдельно: ошибка при сохранении не помешает вызову `OnClosed()` и освобождению файла. `OnClosedCommand`, как и `OnLoadedCommand`, выполняется один раз — повторные вызовы игнорируются.

### Смена файла конфигураций

Только для `ConfigurationsContext`, если администратор задал несколько файлов конфигураций и пользователь переключил файл в открытом окне. Окно при этом остаётся открытым, поэтому `OnLoaded()` и `OnClosed()` **не вызываются**:

1. Вызывается ваш `OnConfigurationsSaving()` — для старого файла.
2. `Configuration` сбрасывается в `null` (срабатывает `OnConfigurationChanging` для текущей конфигурации старого файла).
3. Конфигурации сохраняются в старый файл.
4. Конфигурации загружаются из нового файла — так же, как в шагах 3–5 открытия окна: `PrepareConfiguration()` для каждой, выбор сохранённой, `OnConfigurationsLoaded()`.

Для `ConfigurationsContextWithoutOrgSettings` источник один и не меняется, поэтому `OnConfigurationsLoaded()` и `OnConfigurationsSaving()` вызываются ровно по одному разу — при открытии и закрытии окна. Контракт методов при этом тот же, поэтому плагин переводится с одного контекста на другой без изменения логики.

### Ошибки

| Где произошла ошибка | Что происходит |
|---|---|
| `OnLoaded()` | Сообщение об ошибке, окно закрывается. Конфигурации не загружаются и не сохраняются, `OnClosed()` **не вызывается** |
| Чтение конфигураций, `PrepareConfiguration()`, `OnConfigurationsLoaded()` | Сообщение об ошибке, окно закрывается **без сохранения** конфигураций — файл не будет перезаписан неполными данными. `OnClosed()` вызывается, так как `OnLoaded()` выполнился успешно |
| `OnConfigurationsSaving()`, сохранение | Сообщение об ошибке. `OnClosed()` всё равно вызывается, файл организации освобождается |

`SaveConfigurations()` тоже защищён: до завершения загрузки списка (в том числе в `OnLoaded()` и `PrepareConfiguration()`) и после ошибки чтения конфигураций вызов ничего не делает — пустой или частично заполненный список не затрёт сохранённые настройки. Начиная с `OnConfigurationsLoaded()` метод работает.

### Переключение конфигурации

Когда пользователь выбирает другую конфигурацию в списке:

1. Срабатывает событие `OnConfigurationChanging` — в аргументе **предыдущая** конфигурация. Если до этого ни одна конфигурация не была выбрана, событие не срабатывает.
2. Меняется свойство `Configuration`.
3. Имя новой конфигурации сразу записывается в `mpConfig.mpcf` — плагину делать это не нужно.
4. Срабатывает событие `OnConfigurationChanged` — в аргументе **новая** конфигурация (может быть `null`).

### Добавление конфигураций

`PrepareConfiguration()` вызывается для каждой конфигурации, попавшей в список **любым способом**: загрузка, создание, дублирование, импорт, а также добавление из кода в `Configurations`. Поэтому миграцию старого формата и заполнение данных, зависящих от документа, достаточно написать в одном месте.

!!! warning ""
    `PrepareConfiguration()` вызывается из обработчика `CollectionChanged`, поэтому изменять коллекцию `Configurations` внутри него нельзя. Конфигурация в момент вызова уже находится в списке, но может быть ещё не выбрана.

### Удаление конфигурации

После удаления текущей конфигурации выбирается предыдущая в списке. После удаления последней конфигурации `Configuration` становится `null`.

### Импорт

Конфигурация добавляется в список только после успешного чтения файла. Если файл не читается или в нём нет конфигурации нужного типа, пользователь видит сообщение, а список не меняется.

### Имена конфигураций

- Список всегда отсортирован по имени.
- Имена уникальны: при совпадении к имени добавляется номер — `Фасады (1)`, `Фасады (2)`.
- При дублировании и импорте конфигурация получает имя исходной с номером.
- Если создание без запроса имени (`askNameOnConfigurationCreation: false`), новые конфигурации называются «Конфигурация 1», «Конфигурация 2» и т.д.

!!! note ""
    Выбранная конфигурация запоминается **по имени**. Имя сохраняется при выборе, при переименовании и ещё раз при закрытии окна (на случай, если имя изменили через привязку к `Configuration.Name`), поэтому после переименования конфигурация всё равно будет выбрана при следующем открытии.

---

## Примеры

### Подписка на события документа

Подписки делаются в `OnLoaded()`, а снимаются в `OnClosed()` — базовый класс этого не делает.

```csharp
/// <inheritdoc />
public override void OnLoaded()
{
    _application.DocumentChanged += OnDocumentChanged;
}

/// <inheritdoc />
public override void OnClosed()
{
    _application.DocumentChanged -= OnDocumentChanged;
}
```

### Конфигурация по умолчанию

```csharp
/// <inheritdoc />
public override void OnConfigurationsLoaded()
{
    if (!HasConfigurations)
        AddNewConfiguration(string.Empty, string.Empty);
}
```

`AddNewConfiguration()` сам делает новую конфигурацию текущей. В `ConfigurationsContext` метод вызывается и после смены файла конфигураций, так что пустой файл тоже получит конфигурацию по умолчанию.

### Перенос старых настроек

Если раньше плагин хранил настройки без конфигураций, их можно один раз перенести в конфигурацию в `OnConfigurationsLoaded()` и сразу сохранить, не дожидаясь закрытия окна:

```csharp
/// <inheritdoc />
public override void OnConfigurationsLoaded()
{
    if (HasConfigurations)
        return;

    AddNewConfiguration(string.Empty, string.Empty);
    Configuration.Prefix = UserConfigFileUtils.GetValue("Prefix");

    SaveConfigurations();
}
```

### Состояние UI, не связанное привязками

Если часть состояния окна хранится в контексте (например, отмеченные элементы списка), а не в свойствах конфигурации, переносите его в конфигурацию в `OnConfigurationsSaving()` — и при закрытии окна, и перед сменой файла конфигураций. Если состояние должно переноситься и при переключении конфигураций, используйте событие `OnConfigurationChanging`: оно срабатывает также при смене файла, причём до сохранения.

```csharp
/// <inheritdoc />
public override void OnConfigurationsSaving()
{
    if (Configuration != null)
        Configuration.SelectedCategories = SelectedCategories.Select(c => c.Id).ToList();
}
```

### Получение конфигурации без показа окна

`OnLoadedCommand` можно выполнить вручную, не показывая окно, — например, чтобы получить выбранную пользователем конфигурацию и выполнить команду плагина с её настройками:

```csharp
var context = new MainContext(ModPlusConnector.Instance, new MainWindow());
context.OnLoadedCommand.Execute(null);

var configuration = context.Configuration;
```

Окно не показывается, поэтому событие `Closed` не наступает: автоматического сохранения конфигураций и вызова `OnClosed()` не будет. Если они нужны (например, чтобы отписаться от событий, на которые подписался `OnLoaded()`), выполните `OnClosedCommand` вручную. Для `ConfigurationsContext` это же действие снимает блокировку файла организации — без него файл останется заблокирован.

---

## Где хранятся данные

| Что | `ConfigurationsContext` | `ConfigurationsContextWithoutOrgSettings` |
|---|---|---|
| Список конфигураций | Пользовательский файл `UserData\<Имя плагина>.xml` или файл организации, заданный администратором | Всегда пользовательский файл `UserData\<Имя плагина>.xml` |
| Имя выбранной конфигурации | `mpConfig.mpcf`, отдельно для пользовательского файла и для каждого файла организации | `mpConfig.mpcf` |
| Экспортированная конфигурация | Отдельный XML-файл, путь выбирает пользователь | То же |

Имя выбранной конфигурации всегда хранится в личном `mpConfig.mpcf`, даже при работе с файлом организации: этот файл может быть открыт только на чтение, а выбор у каждого пользователя свой.

В `ConfigurationsContext` выбор запоминается отдельно для каждого источника: при переключении между пользовательским файлом и файлами организации у каждого восстанавливается своя выбранная конфигурация. Для пользовательского файла используется тот же ключ, что и в устаревших контекстах, поэтому выбор пользователя при переходе на новый контекст сохраняется.

Подробнее о файлах пользовательских настроек — в статье [Работа с пользовательскими настройками](user-settings.md).

---

## Справочник API

### Конструктор контекста

Оба контекста имеют одинаковый конструктор:

```csharp
protected ConfigurationsContext(
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

`UserConfigFileUtils` и `StorageFile` можно использовать не только для конфигураций, но и для любых других настроек плагина:

- `UserConfigFileUtils` — для простых значений, которые не входят в конфигурацию: состояние окна, последний выбранный элемент и т.п.
- `StorageFile` — для дополнительных данных, которые должны храниться рядом с конфигурациями. В `ConfigurationsContext` это `OrgSettingsService`: методы `Save(data, name)`/`Load<T>(name)`, данные подчиняются Настройкам организации. В `ConfigurationsContextWithoutOrgSettings` это `UserSettingsService`: методы `Set(data, name)`/`Get<T>(name)`, данные всегда в пользовательском файле.

!!! warning ""
    Не используйте в `StorageFile` имя узла `Configurations` — под этим именем хранится сам список конфигураций. В `ConfigurationsContext` не обращайтесь к `StorageFile` в `OnClosed()`: сразу после него файл освобождается, и данные, которые должны попасть в файл организации, сохраняйте в `OnConfigurationsSaving()`.

### События

| Событие | Когда срабатывает |
|---|---|
| `OnConfigurationChanging` | Перед сменой выбранной конфигурации. Аргумент — предыдущая конфигурация. Не срабатывает, если текущей конфигурации не было. При смене файла конфигураций срабатывает до сохранения, поэтому изменения, внесённые в обработчике, будут сохранены |
| `OnConfigurationChanged` | После смены выбранной конфигурации (после `PropertyChanged` для `Configuration`). Аргумент — новая конфигурация, может быть `null` |
| `OnConfigurationPropertyChanged` | При изменении любого свойства текущей конфигурации. `sender` — конфигурация, `e.PropertyName` — имя свойства |

### Абстрактные методы

| Метод | Когда вызывается | Для чего |
|---|---|---|
| `OnLoaded()` | Один раз при открытии окна, **до** загрузки конфигураций | Подписка на внешние события, сбор данных, нужных в `PrepareConfiguration()` |
| `PrepareConfiguration(TConfiguration)` | Для каждой конфигурации, добавленной в список: загрузка, создание, дублирование, импорт | Миграция старого формата, заполнение данных, зависящих от документа. Изменять `Configurations` здесь нельзя |
| `OnConfigurationsLoaded()` | После каждой загрузки конфигураций: при открытии окна и после смены файла. Текущая конфигурация уже выбрана | Конфигурация по умолчанию для пустого списка, перенос старых настроек (с явным `SaveConfigurations()`), обновление UI, зависящего от набора конфигураций |
| `OnConfigurationsSaving()` | Перед каждым сохранением: при закрытии окна и перед сменой файла. Не вызывается, если загрузка завершилась ошибкой | Перенос в конфигурации состояния UI, не связанного привязками |
| `OnClosed()` | Один раз при закрытии окна, **после** сохранения конфигураций. Только если `OnLoaded()` выполнился без ошибок | Отписка от событий, освобождение ресурсов, захваченных в `OnLoaded()` |

### Публичные методы

| Метод | Описание |
|---|---|
| `AddNewConfiguration(string baseName, string description)` | Добавляет конфигурацию, делает её текущей и сортирует список. Пустое `baseName` — автоматическое имя «Конфигурация N» |
| `RenameConfiguration(string baseName, string description)` | Переименовывает текущую конфигурацию с учётом уникальности имени и обновляет сохранённый выбор |
| `SaveConfigurations()` | Сохраняет список конфигураций немедленно, не дожидаясь закрытия окна. Работает начиная с `OnConfigurationsLoaded()`; раньше (в `OnLoaded()`, `PrepareConfiguration()`) и после ошибки чтения конфигураций ничего не делает |

### Команды

Команды уже привязаны в `ConfigurationsControl`, но их можно использовать и в собственной разметке — например, если нужен нестандартный UI.

| Команда | Действие |
|---|---|
| `OnLoadedCommand` | Загрузка конфигураций — привязывается к `Loaded` окна. Выполняется один раз, можно вызвать вручную без показа окна |
| `OnClosedCommand` | Сохранение конфигураций — привязывается к `Closed` окна. Выполняется один раз |
| `AddNewConfigurationCommand` | Создать конфигурацию |
| `RenameConfigurationCommand` | Переименовать текущую |
| `CloneConfigurationCommand` | Дублировать текущую |
| `ExportCurrentConfigurationCommand` | Экспортировать текущую в XML-файл. Недоступна, если конфигурация не выбрана |
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

При работе с `ConfigurationsContext` кнопки создания, переименования, дублирования, импорта и удаления автоматически блокируются, если у пользователя нет прав на редактирование файла организации.

---

## Переход со старых контекстов

Старые контексты остались в ModPlusAPI и продолжают работать, но при сборке компилятор выдаёт предупреждение `CS0618` с указанием, на что заменить класс. Причины, по которым они заменены:

- **`BaseConfigurationsContext`** — `OnLoaded()` и `OnClosed()` вызывались не только при открытии и закрытии окна, но и при каждой смене файла конфигураций. Подписки, сделанные в `OnLoaded()`, дублировались, а ресурсы, освобождённые в `OnClosed()`, пропадали при открытом окне.
- **`BaseConfigurationsContextWithoutOrgSettings`** — не было обработки конфигураций, добавляемых при создании, дублировании и импорте, а после удаления последней конфигурации выбранной оставалась удалённая.

Данные пользователей переносить не нужно: новые контексты читают те же файлы и узлы.

### Порядок перехода

1. Обновите ModPlusAPI до 44.0.
2. Замените базовый класс:

    | Было | Стало |
    |---|---|
    | `BaseConfigurationsContext<T>` | `ConfigurationsContext<T>` |
    | `BaseConfigurationsContextWithoutOrgSettings<T>` | `ConfigurationsContextWithoutOrgSettings<T>` |

3. Разнесите код старых `OnLoaded()` и `OnClosed()` по новым методам. Главное отличие: в старом контексте `OnLoaded()` вызывался **после** загрузки конфигураций, а `OnClosed()` — **до** сохранения. В новом — наоборот.

    | Код в старом контексте | Куда перенести |
    |---|---|
    | `OnLoaded()`: подписки на события, сбор данных документа | `OnLoaded()` |
    | `OnLoaded()`: обработка каждой загруженной конфигурации (миграция, заполнение данных) | `PrepareConfiguration()` |
    | `OnLoaded()`: работа с `Configurations`/`Configuration` (конфигурация по умолчанию, обновление UI) | `OnConfigurationsLoaded()` |
    | `OnClosed()`: запись данных в конфигурации, `StorageFile.Save(...)`/`Set(...)` | `OnConfigurationsSaving()` |
    | `OnClosed()`: отписка от событий, освобождение ресурсов | `OnClosed()` |

4. Удалите из кода плагина ручную запись имени выбранной конфигурации в `UserConfigFileUtils`, если она была, — новый контекст делает это сам.
5. Проверьте сценарии: первый запуск без конфигураций, создание, дублирование, импорт, удаление последней конфигурации, закрытие и повторное открытие окна. Для `ConfigurationsContext` — ещё и переключение файла конфигураций, если для плагина заданы Настройки организации.

!!! warning "Код в `OnLoaded()`, обращающийся к `Configuration`"
    Это самая частая ошибка при переходе. В новом `OnLoaded()` конфигурации ещё не загружены, `Configuration` равен `null`, и код, перенесённый без изменений, либо упадёт с `NullReferenceException`, либо молча ничего не сделает. Такой код должен быть в `OnConfigurationsLoaded()`.

---

## Частые ошибки

**Конфигурации не загружаются или не сохраняются.**
Не привязаны события окна `Loaded`/`Closed` к `OnLoadedCommand`/`OnClosedCommand` (шаг 3).

**Изменения в полях не отражаются или не сохраняются.**
В сеттерах свойств конфигурации нет `OnPropertyChanged()`, либо свойство не публичное или без публичного сеттера.

**Изменения, внесённые в `OnClosed()`, не сохраняются.**
`OnClosed()` вызывается после сохранения. Переносите данные в конфигурации в `OnConfigurationsSaving()`.

**В `OnLoaded()` `Configuration` равен `null`.**
Так и должно быть: `OnLoaded()` вызывается до загрузки конфигураций. Код, работающий с конфигурациями, перенесите в `OnConfigurationsLoaded()`.

**У дубликата часть настроек сброшена.**
Новое свойство не добавлено в `Clone()`.

**Сборка падает с ошибкой «тип `ConfigurationsContext` не найден».**
В проекте подключён ModPlusAPI ниже 44.0 — обновите версию пакета.

**Предупреждение `CS0618` о `BaseConfigurationsContext` или `BaseConfigurationsContextWithoutOrgSettings`.**
Используется устаревший контекст — переведите плагин на новый по разделу [Переход со старых контекстов](#переход-со-старых-контекстов).

**Файл с конфигурациями вдруг стал общим для всех пользователей.**
Для плагина заданы Настройки организации, а используется `ConfigurationsContext`. Если плагину это не нужно — используйте `ConfigurationsContextWithoutOrgSettings`.
