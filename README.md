# Экспериментирование с использованием Rust для разработки web-приложений

Rust кажется очень перспективным инструментом разработки, в том числе, применимым для разработки web-приложений. Rust позволяет генерировать компактные и быстрые нативные приложения. Забегая чуть вперёд, можно утверждать, что исполняемый файл (exe) для Windows x64-86, с примитивной реализацией web-сервера занимает всего 1,5 Мб и стартует быстрее, чем за секунду.

Rust часто сравнивают с языком программирования Go, но у этих языков разные цели и области применения. "_Gophers value simplicity, Rustaceans value safety_". Оба языка являются компилируемыми, но в Go используется Garbage Collector. Производительность и надёжность кода у Rust выше. Соответственно, ключевые области применения Rust: _mission critical code_ и приложения, требующие максимальной вычислительной эффективности.

На Rust был переписан **репозитарий npm** и DNS-сервера от Cloudflare. Отмечается, что Rust-приложения обладают отличной предсказуемостью операций выделения памяти, чем не могут похвастаться приложения на Go, Java, JavaScript и платформы .NET. 

Rust используют такие компании, как VK, Twitter и Dropbox. Также на Rust написан движок Quantum (Mozilla Firefox).

В статье [Размышление о двух подходах к C++](https://habr.com/ru/companies/ispsystem/articles/867992/) by omyhosts, указывается, что из-за разногласий в комитете по стандартизации C++, этот язык стагнирует и крупные IT-компании смотрят именно в сторону Rust:

- [Microsoft is busy rewriting core Windows code in memory-safe Rust](https://www.theregister.com/2023/04/27/microsoft_windows_rust/) - апрель 2023
- Google также активно присматривается к Rust, см. [Rust in the Android platform](https://security.googleblog.com/2021/04/rust-in-android-platform.html) - апрель 2021
- AWS применяет Rust в своих продуктах

Важно отменить, что на Rust осуществляются попытки разработать операционные системы, в частности - [Redox 0.9](https://www.redox-os.org/). Также существуют попытки переписать часть Linux на Rust, в частности, драйвера операционной системы, которые работают на уровне ядра. Однако, существуют определённые трения между разработчиками Linux, выбирающими Rust и Си.

Ещё одна важная возможность Rust - использование **inline asm**, что позволяет создавать супер-эффективные (в контексте расхода вычислительных ресурсов) приложения:

```rs
use std::arch::asm;

let mut x: u64 = 4;
unsafe {
    asm!(
        "mov {tmp}, {x}",
        "shl {tmp}, 1",
        "shl {x}, 2",
        "add {x}, {tmp}",
        x = inpot(reg) x,
        tmp = out(reg) _,
    );
}
assert_eq!(x, 4 * 6);
```

Также следует отметить статью [Microsoft seeks Rust developers to rewrite core C# code](https://www.theregister.com/2024/01/31/microsoft_seeks_rust_developers/) by Richard Speed, в которой указывается, что Microsoft ищет специалистов для переписывания части инфраструктуры с C# на Rust (январь 2024 года). Требования к соискателям буквально следующие: "_guiding technical direction, design and implementation of Rust component libraries, SDKs, and re-implementation of existing global scale C# based services to Rust_". Справедливости ради, объявление о вакансии было размещено в Чехии и связано с переработкой облачных сервисов, для которых время поднятия контейнера - критически важная метрика, т.е. в данном случае, конечно же нет разговора об отказе от использования C#, но совершенно чётко указана ниша, в которой Rust значительно превосходит C#.

На конференции Open Source Summit 2023 Japan, Линус Торвальдс, выступая в keynotes, сообщил, что Rust пока ещё не доказал, что это "следующая большая вещь", но в 2024 году в Linux появятся драйвера, написанные на Rust. Торвальдс не пишет код на Rust, но читает его и может принимать решения о включении Rust-кода в Linux.

Приблизительно в том же направлении движется Google, который [Google Spends $1 Million to Make Rust, C+ ‘Interoperable’](https://thenewstack.io/google-spends-1-million-to-make-rust-c-interoperable/) - это статья от 5 февраля 2024 года.

Беглый анализ Tutorial-ов позволил сделать заключение о том, что одной из наиболее популярных библиотек для разработки backend является [Rocket](https://rocket.rs/).

## Первый опыт использования

Rustc очень подробен в выводе сообщений об ошибках. В компилятор встроен очень мощный Linter, который реально помогает написать качественный код.

Синтаксически, Rust требует максимальной определённости конструкций, например, не допускает смешения типов в арифметических операциях. Если требуется осуществить преобразование типов, это необходимо сделать явным образом, например:

```rs
let freq: u32 = read_input();
let bandwidth: u32 = read_input();
let zip: u32 = read_input();

let top : f64 = (bandwidth as f64) * 8.0 * 1024.0 * 0.5;
let bottom : f64  = (freq as f64) * 1000.0 * (1.0 - (zip as f64) / 100.0 );
let result = ( top as u32) / (bottom as u32);
```

Использование макросов для реализации операций типо-безопасного вывода увеличивает размер кода, но повышает его производительность. В общем случае, не оптимизированный код на Rust может быть более эффективным, чем не оптимизированный код на C++.

В Rust очень много специфичных конструкций, которые не очевидны и их нужно выучить. Например, объединение как элементов массива char-ов в строку, так и массива строк в одну строку можно выполнить так:

```rs
let output: String = values.iter().collect();
```

Но если мы хотим объединить через join() с указанием символа-разделителя, то для массива строк мы это сделать можем, а для массива char-ов не можем.

В синтаксисе Rust много удобных конструкций, делающих его, по степени удобства, похожим на C\#, но при этом производительным как C++. Например сопоставление:

```rs
match index {
    1 => println!("Это первый элемент"),
    3 => println!("Это третий элемент", tup),
    5 => println!("А это пятый элемент", tup),
    _ => {}
}
```

Интроспекция, отсутствующая в языке Rust, сильно затрудняет разработку простых и эффективных вспомогательных инструментов. Например, в курсе по Rust на Stepik.org есть тесты, которые проверяют вывод приложений, но из-за отсутствия интроспекции, крайне сложно (если возможно) написать тест, который проверит, что учащийся использует именно кортеж, а не массив. Я видел не одно решение, которое обманывает тесты. Обозначенная проблема может иметь критичное значение в использовании TDD - не аккуратный разработчик может легко обходить тесты.

Индекс массива - это отдельный тип **usize**.

Rust - лаконичный язык. Ниже приведена функция для ввода значения из консоли и преобразования её к указанному типу. Для того, чтобы адаптировать функцию для возврата значения другого типа, достаточно заменить только тип возвращаемого значения в заголовке функции - всего в одном месте:

```rs
fn read_input() -> usize {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}
```

Сравнивая с другими языками программирования, нам бы потребовалось явным образом выполнять преобразование значения, например: `Int32.TryParse()`.

Хотелось бы привести ещё один пример лакончиности Rust в случае использования идеоматических конструкций.

```rs
*counts.entry(value).or_insert(0) += 1;
```

В данном случае приведён пример API для работы HashMap. В этом коде, в одной строке либо добавляется новое значение в хэш-таблицу, либо увеличивается на единицу уже существующее значение. В C\# потребовалось бы использовать явное условие и, как минимуми, две строки для изменения, или добавления кода.

> На самом деле, в C\# 8 и выше можно было бы использовать вот такой код:
>
>```csharp
static void AddOrUpdate(Dictionary<string, int> dictionary, string key, int value)
{
    dictionary[key] = dictionary.GetValueOrDefault(key) + value;
}
```
> Т.е.формально это то же oneliner, хотя и чуть менее красивый.

Очень непривычным является использование строк. В Rust существует тип String с передачей ownership и тип str с динамическим размером. Эти типы не совместимы между собой и требуют преведения. Например, str преобразовать в String можно используя функцию `to_string()`. Функции могут возвращать только строки типа String. Использование str и передача среза строки посредством &str гораздо более эффективно, т.к. позволяет избежать создания копии строки.

Важно заметить, что все данные хранящиеся в стеке должны иметь фиксированный размер. Данные, размер которых неизвестен, или может изменяться - хранится в куче.

Очень крутая _feature_ - затенение, т.е. возможности использования двух последовательно используемых переменных с одинаковым именем, которые хранят одно и тоже значение, но разных типов. Например, сначала считываем из консоли строку value, а потом конвертируем её в value с типом u32. Это позволяет минимизировать замусоривание пространства имён переменных.

Циклы в Rust также выглядят очень компактно и лаконично. Переменная цикла определяется автоматически, диапазон можно указать явным образом:

```rs
for index in 3..10 {
    // ...
}
```

Также можно использовать нижнее подчеркивание для указания локальной переменной, а в диапазоне указывать имена переменных:

```rs
for _ in start..end {
    print!("{input}");
}
```

Необычная конструкция - можно указать, что завершающее значение нужно включать в диапазон:

```rs
for index in 1..=5 {
    // ...
}
```

Очень понравился plugin [rust-analyzer](https://code.visualstudio.com/docs/languages/rust) для Visual Studio Code, который показывает, каким образом будет выполняться преобразование типов в вычислениях. Это позволяет увидеть потенциальные ошибки в коде.

## Выявленные негативные аспекты

Размер собираемого web-проекта (папка target) может составлять более 2,5Гб, что предполагает вместительный накопитель - как минимум в 512Гб. Особенно серьёзно этот аспект может влиять на сервера сборок с большим количеством проектов.

Для информации, размер папки "target" для web-проектов:

- с использованием Rocket 0.4 (Debug): 1164Мб
- с использованием Rocket 0.5 (Debug+Release): 2675Мб

Время загрузки зависимостей может составлять минуты, компиляция - десятки секунд.

В статье [Rust без прикрас: где мы ошибаемся](https://habr.com/ru/companies/beget/articles/857168/) by morett1m, отмечается большое количество ошибок связанных с некорректным использованием функции **unwrap**(). Эта функция используется при извлечении значений из типов **Option** и **Result**. Функция может возвращать значение None (для типа **Option**), или Err (для типа **Result**), что может приводить к возникновению _Runtime Panic_ при дальнешей работе с этим результатом. Чтобы избежать проблему, следует использовать оператор `match`, конструкцию `let if`, или другие способы.

Пример ошибочного кода:

```rs
let none_value: Option<i32> = None;
let value = none_value.unwrap(); // This will panic!
```

Также автор статьи отмечает, что программисты на Rust склонны избыточно использовать операцию клонирования, что может значительно ухудшать производительность приложения.

## Пример исходного кода

В соответствии с _conventions_ для имени проекта с Rust-кодом используется _underscore names_. Соответственно, стартовый проект был назван **rust_web_server**. Шаблонный код проекта на Rust был сгенерирован командой:

``` shell
cargo new rust_web_server --bin
```

Флаг `--bin` указывает, что cargo должен сгенерировать binary-based project.

Описание дополнительных команд CLI доступно [по ссылке](./cli.md).

После генерации проекта сразу же возникает несколько чудес, которые могут причинить небольшие ментальные травмы, если эти чудеса сразу же не задавить. Во-первых, может потребовааться переключение Rust на nightly-версию, если рекомендуемый пример кода не работает. Часть синтаксических возможностей компилятора доступна только в nightly-версии. 

В модели разработки языка Rust есть три _канала выпуска_ (правила выпуска): ночной (Nightly), бета (Beta) и стабильный (Stable). Для большинства разработчиков подходит **Stable**-канал выпуска. Однако, если нужны некоторые экспериментальные возможности, может потребоваться выбрать Beta, или Nightly-канал. Выбор Beta-канала чаще всего обусловлен необходимостью понимания расхождений кодовой базы и новой версии языка и это канал используется на CI/CD.

Переход на ночной канал выпуска можно выполнить следующим образом:

``` shell
rustup override set nightly
```

Проверочная команда:

``` shell
cargo version
```

Более подробная информация о необходимости использования Nightly-версии компилятора с Rocket доступна [здесь](./nightlybuild.md).

Вторая проблема - добиться когерентности между примером кода и устанавливаемой версией Rocket. При автоматическом добавлении зависимости командой `cargo add rocket` была установлена версия Rocket 0.4.11. Чтобы увидеть версию, следует открыть файл "Cargo.toml", в котором есть раздел зависимостей:

``` cargo
[dependencies]
rocket = "0.4.11"
```

Стартовый пример приложения рекомендуется брать из раздела сайта Rocket, посвящённого конкретной версии продукта. Например, для Rocket 0.4 пример нужно брать именно из раздела [версии 0.4](https://rocket.rs/v0.4/).

Пример с которого ознакомление с Rocket начал я был таким (Rocket 0.4):

``` rs
#![feature(proc_macro_hygiene, decl_macro)]

#[macro_use] extern crate rocket;

#[get("/hello/<name>/<age>")]
fn hello(name: String, age: u8) -> String {
    format!("Hello, {} year old named {}!", age, name)
}

fn main() {
    rocket::ignite().mount("/", routes![hello]).launch();
}
```

Сборка и запуск приложения осуществляются в Command Prompt (не PowerShell) следующими командами:

``` shell
cd .\rust_web_server\
cargo build --release
cargo run
```

После запуска приложения к нему можно подключиться по URL: http://localhost:8000/hello/max/49

Сравнение версий Rocket приведено по ссылке [0.4.11 vs 0.5.0-rc2](./rocket_versions.md).

## Ссылки по теме

- [SergioBenitez/Rocket](https://github.com/SergioBenitez/Rocket) - официальный репозитарий библиотеки Rocket, который также содержит примеры кода
- [Creating a Web Service API Using Rust Rocket](https://betterprogramming.pub/creating-a-web-server-using-rust-rocket-1e4939e582df) by Joseph Talon. Свежая статья, в которой используется Rocket 0.5.0-rc.2. Поскольку, по cargo установил 0.4.11, пример не собрался - не был найден атрибут #[launch]
- [Building web apps with Rust using the Rocket framework](https://blog.logrocket.com/rust-web-apps-using-rocket-framework/) by Ebenezer Don. В примере кода использовался Rocket 0.4.5 близкий к тому, что устанавливает Cargo
- [Creating a Rust Web App with Rocket and Diesel](https://medium.com/itnext/creating-a-rust-web-app-with-rocket-and-diesel-58f5f6cacd27) by Raimi Karim. [Diesel](https://diesel.rs/) - это ORM и Query Builder для Rust

Следует заметить, что Rocket имеет _build-in_ поддержку баз данных, таких как Postgres, MySQL, SQLite, Redis, Memcache. См. [официальную документацию](https://rocket.rs/v0.4/guide/state/#databases). В качестве облачной базы данных предлагается использовать [SurrealDB](https://surrealdb.com/docs/integration/libraries/rust)

Rust подходит для разработки ПО для микроконтроллеров. Об этом, в частности, написана статья [Rust on an STM32 microcontroller](https://medium.com/digitalfrontiers/rust-on-a-stm32-microcontroller-90fac16f6342) by Marco Amann.

## Популярные crates

- [Tokio](https://crates.io/crates/tokio) - управляемая по сообщениям (event-driven), не блокирующая платформа ввода/вывода (non-blocking I/O platform) для разработки асинхронных приложений ввода/вывода (back-end). Высокая популярность Tokio связана именно с асинхронностью. В базовых библиотеках Rust есть свой собственный web-сервер, но он не является асинхронным и очень сильно проигрывает в производительности, при сравнении с Tokio
- [Serde](https://crates.io/crates/serde) - framework для сериализации/десериализации данных, например, JSON. Рекомендуется для ознакомления статья [Кратко про Serde в Rust](https://habr.com/ru/companies/otus/articles/806247/) by **artem badcasedaily1**.

## Альтернативные варианты разработки Desktop-приложений

Один из продвигаемых на Medium фреймворков называется [Yew](https://yew.rs/). Рекомендуется к прочтению статья [Exploring Yew, the rust-based frontend framework as a React Developer](https://dev.to/hackmamba/exploring-yew-the-rust-based-frontend-framework-as-a-react-developer-52l) by Demola Malomo.

[Slint](https://slint.dev/) - ещё один Framework для разработки Desktop-приложений на Rust.

## Использование Rust для разработки прошивок микроконтроллеров

Ключевая статья: [Rust on an STM32 microcontroller](https://medium.com/digitalfrontiers/rust-on-a-stm32-microcontroller-90fac16f6342) by Marco Amann.

Установка кросс-компиляторов для Rust осуществляется таким образом:

``` shell
rustup update
rustup component add llvm-tools-preview
rustup target add thumbv7em-none-eabihf
cargo install cargo-binutils cargo-embed cargo-flash cargo-expand
```

Также можно почитать о пректе [Slint](https://slint-ui.com/) - это Framework для разработки GUI-приложений для Embedded-устройств. Бывшие разработчики Qt из Trolltech сделали новый Framework, который работает даже на микроконтроллерах с 256 Кб ОЗУ.

### Кросс-компиляция для Linux на ARM-процессорах

Установить компилятор Rust можно следующим набором команд:

```shell
sudo apt update
sudo apt install curl

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

Команда загружает shell-скрипт и выполняет его. Скрипт устанавливает Rust под конкретную целевую платформу.

Команда `source` работает также как запуск shell-скрипта, но этот скрипт запускается не в отдельном shell-е, а в уже запущенном, что позволяет избежать использования sudo и ввода пароля.

Проверка успешности установки инструментальных средств.

```shell
cargo --version
rustc --version
```

Обновить инструментальные средства Rust можно командой:

```shell
rustup update
```

Необходимо добавить целевую платформу. Получит список доступных платформ можно командой:

```shell
rustup target list
```

Примеры:

- `aarch64-unknown-linux-gnu`: 64-bit ARM (ARMv8) Linux
- `aarch64-unknown-linux-musl`: 64-bit ARM (ARMv8) Linux with musl libc
- `armv7-unknown-linux-gnueabihf`: 32-bit ARM (ARMv7) Linux with hard float
- `armv7-unknown-linux-musleabihf`: 32-bit ARM (ARMv7) Linux with musl libc
- `riscv64gc-unknown-linux-gnu`: RISC-V 64-bit Linux

Предполим, что мы хотели бы собирать приложение для ARMv7 для Raspberry PI OS. В этом случае, нам бы потребовалось выполнить следующую команду:

```shell
rustup target add armv7-unknown-linux-gnueabihf
```

Команда `rustup target add <target>` загружает необходимые компоненты для целевой платформы. Эти файлы включают стандартные библиотеки и вспомогательные инструментальные средства.

Добавления стандартных библиотек для rustc недостаточно. Компилятор rustc полагается на использование внешних компиляторов и их нужно устанавливать дополнительно.

Для сборки Linux-приложений, работающих на ARMv7 с поддержкой аппаратной математики, обычно устанавливается компилятор gcc-arm-linux-gnueabihf, который можно установить в Debian командой:

```shell
sudo apt-get install gcc-arm-linux-gnueabihf
```

Собрать проект следует явно указывая target:

```shell
cargo build --target armv7-unknown-linux-gnueabihf
```

Однако сборка завершилась с ошибкой:

```output
warning: unused manifest key: target.armv7-unknown-linux-gnueabihf.linker
...
/usr/bin/ld: /home/developer/projects/stepik/target/armv7-unknown-linux-gnueabihf/debug/deps/stepik-b0d9d777cdea05d9.0dm5krgeozpp4c8e4fn20bqoy.rcgu.o: error adding symbols: file in wrong format
collect2: error: ld returned 1 exit status
```

Эта ошибка указывает на то, что линковка не балы успешной. Если посмотреть на какой-нибудь ".o"-файл из папки "./target/armv7-unknown-linux-gnueabihf/debug/deps", то можно увидеть, что его платформа соответствует целевой: 

```output
file stepik-b0d9d777cdea05d9.eevt77jxka9iw1eeoluea0pzz.rcgu.o 
stepik-b0d9d777cdea05d9.eevt77jxka9iw1eeoluea0pzz.rcgu.o: 
ELF 32-bit LSB relocatable, ARM, EABI5 version 1 (SYSV), with debug_info, not stripped
```

Очевидно, что проблема возникла при линковке этих файлов с библиотеками. Решается проблема следующим образом: в домашней папке пользователя есть невидимая папка ".cargo". Следует перейти в эту папку, создать файл `config.toml` и добавить в этот файл целевую платформу:

```toml
[target.armv7-unknown-linux-gnueabihf]
linker = "arm-linux-gnueabihf-gcc"
```

Т.е. осуществляя кросс-компиляцию под конкретную аппаратную платформу, мы должны указывать платформу (armv7-unknown-linux-gnueabihf) и компилятор (gcc-arm-linux-gnueabihf). Замечу, что в этом есть смысл, поскольку компиляторы могут быть разные и кросс-компилятор можно собрать их исходных текстов, например, используя скрипт [crosstool-NG](https://github.com/crosstool-ng/crosstool-ng).

После изменения `~/.cargo/config.toml` сборка завершиться успешно.

Для запуска приложения непосредственно на целевой платформе следует собрать приложение в Release-сборке:

```shell
cargo build --target armv7-unknown-linux-gnueabihf --release
```

Однако следует заметить, что приложение будет собрано с динамической линковкой и к нему не будет применена операция strip. При необходимости следует выполнить дополнительные настроечные шаги.

После копирования исполняемого файла "stepik-6d9d1cc5e0992164" на целевую платформу (Raspberry Pi 3) и смены прав запуска:

```shell
chmod a+x ./stepik-6d9d1cc5e0992164
```

Приложение успешно запустилось и выполнило свою работу.

## Портирование приложений с Си на Rust

Материал взят из видео [Migrating from C to Rust - Part 1: Calling Rust Code from C](https://www.youtube.com/watch?v=WsnFZk5-xwQ&ab_channel=GaryExplains) из канала **Gary Explains**.

Решаются обе задачи: вызов Rust-кода из Си и Си кода из Rust. Однако для целей миграции с Си на Rust, более разумным кажется поэтапным (функция за функцией) перевод отдельных функций из Си в Rust.

Предположим, что у нас есть некоторый код на Си:

```c
static unsigned long x = 123456789;

unsigned long vrandom(void) {
    return 69069 * x + 362437;
}

void vseed(unsigned long seed) {
    x = seed;
}
```

Сборка библиотки средствами gcc может выглядеть следующим образом:

```shell
gcc -c vrandom.c                    # Компилируем исходные тексты библиотеки
ar rcs libvrandom.a vrandom.o       # Создаём библиотечный файл из объектного
gcc -o main main.c -L. -lvrandom    # Собираем исполняемый файл
```

Задача - заменить файл vrando.c на Rust-файл, который делает всё тоже самое, что и "vrandom.c".

Код на Rust может выглядеть следующим образом:

```rs
static mut X: u64 = 123456789;

#[no_mangle]
pub extern "C" fn vrandom() -> u64 {
    unsafe {
        X = X.wrapping_mul(69069).wrapping_add(362437);
        X
    }
}

#[no_mangle]
pub extern "C" fn vseed(seed: u64) {
    unsafe {
        X = seed;
    }
}
```

Мы используем ключевое слово **unsafe**, когда необходимо транслировать Си точно. Функции wrapping_mul() и wrapping_add() необходимы, т.к. Rust-код является safe language, а операции умножения и сложения могут приводить к переполнению, которое перехватывает Rust runtime.

Кроме этого нам требуется добавить аннотацию `#[no_mangle]` для каждой функции, для того, чтобы избежать создания уникальных имён функций при генерации машинного кода. Смысл _mangling_ состоит в том, что в приложении может быть определено несколько функций с одинаковым именем. Чтобы избежать коллизий, Rust даёт каждой функции уникальное имя, кроторое включает имя crate, имя модуля, или типа, имя функции, а также hash-код. 

Например, есть некоторое определение:

```rs
struct One;
struct Two;

impl One { fn foo() { ... }}
impl Two { fn foo() { ... }}
```

Для функции foo() в реализации структуры One, mangled name может выглядеть так: `_ZN7example3One3foo17h16fcc82fa6043ccbE`

Также нам требуется указать явным образом нотацию функций, добавив `extern "C"`.

Для сборки проекта следует использовать следующий "Cargo.toml":

```
[package]
name = "vrandom"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["staticlib"]  # Создаём статическую библиотеку

[dependencies]
```

Собрать библиотеку компилятором Rust можно следующим образом:

```shell
cargo build --release
```

Далее можно прилинковать библиотеку на Rust к Си-приложению, в котором реализована функция main():

```shell
gcc -o main main.c -Lvrandom/target/release/ -lvrandom
```

### Идеоматический подход

Идеоматический подход Rust (создание обёртки в виде структуры) выглядит следующим образом:

```rs
#[repr(C)]
pub struct VRandom {
    x: u64;
}

impl VRandom {
    #[no_mangle]
    pub extern "C" fn newVRandom(seed: u64) -> Self {
        VRandom { x: seed }
    }

    // Реализация функций nextVRandom() и seedVRandom()...
    #[no_mangle]
    pub extern "C" fn nextVRandom(&mut self) -> u64 {
        self.x = self.x.wrapping_mul(69069).wrapping_add(362437);
        self.x
    }

    #[no_mangle]
    pub extern "C" fn seedVRandom(&mut self, seed: u64) {
        self.x = seed;
    }
}
```

В самом конце, когда весь код портирован, следует удалить все unsafe-секции, чтобы добавить встроенный контроль Rust.

Существует инструмент, который позволяет сгенерировать header files на Си для Rust-кода. Установить инструмент можно командой:

```shell
cargo install cbindgen
```

Затем нужно запустить эту утилиту на верхнем уровне вашего проекта.

## Code covegare in Rust

Рекомендуется для ознакомления автор - [Dotan Nahum](https://jondot.medium.com/). Начать исследование темы покрытия кода тестами можно со статьи [How to do code coverage in Rust](https://jondot.medium.com/how-to-do-code-coverage-in-rust-9548e0fbacce).
