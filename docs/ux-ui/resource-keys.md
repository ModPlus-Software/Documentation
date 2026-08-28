# Новые ключи ресурсов

Для использования необходимо подключить nuget-пакет ModPlusStyle версии не ниже 16.0 (если в пакете модуля версия ниже)!

```<PackageReference Include="ModPlusStyle" Version="16.0.0" ExcludeAssets="runtime" />```

---

## Что изменилось

В версии 16.0.0 все ключи ресурсов библиотеки оформления переименованы по единой схеме
`ModPlus.<Категория>.<Имя>`. Раньше имена складывались стихийно: `AccentColorBrush`,
`ModPlusFlatButton`, `DarkGray`, `TrueValue` — по имени нельзя было понять ни тип ресурса,
ни то, откуда он берётся. Теперь префикс сразу говорит, что это за ресурс, а IntelliSense
показывает все ресурсы одной категории, если начать набирать `ModPlus.Brushes.`.

В таблице ниже — 275 пар «старый ключ → новый ключ».

> Старые ключи продолжают работать, удалять их из библиотеки нельзя. Они сохранены двумя способами:
>
> * значения тем и размеров (цвета, кисти, шрифты, скругления, отступы) — через словарь
>   `LegacyResourceKeys`: при применении темы `ThemeManager` кладёт в ресурсы окна вместе с каждым
>   актуальным ключом его устаревший псевдоним с тем же значением;
> * стили — прямо в словарях библиотеки, отдельной записью вида
>   `<Style x:Key="ModPlusFlatButton" BasedOn="{StaticResource ModPlus.Styles.FlatButton}" />`.
>
> Благодаря этому плагины, выпущенные до переименования, продолжают работать без изменений.
> **Но в новом коде нужно использовать только новые ключи.**

## Схема именования

| Префикс | Что содержит |
|---|---|
| `ModPlus.Colors.*` | Цвета (`Color`) |
| `ModPlus.Brushes.*` | Кисти (`Brush`) |
| `ModPlus.Fonts.*` | Семейства шрифтов |
| `ModPlus.Fonts.Sizes.*` | Размеры шрифта |
| `ModPlus.Sizes.*` | Размеры, отступы, толщины (`double`, `Thickness`) |
| `ModPlus.Radius.*` | Радиусы скругления (`CornerRadius`) |
| `ModPlus.Styles.*` | Стили контролов |
| `ModPlus.Converters.*` | Конвертеры, объявленные в словарях библиотеки |
| `ModPlus.Shadows.*` | Эффекты тени |
| `ModPlus.Easing.*` | Функции сглаживания анимаций |
| `ModPlus.Animation.*` | Длительности анимаций |
| `ModPlus.Opacity.*` | Значения прозрачности |
| `ModPlus.Values.*` | Логические значения (`True` / `False`) |
| `ModPlus.Visuals.*` | Изображения и визуальные элементы |

Составные имена читаются слева направо от общего к частному:
`ModPlus.Brushes.Button.HoverBackground` — кисть, кнопка, фон при наведении.

Пример перехода на новые ключи:

```xml
<!-- Было -->
<Button Background="{DynamicResource AccentColorBrush}"
        Style="{StaticResource ModPlusFlatButton}"
        Margin="{DynamicResource ModPlusDefaultMargin}" />

<!-- Стало -->
<Button Background="{DynamicResource ModPlus.Brushes.Accent}"
        Style="{StaticResource ModPlus.Styles.FlatButton}"
        Margin="{DynamicResource ModPlus.Sizes.Margin.All}" />
```

> Обратите внимание: ключи стилей содержат точки, поэтому в разметке они всегда указываются
> через `{StaticResource ...}` или `{DynamicResource ...}` — сокращённая запись
> `Style="{StaticResource ModPlus.Styles.FlatButton}"` работает так же, как и раньше.

## Таблица соответствия

Ключи сгруппированы по категориям нового имени. Поиск по странице (<kbd>Ctrl</kbd> + <kbd>F</kbd>)
по старому имени быстро приведёт к нужной строке.

