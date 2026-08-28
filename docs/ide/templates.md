# Гайд по созданию шаблонов

## 1. Предисловие

В этом гайде рассматриваются стандартные способы создания всех 3 видов шаблонов, поддерживаемые в Visual Studio (и, может быть в VS Code), а именно:<br/>
1. Шаблоны проектов;<br/><br/>
2. Шаблоны элементов;
3. Фрагменты кода (code snippet) в чистой Visual Studio;<br/>
4. Фрагменты кода (code snippet) с ReSharper.<br/>

## 2. Шаблоны проектов 

2.1 Создаем основу, это можно сделать и на пустом проекте<br/>

2.1.1. Открываем мастер шаблонов<br/>

<p align="center">
<img width="403" height="461" alt="image" src="https://github.com/user-attachments/assets/76d515f9-997e-41f1-b85a-137a20b6c25d" /><br/>
</p>

2.1.2. Выбираем нужный тип шаблона<br/>

<p align="center">
<img width="974" height="738" alt="image" src="https://github.com/user-attachments/assets/0c573407-4d6e-4d76-919e-1eeebcca0943" /><br/>
</p>

2.1.3. По желанию сразу заполняем название/описание/добавляем иконку (можно не делать сразу, а вписать потом). Так же желательно сразу не импортировать шаблон, т.к. он импортируется без дополнительного форматирования, которое будет нужно почти в любом случае. Но если не надо, то дальнейшие шаги можно пропустить.<br/>

<p align="center">
<img width="974" height="740" alt="image" src="https://github.com/user-attachments/assets/050809f8-f7f3-48c3-a708-0c6b5a38a61b" /><br/>
</p>

2.2. Редактируем шаблон<br/>

2.2.1. Открываем папку куда импортировали шаблон и распаковываем архив. Получаем примерно следующее:<br/>

<p align="center">
<img width="974" height="203" alt="image" src="https://github.com/user-attachments/assets/d900606f-5fa9-415e-b670-ce5fb0a01aa2" /><br/>
</p>

Что это такое:<br/>
1._TemplateIcon.ico – стандартная иконка, которая вставляется в шаблон, если не выбрана кастомная.<br/>
2.MyTemplate.vstemplate – файл с метаданными по шаблону (название, описание и пр.).<br/>
3.Всё остальное – непосредственно файлы шаблона.<br/>

2.2.2. Редактируем шаблон (*.csproj открывает сразу всю папку и даже синтаксические ошибки будут подсвечиваться).<br/>

На этом этапе крайне желательно докрутить вообще всё (лучше не оставлять даже мелкие недочеты, т.к. потом их исправлять будет уже лень).<br/>
Так же надо расставить форматирование шаблонов. В общем случае это выглядит так:<br/>
`$TemplateSchemaVariableName$`<br/>
Список элементов можно посмотреть тут:<br/>
https://learn.microsoft.com/en-us/visualstudio/ide/template-parameters?view=vs-2022<br/>
А наглядно посмотреть что они делают тут:<br/>
https://www.reddit.com/r/learnprogramming/comments/w24f5q/visual_studio_2019_templates_safeprojectname/<br/>
**Важно:** не все элементы форматирования ведут себя одинаково в шаблонах проектов и шаблонах элементов.<br/>

<p align="center">
<img width="379" height="366" alt="image" src="https://github.com/user-attachments/assets/0e4c89ca-add6-4020-bb03-3ef47cfb0576" /><br/>
</p>

Более продвинутые способы форматирования описаны в «дополнительно».<br/>

2.2.3. Редактируем метаданные шаблона<br/>

<p align="center">
<img width="974" height="387" alt="image" src="https://github.com/user-attachments/assets/15090f51-1d05-4e21-b10a-e80738d43f2e" /><br/>
</p>

