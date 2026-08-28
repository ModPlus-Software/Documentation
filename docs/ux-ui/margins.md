# Отступы: ресурсы Margin и расширение MarginExtension

Для использования необходимо подключить nuget-пакет ModPlusStyle версии не ниже 16.0 (если в пакете модуля версия ниже)!

```<PackageReference Include="ModPlusStyle" Version="16.0.0" ExcludeAssets="runtime" />```

---

## Зачем это нужно

В версии 16.0.0 в библиотеке появился **компактный режим** — уменьшенные отступы и высоты
элементов для тех, кто работает на небольших экранах. Режим включается через
`ThemeManager.IsCompactMode` (начальное значение читается из реестра
`HKCU\Software\ModPlus`, параметр `CompactMode`) и применяется словарём
`Density.Compact.xaml` поверх темы.

Из этого следует главное правило разметки:

> **Не задавайте отступы числами.** `Margin="8 4 8 4"` останется таким же и в компактном режиме,
> и окно перестанет быть плотным. Используйте либо готовые ресурсы `ModPlus.Sizes.Margin.*`,
> либо разметочное расширение `{mps:Margin}` — они пересчитываются при переключении режима,
> в том числе на лету.

## Базовая единица плотности

Вся сетка отступов строится от одного ресурса — `ModPlus.Sizes.Unit`:

| Ресурс | Обычный режим | Компактный режим |
|---|---|---|
| `ModPlus.Sizes.Unit` | `4` | `2` |

Все значения ниже — это множители этой единицы.

## Готовые ресурсы отступов

Подключаются как `{DynamicResource ...}`. Именно `DynamicResource`, а не `StaticResource`:
только динамическая ссылка переживает переключение темы и компактного режима.

### Базовый отступ

| Ключ ресурса | Обычный | Компактный |
|---|---|---|
| `ModPlus.Sizes.DefaultMargin` | `4` | `2` |
| `ModPlus.Sizes.Margin.All` | `4` | `2` |
| `ModPlus.Sizes.Margin.Horizontal` | `4 0` | `2 0` |
| `ModPlus.Sizes.Margin.Vertical` | `0 4` | `0 2` |
| `ModPlus.Sizes.Margin.Left` | `4 0 0 0` | `2 0 0 0` |
| `ModPlus.Sizes.Margin.Top` | `0 4 0 0` | `0 2 0 0` |
| `ModPlus.Sizes.Margin.Right` | `0 0 4 0` | `0 0 2 0` |
| `ModPlus.Sizes.Margin.Bottom` | `0 0 0 4` | `0 0 0 2` |
| `ModPlus.Sizes.Margin.ExceptLeft` | `0 4 4 4` | `0 2 2 2` |
| `ModPlus.Sizes.Margin.ExceptTop` | `4 0 4 4` | `2 0 2 2` |
| `ModPlus.Sizes.Margin.ExceptRight` | `4 4 0 4` | `2 2 0 2` |
| `ModPlus.Sizes.Margin.ExceptBottom` | `4 4 4 0` | `2 2 2 0` |

`ModPlus.Sizes.DefaultMargin` и `ModPlus.Sizes.Margin.All` дают одно и то же значение.
Первый — это переименованный старый ключ `ModPlusDefaultMargin`, он объявлен в каждом файле темы,
второй — часть новой сетки отступов из `Sizes.xaml`. В новом коде предпочтительнее `ModPlus.Sizes.Margin.All`: рядом с ним
лежат все остальные варианты.

### Половина базового отступа

Нужна там, где элементы стоят вплотную друг к другу: соседние кнопки в панели,
ячейки внутри строки.

| Ключ ресурса | Обычный | Компактный |
|---|---|---|
| `ModPlus.Sizes.Margin.Half` | `2` | `1` |
| `ModPlus.Sizes.Margin.Half.Horizontal` | `2 0` | `1 0` |
| `ModPlus.Sizes.Margin.Half.Vertical` | `0 2` | `0 1` |
| `ModPlus.Sizes.Margin.Half.Left` | `2 0 0 0` | `1 0 0 0` |
| `ModPlus.Sizes.Margin.Half.Top` | `0 2 0 0` | `0 1 0 0` |
| `ModPlus.Sizes.Margin.Half.Right` | `0 0 2 0` | `0 0 1 0` |
| `ModPlus.Sizes.Margin.Half.Bottom` | `0 0 0 2` | `0 0 0 1` |

### Пример

```xml
<StackPanel Margin="{DynamicResource ModPlus.Sizes.Margin.All}">

    <TextBlock Text="Имя листа"
               Margin="{DynamicResource ModPlus.Sizes.Margin.Bottom}" />

    <TextBox Margin="{DynamicResource ModPlus.Sizes.Margin.Vertical}" />

    <StackPanel Orientation="Horizontal"
                HorizontalAlignment="Right"
                Margin="{DynamicResource ModPlus.Sizes.Margin.Top}">
        <Button Content="ОК"
                Margin="{DynamicResource ModPlus.Sizes.Margin.Half.Right}" />
        <Button Content="Отмена"
                Margin="{DynamicResource ModPlus.Sizes.Margin.Half.Left}" />
    </StackPanel>

</StackPanel>
```

## Расширение {mps:Margin}

Готовых ресурсов хватает не всегда: бывает нужен отступ в три единицы слева, полторы сверху
и ноль по остальным сторонам. Для таких случаев есть разметочное расширение `MarginExtension`.

Подключите пространство имён:

