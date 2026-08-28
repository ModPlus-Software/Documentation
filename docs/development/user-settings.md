# Работа с пользовательскими настройками

В проекте реализованы три уровня работы с пользовательскими настройками:

1. **UserConfigFile** — низкоуровневый статический сервис для работы с единым общим файлом настроек (`mpConfig.mpcf`).
   ⚠️ **Важно:** в своих проектах использовать напрямую не нужно — всегда используйте `UserConfigFileUtils`.
2. **UserConfigFileUtils** — удобная обёртка над `UserConfigFile` для плагинов. Позволяет хранить простые значения, привязанные к имени плагина.
3. **UserSettingsService** — сервис для работы с отдельными XML-файлами, когда нужно хранить сложные объекты или коллекции.

---

## 1. UserConfigFile

* Управляет единым файлом `%appdata%/ModPlus Data/mpConfig.mpcf`.
* Позволяет читать и записывать строковые значения в XML-структуру.
* Использовать напрямую не требуется — только через `UserConfigFileUtils`.

---

## 2. UserConfigFileUtils

### Назначение

* Работает с файлом `%appdata%/ModPlus Data/mpConfig.mpcf`.
* Автоматически создаёт узел для вашего плагина (по имени плагина).
* Упрощает работу с простыми типами: `string`, `int`, `double`, `bool`, `enum`.
* Позволяет работать через свойства, без ручного указания строк.

### Пример использования

```csharp
public class MyPluginSettings
{
    private readonly UserConfigFileUtils _settings;

    public MyPluginSettings()
    {
        _settings = new UserConfigFileUtils(ModPlusConnector.Instance.Name);
    }

    public string UserName
    {
        get => _settings.GetValue("DefaultUser");
        set => _settings.SetValue(value);
    }

    public int WindowWidth
    {
        get => _settings.GetValue(800);
        set => _settings.SetValue(value);
    }

    public bool EnableFeatureX
    {
        get => _settings.GetValue(false);
        set => _settings.SetValue(value);
    }

    public DisplayMode Mode
    {
        get => _settings.GetValue(DisplayMode.Normal);
        set => _settings.SetValue(value);
    }
}
```

👉 В большинстве случаев этого сервиса **достаточно для плагинов**.

---

## 3. UserSettingsService

### Назначение

* Работает с отдельными XML-файлами.
* По умолчанию сохраняет в `%appdata%/ModPlus UserData` (или в подпапке, если указан `subFolder`).
* Поддерживает сериализацию объектов и коллекций.
* При необходимости можно явно указать путь с помощью `SetToFile` и `GetFromFile`.

---

### Пример: сохранение и загрузка объекта

```csharp
public class MyCustomData
{
    public string Name { get; set; }
    public int Value { get; set; }
}

var service = new UserSettingsService(ModPlusConnector.Instance.Name);

// Сохранение объекта
service.Set(new MyCustomData { Name = "Test", Value = 42 });

// Загрузка объекта
var data = service.Get<MyCustomData>();
Console.WriteLine($"{data.Name} = {data.Value}");
```

### Пример: работа с коллекцией

```csharp
var list = new List<object>
{
    new MyCustomData { Name = "Item1", Value = 10 },
    new MyCustomData { Name = "Item2", Value = 20 }
};

service.Set(list, new List<string> { "FirstItem", "SecondItem" });

// Загрузка по конкретному имени узла
var second = service.GetFromFile<MyCustomData>(
    Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
                 "ModPlus UserData",
                 ModPlusConnector.Instance.Name + ".xml"),
    "SecondItem");
```

### Пример: обновление через `Set<T>`

```csharp
service.Set<MyCustomData>(data =>
{
    // Загруженные данные или новый объект, если файла ещё нет
    // Здесь мы обновляем только нужные поля
    data.Value += 1;
    data.Name = DateTime.Now.ToShortDateString();
});
```

👉 Метод `Set<T>` позволяет:

1. Загрузить данные (или создать новый объект, если данных нет).
2. Изменить их в переданном `Action<T>`.
3. Сразу сохранить обратно в файл.

Это **короткий способ обновления настроек**, без необходимости вручную вызывать `Get → изменить → Set`.

---

### Нюансы UserSettingsService

* Сохраняемые классы должны иметь:

  * публичный конструктор без параметров;
  * свойства с `get; set;`.
* Для **value-type (int, double, bool)** и для **коллекций** обязательно указывать `nodeName`.
* В XML удаляются `xmlns`-атрибуты для читаемости.

---

## Сравнительная таблица

| Сервис                  | Где хранит данные                         | Что хранит лучше всего             | Когда использовать                                                                | Пример узла                                                    |
| ----------------------- | ----------------------------------------- | ---------------------------------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| **UserConfigFile**      | `%appdata%/ModPlus Data/mpConfig.mpcf`    | Простые строки                     | ⚠️ Не использовать напрямую. Только через `UserConfigFileUtils`.                  | `<MyPlugin ... />`                                             |
| **UserConfigFileUtils** | `%appdata%/ModPlus Data/mpConfig.mpcf`    | Строки, числа, bool, enum          | Для **плагинов**: хранение простых параметров через свойства классов              | `<MyPlugin WindowWidth="800" EnableFeatureX="true" />`         |
| **UserSettingsService** | `%appdata%/ModPlus UserData/[Plugin].xml` | Объекты и коллекции (сериализация) | Когда нужно хранить **сложные структуры** или коллекции, отдельно от общего файла | `<MyPlugin><MyCustomData Name="Test" Value="42" /></MyPlugin>` |