Что делают основные теги (их сильно больше, может в «дополнительно» будет их описание):<br/>
* Name – название шаблона, отображаемое в меню выбора шаблонов;<br/>
* Description – описание шаблона, отображаемое в меню выбора шаблонов;<br/>
* ProjectType– основной тип проекта по языку. Можно написать что угодно, но при изменении крайне желательно нагуглить стандартные названия. Используется при фильтрации в меню выбора шаблонов;<br/>
* ProjectSubType – подтип проекта по технологии/назначению. Можно писать что угодно, для попадания в одну из уже существующих категорий в меню выбора шаблонов, достаточно просто указать её название (чувствительно к регистру);<br/>
* SortOrder – чем выше – тем выше по списку будет шаблон. В документации написано, что для пользовательских шаблонов ничего не делает;<br/>
* CreateNewFolder – нужно ли создавать новую папку под шаблон. Может меняться пользователем при создании проекта из шаблона;<br/>
* DefaultName – название по умолчанию для проекта, создаваемого по шаблону. Может меняться пользователем при создании проекта из шаблона;<br/>
* ProvideDefaultName – использовать ли название из DefaultName при создании;<br/>
* LocationField – отображать ли поле с путем сохранения при создании проекта из шаблона;<br/>
* EnableLocationBrowserButton – отображать ли кнопку для вызова проводника для выбора пути сохранения при создании проекта из шаблона;<br/>
* CreateInPlace – определяет где *создаются* файлы для проекта. True – создаются прямо по указанному пути. False – создаются в месте к которому есть доступ у VisualStudio и потом копируются по указанному пути. Нужно для запар с доступом;<br/>
* Icon – имя иконки, отображаемой в меню выбора шаблонов. Должно соответствовать имени иконки из архива;<br/>
* TemplateContent – здесь располагаются ссылки на все элементы шаблона. Мультипроектные шаблоны будут описаны в «дополнительно»;<br/>
* Project – тег с описанием проекта, “File” должен указывать на *.csproj проекта;<br/>
* ProjectItem – отдельные файлы из шаблона. Название внутри тега должно соответствовать названию файла в архиве;<br/>
* TargetFileName – название файла после создания проекта по шаблону. Можно создавать папки, для этого надо указать путь через «/» пример на скриншоте. Внутри можно использовать элементы форматирования;<br/>
* ReplaceParameters – нужно ли искать в файле элементы форматирования;<br/>

2.2.4. Импортируем шаблон в Visual Studio.<br/>

Архивируем все файлы шаблона в архив .zip (названия файлов значения не имеют, они будут заменены согласно настройкам, главное чтобы соответствовали указанным в файле метаданных).<br/>
Вставляем ахрив по пути (можно не ложить в отдельную папку для языков, но желательно):<br/>
`%USERPROFILE%\Documents\Visual Studio <version>\Templates\ProjectTemplates`<br/>
И всё - шаблон готов к использованию после перезагрузки студии. <br/>
**Баг:** если создавать шаблон с открытым файлом из шаблона (или файлом с таким же названием), то может не работать замена элементов форматирования.<br/>

<p align="center">
<img width="974" height="652" alt="image" src="https://github.com/user-attachments/assets/b086780d-0ef9-457e-b0f4-d50bd777d938" /><br/>
<img width="548" height="602" alt="image" src="https://github.com/user-attachments/assets/8928d0aa-e4da-466c-93d7-a21950cca45f" /><br/>
<img width="727" height="273" alt="image" src="https://github.com/user-attachments/assets/aa8f7b9e-1985-4dfe-bd81-d7d8ebb7773b" /><br/>
</p>

**Примечание:** Важно отметить, что код из шаблона не обязательно должен компилироваться, или даже соответствовать синтаксису, для создания проекта по шаблону.<br/>

## 3. Шаблоны элементов 

Создание почти полностью соответствует шаблону проекта. Отличия по порядку создания:<br/>

3.1. При изначальном экспорте надо выбрать другой пункт<br/>

<p align="center">
<img width="974" height="738" alt="image" src="https://github.com/user-attachments/assets/b5682cb0-8e23-4b5f-9e37-d7f188fba77f" /><br/>
<img width="974" height="738" alt="image" src="https://github.com/user-attachments/assets/b7fd4ecd-05fc-41e0-a7ea-5fd1cc1078d2" /><br/>
<img width="974" height="741" alt="image" src="https://github.com/user-attachments/assets/3b2228fc-752d-4fcd-a6fb-7605a977dab5" /><br/>
</p>

3.2. В файле с метаданными поменьше тегов и отсутствует тег «Project»:<br/>

<p align="center">
<img width="974" height="264" alt="image" src="https://github.com/user-attachments/assets/6f9c2885-6607-4257-bd7f-7eb63a0e99ed" />
</p>

**Важно:** не все элементы форматирования ведут себя одинаково в шаблонах проектов и шаблонах элементов.<br/>
**Баг:** если создавать шаблон с открытым файлом из шаблона (или файлом с таким же названием), то может не работать замена элементов форматирования.<br/>

## 4. Фрагменты кода (code snippet) в чистой Visual Studio

4.1. Создаем пустой файл и открываем его Visual Studio (желательно создать .xml чтобы подсветка кода работала, но не обязательно).<br/>

<p align="center">
<img width="614" height="67" alt="image" src="https://github.com/user-attachments/assets/0971b9d5-f63f-4791-87b2-f6af9a85af99" />
</p>

4.2. Вставляем в него стандартный шаблон фрагмента кода и редактируем теги:<br/>

```
<?xml version="1.0" encoding="utf-8"?>
<CodeSnippets xmlns="http://schemas.microsoft.com/VisualStudio/2005/CodeSnippet">
    <CodeSnippet Format="1.0.0">
        <Header>
            <Title></Title>
	    <Shortcut></Shortcut>
	    <Description></Description>
	    <Author></Author>
	    <SnippetTypes>
		<SnippetType>SurroundsWith</SnippetType>
            </SnippetTypes>
        </Header>
        <Snippet>
            <Code Language="csharp">
                <![CDATA[]]>
            </Code>
        </Snippet>
    </CodeSnippet>
</CodeSnippets>
```

