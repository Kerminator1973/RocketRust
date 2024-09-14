# Мои решения тестов не курсе по Rust на сайте Stepik.ru

Задача ввода нескольких значений в консоли:

```rs
use std::io;

fn main() {
    let mut values: [String; 3] = Default::default();

    for i in 0..values.len() {
        io::stdin().read_line(&mut values[i]).expect("Failed to get input");
    }

    for value in &values {
        print!("{}", value);
    }
}
```

## Задача вывода введённых данных в обратном порядке

Решение:

```rs
use std::io;

fn main() {
    let mut values: [String; 4] = Default::default();

    for value in &mut values {
        io::stdin().read_line(value).expect("Failed to get input");
    }

    let output: String = values.iter()
        .rev() // Итератор будет реверсивным, т.к. следует от конца к началу
        .map(|s| s.trim()) // К каждой строке будет применён trim()
        .collect::<Vec<&str>>() // Собираем строковые срезы в массив
        .join(" "); // Объединяем элементы массива в строку с разделителем

    print!("{}", output);
}
```

В приведённом выше примере используется замыкание (_closure_): `.map(|s| s.trim())`. Метод map() применяется к итератору по коллекции и указывает, что для каждого элемента коллекции, который будет назван `|s|`, следует применить метод `s.trim()`, для того, чтобы убрать пробелы по краям строки.

Вызов `.collect::<Vec<&str>>()` трансформирует итератор в коллекцию. Часть вызова `::<Vec<&str>>` является аннотацией типа коллекции, которая будет создаваться из итератора. Тип `&str` указывает, что строковые данные заимствуются (_borrowed string data_), а не происходит передача владения.