### Цвета — `ModPlus.Colors.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `AccentColor` | `ModPlus.Colors.Accent` |
| `BlackColor` | `ModPlus.Colors.Black` |
| `BlueColor` | `ModPlus.Colors.Blue` |
| `ButtonAccentBackgroundColor` | `ModPlus.Colors.Button.AccentBackground` |
| `ButtonAccentBorderColor` | `ModPlus.Colors.Button.AccentBorder` |
| `ButtonAccentForegroundColor` | `ModPlus.Colors.Button.AccentForeground` |
| `ButtonHoverBackgroundColor` | `ModPlus.Colors.Button.HoverBackground` |
| `ButtonHoverBorderColor` | `ModPlus.Colors.Button.HoverBorder` |
| `ButtonHoverForegroundColor` | `ModPlus.Colors.Button.HoverForeground` |
| `DarkGray` | `ModPlus.Colors.DarkGray` |
| `ErrorColor` | `ModPlus.Colors.Error` |
| `FlatButtonGray` | `ModPlus.Colors.FlatButtonGray` |
| `IdealForegroundColor` | `ModPlus.Colors.IdealForeground` |
| `LightBlackColor` | `ModPlus.Colors.LightBlack` |
| `LightBlueColor` | `ModPlus.Colors.LightBlue` |
| `LightBrownColor` | `ModPlus.Colors.LightBrown` |
| `LightBurgundyColor` | `ModPlus.Colors.LightBurgundy` |
| `LightGrayColor` | `ModPlus.Colors.LightGray` |
| `LightGreenColor` | `ModPlus.Colors.LightGreen` |
| `LightOrangeColor` | `ModPlus.Colors.LightOrange` |
| `LightPurpleColor` | `ModPlus.Colors.LightPurple` |
| `LightRedColor` | `ModPlus.Colors.LightRed` |
| `LightTealColor` | `ModPlus.Colors.LightTeal` |
| `LightYellowColor` | `ModPlus.Colors.LightYellow` |
| `MidGray` | `ModPlus.Colors.MidGray` |
| `OrangeColor` | `ModPlus.Colors.Orange` |
| `ProgressBarBackgroundColor` | `ModPlus.Colors.ProgressBarBackground` |
| `PurpleColor` | `ModPlus.Colors.Purple` |
| `SuccessColor` | `ModPlus.Colors.Success` |
| `WhiteColor` | `ModPlus.Colors.White` |
| `WindowActiveBorderColor` | `ModPlus.Colors.WindowActiveBorder` |
| `WindowBackgroundColor` | `ModPlus.Colors.WindowBackground` |
| `WindowIconColor` | `ModPlus.Colors.WindowIcon` |
| `WindowNotActiveBorderColor` | `ModPlus.Colors.WindowInactiveBorder` |
| `ModPlusShadow` | `ModPlus.Colors.Shadow` |

