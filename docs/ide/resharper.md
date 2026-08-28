# Настройка ReSharper

## 1. Основные требования к формату кода

1. В конце файла без отступа.<br/>
2. Внутри файла отступы по 1 пустой строчке (внутри методов можно в принципе можно и несколько, но лучше по 1).<br/>
3. Стиль именования переменных соответствует стандартному в ReSharper.<br/>
4. Порядок расположения членов класса/прочих структур соответствует стандартному в ReSharper.<br/>
5. Строки длиннее 120 знаков (с учетом отступов) разбивать везде, где возможно, про формат разбивки в «дополнительно».<br/>
6. Все using внутри namespace. Сам namespace, крайне желательно, с ";".<br/>
7. У всего публичного крайне желательна XML-документация (три ///).<br/>

## 2. Быстрая настройка

<p align="justify">
Только через ReSharper нельзя настроить автоматическое расположение using и namespace, это надо ещё и в самой студии немного повозиться. Описание что надо делать будет ниже.<br/><br/>
Вот файл конфигурации ReSharper и архив с языковыми библиотеками:<br/>
</p>
<p align="center">
<a href="https://www.dropbox.com/scl/fi/l0i7q67cbofmol9cpmgxm/Settings.DotSettings?rlkey=08gp05z7b7n3mzy5yhderk65t&st=b5060mjy&dl=0)">Файл настроек</a><br/>
<a href="https://www.dropbox.com/scl/fi/jqwyqd2j6jgtmzzq81sfu/Dictionaries.rar?rlkey=odxbjaiuzykmrzln2a0b1t5i1&st=3qrjh1hk&dl=0)">Языковые библиотеки</a><br/>
<a href="https://www.dropbox.com/scl/fi/rrvyxbgr7soxuzj6wmtjk/Exported-2025-11-18.vssettings?rlkey=o0avysby1fjv0nojlehsdx6w2&st=ctrghi4o&dl=0">Настройки XAML Styler</a><br/>
</p>
Вот инструкция в картинках куда их сувать:<br/>

<p align="center">
<img width="900" height="400" alt="image" src="https://github.com/user-attachments/assets/350c05fe-26a2-4ef8-8112-e47c4cb35f44" /><br/>
<img width="900" height="400" alt="image" src="https://github.com/user-attachments/assets/b635329e-88bf-4570-911e-b4bffc8e8146" /><br/>
<img width="900" height="400" alt="image" src="https://github.com/user-attachments/assets/380335e7-dec0-4bb9-8962-60d136a8cc83" /><br/>
<img width="900" height="500" alt="image" src="https://github.com/user-attachments/assets/09a4b406-2666-4e5c-8469-9f6b1b871bb0" /><br/>
<img width="900" height="500" alt="image" src="https://github.com/user-attachments/assets/212d3a8a-7a4a-4902-bdef-2c72819b8437" /><br/>
</p>

Для XAML Styler:

<p align="center">
<img width="900" height="600" alt="image" src="https://github.com/user-attachments/assets/b90d0bfa-1364-47b2-8532-d38eb502975c" />
<img width="900" height="700" alt="image" src="https://github.com/user-attachments/assets/45e086e9-5988-42ff-bb2f-0a52b280a994" />
<img width="900" height="700" alt="image" src="https://github.com/user-attachments/assets/b2b5250e-33f2-48fe-b102-1e9196802b58" />
<img width="900" height="700" alt="image" src="https://github.com/user-attachments/assets/aeb04cc0-b22a-48ac-9762-612ff112eb24" />
</p>

## 3. Действия для ручной настройки

1. В конце файла без отступа:<br/>
По умолчанию выполняется в ReSharper. Если слетело, исправлять тут: <br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/c2366479-0c1b-4784-abe9-828e73104688" /><br/>
</p>

2. Внутри файла отступы по 1 пустой строчке (внутри методов можно в принципе можно и несколько, но лучше по 1):<br/>
По умолчанию стоит возможность 2 пустых строк в коде, изменить можно тут:<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/cfaef1cf-68c9-4796-a697-a70376bce4ff" /><br/>
</p>

Также, по умолчанию в ReSharper не отмечается отсутствие отступов между методами. Чтобы было надо проставить все галочки в "Inspection Severity" -> "Incorrect blank lines":<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/b0d1251a-98e4-437c-ac92-50857186f409" /><br/>
</p>

3. Стиль именования переменных соответствует стандартному в ReSharper.<br/>
По умолчанию ReSharper подстраивается под стандарты именования в проекте, а значит может сбиться сам по себе. Чтобы это отключить, надо убрать галочку тут:<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/c424499d-e114-4720-91f8-c52a0bf7c68d" /><br/>
</p>

4. Порядок расположения членов класса/прочих структур соответствует стандартному в ReSharper:<br/>
Само по себе сбиться, вроде, не может, но если такое произойдет (или просто захочется) – копаться надо тут:<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/365e56a6-ba88-4d34-9447-7dbcc3d6b603" /><br/>
</p>

<p align="justify">
5. Строки длиннее 120 знаков (с учетом отступов) разбивать везде, где возможно, про формат разбивки в «дополнительно»:<br/>
По умолчанию стоит 120, если сбилось, то устанавливается тут:<br/>
</p>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/c55ecc7c-f080-4691-bbb9-533b8ebe1b66" /><br/>
</p>

6. Все using внутри namespace. Сам namespace, крайне желательно, с ";":<br/>
Сделать это чисто через ReSharper нельзя, надо ещё и в студии сделать пару настроек:<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/7e43fbee-ed4c-47d5-a414-fdd206fa8d49" /><br/>
</p>

Настройки в студии:<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/d937a0fe-9e7a-4a22-8681-d048852837d5" /><br/>
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/48e5671f-b2bc-4963-9e40-4e5d1511dee1" /><br/>
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/b8e589f1-b4d6-469e-9d2f-75816a2cf78d" /><br/>
</p>

## 3½. Принудительное комментирование публичных членов

Простого чекбокса для этого нет и в итоге процесс костыльный.<br/>
1. Идем в «Inspection severity» => «C#» => «Compiler warnings» и ищем «Missing XML comment for publicly visible type or member» и меняем «Warning» на «Error».<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/64eed5a9-4aae-4bc2-b7a5-ecb3b7c3d624" /><br/>
</p>

2. В свойствах проекта (да, если в решении несколько проектов, надо все протыкать) в «Build», находим чекбокс для создания XML с документацией и проставляем его (путь можно не проставлять)<br/>

<p align="center">
<img width="700" height="300" alt="image" src="https://github.com/user-attachments/assets/e6cc361a-abee-46f8-896e-c5e9b106abeb" /><br/>
</p>

3.      В «ошибки и предупреждения» проставить код нужной ошибки (CS1591) чтобы считать предупреждение за ошибку. Теперь твой проект просто не соберется пока ты не напишешь все доки :3<br/>

<p align="center">
<img width="700" height="300" alt="image" src="https://github.com/user-attachments/assets/e1e5d197-1050-4102-b3f1-34eb23b8e296" /><br/>
</p>

P.S. Это делать прям совсем необязательно, т.к. теперь при сборке будет создаваться дополнительный файл, а чтобы его не было надо вспомнить и отключить чекбокс со второго пункта. Но если хочется посмотреть не забыл ли где комментарий - хорошая проверка.<br/>

4. Мо умолчанию эти предупреждения будут работать только в открытых файлах, но можно включить анализ кода во всём решении. При этом может немного пострадать производительность, но что поделать:<br/>

<p align="center">
<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/3c679236-88c1-4366-970c-8faae192f435" />
</p>

5. По умолчанию ReSharper глушит оригинальные подчеркивания из студии, но это можно исправить вот тут, если включить назад стандартное выделение из студии (обе галочки должны быть выключены):<br/>

<p align="center">
<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/d19a8a09-c665-4e12-a74e-663e8dc67c70" />
</p>

## 4. Дополнительно

В этом разделе просто набор интересных настроек. Сюда можно свободно добавлять всё что угодно. Часть этих настроек изменена в стандартном файле выше.
1. Выравнивания внутристрочных комментариев (с криком души).<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/e0035ed5-8d1e-45d2-be85-8944b73e1d93" /><br/>
</p>

2. Разные настройки переноса длинных строк при вызове/определении всего у чего есть «()»:<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/2c1968c3-2f3f-46ec-8b94-8b6cfd4b37eb" /><br/>
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/bfd16801-4b75-4f6f-b7ab-cb255884462c" /><br/>
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/0e01eaf8-fef5-4ee6-b17c-51eb2bf8675e" /><br/>
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/5f15a4b1-8d31-4081-8071-4435ab09af91" /><br/>
</p>

3. Отступы для # штук:<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/9d3a10ff-53f7-4121-9587-89db24e96954" /><br/>
</p>

4. Отображение предупреждения при неправильной расстановке пробелов перед/в/после скобок:<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/89ee1fec-9113-4b29-9390-fc44d142795b" /><br/>
</p>

5. Если хочется сделать конкретное изменение в стиле, но неясно, что на него влияет, можно выделить конкретный кусочек кода, нажать Alt+Enter, перейти в "Reformat and cleanup" -> "Configure code style" и в открывшимся окне будут все настройки, которые влияют на конкретно выделенный кусочек кода.<br/>

<p align="center">
<img width="595" height="104" alt="image" src="https://github.com/user-attachments/assets/bc7c33af-9c5f-41ab-8ff5-ce47a6d71920" /><br/>
<img width="292" height="163" alt="image" src="https://github.com/user-attachments/assets/05fface1-1f4f-4b47-8e2b-3d614351dd36" /><br/>
<img width="467" height="205" alt="image" src="https://github.com/user-attachments/assets/c813f128-297f-40e2-adae-5209951e3e88" /><br/>
<img width="1020" height="525" alt="image" src="https://github.com/user-attachments/assets/a10dfe47-b443-48e4-82c8-a5da641fd2ee" /><br/>
</p>

6. Уточняющие скобки в бинарных/арифметических выражениях могут:<br/>
Сперва надо в "InspectionSeverity" найти нужный пункт и проставить галочку и "Warning" или "Erorr".<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/b355db72-b9f8-4dd9-8a8e-9748a9c44df0" /><br/>
</p>

После чего необходимо в "Syntax Style" проставить галочки у всех нужных операторов.<br/>

<p align="center">
<img width="700" height="560" alt="image" src="https://github.com/user-attachments/assets/c0e675d2-7573-4f9b-93db-db9101ca50aa" /><br/>
</p>

Важно: это только дополнительные, уточняющие скобки, они не будут влиять на функционал.
