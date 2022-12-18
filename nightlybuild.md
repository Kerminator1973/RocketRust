# Можно ли обойтись без nightly-версии?

Установить Stable-версию можно командой:

``` shell
rustup override set stable
```

Проверочная команда: `cargo --version` должна выдать версию без _nightly_, например: `cargo 1.65.0 (4bc8f24d3 2022-10-20)`

Сборка кода с Rocket 0.4.11 завершилась ошибками:

error: failed to run custom build command for `pear_codegen v0.1.5`

```
Caused by:
  process didn't exit successfully: `D:\Sources\RocketRust\rust_web_server\target\debug\build\pear_codegen-998a7851d9d765e0\build-script-build` (exit code: 101)
  --- stderr
  Error: Pear requires a 'dev' or 'nightly' version of rustc.
  Installed version: 1.65.0 (2022-11-02)
  Minimum required:  1.31.0-nightly (2018-10-05)
  thread 'main' panicked at 'Aborting compilation due to incompatible compiler.', C:\Users\kermi\.cargo\registry\src\github.com-1ecc6299db9ec823\pear_codegen-0.1.5\build.rs:24:13
  note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Такое же сообщение было выдано для Rocket (codegen) и Rocket (core).

Пересборка с дополнительной трассировкой:

``` shell
set RUST_BACKTRACE=1
cargo build
```

Описание ошибки было чуть более информативным:

```
  thread 'main' panicked at 'Aborting compilation due to incompatible compiler.', C:\Users\kermi\.cargo\registry\src\github.com-1ecc6299db9ec823\pear_codegen-0.1.5\build.rs:24:13
  stack backtrace:
     0: std::panicking::begin_panic<str>
               at /rustc/897e37553bba8b42751c67658967889d11ecd120\library\std\src\panicking.rs:616
     1: build_script_build::main
               at .\build.rs:24
     2: core::ops::function::FnOnce::call_once<void (*)(),tuple$<> >
               at /rustc/897e37553bba8b42751c67658967889d11ecd120\library\core\src\ops\function.rs:248
  note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
warning: build failed, waiting for other jobs to finish...
error: failed to run custom build command for `rocket_codegen v0.4.11`    
```

Увеличить уровень логирования компилятора можно командой:

```
set RUST_BACKTRACE=full
cargo build
```

В full-режиме выдаётся детальная информация о том, в каком месте исходных текстов возникли проблемы. Лог здесь не привожу, простых подсказок там нет. Тем не менее, можно предположить, что версия Rocket 0.4.0 разрабатывалась для новых features компилятора четырёхлетней давности (1.31.0-nightly 2018-10-05). Разумным кажется попробовать установить Rocket 0.5.xx и попробовать собрать приложение с новой библиотекой, новым компилятором.

## Проект с Rocket 0.5.0-rc.2

Был создан новый проект:

``` shell
cargo new rust_web_rocket050 --bin
```

В "Cargo.toml" был внесена более свежая зависимость (Rocket 0.5.0-rc.2):

``` Cargo.toml
[dependencies]
rocket = "0.5.0-rc.2"
```

Заметим, что если перейти на сайт [crates.io](https://crates.io/crates/rocket), то можно легко убедиться в том, что формат версий зависимостей - произвольный. Это просто строка текста.

Установлен стабильный компилятор:

``` shell
rustup override set stable
```

Пример приложения взят из репозитария [SergioBenitez/Rocket](https://github.com/SergioBenitez/Rocket/blob/master/examples/hello/src/main.rs).

``` shell
cargo build --release
```

Поскольку Release-сборка включает оптимизацию кода, она занимает на 25-30% больше времени, что Debug-сборка.

Как результат, размер исполняемого файла составляет ~3,7 Мб, что в 2+ раза больше, чем пример для Rocket 0.4.11, но и функциональных возможностей у свежено примера существенно больше. В частности, он поддерживаем национальные языки в routes. Т.е. можно использовать вот такую ссылку в браузере: `http://127.0.0.1:8000/hello/%D0%BC%D0%B8%D1%80`

Отмечу, что русский язык по прежнему часто используется для тестирования i18n.
