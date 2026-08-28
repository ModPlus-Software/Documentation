# Передача прокрутки в родительский элемент

Для использования необходимо подключить nuget-пакет ModPlusStyle версии не ниже 12.0 (если в пакете модуля версия ниже)!

```<PackageReference Include="ModPlusStyle" Version="12.0.0" ExcludeAssets="runtime" />```

---

## PropagateMouseWheelAtScrollEdgesBehavior

### Назначение

Поведение предназначено для корректной обработки колеса мыши в списочных элементах управления WPF (ListBox, ListView, TreeView, DataGrid), вложенных во внешний контейнер с прокруткой (например, ScrollViewer, Grid и т.д.).

При прокрутке, если внутренний список достиг верхней или нижней границы, или если в списке отсутствует собственная вертикальная прокрутка, событие передаётся родительскому элементу. Это позволяет реализовать «сквозную» прокрутку вложенных контролов и избежать ситуации, когда колесо мыши перестаёт работать при упоре во внутренний список.

### Основные возможности

* Поддерживаются все стандартные списочные элементы: **ListBox, ListView, TreeView, DataGrid**.
* Событие прокрутки передаётся родителю:

  * если список достиг **верхней** границы и пользователь прокручивает **вверх**,
  * если список достиг **нижней** границы и пользователь прокручивает **вниз**,
  * если список **не имеет вертикального скроллбара** (мало элементов).
* Работает с вложенными и сложными иерархиями контролов.

### Использование

Подключите пространство имён:

```xml
xmlns:b="https://modplus.org/xaml/behaviors"
xmlns:behaviors="clr-namespace:ModPlusStyle.Behaviours.Public;assembly=ModPlusStyle"
```

Пример применения в XAML:

```xml
<Grid>
    <b:Interaction.Behaviors>
        <behaviors:PropagateMouseWheelAtScrollEdgesBehavior/>
    </b:Interaction.Behaviors>

    <ScrollViewer VerticalScrollBarVisibility="Auto">
        <StackPanel>
            <ListBox Height="200">
                <ListBoxItem>Элемент 1</ListBoxItem>
                <ListBoxItem>Элемент 2</ListBoxItem>
                <ListBoxItem>Элемент 3</ListBoxItem>
                <!-- ... -->
            </ListBox>

            <ListView Height="200">
                <!-- ... -->
            </ListView>

            <TreeView Height="200">
                <!-- ... -->
            </TreeView>

            <DataGrid Height="200" AutoGenerateColumns="True">
                <!-- ... -->
            </DataGrid>
        </StackPanel>
    </ScrollViewer>
</Grid>
```

### Технические детали

* При обработке `PreviewMouseWheel` поведение ищет ближайший родитель-контрол типа ListBox, ListView, TreeView или DataGrid.
* Если у контрола найден `ScrollViewer`, проверяется его свойство `VerticalOffset` и диапазон `ScrollableHeight`.
* Если условия срабатывают, событие помечается как обработанное (`e.Handled = true`), а в родительский элемент поднимается новое событие `MouseWheelEvent`.
* Если у внутреннего списка **вообще нет ScrollViewer** либо `ScrollableHeight <= 0`, событие также сразу передаётся наверх.

### Когда использовать

* При вложении списочных контролов в области с прокруткой (например, `ScrollViewer`, `DockPanel`, `Grid` и т.п.).
* Когда требуется единое «сквозное» поведение колеса мыши для пользователя, без необходимости вручную прокручивать вложенный список до конца.
