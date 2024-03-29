# 13.01.2024 Эволюция средств обеспечения безопасности памяти

## Содержание

- [13.01.2024 Эволюция средств обеспечения безопасности памяти](#13012024-эволюция-средств-обеспечения-безопасности-памяти)
    - [Содержание](#содержание)
    - [Введение](#введение)
    - [Безопасность и ошибки при работе с памятью](#безопасность-и-ошибки-при-работе-с-памятью)
    - [Эволюция средств обеспечения безопасности памяти](#эволюция-средств-обеспечения-безопасности-памяти)
        - [Сборка мусора. Lisp](#сборка-мусора-lisp)
        - [Структурные блоки. ALGOL 60](#структурные-блоки-algol-60)
        - [Проверка выхода за границы массива. ALGOL 60](#проверка-выхода-за-границы-массива-algol-60)
        - [Тэгированная архитектура. B5000](#тэгированная-архитектура-b5000)
        - [Сегментная память. B5000](#сегментная-память-b5000)
        - [RAII. C++](#raii-c)
        - [Stack protector. GCC](#stack-protector-gcc)
        - [Рандомизация размещения адресного пространства. Linux](#рандомизация-размещения-адресного-пространства-linux)
        - [Интерпретация машинного кода. Valgrind](#интерпретация-машинного-кода-valgrind)
        - [Аллокатор спец. назначения. DieHard](#аллокатор-спец-назначения-diehard)
        - [Инструменация кода. Address Sanitazer](#инструменация-кода-address-sanitazer)
        - [Тэгированная архитектура. LowRISC](#тэгированная-архитектура-lowrisc)
        - [Тэгированная архитектура. Arm MTE](#тэгированная-архитектура-arm-mte)
        - [Lifetimes \& Borrow Checker. Rust](#lifetimes--borrow-checker-rust)
    - [Вывод](#вывод)
    - [Ссылки](#ссылки)

## Введение

Ручное управление памятью сложно и подвержено ошибкам, которые могут повлечь серьезные проблемы с безопасностью. Поэтому очень важно ответственно подходить к разработке и использовать весь доступный инструментарий для обеспечения работы с памятью.

## Безопасность и ошибки при работе с памятью

Безопасность памяти - состояние программы при котором все ее доступы к памяти корректны.

Доступ к памяти корректен тогда и только тогда, когда его выполнение не может привести к ошибке программы.

Ошибка программы - ошибка, приводящая к неожиданному поведению программы и, как следствие, выдаче некорректного результата.

## Эволюция средств обеспечения безопасности памяти

Все время развития компьютерных наук человечество пытается решить обсуждаемую проблему различными способами. Давайте рассмотрим некоторые из них.

### Сборка мусора. Lisp

Поскольку ручное управление памятью привносит с собой возможность ошибиться, то можно попытаться избежать таких ошибок, введя механизм автоматического управления памятью - сборку мусора - который сам бы выделял и освобождал память, не допустив нас до опасного инструментария. Такое решение было принято при дизайне языка программирования LISP в 1959 году. В рантайм языка ввели процесс reclamation, позволявший программисту не думать о памяти, он отмечал недоступную память как мусор. Кроме того, сам язык был достаточно высокоуровневым и не оперировал такими понятиями, как указатель и массив. Ввиду отсутствия возможности ручного управления памятью LISP, многие считают его memory-safe языком. В будущем идея сборки мусора обеспечит успех вычислительной платформе Java.

- <http://www-formal.stanford.edu/jmc/recursive.pdf>
- <https://www.gnu.org/software/emacs/manual/html_node/elisp/Garbage-Collection.html>

### Структурные блоки. ALGOL 60

Код на языке Алгол 60 состоял из блоков. Блок представлял из себя объявления локальных переменных и список инструкций.

```ada
begin integer X;
  X := 5;
  begin integer X;
    X := 4;
  end
  print (X);
end;
```

Выделение памяти под локальные переменные происходило на стеке при входе в блок, а освобождение - при выходе. Возможность аллокации на куче отсутствовала, что решало обсуждаемую проблему на уровне пользователя.

### Проверка выхода за границы массива. ALGOL 60

Кроме того, в Алгол 60 была введена обязательная проверка нахождения индекса в рамках массива (range-check), вставляемая компилятором. С тех пор большинство языков программирования предпочитают не опускать проверки при доступе к элементам массива. Ненужные вхождения range-check могут быть убраны компилятором, что снижает накладные расходы.

- <https://en.wikipedia.org/wiki/ALGOL_60>
- <https://web.eecs.umich.edu/~bchandra/courses/papers/Naure_Algol60.pdf>
- <https://www.cl.cam.ac.uk/teaching/0910/ConceptsPL/Algol-Pascal.pdf>
- <https://jcsites.juniata.edu/faculty/rhodes/lt/blockstr.htm>

### Тегированная память. B5000

В 1961 году мир увидел мэйнфрейм B5000 со стековой архитектурой. Одной из интересных нововведений данной машины было использование нескольких бит в машинных словах для кодирования типа хранимых данных, чтобы различать неинициализированные данные, код, индексы, числа с плавающей точкой, адреса возврата и др. Это помогало предотвращать перезапись адреса возврата, использование неинициализированных переменных, неправильную интерпретацию данных.

- <https://www.memorymanagement.org/glossary/t.html#tagged.architecture>
- <https://homepage.cs.uiowa.edu/~dwjones/arch/notes/11burroughs.html>
- <https://dl.acm.org/doi/pdf/10.1145/3533704>
- <https://en.wikipedia.org/wiki/Tagged_architecture>
- <https://en.wikipedia.org/wiki/Burroughs_Large_Systems>

### Сегментная память. B5000

Также B5000 имел сегментную память с разграничением доступа. Каждому сегменту соответствовал набор флагов, характеризующих ограничения на доступ к памяти: чтение, запись, исполнение.

- <https://en.wikipedia.org/wiki/Memory_segmentation>
- <https://homes.cs.washington.edu/~levy/capabook/Chapter1.pdf>

### RAII. C++

С++ зародился в 1985 году и представил миру идиому Resource Acquisition Is Initialization - механизм, связывающий захват и освобождение ресурса с временем жизни соответствующего ему объекта. Идиома использует структурные блоки, определяющие время жизни объекта. При создании объекта ресурс захватывается в коде конструктора, а отпускается в деструкторе, который вызывается в конце блока, либо во время раскрутки стека. Для того, чтобы объект смог пережить область видимости, в которой объявлен, была введена семантика перемещения. Таким образом мы не управляем памятью вручную, а делегируем это высокоуровневым абстракциям. Подход используется в том числе и языке программирования Rust!

- <https://en.cppreference.com/w/cpp/language/raii>
- <https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization>

### Stack protector. GCC

Канарейки, защищающие адрес возврата из функции, предотвращают атаки с помощью переполнения буфера на стеке. Компилятор инструментирует код программы, вставляя в сгенерированный ассемблер инструкции, помещающие специальное значение перед адресом возврата, а далее проверяющие то, что оно не изменилось перед выходом из функции. Таким образом при попытке перезаписи адреса возврата будет перезаписана канарейка, наш код это заметит и аварийно завершит выполнение, не допустив атаку. Чтобы атакующий не смог подменить канарейку, ее значение генерируется случайно средствами ОС.

- <https://en.wikipedia.org/wiki/Buffer_overflow_protection#Canaries>

### Рандомизация размещения адресного пространства. Linux

В 2001 году в Linux была добавлена "Рандомизация расположения адресного пространства". Этот метод защиты, который привносит случайности в адреса памяти процессов в попытке предотвратить атаки, основанные на знании точного местоположения объектов процесса.

- <https://www.mdpi.com/2076-3417/9/14/2928>

### Интерпретация машинного кода. Valgrind

В 2002 году был создан Valgrind - набор инструментов, которые могут автоматически обнаруживать множество ошибок управления памятью и потоковой передачей и детально профилировать программы. Его преимуществом является хорошая точность и отсутствие необходимости иметь исходный код программы. Valgrind Memcheck следит за корректностью выполняемых действий в программе, интерпретируя машинный код, из-за чего сильно замедляет выполнение, что делает его непригодным для отладки больших приложений и запуска в продуктовой среде.

- <https://valgrind.org/docs/pubs.html>

### Аллокатор спец. назначения. DieHard

Очередными неинтрузивными инструментами для отладки работы с памятью являются специальные аллокаторы памяти, использующие механизм виртуальной памяти для аллокации блоков так, чтобы выход за их пределы вызывал Segmentation Fault. Они размещают блоки в конце или начале новой страницы, а следующую за ней страницу отмечают недоступной для записи и чтения. Это позволяет с большей вероятностью обнаружить выход за пределы массива.

- <https://github.com/emeryberger/DieHard>

### Инструментация кода. Address Sanitazer

AddressSanitizer - инструмент на основе компилятора для обнаружения ошибок памяти. Он обнаруживает: stack/heap buffer overflow, use after free/scope, double/wild free. Принцип работы заключается в том, чтобы аннотировать недоступную память и во время выполнения проверять доступы к памяти на корректность. Делается это через инструментацию кода. В среднем замедляет программу в 2 раза, но может быть гибко настроен для использования в продуктовой среде, а также имеет Hardware Accelerated реализации на основе тегирования памяти.

- <https://www.usenix.org/system/files/conference/atc12/atc12-final39.pdf>

### Тегированная память. LowRISC

Еще пример, в 2014 году LowRISC предложили ввести по 2битному тегу, хранимые в отдельной области памяти на каждый байт информации в памяти.

- 00 - No effect
- 01 - Exception on read
- 10 - Exception on write
- 11 - Exception on read or write

С их помощью аппаратура может проверять доступ к памяти на корректность и кидать исключение при обнаружении ошибки. Для снижения отрицательного эффекта на производительность программы предлагается делать это параллельно. Для поддержки данного механизма требуется добавление команд для управления тегами в систему команд, а также обновление компиляторов и аллокаторов. Также заметим, что такой метод не позволяет обнаружить ошибочный доступ к объекту А, попавший в объект Б, в отличие от следующего решения.

- <https://www.cl.cam.ac.uk/~jrrk2/downloads/lowRISC-memo-2014-001.pdf>

### Тегированная архитектура. Arm MTE

ARM Memory Tagging Extension - было внедрено в их процессоры в 2015 году. Идея заключается в том, чтобы отдать 4 неиспользуемые бита адреса указателя на некоторый тэг, означающий цвет. Байты памяти также аннотируются цветом в отдельно выделенной области памяти. При разыменовании указателя аппаратура асинхронно проверяет совпадение цветов указателя и соответствующей памяти. Поскольку есть шанс совпадения цветов, данный метод защиты является вероятностным.

- <https://www.youtube.com/watch?v=UwMt0e_dC_Q>
- <https://arxiv.org/pdf/2209.00307.pdf>

### Lifetimes & Borrow Checker. Rust

Главной особенностью языка Rust является расширение системы типов специальными тэгами, аннотирующими время жизни объекта и позволяющие компилятору на этапе семантического анализа находить dangling-reference и use-after-free. С помощью аннотаций времени жизни Rust дополняет RAII, делает его удобнее и безопаснее.

- <https://doc.rust-lang.org/1.8.0/book/references-and-borrowing.html>

## Вывод

Во времена, когда люди много писали на ассемблере, были популярны аппаратные методы защиты памяти. Далее, с развитием языков программирования, мы получили инструменты статического анализа кода и автоматического управления памятью - эти направления активно развиваются и по сей день. Кроме того, мы начали инструментировать код, добавляя в него дополнительные проверки, в том числе срабатывающие лишь с определенной вероятностью - это оказывается хорошим обменом на производительность на практике. Однако замедление программ из-за инструментации ощутимо, поэтому все чаще мы узнаем о новых методах аппаратной защиты памяти, напоминающие нам те самые тегированные архитектуры.

## Ссылки

1. <https://www.cse.cuhk.edu.hk/~taoyf/course/comp3506/lec/ram.pdf>
2. <https://digital.car.chula.ac.th/cgi/viewcontent.cgi?article=3284&context=chulaetd>
3. <https://en.wikipedia.org/wiki/Memory_safety>