### Кисти — `ModPlus.Brushes.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `AccentColorBrush` | `ModPlus.Brushes.Accent` |
| `BlackBrush` | `ModPlus.Brushes.Black` |
| `BlueBrush` | `ModPlus.Brushes.Blue` |
| `BoxesBorderBrush` | `ModPlus.Brushes.BoxesBorder` |
| `ButtonAccentBackgroundBrush` | `ModPlus.Brushes.Button.AccentBackground` |
| `ButtonAccentBorderBrush` | `ModPlus.Brushes.Button.AccentBorder` |
| `ButtonAccentForegroundBrush` | `ModPlus.Brushes.Button.AccentForeground` |
| `ButtonHoverBackgroundBrush` | `ModPlus.Brushes.Button.HoverBackground` |
| `ButtonHoverBorderBrush` | `ModPlus.Brushes.Button.HoverBorder` |
| `ButtonHoverForegroundBrush` | `ModPlus.Brushes.Button.HoverForeground` |
| `CardBackground` | `ModPlus.Brushes.CardBackground` |
| `ControlsValidationBrush` | `ModPlus.Brushes.ControlsValidation` |
| `DarkGrayBrush` | `ModPlus.Brushes.DarkGray` |
| `ErrorBrush` | `ModPlus.Brushes.Error` |
| `FlatButtonGrayBrush` | `ModPlus.Brushes.FlatButtonGray` |
| `FlatButtonMouseOverForeground` | `ModPlus.Brushes.FlatButtonMouseOverForeground` |
| `FlyoutBackgroundBrush` | `ModPlus.Brushes.FlyoutBackground` |
| `FlyoutForegroundBrush` | `ModPlus.Brushes.FlyoutForeground` |
| `FlyoutModalessBackgroundBrush` | `ModPlus.Brushes.FlyoutModalessBackground` |
| `FlyoutModalessForegroundBrush` | `ModPlus.Brushes.FlyoutModalessForeground` |
| `ForegroundForAccentedBrush` | `ModPlus.Brushes.ForegroundForAccented` |
| `IdealForegroundColorBrush` | `ModPlus.Brushes.IdealForeground` |
| `IdealForegroundDisabledBrush` | `ModPlus.Brushes.IdealForegroundDisabled` |
| `LightBlackBrush` | `ModPlus.Brushes.LightBlack` |
| `LightBlueBrush` | `ModPlus.Brushes.LightBlue` |
| `LightBrownBrush` | `ModPlus.Brushes.LightBrown` |
| `LightBurgundyBrush` | `ModPlus.Brushes.LightBurgundy` |
| `LightGrayBrush` | `ModPlus.Brushes.LightGray` |
| `LightGreenBrush` | `ModPlus.Brushes.LightGreen` |
| `LightOrangeBrush` | `ModPlus.Brushes.LightOrange` |
| `LightPurpleBrush` | `ModPlus.Brushes.LightPurple` |
| `LightRedBrush` | `ModPlus.Brushes.LightRed` |
| `LightTealBrush` | `ModPlus.Brushes.LightTeal` |
| `LightYellowBrush` | `ModPlus.Brushes.LightYellow` |
| `MidGrayBrush` | `ModPlus.Brushes.MidGray` |
| `ModPlusShadowBrush` | `ModPlus.Brushes.Shadow` |
| `ModPlusWindowIconBrush` | `ModPlus.Brushes.WindowIcon` |
| `ModPlusWindowOverlayBrush` | `ModPlus.Brushes.WindowOverlay` |
| `OrangeBrush` | `ModPlus.Brushes.Orange` |
| `ProgressBarBackgroundBrush` | `ModPlus.Brushes.ProgressBarBackground` |
| `PurpleBrush` | `ModPlus.Brushes.Purple` |
| `SemiTransparentWhiteBrush` | `ModPlus.Brushes.SemiTransparentWhite` |
| `SuccessBrush` | `ModPlus.Brushes.Success` |
| `TabHeaderBrush` | `ModPlus.Brushes.TabHeader` |
| `TabHeaderOverBrush` | `ModPlus.Brushes.TabHeaderOver` |
| `TabHeaderOverSelectedBrush` | `ModPlus.Brushes.TabHeaderOverSelected` |
| `TabHeaderSelectedBrush` | `ModPlus.Brushes.TabHeaderSelected` |
| `ToggleButtonThumbBrush` | `ModPlus.Brushes.ToggleButtonThumb` |
| `TransparentWhiteBrush` | `ModPlus.Brushes.TransparentWhite` |
| `ValidationTextBrush` | `ModPlus.Brushes.ValidationText` |
| `WhiteBrush` | `ModPlus.Brushes.White` |
| `WindowActiveBorderColorBrush` | `ModPlus.Brushes.WindowActiveBorder` |
| `WindowBackgroundColorBrush` | `ModPlus.Brushes.WindowBackground` |
| `WindowNotActiveBorderColorBrush` | `ModPlus.Brushes.WindowInactiveBorder` |

