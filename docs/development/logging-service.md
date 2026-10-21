# Сервис логирования для плагинов

Для использования необходимо подключить nuget-пакет ModPlusAPI версии не ниже 45.0 (если в пакете модуля версия ниже)!

```<PackageReference Include="ModPlusAPI" Version="45.0.0" ExcludeAssets="runtime" />```

---
Сервис логирования для плагинов ModPlus. Пишет текстовые лог файлы в папку пользовательских данных ModPlus, не требует настройки и не бросает исключений при проблемах с записью в файл.
 
Пространство имён: `ModPlusAPI.Services`
Сборка: `ModPlusAPI`
 
## Содержание
 
- [Быстрый старт](#быстрый-старт)
- [Расположение лог файлов](#расположение-лог-файлов)
- [Время жизни лог файла](#время-жизни-лог-файла)
- [Формат записи](#формат-записи)
- [Логирование исключений](#логирование-исключений)
- [Несколько лог файлов в подпапке](#несколько-лог-файлов-в-подпапке)
- [Потокобезопасность](#потокобезопасность)
- [Обработка ошибок](#обработка-ошибок)
- [Справочник API](#справочник-api)
- [Рекомендации](#рекомендации)
 
## Быстрый старт
 
```csharp
using ModPlusAPI.Enums;
using ModPlusAPI.Services;
 
var logger = new LoggerService("mprDemoFunction");
 
logger.LogInfo("Плагин запущен");
logger.LogWarning("Не найден файл настроек, используются значения по умолчанию");
 
try
{
    // полезная работа
    logger.LogSuccess("Обработано 42 элемента");
}
catch (Exception exception)
{
    logger.Log(exception);
}
```
 
В каждом плагине есть класс `ModPlusConnector`, реализующий `IModPlusPluginGeneral`. Если передать его экземпляр в конструктор, имя лог файла указывать не нужно — оно будет взято из свойства `Name`:
 
```csharp
/// <summary>
/// Коннектор плагинов ModPlus
/// </summary>
public class ModPlusConnector : IModPlusPluginGeneral
{
    /// <inheritdoc />
    public SupportedProduct SupportedProduct => SupportedProduct.Revit;
 
    /// <inheritdoc />
    public string Name => nameof(DemoFunction);
 
    /// <inheritdoc />
    public string Price => "0";
}
```
 
Сам сервис логирования создаётся там, где начинается работа плагина, — в команде или в контексте окна:
 
```csharp
[Transaction(TransactionMode.Manual)]
public class DemoFunctionCommand : IExternalCommand
{
    public Result Execute(ExternalCommandData commandData, ref string message, ElementSet elements)
    {
        var logger = new LoggerService(new ModPlusConnector(), LogFileLifetime.Daily);
        logger.LogInfo("Плагин запущен");
 
        try
        {
            var context = new MainContext(logger);
            // ...
            return Result.Succeeded;
        }
        catch (Exception exception)
        {
            logger.Log(exception);
            return Result.Failed;
        }
    }
}
```
 
## Расположение лог файлов
 
Все лог файлы создаются в корневой папке логов:
 
```
%AppData%\ModPlus\UserData\_logs
```
 
Путь доступен через статическое свойство `LoggerService.LogsRootDirectory`. Папка создаётся автоматически при обращении к свойству, если её нет.
 
При указании подпапки в конструкторе файлы создаются в ней:
 
```
%AppData%\ModPlus\UserData\_logs\<subDirectory>
```
 
Полный путь к папке текущего экземпляра сервиса — свойство `LogsDirectory`, полный путь к основному лог файлу — свойство `LogFilePath`.
 
Расширение лог файла — `.log` (константа `LogFileExtension`). Имя файла в конструкторе указывается **без расширения**; если расширение всё же указано, оно отбрасывается.
 
> Имя файла и имя подпапки не должны содержать недопустимых для имени файла символов, а также `/`, `\` и `:`. Иначе конструктор бросит `ArgumentException`. Значения обрезаются по краям (`Trim`).
 
## Время жизни лог файла
 
Задаётся перечислением `LogFileLifetime` (параметр `lifetime` конструктора).
 
| Значение | Поведение | Имя файла |
|---|---|---|
| `Session` (по умолчанию) | Файл **перезаписывается** при создании экземпляра сервиса. Лог содержит записи только текущего запуска плагина | `mprDemoFunction.log` |
| `Daily` | На каждый день создаётся отдельный файл с суффиксом даты. Устаревшие файлы удаляются | `mprDemoFunction_2026-08-10.log` |
| `Daily` с хранением 1 день | Ведётся один файл без даты в имени, который пересоздаётся каждый день | `mprDemoFunction.log` |
 
### Срок хранения дневных лог файлов
 
По умолчанию дневные лог файлы хранятся `LoggerService.DailyLogFilesLifetimeInDays` дней (7 дней). Срок можно уменьшить, передав в конструктор параметр `dailyLogFilesLifetimeInDays` — от `MinDailyLogFilesLifetimeInDays` (1) до `DailyLogFilesLifetimeInDays` (7) включительно. Значение вне диапазона приводит к `ArgumentOutOfRangeException`. Заданный срок доступен через свойство `DailyLogFilesLifetimeDays`; при времени жизни `Session` параметр не учитывается.
 
```csharp
// Дневные лог файлы хранятся 3 дня
var logger = new LoggerService(new ModPlusConnector(), LogFileLifetime.Daily, 3);
 
// С подпапкой
var subDirectoryLogger = new LoggerService("mprDemoFunction", LogFileLifetime.Daily, 3, "DemoFunction");
```
 
Очистка устаревших файлов выполняется при создании экземпляра сервиса и при первой записи в новый дневной файл (например, после смены даты).
 
Удаляются **только файлы того же ряда** — то есть файлы вида `<имя>_*.log` в той же папке. Логи других плагинов и файлы с временем жизни `Session` (у них нет суффикса с датой) не затрагиваются.
 
Файл считается устаревшим, только если устарели **все** доступные даты: дата из имени файла, дата создания и дата последнего изменения. Берётся наиболее поздняя из них. Если файл был переименован извне и дату из имени получить нельзя, учитываются только даты файловой системы — такой файл всё равно будет удалён по истечении срока.
 
### Единственный дневной лог файл
 
Если задать срок хранения равным `MinDailyLogFilesLifetimeInDays` (1 день), ведётся **один лог файл без даты в имени** — имя такое же, как при `Session`:
 
```csharp
var logger = new LoggerService(new ModPlusConnector(), LogFileLifetime.Daily, LoggerService.MinDailyLogFilesLifetimeInDays);
```
 
Поведение такого файла:
 
- если файл создан в предыдущие дни (или его нет), он создаётся заново; в пределах одного дня записи накапливаются, в том числе от разных экземпляров сервиса и разных запусков плагина — в отличие от `Session`, файл не перезаписывается при каждом создании экземпляра;
- дата создания определяется по атрибутам файла. Проверка выполняется при создании экземпляра сервиса и при первой записи после смены даты, поэтому долго работающий экземпляр сам начнёт новый файл на следующий день;
- если файл не удалось пересоздать (например, он заблокирован другим процессом), запись всё равно выполняется, а попытка пересоздания повторяется при следующей записи;
- после пересоздания удаляются устаревшие файлы того же ряда с датой в имени — например, оставшиеся с тех пор, когда срок хранения был больше одного дня.
 
## Формат записи
 
Каждый вызов метода логирования добавляет в файл одну запись (одну строку, если текст однострочный):
 
```
[2026-08-10 14:23:07.512] [INFO] [Execute] Плагин запущен
```
 
Состав записи:
 
1. `[дата и время]` — локальное время в формате `yyyy-MM-dd HH:mm:ss.fff`;
2. `[ТИП]` — тип записи в верхнем регистре. Отсутствует, если тип `ResultItemType.None`;
3. `[имя метода]` — имя метода, из которого выполнялось логирование. Подставляется автоматически через `[CallerMemberName]`, но может быть задано явно последним параметром;
4. текст записи.
 
Многострочный текст смещается вправо на 4 пробела, начиная со второй строки, — так одна запись визуально отделяется от следующей:
 
```
[2026-08-10 14:23:08.004] [ERROR] [Execute] System.InvalidOperationException: Элемент не найден
       в Demo.Function.Execute()
       в Demo.Command.Run()
```
 
Кодировка файла — UTF-8, записи добавляются в конец файла.
 
Доступные типы записи (`ResultItemType`): `None`, `Info`, `Warning`, `Error`, `Success`.
 
## Логирование исключений
 
Перегрузка `Log(Exception)` разворачивает всю цепочку внутренних исключений: для каждого уровня пишутся полное имя типа, сообщение и стек вызовов, уровни разделяются строкой `--- Внутреннее исключение ---`.
 
```csharp
catch (Exception exception)
{
    logger.Log(exception);              // тип записи ERROR по умолчанию
    logger.Log(exception, ResultItemType.Warning);
}
```
 
Если передать `null`, будет брошено `ArgumentNullException`.
 
## Несколько лог файлов в подпапке
 
Когда плагину нужно несколько лог файлов, задайте подпапку — тогда станут доступны методы записи в произвольные файлы внутри неё:
 
```csharp
var logger = new LoggerService("mprDemoFunction", LogFileLifetime.Daily, "DemoFunction");
 
logger.LogInfo("Общее сообщение");                         // ...\_logs\DemoFunction\mprDemoFunction_2026-08-10.log
logger.LogToFile("export", "Экспортировано 10 файлов");    // ...\_logs\DemoFunction\export_2026-08-10.log
logger.LogToFile("export", exception);
 
var path = logger.GetLogFilePath("export");
```
 
Методы `LogToFile` и `GetLogFilePath` требуют заданной подпапки: без неё они бросают `InvalidOperationException`. Время жизни у дополнительных файлов такое же, как у основного, и обрабатываются они по тем же правилам.
 
## Потокобезопасность
 
Сервис потокобезопасен. Запись в файл синхронизируется по полному пути к файлу через статическую коллекцию объектов синхронизации, поэтому корректно работают и несколько потоков, и несколько экземпляров `LoggerService`, пишущих в один файл в пределах процесса.
 
Запись добавляется в файл сразу при вызове метода логирования — буферизации нет, освобождать сервис (`Dispose`) не требуется.
 
## Обработка ошибок
 
Логирование не должно нарушать работу плагина, поэтому ошибки файловых операций **не приводят к исключениям**:
 
- при неудачной записи выполняется до 5 попыток с паузой 30 мс (файл может быть временно заблокирован другим процессом), после чего запись молча пропускается;
- ошибки удаления файлов (перезапись `Session`-файла, очистка устаревших `Daily`-файлов) игнорируются.
 
Исключения бросают только конструктор и методы, работающие с подпапкой, — при некорректных аргументах:
 
| Исключение | Причина |
|---|---|
| `ArgumentException` | Имя файла или подпапки не задано либо содержит недопустимые символы |
| `ArgumentOutOfRangeException` | Срок хранения дневных лог файлов вне диапазона 1–7 |
| `ArgumentNullException` | Не задан плагин в конструкторе или не задано исключение в `Log`/`LogToFile` |
| `InvalidOperationException` | Вызов `LogToFile`/`GetLogFilePath` без заданной подпапки |
 
## Справочник API
 
### Конструкторы
 
```csharp
LoggerService(string fileName, LogFileLifetime lifetime = LogFileLifetime.Session, string subDirectory = null)
LoggerService(string fileName, LogFileLifetime lifetime, int dailyLogFilesLifetimeInDays, string subDirectory = null)
LoggerService(IModPlusPluginGeneral plugin, LogFileLifetime lifetime = LogFileLifetime.Session, string subDirectory = null)
LoggerService(IModPlusPluginGeneral plugin, LogFileLifetime lifetime, int dailyLogFilesLifetimeInDays, string subDirectory = null)
```
 
| Параметр | Описание |
|---|---|
| `fileName` | Имя основного лог файла без расширения |
| `plugin` | Плагин, имя лог файла берётся из `IModPlusPluginGeneral.Name` |
| `lifetime` | Время жизни лог файла |
| `dailyLogFilesLifetimeInDays` | Срок хранения дневных лог файлов в днях — от 1 до 7 включительно. Учитывается только при `LogFileLifetime.Daily`. В конструкторах без этого параметра — 7 |
| `subDirectory` | Имя подпапки в корневой папке логов. `null` — файлы создаются в корневой папке |
 
Конструктор сразу подготавливает основной файл: для `Session` — удаляет старый файл, для `Daily` — удаляет устаревшие файлы, а для единственного дневного файла ещё и пересоздаёт его, если он создан в предыдущие дни. Нужные папки при этом создаются.
 
### Свойства
 
| Свойство | Тип | Описание |
|---|---|---|
| `LogsRootDirectory` | `static string` | Корневая папка всех лог файлов — `%AppData%\ModPlus\UserData\_logs` |
| `FileName` | `string` | Имя основного лог файла без расширения |
| `SubDirectory` | `string` | Имя подпапки или `null` |
| `Lifetime` | `LogFileLifetime` | Время жизни лог файла |
| `DailyLogFilesLifetimeDays` | `int` | Срок хранения дневных лог файлов в днях. Учитывается только при `Daily` |
| `LogsDirectory` | `string` | Папка лог файлов текущего экземпляра сервиса |
| `LogFilePath` | `string` | Полный путь к основному лог файлу |
 
> `LogsRootDirectory` и `LogsDirectory` создают папку при обращении к свойству, если её нет.
 
### Методы
 
| Метод | Описание |
|---|---|
| `Log(string message, ResultItemType type = None, string memberName = null)` | Запись в основной файл с указанным типом |
| `Log(Exception exception, ResultItemType type = Error, string memberName = null)` | Запись об исключении в основной файл |
| `LogInfo(string message, string memberName = null)` | Запись типа `Info` |
| `LogWarning(string message, string memberName = null)` | Запись типа `Warning` |
| `LogError(string message, string memberName = null)` | Запись типа `Error` |
| `LogSuccess(string message, string memberName = null)` | Запись типа `Success` |
| `LogToFile(string fileName, string message, ResultItemType type = None, string memberName = null)` | Запись в отдельный файл в подпапке |
| `LogToFile(string fileName, Exception exception, ResultItemType type = Error, string memberName = null)` | Запись об исключении в отдельный файл в подпапке |
| `GetLogFilePath(string fileName)` | Полный путь к файлу с указанным именем в подпапке |
 
Параметр `memberName` помечен атрибутом `[CallerMemberName]` и заполняется компилятором — передавать его вручную нужно только в особых случаях (например, при логировании из общего вспомогательного метода).
 
### Константы
 
| Константа | Значение | Описание |
|---|---|---|
| `LogsDirectoryName` | `"_logs"` | Имя папки с лог файлами |
| `LogFileExtension` | `".log"` | Расширение лог файла |
| `DailyLogFilesLifetimeInDays` | `7` | Срок хранения дневных лог файлов по умолчанию и максимально допустимый срок |
| `MinDailyLogFilesLifetimeInDays` | `1` | Минимально допустимый срок хранения дневных лог файлов. При этом значении ведётся один файл без даты в имени |
 
## Рекомендации
 
- Создавайте один экземпляр `LoggerService` на плагин и переиспользуйте его: каждое создание экземпляра при `Session` перезаписывает лог файл, а при `Daily` запускает очистку устаревших файлов.
- Используйте `LogFileLifetime.Session` для отладки текущего запуска, `LogFileLifetime.Daily` — когда нужна история за несколько дней, а `Daily` со сроком хранения 1 день — когда нужен лог всех запусков за текущий день в одном файле с постоянным именем.
- Задавайте подпапку, если плагин пишет более одного лог файла: так логи не смешиваются с логами других плагинов в корневой папке.
- В имени лог файла используйте уникальное имя плагина с префиксом разработчика (например, `mprDemoFunction`) — файлы всех плагинов лежат рядом.
- Не логируйте персональные данные и содержимое, которое пользователь не готов передать в поддержку: лог файлы обычно прикладываются к обращениям.
