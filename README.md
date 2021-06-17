# Архиватор

В данном задании вам предстоит реализовать архиватор посредством алгоритма Хаффмана.

EN: https://en.wikipedia.org/wiki/Huffman_coding

RU: https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D0%B4_%D0%A5%D0%B0%D1%84%D1%84%D0%BC%D0%B0%D0%BD%D0%B0

### Предварительная установка

Для выполнения задания потребуются утилиты:
* [git](https://git-scm.com/)
* [cmake](https://cmake.org/)

А также один из компиляторов C++:
* g++
* clang++
* msvc (поставляется с Microsoft Visual Studio)

#### Ubuntu
`sudo apt install git cmake g++ clang++`

#### OSX

`brew install cmake`

`brew install git`

#### Windows

* https://cmake.org/download/
* https://git-scm.com/download/win
* https://visualstudio.microsoft.com/

### Клонирование проекта

1. Создайте в вашей системе директорию для проекта и перейдите в нее в консоли:

   `cd path/to/your/project`
1. Скачайте репозиторий командой:

   `git clone https://gitlab.com/s-mx/archiver.git`
  

### Сборка проекта

В первую очередь необходимо загрузить подмодули gtests, gflags командой:

`git submodule update --init --recursive`

#### Clion

Создайте CMake проект из корня проекта. Должен появиться executable Huffman по умолчанию.
Можете собрать его нажатием `Build/Build Project`. В директории `cmake-build-debug` или `cmake-build-release`
появится исполняемый файл.

Для Release сборки в `Settings/Build, Execution, Deployment/CMake` добавьте release build.
После этого достаточно в выборе конфигурации для сборки выбирать Release.

#### Microsoft Visual Studio

Настоятельно рекомендуем с Visual Studio перейти на связку Mingw + Clion. [Инструкция](https://www.jetbrains.com/help/clion/quick-tutorial-on-configuring-clion-on-windows.html)

По [инструкции](https://docs.microsoft.com/en-us/cpp/build/cmake-projects-in-visual-studio?view=vs-2019)
необходимо установить плагин и настроить проект.

Открываете проект посредством `Open a local folder` в главном меню Visual Studio. В последствии в `Solution Explorer` по кнопке `Switch views` переходите на `CMake Targets View`. Раскрываете список `Huffman Project`. Вы увидете
список целей. Можете раскрыть цель `Huffman (executable)` чтобы увидеть соответствующие исходные коды. Нажатием на цель правой кнопкой мыши выпадает
контекстное меню, в котором можно нажать кнопку `Build`. Также имеются цели `Tests_run (executable)` с юнит тестами, `HuffmanLibrary(static library)`
с файлом, в который вам нужно написать ваше решение. 

#### Linux

1. В корне проекта создайте папку для сборки и перейдите в нее.

   `mkdir build-debug && cd build-debug`

1. `cmake ..`
1. `make`
1. Появится исполняемый файл Huffman.

Для Release сборки нужно написать `cmake -DCMAKE_BUILD_TYPE=Release ..`

### Запуск проекта

[Пример](https://gist.github.com/s-mx/014e788847f1dc4218da91e2c5507a0d) запуска программы.

В файле `main.cpp` написана реализация функции `main()`.

Программа в качестве входных параметров принимает следующие параметры:
* `-compress` - сжать файл.
* `-decompress` - расжать файл.
* `-input` - путь к входному файлу.
* `-output` - путь, по которому должен появиться результат.

### Алгоритм

Алгоритм сжатия  устроен следующим образом:
1. Подсчитывается частотность символов в файле. В задании для упрощения можно бинарный файл считать как массив байтов типа char.
1. Строится бинарный [бор](https://en.wikipedia.org/wiki/Trie) кодирования следующей процедурой:
   1. Для каждого символа добавляется вершина в очередь с приоритетом равным частоте символа в файле.
   1. Пока в очереди больше одного элемента, достаются 2 вершины A и B с минимальными приоритетами. Создается новая вершина С, у которой детьми являются вершины A и B. 
      Вершина C кладется в очередь с приоритетом равным сумме приоритетов вершин A и B.
   1. По окончанию процедуры в очереди остается ровно одна вершина, которая является корнем построенного бора. Листовые вершины являются терминальными.
      В каждой терминальной вершине был написан символ из исходного файла. В дереве ребра к сыновьям могут быть левыми или правыми.
      Левое ребро соответствует биту 0, правое ребро соответствует биту 1.
      Каждой терминальной вершине соответствует битовая последовательность, получающаяся спуском из корня бора к терминальной вершине и выписыванием битов всех пройденных ребер.
      Для наглядности можно воспользоваться следующими иллюстрациями:
   * [gif demo](https://commons.wikimedia.org/wiki/File:Huffmantree_ru_animated.gif?uselang=ru)
   * [gif demo](https://commons.wikimedia.org/wiki/File:Huffman_huff_demo.gif)
   * [graphic demo](https://people.ok.ubc.ca/ylucet/DS/Huffman.html). 
1. Всем символам ставится в соответствие бинарная последовательность посредством построенного бора.
   Остается только заменить все символы файла на соответствующие бинарные последовательности.
   Для возможности обратно раскодировать файл также необходимо в файл записать таблицу кодирования.
   Мы предлагаем вам самим придумать формат кодирования таблицы.
   
Алгоритм декодирования устроен следующим образом:
1. Из файла считывается таблица кодирования.
1. По таблице кодирования строится бинарный бор.
1. Производится проход по бинарным последовательностям и в соответствии с бором в выходной файл выписываются символы из исходного файла.


### Требование к решению

Функции, которые требуется реализовать находятся в файле `lib/Huffman.cpp`. 

```cpp
int Arch::compress(std::istream *in, std::ostream *out) {

}

int Arch::decompress(std::istream *in, std::ostream *out) {

}
```
Только этот файл
в последствии нужно будет отправлять в контест. По этой причине в этой задаче не получится
разделять реализацию на несколько файлов.


Функция `compress` принимает входной, выходной потоки и печатает сжатые данные из потока `in` в поток `out`.
В случае успешной работы, функция возвращает код возврата 0. В случае ошибки, возвращается ненулевой код возврата.
Выкидывать исключения из функции `compress` запрещено.

Функция `decompres` принимает входной, выходной потоки и печатает расжатые данные из потока `in` в поток `out`.
В случае успешной работы, функция возвращает код возврата 0. В случае ошибки, возвращается ненулевой код возврата.
Выкидывать исключения из функции `decompress` запрещено.

Программа должна быть реализована так, чтобы ее можно было запускать на файлах любого размера. Например, 50 GiB. Поэтому,
будет неправильно весь файл поднимать в оперативную память, потому что на современных компьютерах даже 32 GiB RAM роскошь.
Для блочной работы с файлом рекомендуем использовать методы [read](https://en.cppreference.com/w/cpp/io/basic_istream/read) и
[write](https://en.cppreference.com/w/cpp/io/basic_ostream/write).

Рекомендуем следовать принципу [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself):
* Выносите общий код в функции и классы
* Избегайте копипасты кода

Цель данной задачи закрепить знания, полученные в курсе. Поэтому, мы ожидаем от вас
использование стандартной библиотеки C++ STL. Использование Windows API, POSIX API, библиотек из языка C, сторонних библиотек
запрещено. В решении нужно будет работать с файлом в бинарном режиме.
Можете попрактивоваться с этим в задачах 5-ого домашнего контеста.

В вашем коде не должно быть никаких багов, Undefined Behavior и утечек памяти.

Полезные ссылки:
* [Containers](https://en.cppreference.com/w/cpp/container)
* [IO Library](https://en.cppreference.com/w/cpp/io)

### Тестирование решения

Для вас подготовлены юнит тесты в директории `tests/`. Используется фреймворк [Google Test](https://github.com/google/googletest).
В файле `tests/main_tests.cpp` реализован набор тестов `MainTests`, который ваше решение должно полностью проходить.

Также для вашего удобства подготовлен шаблон `StudentTests` в файле `tests/student_tests.cpp`. Туда вы можете писать
свои собственные тесты на вашу реализацию. Это поможет вам сохранить много времени и нервов при отладке. 
[Инструкция](https://github.com/google/googletest/blob/master/googletest/docs/primer.md) по написанию тестов.

В наборе тестов `MainTests` содержатся следующие тесты:
1. `CorrectCompressionDecompression`. Проверяется, что ваше решение вообще работает. Для заданного набора входных строк
   производится компрессия и обратная декомпрессия. Вывод декомпрессии сравнивается с изначальным входом.
   Обратите внимание на возможные крайние случаи в тестах и подумайте, как ваша реализация должна справляться с ними.
1. `MalformedInputInDecompression`. Ваша реализация функции decompress должна быть преспособлена к случаям, когда
   программе на вход подают по ошибке не тот файл. Тест генерирует случайную строку, вызывает от нее `compress` и
   получает сжатые данные. После этого пытается по очереди сломать первые 8 байт побитовым дополнением. Также он
   проверяет, как что функция decompress справляется со случаями, когда в конце файла появляются лишние данные, а также, когда
   наоборот не хватает данных.
1. `CheckSizeOfCompressedContent`. Тест проверяет, что ваше решение вообще сокращает размер файла, а не увеличивает его.

### Clion

Можно добавить конфигурацию по [инструкции](https://www.jetbrains.com/help/clion/creating-google-test-run-debug-configuration-for-test.html#gtest-config)
и запускать тесты кнопкой.

### Microsoft Visual Studio

Также, как в секции про сборку проекта, выбираете цель `Tests_run (executable)`, нажимаете правой кнопкой по целе, в контекстном меню нажимаете
кнопку `Debug`.

### Linux

Достаточно выполнить команды из секции `Сборка проекта` и в директории `build-debug/tests/` или `build-release/tests/`
появится исполняемый файл `Tests_run`, который исполняет тесты.

### Отправка решения

Необходимо отправить решение в контест по ссылке:
https://official.contest.yandex.ru/contest/20948/problems/

### Оценивание

Необходимым условием зачета вашего решения является прохождение тестов `MainTests`.
Также необходимым условием является исправление всех замечаний по коду от ревьюеров.