### Скругления — `ModPlus.Radius.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `ButtonCornerRadius` | `ModPlus.Radius.Button` |
| `CardCornerRadius` | `ModPlus.Radius.Card` |
| `DialogCornerRadius` | `ModPlus.Radius.Dialog` |
| `ExpanderCornerRadius` | `ModPlus.Radius.Expander` |
| `FlatButtonCornerRadius` | `ModPlus.Radius.FlatButton` |
| `FlipperCornerRadius` | `ModPlus.Radius.Flipper` (контрол Flipper удален, ключ оставлен) |
| `GroupBoxCornerRadius` | `ModPlus.Radius.GroupBox` |
| `ModPlusBoxesCornerRadius` | `ModPlus.Radius.Boxes` |
| `ModPlusButtonsCornerRadius` | `ModPlus.Radius.Buttons` |
| `ModPlusValidationCornerRadius` | `ModPlus.Radius.Validation` |
| `PopUpCornerRadius` | `ModPlus.Radius.Popup` |
| `ScrollBarCornerRadius` | `ModPlus.Radius.ScrollBar` |

### Анимация, прозрачность, базовый отступ

| Устаревший ключ | Новый ключ |
|---|---|
| `AnimateMouseDuration` | `ModPlus.Animation.MouseDuration` |
| `ModPlusDefaultMargin` | `ModPlus.Sizes.DefaultMargin` |
| `ModPlusDisableOpacity` | `ModPlus.Opacity.Disabled` |

### Шрифты — `ModPlus.Fonts.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `ClearTextButtonFontSize` | `ModPlus.Fonts.Sizes.ClearTextButton` |
| `ContentFontFamily` | `ModPlus.Fonts.Content` |
| `ContentFontSize` | `ModPlus.Fonts.Sizes.Content` |
| `ContextMenuFontSize` | `ModPlus.Fonts.Sizes.ContextMenu` |
| `DefaultFont` | `ModPlus.Fonts.Default` |
| `DialogMessageFontSize` | `ModPlus.Fonts.Sizes.DialogMessage` |
| `DialogTitleFontSize` | `ModPlus.Fonts.Sizes.DialogTitle` |
| `FlatButtonFontSize` | `ModPlus.Fonts.Sizes.FlatButton` |
| `FloatingWatermarkFontSize` | `ModPlus.Fonts.Sizes.FloatingWatermark` |
| `FlyoutHeaderFontSize` | `ModPlus.Fonts.Sizes.FlyoutHeader` |
| `HeaderFontFamily` | `ModPlus.Fonts.Header` |
| `HeaderFontSize` | `ModPlus.Fonts.Sizes.Header` |
| `MenuFontSize` | `ModPlus.Fonts.Sizes.Menu` |
| `NormalFontSize` | `ModPlus.Fonts.Sizes.Normal` |
| `StatusBarFontSize` | `ModPlus.Fonts.Sizes.StatusBar` |
| `SubHeaderFontSize` | `ModPlus.Fonts.Sizes.SubHeader` |
| `TabItemFontSize` | `ModPlus.Fonts.Sizes.TabItem` |
| `ToggleSwitchFontFamily` | `ModPlus.Fonts.ToggleSwitch` |
| `ToggleSwitchFontFamily.Win10` | `ModPlus.Fonts.ToggleSwitch.Win10` |
| `ToggleSwitchFontSize` | `ModPlus.Fonts.Sizes.ToggleSwitch` |
| `ToggleSwitchFontSize.Win10` | `ModPlus.Fonts.Sizes.ToggleSwitch.Win10` |
| `ToggleSwitchHeaderFontFamily` | `ModPlus.Fonts.ToggleSwitchHeader` |
| `ToggleSwitchHeaderFontFamily.Win10` | `ModPlus.Fonts.ToggleSwitchHeader.Win10` |
| `ToggleSwitchHeaderFontSize` | `ModPlus.Fonts.Sizes.ToggleSwitchHeader` |
| `ToggleSwitchHeaderFontSize.Win10` | `ModPlus.Fonts.Sizes.ToggleSwitchHeader.Win10` |
| `TooltipFontSize` | `ModPlus.Fonts.Sizes.Tooltip` |
| `UpperCaseContentFontSize` | `ModPlus.Fonts.Sizes.UpperCaseContent` |
| `WindowTitleFontSize` | `ModPlus.Fonts.Sizes.WindowTitle` |

