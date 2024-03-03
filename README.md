# Экспериментирование с использованием Rust для разработки web-приложений

Rust кажется очень перспективным инструментом разработки, в том числе, применимым для разработки web-приложений. Rust позволяет генерировать компактные и быстрые нативные приложения. Забегая чуть вперёд, можно утверждать, что исполняемый файл (exe) для Windows x64-86, с примитивной реализацией web-сервера занимает всего 1,5 Мб и стартует быстрее, чем за секунду.

Rust часто сравнивают с языком программирования Go, но у этих языков разные цели и области применения. "_Gophers value simplicity, Rustaceans value safety_". Оба языка являются компилируемыми, но в Go используется Garbage Collector. Производительность и надёжность кода у Rust выше. Соответственно, ключевые области применения Rust: mission critical code и приложения, требующие максимальной вычислительной эффективности.

На Rust был переписан **репозитарий npm** и DNS-сервера от Cloudflare. Отмечается, что Rust-приложения обладают отличной предсказуемостью операций выделения памяти, чем не могут похвастаться приложения на Go, Java, JavaScript и платформы .NET.

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

## Выявленные негативные аспекты

Размер собираемого web-проекта (папка target) может составлять более 2,5Гб, что предполагает вместительный накопитель - как минимум в 512Гб. Особенно серьёзно этот аспект может влиять на сервера сборок с большим количеством проектов.

Для информации, размер папки "target" для web-проектов:

- с использованием Rocket 0.4 (Debug): 1164Мб
- с использованием Rocket 0.5 (Debug+Release): 2675Мб

Время загрузки зависимостей может составлять минуты, компиляция - десятки секунд.

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

- [Tokio](https://crates.io/crates/tokio) - управляемая по сообщениям (event-driven), не блокирующая платформа ввода/вывода (non-blocking I/O platform) для разработки асинхронных приложений ввода/вывода (back-end)
- [Serde](https://crates.io/crates/serde) - framework для сериализации/десериализации данных, например, JSON

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

## Code covegare in Rust

Рекомендуется для ознакомления автор - [Dotan Nahum](https://jondot.medium.com/). Начать исследование темы покрытия кода тестами можно со статьи [How to do code coverage in Rust](https://jondot.medium.com/how-to-do-code-coverage-in-rust-9548e0fbacce).