Title – Название фрагмента кода, отображаемое пользователю;<br/>
Shortcut – сокращение, по которому можно будет быстро вставить фрагмент;<br/>
Description – описание фрагмента кода, отображаемое пользователю;<br/>
Author – автор, отображается только в отдельном меню. Необязательный тег;<br/>
SnippetTypes – теги для фрагмента кода, нужны для сортировки в отдельном меню;<br/>
SurroundsWith – указывает, что фрагментом кода можно окружить выделенную область (используются $selected$ и $end$).<br/>
4.3. Вставляем код в CDATA[]:<br/>

```
<?xml version="1.0" encoding="utf-8"?>
<CodeSnippets xmlns="http://schemas.microsoft.com/VisualStudio/2005/CodeSnippet">
    <CodeSnippet Format="1.0.0">
        <Header>
            <Title>MySnippet</Title>
	    <Shortcut>snippet</Shortcut>
	    <Description>snippet</Description>
	    <Author>ME</Author>
	    <SnippetTypes>
		<SnippetType>SurroundsWith</SnippetType>
            </SnippetTypes>
        </Header>
        <Snippet>
            <Code Language="csharp">
                <![CDATA[var a = "myString";

		$selected$ $end$

                var b = 8;
		]]>
            </Code>
        </Snippet>
    </CodeSnippet>
</CodeSnippets>
```

4.4. Импортируем фрагмент кода:<br/>

Меняем расширение у файла на .snippet и закидываем по пути:<br/>
`%USERPROFILE%\Documents\Visual Studio <version>\Code Snippets\Visual C#\My Code Snippets`<br/>
"My Code Snippets" можно заменить на другое название, или вообще положить прямо в папку языка, это влияет только на то, где надо будет искать фрагмент при долгом способе вставки.<br/>
Вот и всё, фрагмент кода готов к использованию после перезагрузки студии. Для использования достаточно вбить сокращение, указанное в теге <Shortcut>, или вставить его через сочетание Ctrl+K+S.<br/>
Важно: у фрагментов кода, созданных данным способом не будут работать шорткаты с установленным ReSharper, но их всё ещё можно будет использовать через сочетание клавиш.<br/>

## 5. Фрагменты кода (code snippet) с ReSharper

У них документация сильно лучше чем у microsoft, так что можно просто там про это посмотреть:<br/>
https://www.jetbrains.com/help/resharper/Reference__Template_Editor.html#general<br/>
Но если хочется только тут, то вот минимальный набор:<br/>

5.1. Открываем меню фрагментов кода:<br/>

<p align="center">
<img width="917" height="585" alt="image" src="https://github.com/user-attachments/assets/a704e169-aaf7-4322-a939-bf05ff63cf79" />
</p>

5.2. Открываем меню создание новых фрагментов кода:<br/>

<p align="center">
<img width="317" height="107" alt="image" src="https://github.com/user-attachments/assets/43bdf66f-c47c-4f21-9e6e-29a22c55b855" />
</p>

5.3. Вводим основные данные по фрагменту кода:<br/>

<p align="center">
<img width="299" height="260" alt="image" src="https://github.com/user-attachments/assets/8c92a5de-6cce-4d1d-8dc4-667d1b1bf823" />
</p>

* Shortcut – сокращение, по которому можно будет быстро вставить фрагмент;<br/>
* Description – описание фрагмента кода, отображаемое пользователю;<br/>
* Reformat – форматировать ли фрагмент кода согласно прочим настройкам ReSharper при вставке;<br/>
* Shorten qualified references – нужно ли сокращать ссылки на namespace по возможности при вставке фрагмента кода;<br/>
* Availiability – указывает, где доступен фрагмент кода;<br/>
* Use in - при каких условиях доступен фрагмент кода (при написании шортката, при окружении, и там и там);<br/>
* Mnemonic - ещё более быстрый шорткат;<br/>
* Show in context action - добавляет фрагмент кода в контекстное окно при окружении.<br/>

5.4. Создаем нужный фрагмент (описание как работать с параметрами в «дополнительно»):<br/>

<p align="center">
<img width="232" height="88" alt="image" src="https://github.com/user-attachments/assets/f5eba9de-db5d-4d66-89b2-2bff2a736cab" /><br/>
</p>

Вот и всё, фрагмент готов к использованию (надо прожать Ctrl+S в меню создания):<br/>

<p align="center">
<img width="648" height="399" alt="image" src="https://github.com/user-attachments/assets/7b64b35d-5781-4639-ace1-cba2b544b590" /><br/>
<img width="547" height="141" alt="image" src="https://github.com/user-attachments/assets/d1818e1a-ae3c-41c9-a5d1-43e7f9c52559" /><br/>
<img width="314" height="83" alt="image" src="https://github.com/user-attachments/assets/cf94139a-333a-47e5-a21c-0dc96073a934" /><br/>
</p>
