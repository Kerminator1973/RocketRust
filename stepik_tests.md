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

Важно обратить внимание на тот факт, что в цикле for не используются круглые скобки, как это принято в C/C++/C\#.

Реализация print!() и println!() как макросов, а не функций, связана с тем, что макросы позволяют генерировать код на этапе компиляции, принимая во внимание строку форматирования. По сути, под каждую операцию вывода в консоль генерируется свой уникальный бинарный код, что, несомненно, положительно влияет на производительность приложения.

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

Данный алгоритм можно считать эффективно выделяющим память, т.к. когда создаётся новый массив, перед вызовом join(), это не массив строк, а массив строковых срезов, т.к. объекты фиксированной длины и минимального размера. В оптимальном случае, вызов collect() выделит только одну область памяти, для хранения всей новой коллекции.

## Использование mem::swap()

```rs
use std::io;
use std::mem;

fn main() {

    let mut str_first = String::new();
    io::stdin().read_line(&mut str_first).expect("Failed to get input");
    let mut first_num : i32 = str_first.trim().parse().expect("Failure to parse");

    let mut str_second = String::new();
    io::stdin().read_line(&mut str_second).expect("Failed to get input");
    let mut second_num : i32 = str_second.trim().parse().expect("Failure to parse");

    mem::swap(&mut first_num, &mut second_num);

    println!("{first_num}");
    println!("{second_num}");
}
```

## Вывод целочисленных значений в разных форматах

```rs
use std::io;

fn main() {

    let mut str_first = String::new();
    io::stdin().read_line(&mut str_first).expect("Failed to get input");
    let first_num : i32 = str_first.trim().parse().expect("Failure to parse");

    let mut str_second = String::new();
    io::stdin().read_line(&mut str_second).expect("Failed to get input");
    let second_num : i32 = str_second.trim().parse().expect("Failure to parse");

    trace_result(first_num, second_num, first_num + second_num, '+');
    trace_result(first_num, second_num, first_num - second_num, '-');
    trace_result(first_num, second_num, first_num * second_num, '*');
    trace_result(first_num, second_num, first_num / second_num, '/');
    trace_result(first_num, second_num, first_num % second_num, '%');
}

fn trace_result(first: i32, second: i32, result: i32, oper: char )
{
    println!("{first:#b} {oper} {second:#b} = {result:#b}");
    println!("{first:#o} {oper} {second:#o} = {result:#o}");
    println!("{first:#x} {oper} {second:#x} = {result:#x}");
    println!("");
}
```

## Ввод и вывод вещественных чисел

В первой задаче осуществляется вывод с указанием точности через внешнюю переменную, см. `prec$`:

```rs
use std::io;

fn main() {

    let mut str_float = String::new();
    io::stdin().read_line(&mut str_float).expect("Failed to get input");
    let float : f64 = str_float.trim().parse().expect("Failure to parse");

    let mut str_prec = String::new();
    io::stdin().read_line(&mut str_prec).expect("Failed to get input");
    let prec : usize = str_prec.trim().parse().expect("Failure to parse");

    print!("{:.prec$}", float);
}
```

Во второй задаче проверяется навык сложения переменных разных типов:

```rs
use std::io;

fn main() {

    let mut str_balance = String::new();
    io::stdin().read_line(&mut str_balance).expect("Failed to get input");
    let balance : f64 = str_balance.trim().parse().expect("Failure to parse");

    let mut str_amount = String::new();
    io::stdin().read_line(&mut str_amount).expect("Failed to get input");
    let amount : u32 = str_amount.trim().parse().expect("Failure to parse");

    let sum = balance + amount as f64;

    print!("{:.1}", sum);
}
```

В третьей задаче нужно было разделить вещественное число на мажорную (целочисленную) и минорную (вещественную) части. Мой код выглядел так:

```rs
use std::io;

fn main() {

    let mut str_value = String::new();
    io::stdin().read_line(&mut str_value).expect("Failed to get input");
    let value : f64 = str_value.trim().parse().expect("Failure to parse");

    let int_part = value as i64;
    println!("{int_part}");

    let minor = value - int_part as f64;
    println!("{:.3}", minor);
}
```

ChatGPT предложил следующую оптимизацию:

```rs
// Get the integer part and fractional part
let integer_part = value.trunc() as i64; // Use trunc() to get the integer part
let fractional_part = value.fract(); // Use fract() to get the fractional part
```

## Вывод вещественных чисел

Цель задачи: ввести число в виде `481068.0`, вывести в экспоненциальном (научном) виде `4.81068E5`. Решение:

```rs
use std::io;

fn main() {

    let mut str_first = String::new();
    io::stdin().read_line(&mut str_first).expect("Failed to get input");
    let num : f64 = str_first.trim().parse().expect("Failure to parse");

    println!("{:E}", num);
}
```

## Ввод отдельных символов и их объединение в строку

В задаче требовалось ввести пять японских иероглифов и объединить их в строку:

```text
こ
ん
に
ち
は
```

Вот моё решение:

```rs
use std::io;

fn main()
{
    let mut values: [char; 5] = Default::default();

    for value in &mut values {
        let mut buffer = String::new();
        io::stdin().read_line(&mut buffer).expect("Failed to get input");
        *value = buffer.chars().next().expect("String is empty");
    }

    let output: String = values.iter().collect();
    print!("{}", output);
}
```

Из отличий:

- Потребовалось использовать конструкцию `*value` вместо `value` для записи в элемент массива
- Чтобы получить отдельный символ строки, был получен итератор `chars()` и возвращён первый символ `next()`, после чего итератор был сдвинут на следующий символ (для конкретной задачи это не важно)
- Чтобы собрать отдельные символы в строку, потребовалось получить итератор на массив `iter()` и объединить все элементы `collect()`
