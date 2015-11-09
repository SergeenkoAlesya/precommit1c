﻿## Набор утилит для автоматической разборки/сборки внешних обработок и отчетов, при помещении (commit) в git

### Что к чему
----
* v8files-extractor.os (рекомендуется для разборки файлов) - скрипт для OneScript, получающий список помещаемых файлов при коммите, фильтрующий по расширению только  внешние отчёты/обработки и запускающий внешнюю обработку для распаковки этих файлов.
* * pyv8unpack.py - Python-скрипт, выполняющий такую же задачу + задачу сборки
* [V8Reader.epf](http://infostart.ru/public/106310/) - внешняя обработка 1С, которая с помощью  [v8unpack](http://svn2.assembla.com/svn/V8Unpack/track/) разбирает внешние обработки, определяет нормальные  наименования для каталогов форм, файлов модулей объектов и т. д. и раскладывает их в нормальную структуру папок.
* ibService - сервисная база данных на 1С для запуска V8Reader.epf
* pre-commit - собственно командный файл, вызываемый git перед каждым помещением. Выполняет роль простой запускалки  скрипта pyv8unpack.py

### Установка

1. Зависимости: 
    * OneScript http://oscript.io/ (рекомендуется)
    * * или Python 3.3
    * установленная платформа 1С:Предприятия
    * git
    * в случае запуска из под wine необходим msscriptcontrol

2. По умолчанию считается, что пути к oscript.exe и/или python.exe и git.exe находятся в переменной path, иначе необходимо указать явный путь в файлах pre-commit (для oscript/python) и pyv8unpack.py (для git)

3. Путь к платформе находится автоматически в случае стандартной установки 1С. Если необходимо явно указать путь к платформе, то нужно: указать переменную окружения PATH1C c путём к каталогу, в который установлена 1С
```
set PATH1C = d:\program\
```
или создать ini-файл рядом с файлом скрипта pyv8unpack.py или в домашней папке в корне с именем precommit1c.ini и содержанием:
```
[DEFAULT]
onecplatfrorms = c:\program\1cv8\8.3.5.823\bin\1cv8.exe
```

4. Путь хранения исходных текстов разобранных обработок по умолчанию используется как **src** (для обеспечения совместимости со старыми версиями обработки), однако его можно переназначить в ini-файле
```
[DEFAULT]
source = plugin_source
```

5. Флажок, изменяющий корневую папку хранения исходных текстов разобранных обработок по умолчанию используется как корневая папка (для обеспечения совместимости со старыми версиями обработки), однако его можно переназначить в ini-файле. Если изменить флажок на True - в каждой корневой папке внешних обработок будет создан подкаталог текстов разобранных обработок.
```
[DEFAULT]
source_in_source = False
```

6. Наконец, содержимое каталога необходимо скопировать в каталог .git/hooks/ вашего проекта. 
> *Примечание:* каталог .git по умолчанию скрыт.  

```
.git\
    hooks\
        pre-commit
        V8Reader
        ibService 
        v8files-extractor.os
        pyv8unpack.py
```

##Запуск 

После установки достаточно для проверки сделать commit для любого файла epf/erf, и в вашем репозитории автоматически должна создаться папка *src*, полностью повторяющая структуру проекта, изменённые или добавленные файлы распакуются в папки с аналогичными наименованиями. 

##Командная строка запуска OneScript

```
oscript v8files-extractor.os

Утилита сборки/разборки внешних файлов 1С

Параметры командной строки:
        --decompile inputPath outputPath [--debug]
                Разбор файлов на исходники
				inputPath   Путь к файлу, который распаковывается
				outputPath   Путь к каталогу, куда распаковывать
        --help
                Показ этого экрана
        --git-precommit [outputPath] [--debug]
                Запустить чтение индекса из git и определить список файлов для разбора, разложить их и добавить исходники в индекс
				outputPath  Путь к каталогу, куда распаковывать. Если не задан, используется каталог .\src
        --compile inputPath outputPath [--type TYPE] [--debug]
                Собрать внешний файл/обработку.
				inputPath   Путь к файлу, который распаковывается
				outputPath   Путь к каталогу, куда распаковывать
                В параметре --type указывается тип файла для сборки (epf/erf). Значение по умолчанию - epf

Ключ --debug включает режим формирования полных логов, выводимых в консоль
```

##Командная строка запуска Питона

```
python pyv8unpack.py [-h] [--version] [-v] [--index] [--g] [--compile]
                     [--type TYPE] [--platform PLATFORM]
                     [inputPath] [output]


Утилита для автоматической распаковки внешних обработок

positional arguments:
  inputPath            Путь к файлам, необходимым для распаковки
  output               Путь к каталогу, куда распаковывать

optional arguments:
  -h, --help           Show this help message and exit
  --version            Show program's version number and exit
  -v, --verbose        Increases log verbosity for each occurence
  --index              Добавляем в индекс исходники
  --g                  Запустить чтение индекса из git и определить список
                       файлов для разбора
  --compile            Собрать внешний отчёт/обработку
  --type TYPE          Тип файла для сборки epf, erf. По умолчанию авто epf
  --platform PLATFORM  Путь к платформе 1С
```

##Ограничения

Дополнительно необходима настройка git для возможности использования кириллических наименований внешних обработок ```git config --local core.quotepath false```

Не стоит называть файлы с разным расширением epf и erf одинаковыми именами - каталоги с исходниками создаются только по наименованию без учёта расширения и возможен конфликт имен. 

##Что внутри

Как это работает: v8files-extractor.os/pyv8unpack.py полностью повторяет иерархию папок относительно корня репозитория только в папке SRC (от слова source), для каждой изменённой внешней обработки создаётся своя папка и туда с помощью v8unpack распаковывается помещаемая обработка, с помощью v8reader определяются наименования макетов, форм, модуля обработки и переименовываются, переименования сохраняются в служебном файле renames.txt, те файлы, которые невозможно определить или же носят чисто служебный характер, переносятся в каталог *und*
