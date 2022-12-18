# Команды командной строки

Обновить версию Rust можно командой: `rustup update stable`

Добавить зависимость в проекте можно командой: `cargo add [crate]`

Чтобы установить конкретную версию crate, её нужно указать через символ @ после имени. Например: `cargo add rocket@0.5.0-rc2`

_Crates_ - это названия packages в Rust. Например: rocket, tauri.

Удалить зависимость можно командой: `cargo remove [crate]` (начиная с Rust 1.62).

## Clippy

Clippy - это набор из 550 правил оформления кода (Linters). Запустить команду можно так:

``` shell
cargo clippy
```

Дополнительная информация в [официальной документации](https://github.com/rust-lang/rust-clippy)
