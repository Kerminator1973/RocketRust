# Экспериментирование с использованием Rust для разработки web-приложений

Rust кажется очень перспективным инструментом разработки, в том числе, применимым для разработки web-приложений. Rust позволяет генерировать компактные и быстрые нативные приложения. Забегая чуть вперёд, можно утверждать, что исполняемый файл (exe) для Windows x64-86, с примитивной реализацией web-сервера занимает всего 1,5 Мб и стартует быстрее, чем за секунду.

Беглый анализ Tutorial-ов позволил сделать заключение о том, что одной из наиболее популярных библиотек для разработки backend является [Rocket](https://rocket.rs/).

## Пример исходного кода

В соответствии с _conventions_ для имени проекта с Rust-кодом используется _underscore names_. Соответственно, стартовый проект был назван **rust_web_server**. Шаблонный код проекта на Rust был сгенерирован командой:

``` shell
cargo new rust_web_server --bin
```

Флаг `--bin` указывает, что cargo должен сгенерировать binary-based project.

После генерации проекта сразу же возникает несколько чудес, которые могут причинить небольшие ментальные травмы, если эти чудеса сразу же не задавить. Во-первых, нужно переключить Rust на nightly-версию (_что это такое?_). Если этого не сделать, то компиляция приложения не будет возможна. Корректирующие действия выглядят следующим образом:

``` shell
rustup override set nightly
```

Проверочная команда:

``` shell
cargo version
```

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

`` shell
cd .\rust_web_server\
cargo build --release
cargo run
```

После запуска приложения к нему можно подключиться по URL: http://localhost:8000/hello/max/49

## Ссылки по теме

- [Creating a Web Service API Using Rust Rocket](https://betterprogramming.pub/creating-a-web-server-using-rust-rocket-1e4939e582df) by Joseph Talon
- [Building web apps with Rust using the Rocket framework](https://blog.logrocket.com/rust-web-apps-using-rocket-framework/) by Ebenezer Don
- [SergioBenitez/Rocket](https://github.com/SergioBenitez/Rocket)
