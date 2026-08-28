# Конвертеры

Конвертеры предназначены для использования в привязках WPF (XAML) для преобразования значений перед отображением или управлением визуальными свойствами. Все конвертеры реализуют `IValueConverter` либо `IMultiValueConverter` и могут быть объявлены как ресурсы.

> Хотя конвертеры и относятся к UX/UI, располагаются они в библиотеке ModPlusAPI!

В примерах ниже предполагается, что:

```xml
<Window ...
        xmlns:conv="clr-namespace:ModPlusAPI.Converters;assembly=ModPlusAPI">
    <Window.Resources>
        <conv:BooleanToOpacityConverter x:Key="BoolToOpacity"
                                        TrueOpacity="1.0"
                                        FalseOpacity="0.3" />
        <conv:BooleanToVisibilityConverter x:Key="BoolToVisibility"
                                           TrueVisibility="Visible"
                                           FalseVisibility="Collapsed" />
        <conv:BooleanInverseConverter x:Key="BoolInverse" />
        <conv:EnumToIntConverter x:Key="EnumToInt" />
        <conv:EnumDisplayNameConverter x:Key="EnumDisplayName" />
        <conv:MultiBooleanConverter x:Key="MultiBool" />
    </Window.Resources>
    ...
</Window>
```

---

## 1) BooleanToOpacityConverter

### Назначение

Преобразует `bool` в прозрачность (`double` для свойства `Opacity`). Удобно, когда нужно не «рубить» видимость (`Visibility`), а мягко приглушать элемент.

### Свойства

* `TrueOpacity: double` — значение для `true` (по умолчанию `1.0`)
* `FalseOpacity: double` — значение для `false` (по умолчанию `0.3`)

### Поведение

* `Convert`: `true → TrueOpacity`, `false → FalseOpacity`. Если вход не `bool`, возвращает исходное значение без изменений.
* `ConvertBack`: если вход — `double`, сравнивает его с `TrueOpacity`/`FalseOpacity` с допуском `0.01` и возвращает `bool`. Иначе — возвращает вход.

> Практика: держите `FalseOpacity` в диапазоне `0.2–0.5` — элемент остаётся видимым, но «приглушён».

### Примеры

```xml
<!-- При IsValid = true полностью видим, иначе приглушён -->
<TextBlock Text="Данные валидны"
           Opacity="{Binding IsValid, Converter={StaticResource BoolToOpacity}}" />
```

---

## 2) BooleanToVisibilityConverter

### Назначение

Преобразует `bool` в `Visibility`. Основной способ «показывать/скрывать» элементы.

### Свойства

* `TrueVisibility: Visibility` — значение для `true` (по умолчанию `Visible`)
* `FalseVisibility: Visibility` — значение для `false` (по умолчанию `Collapsed`)

> Если нужна «пустая зона» вместо исчезновения, используйте `FalseVisibility="Hidden"`.

### Поведение

* `Convert`: `true → TrueVisibility`, `false → FalseVisibility`. Не-`bool` возвращается как есть.
* `ConvertBack`: `Visibility` → `bool` (сравнение с `TrueVisibility`/`FalseVisibility`). Иначе — возвращает вход.

### Примеры

```xml
<!-- Кнопка отображается только когда можно сохранить -->
<Button Content="Сохранить"
        Visibility="{Binding CanSave, Converter={StaticResource BoolToVisibility}}" />
```

---

## 3) BooleanInverseConverter

### Назначение

Инвертирует булево значение. Полезно, когда логика «обратная» и плодить лишние свойства во ViewModel не хочется.

### Поведение

* `Convert`/`ConvertBack`: `true ↔ false`. Если вход не `bool`, возвращается вход.

### Примеры

```xml
<!-- Заблокировать кнопку, пока IsBusy = true -->
<Button Content="Выполнить"
        IsEnabled="{Binding IsBusy, Converter={StaticResource BoolInverse}}" />
```

---

## 4) MultiBooleanConverter (мульти-привязка)

### Назначение

Агрегирует несколько `bool` и возвращает `true`, только если **все** они `true`. Любое `false` — результат `false`.

### Поведение

* `Convert(object[] values, ...)`: проходит по массиву и если встречает `bool false` — возвращает `false`.
  `null` и значения, не являющиеся `bool`, **игнорируются** (то есть не делают результат `false`).
