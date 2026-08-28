# Сборка ClosedXML

1. Клонируем решение https://github.com/ClosedXML/ClosedXML.git

2. В свойствах проекта добавляем **CopyLocalLockFileAssemblies** со значением **true**

<img width="584" height="78" alt="image" src="https://github.com/user-attachments/assets/a802dd01-6d05-4624-9473-a1b960511608" />

Почему-то без этой опции в выходной каталог не добавляются все зависимые dll

3. Подключаем к проекту ссылку на [ILRepack](https://github.com/gluck/il-repack). По сути это не обязательно - можно сделать один раз, чтобы нужные dll появились в локальном кэше библиотек из nuget. Можно просто со страницы проекта скачать exe

<img width="831" height="74" alt="image" src="https://github.com/user-attachments/assets/e8b4cb46-4cdf-4c13-afad-6c2417b9d406" />

4. Выбираем нужную ветку (с которой построен нужный/последний релиз), включаем конфигурацию Release и выполняем сборку

5. Переходим в папку с собранными файлами \ClosedXML\ClosedXML\bin\Release\netstandard2.0 и запускаем в ней командную строку (именно cmd, а не PowerShell)

**Мы используем именно net standard 2.0, так как он будет работать и с Net48 и с Net8 и остальными. Net Standard 2.1 не будет работать с Net48**

6. В командной строке пишем и выполняем:

```
"C:\Users\cjmod\.nuget\packages\ilrepack\2.0.41\tools\ILRepack.exe" /internalize /out:"ModPlus.Excel.dll" "ClosedXML.dll" "ClosedXML.Parser.dll" "DocumentFormat.OpenXml.dll" "DocumentFormat.OpenXml.Framework.dll" "ExcelNumberFormat.dll" "RBush.dll" "SixLabors.Fonts.dll" "Microsoft.Bcl.HashCode.dll" "System.Buffers.dll" "System.IO.Packaging.dll" "System.Memory.dll" "System.Numerics.Vectors.dll" "System.Runtime.CompilerServices.Unsafe.dll"
```
