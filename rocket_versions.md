# Различия в коде для разных версий Rocket

Запуск web-сервера выглядит по другому. Версия 0.4.11:

``` rs
fn main() {
    rocket::ignite().mount("/", routes![hello]).launch();
}
```

В версии 0.5.0 такой же код выглядит так:

``` rs
#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![hello])
        .mount("/hello", routes![world, mir])
        .mount("/wave", routes![wave])
}
```

Вместо fancy-функции ignite() используется более традиционное build(). Не нужно вызывать метод launch().

Обработчик запрос в 0.4.11 принимает строку в качестве входного значения:

``` rs
#[get("/hello/<name>/<age>")]
fn hello(name: String, age: u8) -> String {
    format!("Hello, {} year old named {}!", age, name)
}
```

В версии 0.5.0 передаётся строковый срез, что является более эффективным, с точки зрения расхода вычислительных ресурсов и управления владением (ownership):

``` rs
#[get("/<name>/<age>")]
fn wave(name: &str, age: u8) -> String {
    format!("👋 Hello, {} year old named {}!", age, name)
}
```

Оценочное мнение: версия 0.5.0 является более зрелой.

Также следует заметить, что Rust Linter гораздо лучше работает с 0.5.0, чем с 0.4.11.
