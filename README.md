# Экспериментирование с использованием Rust для разработки web-приложений

Rust кажется очень перспективным инструментом разработки, в том числе, применимым для разработки web-приложений. Rust позволяет генерировать компактные и быстрые нативные приложения. Забегая чуть вперёд, можно утверждать, что исполняемый файл (exe) для Windows x64-86, с примитивной реализацией web-сервера занимает всего 1,5 Мб и стартует быстрее, чем за секунду.

Rust часть сравнивают с языком программирования Go, но и этих задач разные цели и области применения. "_Gophers value simplicity, Rustaceans value safety_". Оба языка являются компилируемыми, но в Go используется Garbage Collector. Производительность и надёжность кода у Rust выше. Соответственно, ключевые области применения Rust: mission critical code и приложения, требующие максимальной вычислительной эффективности.

Беглый анализ Tutorial-ов позволил сделать заключение о том, что одной из наиболее популярных библиотек для разработки backend является [Rocket](https://rocket.rs/).

## Пример исходного кода

В соответствии с _conventions_ для имени проекта с Rust-кодом используется _underscore names_. Соответственно, стартовый проект был назван **rust_web_server**. Шаблонный код проекта на Rust был сгенерирован командой:

``` shell
cargo new rust_web_server --bin
```

Флаг `--bin` указывает, что cargo должен сгенерировать binary-based project.

Описание дополнительных команд CLI доступно [по ссылке](./cli.md).

После генерации проекта сразу же возникает несколько чудес, которые могут причинить небольшие ментальные травмы, если эти чудеса сразу же не задавить. Во-первых, нужно переключить Rust на nightly-версию. Если этого не сделать, то компиляция приложения не будет возможна. В модели разработки языка Rust есть три _канала выпуска_ (правила выпуска): ночной (Nightly), бета (Beta) и стабильный (Stable). Для большинства разработчиков подходит **Stable**-канал выпуска. Однако, если нужны некоторые экспериментальные возможности, может потребоваться выбрать Beta, или Nightly-канал. Выбор Beta-канала чаще всего обусловлен необходимостью понимания расхождений кодовой базы и новой версии языка и это канал используется на CI/CD.

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

Стартовый пример приложения рекомендуется взять в разделе сайта Rocket посвящённом именно [версии 0.4](https://rocket.rs/v0.4/).

Работающий пример оказался таким:

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

Следует заметить, что Rocket имеет _build-in_ поддержку баз данных, таких как Postgres, MySQL, SQLite, Redis, Memcache. См. [официальную документацию](https://rocket.rs/v0.4/guide/state/#databases).

## Популярные crates

- [Tokio](https://crates.io/crates/tokio) - управляемая по сообщениям (event-driven), не блокирующая платформа ввода/вывода (non-blocking I/O platform) для разработки асинхронных приложений ввода/вывода (back-end)
- [Serde](https://crates.io/crates/serde) - framework для сериализации/десериализации данных, например, JSON

## Использование Rust для разработки прошивок микроконтроллеров

Ключевая статья: [Rust on an STM32 microcontroller](https://medium.com/digitalfrontiers/rust-on-a-stm32-microcontroller-90fac16f6342) by Marco Amann.

Установка кросс-компиляторов для Rust осуществляется таким образом:

``` shell
rustup update
rustup component add llvm-tools-preview
rustup target add thumbv7em-none-eabihf
cargo install cargo-binutils cargo-embed cargo-flash cargo-expand
```