### Конвертеры — `ModPlus.Converters.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `ArcEndPointConverter` | `ModPlus.Converters.ArcEndPoint` |
| `ArcSizeConverter` | `ModPlus.Converters.ArcSize` |
| `BoolToVisConverter` | `ModPlus.Converters.BoolToVisibility` |
| `BooleanToVisibilityConverter` | `ModPlus.Converters.BooleanToVisibility` |
| `BottomCornerRadiusToZeroConverter` | `ModPlus.Converters.BottomCornerRadiusToZero` |
| `BrushRoundConverter` | `ModPlus.Converters.BrushRound` |
| `CanExpandConverter` | `ModPlus.Converters.CanExpand` |
| `CardClipConverter` | `ModPlus.Converters.CardClip` |
| `EqualityToVisibilityConverter` | `ModPlus.Converters.EqualityToVisibility` |
| `InverseBoolToVisConverter` | `ModPlus.Converters.InverseBoolToVisibility` |
| `InvertedNullVisibilityConverter` | `ModPlus.Converters.InvertedNullVisibility` |
| `LargeArcConverter` | `ModPlus.Converters.LargeArc` |
| `LevelToIndentConverter` | `ModPlus.Converters.LevelToIndent` |
| `LocalizationCommonConverter` | `ModPlus.Converters.LocalizationCommon` |
| `MathAddConverter` | `ModPlus.Converters.MathAdd` |
| `MathMlpMultipleConverter` | `ModPlus.Converters.MathMultiple` |
| `ModPlusListViewItemContainerStyleConverter` | `ModPlus.Converters.ListViewItemContainerStyle` |
| `ModPlusTreeListViewItemContainerStyleConverter` | `ModPlus.Converters.TreeListViewItemContainerStyle` |
| `NotZeroConverter` | `ModPlus.Converters.NotZero` |
| `NotZeroToVisibilityConverter` | `ModPlus.Converters.NotZeroToVisibility` |
| `NullVisibilityConverter` | `ModPlus.Converters.NullVisibility` |
| `NullableToVisibilityConverter` | `ModPlus.Converters.NullableToVisibility` |
| `RangePositionConverter` | `ModPlus.Converters.RangePosition` |
| `RotateTransformCentreConverter` | `ModPlus.Converters.RotateTransformCentre` |
| `RotateTransformConverter` | `ModPlus.Converters.RotateTransform` |
| `StartPointConverter` | `ModPlus.Converters.StartPoint` |
| `StringIsEmptyVisibilityConverter` | `ModPlus.Converters.StringIsEmptyVisibility` |
| `TextBoxContextMenuLocalizationConverter` | `ModPlus.Converters.TextBoxContextMenuLocalization` |
| `TextFieldHintVisibilityConverter` | `ModPlus.Converters.TextFieldHintVisibility` |
| `ThicknessBindingConverter` | `ModPlus.Converters.ThicknessBinding` |
| `ThicknessToDoubleConverter` | `ModPlus.Converters.ThicknessToDouble` |
| `TranslateConverter` | `ModPlus.Converters.Translate` |

### Функции сглаживания — `ModPlus.Easing.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `AnimationEasingFunction` | `ModPlus.Easing.Animation` |
| `BadgeEase` | `ModPlus.Easing.Badge` |

### Тени — `ModPlus.Shadows.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `ModPlusShadowDepth1` | `ModPlus.Shadows.Depth1` |
| `ModPlusShadowDepth2` | `ModPlus.Shadows.Depth2` |
| `ModPlusShadowDepth3` | `ModPlus.Shadows.Depth3` |
| `ModPlusShadowDepth4` | `ModPlus.Shadows.Depth4` |
| `ModPlusShadowDepth5` | `ModPlus.Shadows.Depth5` |

### Размеры — `ModPlus.Sizes.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `NoContentFloatingScale` | `ModPlus.Sizes.NoContentFloatingScale` |
| `PopupContentPresenterExtend` | `ModPlus.Sizes.Popup.ContentPresenterExtend` |
| `PopupLeftRightMargin` | `ModPlus.Sizes.Popup.LeftRightMargin` |
| `PopupTopBottomMargin` | `ModPlus.Sizes.Popup.TopBottomMargin` |

