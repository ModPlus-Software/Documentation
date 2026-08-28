# Компонент ButtonWithOptionsControl

Для использования необходимо подключить nuget-пакет ModPlusAPI версии не ниже 37.0 (если в пакете модуля версия ниже)!

```<PackageReference Include="ModPlusAPI" Version="37.0.0" ExcludeAssets="runtime" />```

---

## ButtonWithOptionsControl

`ButtonWithOptionsControl` — элемент управления WPF, представляющий кнопку с выпадающим списком вариантов.

### Возможности

- Основная кнопка с текстом (`ButtonHeader`)
- До 10 опций (`Option1Header` … `Option10Header`)
- Поддержка привязок (например, для локализации)
- Вызов команды при выборе опции
- Передача информации о выбранном варианте в команду

### Использование

#### Пример XAML

```xml
<controls:ButtonWithOptionsControl 
    Command="{Binding SelectedButtonOptionCommand}"
    ButtonHeader="Actions"
    Option1Header="Create"
    Option2Header="Edit"
    Option3Header="Delete" />
```

Пример с привязкой к ресурсам:

```xml
<controls:ButtonWithOptionsControl 
    Command="{Binding SelectedButtonOptionCommand}"
    ButtonHeader="{Binding Source={StaticResource Lang}, XPath=actions}"
    Option1Header="{Binding Source={StaticResource Lang}, XPath=optCreate}"
    Option2Header="{Binding Source={StaticResource Lang}, XPath=optEdit}" />
```

### Обработка выбора

Команда получает аргументы типа `ButtonWithOptionsArgs`, содержащие:

- `Window` — родительское окно
- `Option` — выбранная опция (`ButtonOption`)

#### Пример команды

```csharp
public ICommand SelectedButtonOptionCommand =>
    new RelayCommand<ButtonWithOptionsArgs>(args =>
    {
        MessageBox.Show($"Selected: {args.Option}");
    });
```

### Свойства

| Свойство | Описание |
|---------|----------|
| `ButtonHeader` | Текст основной кнопки |
| `Option1Header` … `Option10Header` | Тексты опций. Пустые не отображаются |
| `Command` | Команда, вызываемая при выборе |

### Пример из тестового приложения

```xml
<StackPanel Orientation="Horizontal">
    <TextBlock Text="Button with options:" />

    <controls:ButtonWithOptionsControl 
        Command="{Binding SelectedButtonOptionCommand}"
        ButtonHeader="No lang"
        Option1Header="Option 1"
        Option2Header="Option 2" />

    <controls:ButtonWithOptionsControl 
        Command="{Binding SelectedButtonOptionCommand}"
        ButtonHeader="All options"
        Option1Header="Option 1"
        Option2Header="Option 2"
        Option3Header="Option 3"
        Option4Header="Option 4"
        Option5Header="Option 5"
        Option6Header="Option 6"
        Option7Header="Option 7"
        Option8Header="Option 8"
        Option9Header="Option 9"
        Option10Header="Option 10" />
</StackPanel>
```
