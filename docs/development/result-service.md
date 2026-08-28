# Сервис ResultService

Сервис предназначен для **сбора, хранения и отображения результатов работы** в окне результатов. Он позволяет добавлять записи различных типов (информация, предупреждение, ошибка, успех и др.), а затем отображать их в удобном виде — сгруппированными или без группировки.

---

## Основные возможности

* Добавление записей в хранилище:

  * запись с сообщением и типом;
  * запись с сообщением, отдельным элементом (например, идентификатором объекта) и типом.
* Удаление последней добавленной записи.
* Полная очистка хранилища.
* Проверка, пусто ли хранилище.
* Подсчет количества записей определенного типа.
* Отображение результатов в разных режимах:

  * **По типу** — группировка по типу сообщения (`Info`, `Warning`, `Error`, `Success`, `None`).
  * **По времени** — в порядке добавления.
  * **Без группировки** — все записи последовательно, с возможностью отображения времени.

⚠️ **Важно**: если в хранилище отсутствуют результаты, вызов методов `Show`, `ShowByType`, `ShowByTime` или `ShowWithoutGrouping` **не откроет окно результатов**.

---

## Типы результатов

Типы результатов определяются перечислением `ResultItemType`:

* `None` — без типа
* `Info` — информационное сообщение
* `Warning` — предупреждение
* `Error` — ошибка
* `Success` — успешное выполнение

---

## Основные методы

### Добавление записей

```csharp
// Добавление записи с сообщением
void Add(string message, ResultItemType resultItemType = ResultItemType.None);

// Добавление записи с сообщением и элементом
void Add(string message, string item, ResultItemType resultItemType = ResultItemType.None);
```

### Удаление и очистка

```csharp
// Удаляет последнюю добавленную запись
void DeleteLastAdded();

// Очищает все записи
void Clear();
```

### Проверка состояния

```csharp
// Возвращает количество записей указанного типа
int Count(ResultItemType resultItemType);

// Проверяет, есть ли записи
bool IsEmpty();
```

### Отображение результатов

```csharp
// Отображение результатов (по умолчанию — сгруппированные по типу)
void Show(string title = null, string separator = "; ");
void Show(IntPtr ownerHandle, string title = null, string separator = "; ");
void Show(ModPlusWindow ownerWindow, string title = null, string separator = "; ");

// Отображение с группировкой по типу
void ShowByType(string title = null, string separator = "; ");

// Отображение в порядке добавления
void ShowByTime(string title = null, string separator = "; ");

// Отображение без группировки
void ShowWithoutGrouping(string title = null, bool printTime = false);
```

---

## Пояснение по перегрузкам методов `Show*`

Методы `Show`, `ShowByType`, `ShowByTime`, `ShowWithoutGrouping` имеют **перегрузки** с параметрами:

* `IntPtr ownerHandle` — позволяет указать **дескриптор окна-владельца** (например, окно приложения или плагина).
* `ModPlusWindow ownerWindow` — позволяет указать **окно ModPlus** в качестве владельца.

Использование перегрузок с указанием родительского окна рекомендуется для правильного модального поведения окна результатов и привязки его к главному окну приложения.

---

## Пример использования

```csharp
var service = new ResultService();

// Добавляем разные записи
service.Add("Обработка завершена", ResultItemType.Success);
service.Add("Не удалось найти элемент", ResultItemType.Warning);
service.Add("Идентификатор элемента:", "12345", ResultItemType.Info);

// Отображаем результаты по типу
service.ShowByType("Результаты обработки");

// Отображаем результаты по времени
service.ShowByTime("Хронология");

// Отображаем без группировки с выводом времени
service.ShowWithoutGrouping("Все записи", printTime: true);
```