### Стили — `ModPlus.Styles.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `BaseMetroWindowButtonStyle` | `ModPlus.Styles.WindowButton.Base` |
| `ComboBoxFilterTextBoxStyle` | `ModPlus.Styles.ComboBoxFilterTextBox` |
| `FocusVisual` | `ModPlus.Styles.FocusVisual` |
| `ModPlusAccentButton` | `ModPlus.Styles.AccentButton` |
| `ModPlusAccentRoundButton` | `ModPlus.Styles.AccentRoundButton` |
| `ModPlusAccentRoundLargeButton` | `ModPlus.Styles.AccentRoundLargeButton` |
| `ModPlusBadged` | `ModPlus.Styles.Badged` |
| `ModPlusBorderlessTextBox` | `ModPlus.Styles.BorderlessTextBox` |
| `ModPlusButtonForGrayBackground` | `ModPlus.Styles.ButtonForGrayBackground` |
| `ModPlusButtonForTextBox` | `ModPlus.Styles.ButtonForTextBox` |
| `ModPlusCardGroupBox` | `ModPlus.Styles.GroupBox` (стиль CardGroupBox удален, ключ перенаправлен) |
| `ModPlusCheckBox` | `ModPlus.Styles.CheckBox` |
| `ModPlusCircularProgressBar` | `ModPlus.Styles.CircularProgressBar` |
| `ModPlusCloseWindowButtonStyle` | `ModPlus.Styles.WindowButton.Close` |
| `ModPlusComboBox` | `ModPlus.Styles.ComboBox` |
| `ModPlusComboBoxEditableTextBox` | `ModPlus.Styles.ComboBoxEditableTextBox` |
| `ModPlusComboBoxItemSelectedCollapsedStyle` | `ModPlus.Styles.ComboBoxItemSelectedCollapsed` |
| `ModPlusComboBoxItemStyle` | `ModPlus.Styles.ComboBoxItem` |
| `ModPlusComboBoxToggleButton` | `ModPlus.Styles.ComboBoxToggleButton` |
| `ModPlusContentControlEx` | `ModPlus.Styles.ContentControlEx` |
| `ModPlusContextMenu` | `ModPlus.Styles.ContextMenu` |
| `ModPlusDataGrid` | `ModPlus.Styles.DataGrid` |
| `ModPlusDataGridCell` | `ModPlus.Styles.DataGridCell` |
| `ModPlusDataGridCheckBoxColumnEditingStyle` | `ModPlus.Styles.DataGridCheckBoxColumnEditing` |
| `ModPlusDataGridCheckBoxColumnStyle` | `ModPlus.Styles.DataGridCheckBoxColumn` |
| `ModPlusDataGridColumnHeader` | `ModPlus.Styles.DataGridColumnHeader` |
| `ModPlusDataGridComboBox` | `ModPlus.Styles.DataGridComboBox` |
| `ModPlusDataGridComboBoxEditableTextBox` | `ModPlus.Styles.DataGridComboBoxEditableTextBox` |
| `ModPlusDataGridComboBoxItemStyle` | `ModPlus.Styles.DataGridComboBoxItem` |
| `ModPlusDataGridComboBoxToggleButton` | `ModPlus.Styles.DataGridComboBoxToggleButton` |
| `ModPlusDataGridRow` | `ModPlus.Styles.DataGridRow` |
| `ModPlusDataGridRowHeader` | `ModPlus.Styles.DataGridRowHeader` |
| `ModPlusDataGridTextColumnEditingStyle` | `ModPlus.Styles.DataGridTextColumnEditing` |
| `ModPlusDataGridTextColumnPopupEditingStyle` | `ModPlus.Styles.DataGridTextColumnPopupEditing` |
| `ModPlusDialogStyle` | `ModPlus.Styles.Dialog` |
| `ModPlusDiscreteSlider` | `ModPlus.Styles.DiscreteSlider` |
| `ModPlusExpandCollapseToggleStyle` | `ModPlus.Styles.ExpandCollapseToggle` |
| `ModPlusExpander` | `ModPlus.Styles.Expander` |
| `ModPlusExpanderDownHeaderStyle` | `ModPlus.Styles.ExpanderDownHeader` |
| `ModPlusExpanderLeftHeaderStyle` | `ModPlus.Styles.ExpanderLeftHeader` |
| `ModPlusExpanderRightHeaderStyle` | `ModPlus.Styles.ExpanderRightHeader` |
| `ModPlusExpanderToggleButton` | `ModPlus.Styles.ExpanderToggleButton` |
| `ModPlusExpanderUpHeaderStyle` | `ModPlus.Styles.ExpanderUpHeader` |
| `ModPlusFlatButton` | `ModPlus.Styles.FlatButton` |
| `ModPlusFlatButtonForIcon` | `ModPlus.Styles.FlatButtonForIcon` |
| `ModPlusGridColumnHeaderGripper` | `ModPlus.Styles.GridColumnHeaderGripper` |
| `ModPlusGridRowHeaderGripper` | `ModPlus.Styles.GridRowHeaderGripper` |
| `ModPlusGridSplitter` | `ModPlus.Styles.GridSplitter` |
| `ModPlusGridViewItem` | `ModPlus.Styles.GridViewItem` |
| `ModPlusGroupBox` | `ModPlus.Styles.GroupBox` |
| `ModPlusHorizontalTrackDiscreteRepeatButton` | `ModPlus.Styles.HorizontalTrackDiscreteRepeatButton` |
| `ModPlusHorizontalTrackRepeatButton` | `ModPlus.Styles.HorizontalTrackRepeatButton` |
| `ModPlusLightStretchListBoxItem` | `ModPlus.Styles.LightStretchListBoxItem` |
| `ModPlusLinearProgressBar` | `ModPlus.Styles.LinearProgressBar` |
| `ModPlusListBox` | `ModPlus.Styles.ListBox` |
| `ModPlusListBoxItem` | `ModPlus.Styles.ListBoxItem` |
| `ModPlusListView` | `ModPlus.Styles.ListView` |
| `ModPlusMenu` | `ModPlus.Styles.Menu` |
| `ModPlusMenuItem` | `ModPlus.Styles.MenuItem` |
| `ModPlusMultiFloatingActionPopupBox` | `ModPlus.Styles.MultiFloatingActionPopupBox` |
| `ModPlusMultiSelectTreeView` | `ModPlus.Styles.MultiSelectTreeView` |
| `ModPlusMultiSelectTreeViewItem` | `ModPlus.Styles.MultiSelectTreeViewItem` |
| `ModPlusPasswordBox` | `ModPlus.Styles.PasswordBox` |
| `ModPlusPopupBox` | `ModPlus.Styles.PopupBox` |
| `ModPlusRadioButton` | `ModPlus.Styles.RadioButton` |
| `ModPlusRichTextBox` | `ModPlus.Styles.RichTextBox` |
| `ModPlusScrollBar` | `ModPlus.Styles.ScrollBar` |
| `ModPlusScrollBarRepeatButtonLargeStyle` | `ModPlus.Styles.ScrollBarRepeatButtonLarge` |
| `ModPlusScrollBarRepeatButtonSmallStyle` | `ModPlus.Styles.ScrollBarRepeatButtonSmall` |
| `ModPlusScrollBarThumbStyle` | `ModPlus.Styles.ScrollBarThumb` |
| `ModPlusSeparator` | `ModPlus.Styles.Separator` |
| `ModPlusSlider` | `ModPlus.Styles.Slider` |
| `ModPlusStylesWindowButtonCommands` | `ModPlus.Styles.WindowButtonCommands.Base` |
| `ModPlusSwitchToggleButton` | `ModPlus.Styles.SwitchToggleButton` |
| `ModPlusTabControl` | `ModPlus.Styles.TabControl` |
| `ModPlusTabItem` | `ModPlus.Styles.TabItem` |
| `ModPlusTextBox` | `ModPlus.Styles.TextBox` |
| `ModPlusThumb` | `ModPlus.Styles.Thumb` |
| `ModPlusThumbContentControl` | `ModPlus.Styles.ThumbContentControl` |
| `ModPlusToolButton` | `ModPlus.Styles.ToolButton` |
| `ModPlusToolForegroundPopupBox` | `ModPlus.Styles.ToolForegroundPopupBox` |
| `ModPlusToolPopupBox` | `ModPlus.Styles.ToolPopupBox` |
| `ModPlusToolTip` | `ModPlus.Styles.ToolTip` |
| `ModPlusTransparentAccentButton` | `ModPlus.Styles.TransparentAccentButton` |
| `ModPlusTransparentRoundButton` | `ModPlus.Styles.TransparentRoundButton` |
| `ModPlusTreeListView` | `ModPlus.Styles.TreeListView` |
| `ModPlusTreeView` | `ModPlus.Styles.TreeView` |
| `ModPlusTreeViewItem` | `ModPlus.Styles.TreeViewItem` |
| `ModPlusTreeViewItemFocusVisual` | `ModPlus.Styles.TreeViewItemFocusVisual` |
| `ModPlusValidationPopup` | `ModPlus.Styles.ValidationPopup` |
| `ModPlusVerticalTrackRepeatButton` | `ModPlus.Styles.VerticalTrackRepeatButton` |
| `ModPlusWindowButtonCommands` | `ModPlus.Styles.WindowButtonCommands` |
| `OptionMarkFocusVisual` | `ModPlus.Styles.OptionMarkFocusVisual` |
| `WindowCommandsControlStyle` | `ModPlus.Styles.WindowCommands` |
| `WindowTitleThumbStyle` | `ModPlus.Styles.WindowTitleThumb` |

