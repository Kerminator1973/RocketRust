# Cheatsheet по RUST

Инструкция по установке Rust доступна [по ссылке](https://www.rust-lang.org/ru/tools/install). Для Linux, обычно достаточно выполнить следующий скрипт:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Может потребоваться добавить путь к исполняемым файлам Rust в путь к исполняемым файлам текущего пользователя. Для этого необходимо добавить в конец файла `~/.profile` следующую строку:

export PATH="$HOME/.cargo/bin:$PATH"

Проверить успешность установки можно командой:

```shell
rustc --version
```

В Visual Studio Code есть целый ряд plug-in-ов, обеспечивающих поддержку Rust:

- [rust-analyzer](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer) by rust-lang.org
- [CodeLLDB](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb) by Vadim Chugunov - для отладки кода
- [crates](https://marketplace.visualstudio.com/items?itemName=serayuzgur.crates) by Seray Uzgur - управление зависимостями
- [Even Better TOML](https://marketplace.visualstudio.com/items?itemName=tamasfe.even-better-toml) by tamasfe - синтаксическая подсветка TOML

## Базовые синтаксические конструкции Rust

Вывести сообщение в консоль можно макросами print!() и println!():

```rs
fn main() {
    print!("Hello, World!");
}
```

Функция main() - стартовая точка приложения на Rust.

Форматированный вывод переменных выглядит следующим образом:

```rs
let age = 31;
println!("{}", age);
```

В Rust может быть использован _string interpolation_:

```rs
let hello: &str = "Hello";
println!("{hello}");
```

В приведённом выше примере используется т.н. _строковый срез_. Аналог в C++ и C# - `span<>`.

Определить неизменное значение можно с помощью ключевого слово let:

```rs
let outer_var = 100;
```

Вычисляемые значения могут быть константными:

```rs
let outer_var = 20 * another_value + 5;
```

Если мы планируем изменять переменную, потребуется добавить модификатор **mut** (mutable - изменяемый):

```rs
let mut age = 1;
```

Инициализация массивов и строк:

```rs
let sparkle_heart = vec![240, 159, 146, 150];

// Поскольку мы знаем, что переданные в параметре байты корректные, мы можем выполнить преобразование unwrap()
let sparkle_heart = String::from_utf8(sparkle_heart).unwrap();
```