* Обратного преобразования нет (исключение `NotImplementedException`).

### Примеры

```xml
<!-- Кнопка включена, только если два условия одновременно истинны -->
<Button Content="Далее">
    <Button.IsEnabled>
        <MultiBinding Converter="{StaticResource MultiBool}">
            <Binding Path="IsFormValid"/>
            <Binding Path="IsNotBusy"/>
        </MultiBinding>
    </Button.IsEnabled>
</Button>
```

> Лайфхак: если какие-то из условий могут быть `null`, это **не** сломает конвертер — они просто не повлияют на итог.

---

## 5) EnumToIntConverter (двусторонний)

### Назначение

Двустороннее преобразование `Enum` ↔ `int`. Полезно для:

* биндинга к `SelectedIndex` или числовым настройкам
* сериализации числовых значений в настройках

### Требование

В `ConverterParameter` нужно передать **тип перечисления** (`System.Type`).

### Поведение

* `Convert`: `Enum` → `int` через `Enum.Parse` по имени (используется `parameter` как тип).
* `ConvertBack`: `int` → `Enum` (`Enum.Parse` по числу). Если параметр не передан/тип не совпадает — возвращает вход.

> В XAML тип удобно передать как `ConverterParameter={x:Type local:MyEnum}`.

### Примеры

```xml
<!-- Привязываем ComboBox.SelectedIndex к значению enum -->
<ComboBox ItemsSource="{Binding Source={x:Static local:MyEnum.GetValues}}"
          SelectedIndex="{Binding SelectedMode,
                                  Converter={StaticResource EnumToInt},
                                  ConverterParameter={x:Type local:MyEnum},
                                  Mode=TwoWay}" />
```

```csharp
// ViewModel
public MyEnum SelectedMode { get; set; }
public enum MyEnum { None = 0, Fast = 1, Safe = 2 }
```

> Частая ошибка: забыть `ConverterParameter`. Тогда парсинг не знает, к какому типу enum приводить.

---

## 6) EnumDisplayNameConverter

### Назначение

Показывает для элементов `Enum` локализованные/человекочитаемые названия, заданные атрибутом `System.ComponentModel.DataAnnotations.DisplayAttribute` (свойство `Name`). Также умеет искать элемент по строке при обратном преобразовании.

### Подготовка перечисления

```csharp
using System.ComponentModel.DataAnnotations;

public enum OperationMode
{
    [Display(Name = "Быстрый режим")]
    Fast = 0,

    [Display(Name = "Безопасный режим")]
    Safe = 1,

    [Display(Name = "Экономичный режим")]
    Eco = 2
}
```

### Поведение

* `Convert`: берёт поле enum по имени и читает `DisplayAttribute.Name`. Если `value == null` — бросает `ArgumentException`.
* `ConvertBack`: по входной строке перебирает все значения enum и возвращает то, у которого `DisplayAttribute.Name` совпадает со строкой. Если `value` не `string` или `parameter` не `Type` — бросает `ArgumentException`.

> Рекомендация: используйте `Mode=TwoWay` только если действительно ожидаете ввод строки пользователем (например, через `ComboBox`/`AutoComplete`). Для одностороннего отображения оставьте `Mode=OneWay`.

### Примеры

```xml
<!-- Отобразить подпись текущего режима -->
<TextBlock Text="{Binding SelectedMode,
                          Converter={StaticResource EnumDisplayName},
                          Mode=OneWay}" />
```

```xml
<!-- Выпадающий список с локализованными подписями -->
<ComboBox SelectedItem="{Binding SelectedMode, Mode=TwoWay}">
    <ComboBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Converter={StaticResource EnumDisplayName},
                                      ConverterParameter={x:Type local:OperationMode}}"/>
        </DataTemplate>
    </ComboBox.ItemTemplate>
    <ComboBox.ItemsSource>
        <x:Array Type="{x:Type local:OperationMode}">
            <local:OperationMode>Fast</local:OperationMode>
            <local:OperationMode>Safe</local:OperationMode>
            <local:OperationMode>Eco</local:OperationMode>
        </x:Array>
    </ComboBox.ItemsSource>
</ComboBox>
```

> Частая ошибка: не передать тип в `ConverterParameter` для `ConvertBack`. В таком случае сопоставление по строке выполнить нельзя.