### Логические значения — `ModPlus.Values.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `FalseValue` | `ModPlus.Values.False` |
| `TrueValue` | `ModPlus.Values.True` |

### Изображения — `ModPlus.Visuals.*`

| Устаревший ключ | Новый ключ |
|---|---|
| `ModPlusMultiFloatingActionPopupBoxIcon` | `ModPlus.Visuals.PopupBoxIcon` |
| `ModPlusPopupBoxToggleContent` | `ModPlus.Visuals.PopupBoxToggleContent` |
| `NewYearDecorationImage` | `ModPlus.Visuals.NewYearDecoration` |

## Ключи, которые не переименованы

Шаблоны (`ControlTemplate`), раскадровки (`Storyboard`) и два конвертера шаблонов сохранили
исходные имена. Для них нет способа объявить в XAML псевдоним без полного дублирования блока:
конструкция `<StaticResource x:Key="..." ResourceKey="..." />` ломает загрузку словаря, а
дублирование этих блоков добавило бы около 2300 строк.

Список:

- `BadgeChangedStoryboard`
- `CardTemplate`
- `DialogCloseStoryboard`
- `DialogShownStoryboard`
- `HorizontalScrollBar`
- `ModPlusComboBoxItemTemplate`
- `ModPlusDataGridComboBoxEditableTemplate`
- `ModPlusDataGridComboBoxTemplate`
- `ModPlusDialogTemplate`
- `ModPlusDiscreteSliderHorizontal`
- `ModPlusDiscreteSliderThumb`
- `ModPlusFloatingHintComboBoxTemplate`
- `ModPlusListViewTemplateConverter`
- `ModPlusScrollBarThumbMouseOverStoryboard`
- `ModPlusScrollBarThumbPressedStoryboard`
- `ModPlusSliderHorizontal`
- `ModPlusSliderThumb`
- `ModPlusSliderVertical`
- `ModPlusTemplatesWindowButtonCommands`
- `ModPlusTreeListViewTemplateConverter`
- `ModPlusValidationErrorTemplate`
- `OverlayFastSemiFadeIn`
- `OverlayFastSemiFadeOut`
- `PopupContentClassicTemplate`
- `PopupContentDownTemplate`
- `PopupContentUpTemplate`
- `SheetNameTextBoxTemplate`
- `ValidationErrorTemplate`
- `VerticalScrollBar`
- `WindowButtonCommandsTemplate`
- `WindowCommandsButtonTemplate`
- `WindowCommandsTemplate`
- `WindowCommandsToggleButtonTemplate`
- `WindowControlTemplate`
