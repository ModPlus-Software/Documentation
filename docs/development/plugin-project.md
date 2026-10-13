# Создание проекта плагина

Как с нуля создать проект плагина ModPlus для Revit, AutoCAD, nanoCAD или Renga: структура папок, файл решения и файл проекта (csproj) со всеми обязательными свойствами.

---

## Именование {#naming}

Имя плагина состоит из префикса платформы и названия функции латиницей в PascalCase.

| Платформа | Префикс | Пример |
|---|---|---|
| Revit | `mpr` | `mprAlignViews` |
| AutoCAD | `mp` | `mpMultiCopy` |
| nanoCAD | `mnc` | `mncMultiCopy` |
| Renga | `mr` | `mrLevelsList` |

Одно и то же имя используется для папки решения, файла решения, папки проекта, файла csproj и свойства `RootNamespace`.

## Структура папок {#structure}

```text
Functions\<Платформа>\<Имя>\
├── <Имя>.slnx
├── ModPlus.ruleset
└── <Имя>\
    ├── <Имя>.csproj
    ├── Resources\        ← иконки, добавляются перед релизом
    └── ...исходный код
```

`ModPlus.ruleset` берётся из репозитория [ModPlus-Software/Ruleset](https://github.com/ModPlus-Software/Ruleset) и кладётся рядом с файлом решения, а не рядом с csproj: проект ссылается на него через `$(SolutionDir)`.

!!! warning ""
    Не копируйте `ModPlus.ruleset` из других проектов — в них встречаются устаревшие варианты.

## Порядок создания {#steps}

1. В Visual Studio создайте проект **Class Library** (C#) с именем плагина.
2. Полностью замените содержимое csproj [шаблоном](#templates) для своей платформы.
3. Замените в шаблоне `ИМЯ` на имя плагина.
4. Положите `ModPlus.ruleset` из репозитория [ModPlus-Software/Ruleset](https://github.com/ModPlus-Software/Ruleset) рядом с файлом решения.
5. Настройте конфигурации [решения](#solution).
6. Проверьте проект по [чек-листу](#checklist).

Иконки при создании проекта не нужны — обычно их добавляют в конце разработки, перед релизом (см. [Иконки](#icons)).

## Как устроен csproj {#how-it-works}

Все проекты плагинов собираются по одной схеме: **одна конфигурация решения — одна версия целевой программы**. Конфигурации Release нет.

- Имя конфигурации — буква платформы и номер версии: `R2024` (Revit), `A2024` (AutoCAD), `N26.0` (nanoCAD), `R9.2` (Renga).
- Свойство `ExternalVersion` получается из имени конфигурации отрезанием первой буквы: `$(Configuration.Substring(1))`. Для `R2024` это `2024`.
- По `ExternalVersion` выбираются:
    - имя выходной сборки: `mprAlignViews_2024.dll`;
    - константа условной компиляции: `R2024`;
    - пакет API нужной версии: `ModPlus.Revit.Module.2024`;
    - целевая платформа .NET (через условия по конфигурации).
- Конфигурация `Debug` нужна для разработки. В ней `ExternalVersion` задаётся явно: каждый разработчик указывает ту версию программы, на которой тестирует. Значения в шаблонах — только пример.

!!! warning "Константа Debug не нужна"
    Не добавляйте `Debug` в `DefineConstants`: в конфигурации `Debug` SDK сам определяет константу `DEBUG`. В коде используйте только `#if DEBUG` / `#if !DEBUG` — заглавными буквами: директивы препроцессора чувствительны к регистру, и `#if Debug` — это другой символ.

## Обязательные свойства {#properties}

### Идентификация плагина

| Свойство | Значение | Назначение |
|---|---|---|
| `Version` | Четыре числа, например `1.0.0.0` | Версия плагина. Правила изменения — в статье [Версионирование плагинов](plugin-versioning.md) |
| `RootNamespace` | Имя плагина | Корневое пространство имён; от него строится `AssemblyName` |

### Метаданные сборки

Значения одинаковы для всех плагинов, менять их нельзя.

| Свойство | Значение |
|---|---|
| `Company` | `modplus.org` |
| `Authors` | `modplus.org` |
| `Product` | `ModPlus` |
| `AssemblyTitle` | `Plugin for ModPlus software for <Платформа>`, где платформа — `Revit`, `AutoCAD`, `nanoCAD` или `Renga` |
| `Description` | `$(AssemblyTitle)` |
| `Copyright` | `Copyright © ModPlus` |

`AssemblyTitle` должен соответствовать платформе плагина. Частая ошибка — проект nanoCAD, созданный копированием проекта AutoCAD.

### Параметры сборки

| Свойство | Значение | Назначение |
|---|---|---|
| `CodeAnalysisRuleSet` | `$(SolutionDir)\ModPlus.ruleset` | Правила анализа кода (StyleCop) |
| `Platform` | `x64` | Платформа по умолчанию, если csproj собирается без решения (например, `dotnet build` по csproj). При сборке из решения значение приходит из решения |
| `Platforms` | `x64` | Список платформ проекта. Других платформ, кроме x64, быть не должно |
| `PlatformTarget` | `x64` | Архитектура скомпилированной сборки. Программы бывают только 64-битными, поэтому сборка явно помечается как x64 |
| `AppendTargetFrameworkToOutputPath` | `false` | Не добавлять к выходной папке подпапку `net48` / `net8.0-windows` |
| `Optimize` | `false` | Отключает оптимизацию компилятора |
| `Prefer32Bit` | `false` | Не предпочитать 32-битный режим |
| `UseWpf` | `true` | Поддержка WPF (XAML, ресурсы). Нужна во всех плагинах |
| `LangVersion` | `latest` | Последняя версия C#, доступная компилятору |

### Версии программы

| Свойство | Значение | Назначение |
|---|---|---|
| `Configurations` | `Debug;` + список версий (см. [шаблоны](#templates)) | Конфигурации проекта. Каждая версия указывается один раз |
| `TargetFramework` | Зависит от платформы и версии (см. [шаблоны](#templates)) | Целевая платформа .NET |
| `ExternalVersion` | `$(Configuration.Substring(1))`, а в Debug — конкретная версия | Номер версии программы |
| `DefineConstants` | Константа версии (см. [шаблоны](#templates)) | Условная компиляция под версию программы |
| `AssemblyName` | `$(RootNamespace)_$(ExternalVersion)` | Имя DLL с суффиксом версии |

!!! warning "Порядок групп важен"
    Группа с `AssemblyName` должна стоять **после** группы Debug. Иначе в Debug имя сборки будет построено ещё до того, как `ExternalVersion` получит своё значение.

`OutputPath` обязательным не является и в шаблонах не задан. При необходимости его можно указать в группе Debug — путь любой удобный, например:

```xml
<OutputPath>$(appdata)\ModPlus\Functions\Revit\$(RootNamespace)</OutputPath>
```

В других конфигурациях `OutputPath` не задаётся.

### Пакеты

| Пакет | Версия | Атрибуты |
|---|---|---|
| `ModPlus.<Платформа>.Module.$(ExternalVersion)` | Актуальная (см. [шаблоны](#templates)) | `ExcludeAssets="runtime"` |
| `StyleCop.Analyzers` | `1.2.0-beta.556` | `PrivateAssets=all` |

`ExcludeAssets="runtime"` означает, что библиотеки пакета используются только при компиляции и не копируются в выходную папку: во время работы их предоставляет ModPlus.

Остальные пакеты подключаются по необходимости. Пакеты ModPlus (`ModPlusAPI`, `ModPlusStyle` и т. д.) подключаются с `ExcludeAssets="runtime"`.

### Windows Forms

Если плагин использует Windows Forms, добавьте `<UseWindowsForms>true</UseWindowsForms>` в общую группу свойств. По умолчанию это свойство не нужно.

---

## Шаблоны csproj {#templates}

Шаблоны полные: скопируйте целиком и замените `ИМЯ` на имя плагина.

Во всех проектах используется SDK `Microsoft.NET.Sdk`. `Microsoft.NET.Sdk.WindowsDesktop` не используйте: WPF включается свойством `UseWpf`.

=== "Revit"

    - Версии: R2019–R2027.
    - .NET: net48 для 2019–2024, net8.0-windows для 2025–2026, net10.0-windows для 2027.
    - Константа: `R2024`.
    - Debug: в шаблоне версия 2024, указывается своя.

    Если для отладки нужна версия 2025 или новее, добавьте в группу Debug подходящий `TargetFramework`: `net8.0-windows` для 2025–2026, `net10.0-windows` для 2027.

    ```xml title="ИМЯ.csproj"
    <Project Sdk="Microsoft.NET.Sdk">

      <PropertyGroup>
        <Version>1.0.0.0</Version>
        <RootNamespace>ИМЯ</RootNamespace>
      </PropertyGroup>

      <PropertyGroup>
        <Company>modplus.org</Company>
        <Authors>modplus.org</Authors>
        <Product>ModPlus</Product>
        <AssemblyTitle>Plugin for ModPlus software for Revit</AssemblyTitle>
        <Description>$(AssemblyTitle)</Description>
        <Copyright>Copyright © ModPlus</Copyright>
        <CodeAnalysisRuleSet>$(SolutionDir)\ModPlus.ruleset</CodeAnalysisRuleSet>
        <Platform>x64</Platform>
        <Platforms>x64</Platforms>
        <PlatformTarget>x64</PlatformTarget>
        <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
        <Optimize>false</Optimize>
        <Prefer32Bit>false</Prefer32Bit>
        <UseWpf>true</UseWpf>
        <LangVersion>latest</LangVersion>
      </PropertyGroup>

      <PropertyGroup>
        <Configurations>Debug;R2019;R2020;R2021;R2022;R2023;R2024;R2025;R2026;R2027</Configurations>
        <TargetFramework>net48</TargetFramework>
        <ExternalVersion>$(Configuration.Substring(1))</ExternalVersion>
        <DefineConstants>R$(ExternalVersion)</DefineConstants>
      </PropertyGroup>

      <PropertyGroup Condition="'$(Configuration)' == 'R2025' or '$(Configuration)' == 'R2026'">
        <TargetFramework>net8.0-windows</TargetFramework>
      </PropertyGroup>

      <PropertyGroup Condition="'$(Configuration)' == 'R2027'">
        <TargetFramework>net10.0-windows</TargetFramework>
      </PropertyGroup>

      <PropertyGroup Condition="'$(Configuration)' == 'Debug'">
        <ExternalVersion>2024</ExternalVersion>
        <DefineConstants>R$(ExternalVersion)</DefineConstants>
      </PropertyGroup>

      <PropertyGroup>
        <AssemblyName>$(RootNamespace)_$(ExternalVersion)</AssemblyName>
      </PropertyGroup>

      <ItemGroup>
        <PackageReference Include="ModPlus.Revit.Module.$(ExternalVersion)" Version="39.0.0" ExcludeAssets="runtime" />
      </ItemGroup>

      <ItemGroup>
        <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-beta.556">
          <PrivateAssets>all</PrivateAssets>
          <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
      </ItemGroup>

    </Project>
    ```

=== "AutoCAD"

    - Версии: A2013–A2027.
    - .NET: net48 для 2013–2024, net8.0-windows для 2025–2026, net10.0-windows для 2027.
    - Константа: `A2024`.
    - Debug: в шаблоне версия 2024, указывается своя.

    Если для отладки нужна версия 2025 или новее, добавьте в группу Debug подходящий `TargetFramework`: `net8.0-windows` для 2025–2026, `net10.0-windows` для 2027.

    ```xml title="ИМЯ.csproj"
    <Project Sdk="Microsoft.NET.Sdk">

      <PropertyGroup>
        <Version>1.0.0.0</Version>
        <RootNamespace>ИМЯ</RootNamespace>
      </PropertyGroup>

      <PropertyGroup>
        <Company>modplus.org</Company>
        <Authors>modplus.org</Authors>
        <Product>ModPlus</Product>
        <AssemblyTitle>Plugin for ModPlus software for AutoCAD</AssemblyTitle>
        <Description>$(AssemblyTitle)</Description>
        <Copyright>Copyright © ModPlus</Copyright>
        <CodeAnalysisRuleSet>$(SolutionDir)\ModPlus.ruleset</CodeAnalysisRuleSet>
        <Platform>x64</Platform>
        <Platforms>x64</Platforms>
        <PlatformTarget>x64</PlatformTarget>
        <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
        <Optimize>false</Optimize>
        <Prefer32Bit>false</Prefer32Bit>
        <UseWpf>true</UseWpf>
        <LangVersion>latest</LangVersion>
      </PropertyGroup>

      <PropertyGroup>
        <Configurations>Debug;A2013;A2014;A2015;A2016;A2017;A2018;A2019;A2020;A2021;A2022;A2023;A2024;A2025;A2026;A2027</Configurations>
        <TargetFramework>net48</TargetFramework>
        <ExternalVersion>$(Configuration.Substring(1))</ExternalVersion>
        <DefineConstants>A$(ExternalVersion)</DefineConstants>
      </PropertyGroup>

      <PropertyGroup Condition="'$(Configuration)' == 'A2025' or '$(Configuration)' == 'A2026'">
        <TargetFramework>net8.0-windows</TargetFramework>
      </PropertyGroup>

      <PropertyGroup Condition="'$(Configuration)' == 'A2027'">
        <TargetFramework>net10.0-windows</TargetFramework>
      </PropertyGroup>

      <PropertyGroup Condition="'$(Configuration)' == 'Debug'">
        <ExternalVersion>2024</ExternalVersion>
        <DefineConstants>A$(ExternalVersion)</DefineConstants>
      </PropertyGroup>

      <PropertyGroup>
        <AssemblyName>$(RootNamespace)_$(ExternalVersion)</AssemblyName>
      </PropertyGroup>

      <ItemGroup>
        <PackageReference Include="ModPlus.AutoCAD.Module.$(ExternalVersion)" Version="16.0.0" ExcludeAssets="runtime" />
      </ItemGroup>

      <ItemGroup>
        <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-beta.556">
          <PrivateAssets>all</PrivateAssets>
          <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
      </ItemGroup>

    </Project>
    ```

=== "nanoCAD"

    - Версии: N25.0 и N26.0.
    - .NET: net6-windows для всех версий.
    - Константа: точка в номере версии заменяется на подчёркивание — `N26_0` (идентификатор C# не может содержать точку).
    - Debug: в шаблоне версия 26.0, указывается своя.

    ```xml title="ИМЯ.csproj"
    <Project Sdk="Microsoft.NET.Sdk">

      <PropertyGroup>
        <Version>1.0.0.0</Version>
        <RootNamespace>ИМЯ</RootNamespace>
      </PropertyGroup>

      <PropertyGroup>
        <Company>modplus.org</Company>
        <Authors>modplus.org</Authors>
        <Product>ModPlus</Product>
        <AssemblyTitle>Plugin for ModPlus software for nanoCAD</AssemblyTitle>
        <Description>$(AssemblyTitle)</Description>
        <Copyright>Copyright © ModPlus</Copyright>
        <CodeAnalysisRuleSet>$(SolutionDir)\ModPlus.ruleset</CodeAnalysisRuleSet>
        <Platform>x64</Platform>
        <Platforms>x64</Platforms>
        <PlatformTarget>x64</PlatformTarget>
        <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
        <Optimize>false</Optimize>
        <Prefer32Bit>false</Prefer32Bit>
        <UseWpf>true</UseWpf>
        <LangVersion>latest</LangVersion>
      </PropertyGroup>

      <PropertyGroup>
        <Configurations>Debug;N25.0;N26.0</Configurations>
        <TargetFramework>net6-windows</TargetFramework>
        <ExternalVersion>$(Configuration.Substring(1))</ExternalVersion>
        <DefineConstants>N$(ExternalVersion.Replace('.', '_'))</DefineConstants>
      </PropertyGroup>

      <PropertyGroup Condition="'$(Configuration)' == 'Debug'">
        <ExternalVersion>26.0</ExternalVersion>
        <DefineConstants>N$(ExternalVersion.Replace('.', '_'))</DefineConstants>
      </PropertyGroup>

      <PropertyGroup>
        <AssemblyName>$(RootNamespace)_$(ExternalVersion)</AssemblyName>
      </PropertyGroup>

      <ItemGroup>
        <PackageReference Include="ModPlus.nanoCAD.Module.$(ExternalVersion)" Version="2.0.0" ExcludeAssets="runtime" />
      </ItemGroup>

      <ItemGroup>
        <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-beta.556">
          <PrivateAssets>all</PrivateAssets>
          <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
      </ItemGroup>

    </Project>
    ```

=== "Renga"

    - Версии: R5.1–R9.2.
    - .NET: net48 для всех версий.
    - Константа: точка заменяется на подчёркивание — `R9_2`.
    - Debug: в шаблоне версия 9.2, указывается своя.

    Отличия от других платформ:

    - обязательна ссылка `COMFileReference` на `RengaCOMAPI.tlb` из пакета `ModPlus.Renga.API.$(ExternalVersion)` с параметрами `EmbedInteropTypes=False` и `Private=False`;
    - обязательна ссылка `<Reference Include="Microsoft.CSharp" />`;
    - иконки подключаются иначе (см. [Иконки](#icons)).

    ```xml title="ИМЯ.csproj"
    <Project Sdk="Microsoft.NET.Sdk">

      <PropertyGroup>
        <Version>1.0.0.0</Version>
        <RootNamespace>ИМЯ</RootNamespace>
      </PropertyGroup>

      <PropertyGroup>
        <Company>modplus.org</Company>
        <Authors>modplus.org</Authors>
        <Product>ModPlus</Product>
        <AssemblyTitle>Plugin for ModPlus software for Renga</AssemblyTitle>
        <Description>$(AssemblyTitle)</Description>
        <Copyright>Copyright © ModPlus</Copyright>
        <CodeAnalysisRuleSet>$(SolutionDir)\ModPlus.ruleset</CodeAnalysisRuleSet>
        <Platform>x64</Platform>
        <Platforms>x64</Platforms>
        <PlatformTarget>x64</PlatformTarget>
        <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
        <Optimize>false</Optimize>
        <Prefer32Bit>false</Prefer32Bit>
        <UseWpf>true</UseWpf>
        <LangVersion>latest</LangVersion>
      </PropertyGroup>

      <PropertyGroup>
        <Configurations>Debug;R5.1;R6.0;R7.0;R7.4;R8.0;R8.8;R8.9;R9.0;R9.2</Configurations>
        <TargetFramework>net48</TargetFramework>
        <ExternalVersion>$(Configuration.Substring(1))</ExternalVersion>
        <DefineConstants>R$(ExternalVersion.Replace('.', '_'))</DefineConstants>
      </PropertyGroup>

      <PropertyGroup Condition="'$(Configuration)' == 'Debug'">
        <ExternalVersion>9.2</ExternalVersion>
        <DefineConstants>R$(ExternalVersion.Replace('.', '_'))</DefineConstants>
      </PropertyGroup>

      <PropertyGroup>
        <AssemblyName>$(RootNamespace)_$(ExternalVersion)</AssemblyName>
      </PropertyGroup>

      <ItemGroup>
        <PackageReference Include="ModPlus.Renga.Module.$(ExternalVersion)" Version="16.0.0" ExcludeAssets="runtime" />
      </ItemGroup>

      <ItemGroup>
        <COMFileReference Include="$(NuGetPackageRoot)\ModPlus.Renga.API.$(ExternalVersion)\2.0.0\lib\RengaCOMAPI.tlb">
          <EmbedInteropTypes>False</EmbedInteropTypes>
          <Private>False</Private>
        </COMFileReference>
      </ItemGroup>

      <ItemGroup>
        <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-beta.556">
          <PrivateAssets>all</PrivateAssets>
          <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
        </PackageReference>
        <Reference Include="Microsoft.CSharp" />
      </ItemGroup>

    </Project>
    ```

---

## Иконки {#icons}

На этапе разработки иконки не обязательны, поэтому в шаблонах их нет. Обычно их добавляют в конце, перед релизом, и тогда же добавляют в csproj группу элементов с иконками.

Ссылки на иконки добавляются в csproj вместе с самими файлами, не раньше.

=== "Revit, AutoCAD, nanoCAD"

    Нужны четыре файла в папке `Resources`, тип сборки — `Resource`:

    | Файл | Назначение |
    |---|---|
    | `<Имя>_16x16.png` | Маленькая иконка, светлая тема |
    | `<Имя>_16x16_dark.png` | Маленькая иконка, тёмная тема |
    | `<Имя>_32x32.png` | Большая иконка, светлая тема |
    | `<Имя>_32x32_dark.png` | Большая иконка, тёмная тема |

    Если в плагине несколько команд, такой набор из четырёх файлов нужен для каждой команды, с её именем.

    Тип сборки — именно `Resource`, а не `EmbeddedResource`, `Content` или `None`.

    Группа элементов добавляется после групп с пакетами:

    ```xml
    <ItemGroup>
      <Resource Include="Resources\ИМЯ_16x16.png" />
      <Resource Include="Resources\ИМЯ_16x16_dark.png" />
      <Resource Include="Resources\ИМЯ_32x32.png" />
      <Resource Include="Resources\ИМЯ_32x32_dark.png" />
    </ItemGroup>
    ```

=== "Renga"

    Иконки лежат в двух папках, тёмных вариантов нет:

    - `Icons\<Имя>_16x16.png` и `Icons\<Имя>_24x24.png` — тип сборки `EmbeddedResource`;
    - `Resources\<Имя>_16x16.png` и `Resources\<Имя>_32x32.png` — тип сборки `Resource`.

    ```xml
    <ItemGroup>
      <EmbeddedResource Include="Icons\ИМЯ_16x16.png" />
      <EmbeddedResource Include="Icons\ИМЯ_24x24.png" />
    </ItemGroup>

    <ItemGroup>
      <Resource Include="Resources\ИМЯ_16x16.png" />
      <Resource Include="Resources\ИМЯ_32x32.png" />
    </ItemGroup>
    ```

## Решение {#solution}

Решение создаётся только в формате `.slnx`. Формат `.sln` для новых проектов не используется.

Конфигурации решения должны совпадать с конфигурациями проекта. Платформа решения — только `x64`.

```xml title="ИМЯ.slnx (Revit)"
<Solution>
  <Configurations>
    <BuildType Name="Debug" />
    <BuildType Name="R2019" />
    <BuildType Name="R2020" />
    <BuildType Name="R2021" />
    <BuildType Name="R2022" />
    <BuildType Name="R2023" />
    <BuildType Name="R2024" />
    <BuildType Name="R2025" />
    <BuildType Name="R2026" />
    <BuildType Name="R2027" />
    <Platform Name="x64" />
  </Configurations>
  <Project Path="ИМЯ/ИМЯ.csproj">
    <Platform Project="x64" />
  </Project>
</Solution>
```

Для других платформ список `BuildType` берётся из свойства `Configurations` соответствующего [шаблона](#templates).

---

## Чего делать нельзя {#donts}

- **Личные конфигурации** (`DebugИванов`, `Debug2025` и т. п.). Чтобы отлаживать на другой версии, достаточно поменять `ExternalVersion` в группе Debug на своей машине, не коммитя это изменение.
- **Абсолютные пути** (`D:\...`, `C:\Users\...`) везде, кроме `OutputPath` в группе Debug. В остальных местах — только `$(SolutionDir)`, `$(NuGetPackageRoot)` и относительные пути.
- **Константа `Debug`** в `DefineConstants` и **`#if Debug`** в коде. Правильно: `#if DEBUG`.
- **Повторы в `Configurations`** — каждая версия указывается один раз.
- **Одно свойство дважды**, в том числе в другом регистре (`UseWpf` и `UseWPF`, `OutputPath` и `outputpath`).
- **Конфигурация `Release`** и платформа `AnyCPU`.
- **Служебные файлы в репозитории**: `*_wpftmp.csproj`, `* - Backup.csproj`, архивы старых версий проекта. `*_wpftmp.csproj` — временные файлы сборки WPF; если они остались в папке проекта, их надо удалить.
- **`AssemblyTitle` от другой платформы** после копирования проекта.
- **Устаревшие версии пакетов**: `StyleCop.Analyzers 1.1.118`, старые версии Module.

## Чек-лист {#checklist}

Перед первым коммитом:

- [ ] Имя плагина с правильным префиксом, одинаковое для папки, решения, csproj и `RootNamespace`.
- [ ] `ModPlus.ruleset` из репозитория Ruleset лежит рядом с файлом решения.
- [ ] `AssemblyTitle` соответствует платформе.
- [ ] `Platform`, `Platforms` и `PlatformTarget` равны `x64` и стоят в общей группе свойств.
- [ ] Каждая версия в `Configurations` указана один раз; решение содержит те же конфигурации.
- [ ] В Debug: `ExternalVersion` задана, `TargetFramework` ей соответствует, `Debug;` в `DefineConstants` нет.
- [ ] Группа с `AssemblyName` стоит после группы Debug.
- [ ] Пакет Module актуальной версии подключён с `ExcludeAssets="runtime"`.
- [ ] `StyleCop.Analyzers 1.2.0-beta.556` подключён с `PrivateAssets=all`.
- [ ] В csproj нет личных конфигураций и абсолютных путей (кроме `OutputPath` в Debug).
- [ ] `OutputPath`, если задан, — только в группе Debug.
- [ ] Проект собирается в конфигурации Debug и хотя бы в одной конфигурации каждой версии .NET (например, R2024, R2025, R2027).

Перед релизом дополнительно:

- [ ] Для каждой команды добавлены все иконки с нужным типом сборки (см. [Иконки](#icons)).