```xml
xmlns:mps="https://modplus.org/winfx/xaml/controls"
```

### Как записывается значение

**Числа в расширении — это множители, а не пиксели.** Итоговое значение равно
множитель × `ModPlus.Sizes.Unit`, то есть в обычном режиме множитель умножается на 4,
а в компактном — на 2.

Запись множителей повторяет правила `Thickness`:

| Количество чисел | Смысл |
|---|---|
| одно | все четыре стороны |
| два | горизонталь и вертикаль |
| четыре | левый, верхний, правый, нижний |

Разделители — пробел или запятая, дробная часть через **точку** (значение разбирается
в инвариантной культуре, поэтому запятая как десятичный разделитель работать не будет).

> Если множителей больше одного, всю запись нужно взять в **одинарные кавычки**:
> `{mps:Margin '1 0 0 0'}`. Без кавычек XAML воспримет пробел или запятую как разделитель
> аргументов расширения и выдаст ошибку разбора.

### Примеры значений

| Запись | Обычный режим | Компактный режим |
|---|---|---|
| `{mps:Margin 1}` | `4 4 4 4` | `2 2 2 2` |
| `{mps:Margin 0.5}` | `2 2 2 2` | `1 1 1 1` |
| `{mps:Margin '1 0 0 0'}` | `4 0 0 0` | `2 0 0 0` |
| `{mps:Margin '1 0.5'}` | `4 2 4 2` | `2 1 2 1` |
| `{mps:Margin '3 1 1 1'}` | `12 4 4 4` | `6 2 2 2` |
| `{mps:Margin '5 1 1 1'}` | `20 4 4 4` | `10 2 2 2` |

### Пример в разметке

```xml
<mps:ModPlusWindow x:Class="MyPlugin.MainWindow"
                   xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                   xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                   xmlns:mps="https://modplus.org/winfx/xaml/controls"
                   Title="Пример">

    <StackPanel Margin="{mps:Margin 1}">

        <!-- Заголовок группы: обычный отступ снизу -->
        <TextBlock Text="Параметры" Margin="{mps:Margin '0 0 0 1'}" />

        <!-- Вложенный элемент: сдвиг вправо на три единицы -->
        <CheckBox Content="Учитывать высоту" Margin="{mps:Margin '3 0.5 0 0.5'}" />

        <!-- Отступ можно ставить не только на Margin -->
        <Border BorderThickness="1"
                Padding="{mps:Margin '1 0.5'}">
            <TextBlock Text="Содержимое" />
        </Border>

    </StackPanel>

</mps:ModPlusWindow>
```

### Не только Margin

Расширение возвращает `Thickness`, поэтому подходит **любому** свойству этого типа:
`Margin`, `Padding`, `BorderThickness`, `ControlsAssist.HeaderMargin` и так далее.

## Что выбрать: ресурс или расширение

| Ситуация | Что использовать |
|---|---|
| Обычный отступ по всем сторонам или по одной | `{DynamicResource ModPlus.Sizes.Margin.*}` |
| Половинный отступ между соседними элементами | `{DynamicResource ModPlus.Sizes.Margin.Half.*}` |
| Дробный множитель, сдвиг вложенного элемента, нестандартная комбинация сторон | `{mps:Margin '...'}` |
| Контрол создаётся в коде | `DensityAssist.SetMargin` |

Оба способа дают одинаковый результат для одинаковых значений и одинаково пересчитываются
при переключении компактного режима. Ресурсы читаются чуть привычнее, расширение — гибче.

## Отступы для контролов, создаваемых в коде

Если контрол собирается в C#, присваивать `Margin` числом нельзя по той же причине.
Для этого есть статический класс `DensityAssist`:

```csharp
using ModPlusStyle.Controls;

var button = new Button { Content = "ОК" };

// Margin = 1 единица по всем сторонам
DensityAssist.SetMargin(button, 1);

// Margin = "1 0.5" (горизонталь / вертикаль)
DensityAssist.SetMargin(button, 1, 0.5);

// Margin = "3 1 1 1"
DensityAssist.SetMargin(button, 3, 1, 1, 1);

// Любое другое свойство типа Thickness
DensityAssist.SetThickness(button, Control.PaddingProperty, 1, 0.5);
```

Правила те же: один, два или четыре множителя. Другое количество приведёт к
`ArgumentException`. Метод ставит привязку, а не константу, поэтому значение
пересчитывается при переключении режима на лету.

## Технические детали

* Значение отдаётся не числом, а привязкой к наследуемому свойству `DensityAssist.Unit`.
  Именно поэтому оно переживает переключение компактного режима без перезагрузки окна.
* `ModPlusWindow` в конструкторе связывает `DensityAssist.Unit` с ресурсом
  `ModPlus.Sizes.Unit`, и значение наследуется всеми элементами внутри окна.
* Когда цель расширения известна и это `FrameworkElement`, расширение дополнительно
  ставит ссылку на ресурс само. Благодаря этому `{mps:Margin}` работает и в окнах,
  не унаследованных от `ModPlusWindow`, — достаточно, чтобы были подключены словари стилей.
* Если ни того, ни другого нет, `DensityAssist.Unit` берёт значение по умолчанию `4`,
  то есть разметка выглядит как в обычном режиме.

## Смотрите также

* [Размеры: расширение SizeExtension](size-extension.md) — то же самое для ширины, высоты
  и прочих размеров.
* [Новые ключи ресурсов](resource-keys.md) — соответствие старых и новых имён ресурсов.
