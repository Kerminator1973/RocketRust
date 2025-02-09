# Мои решения тестов курса по Rust на сайте Stepik.ru

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

# Задание "Упаковщик"

Задача - ввести пять строковых значений, а затем создать из них tuple с представлением элементов в том же порядке.

Моё решение:

```rs
use std::io;

fn main() {

    let mut values: [String; 5] = Default::default();

    for i in 0..values.len() {
        io::stdin().read_line(&mut values[i]).expect("Failed to get input");
    }

    let tup = (values[0].clone(), values[1].clone(), values[2].clone(), values[3].clone(), values[4].clone());
    print!("{:?}", tup);
}
```

В Rust мы не можем включить существующий элемент в другой объект просто так - будет получено сообщение об ошибке. Но мы можем клонировать элемент и включить tuple уже этот клонированный элемент.

ChatGPT сообщил, что у меня есть ошибка в определении массива values: "_The Default::default() for an array of String will not work as expected because String does not implement the Default trait in a way that initializes an empty string for each element. You should initialize the array with empty strings instead_".

Предложено альтернативное решение:

```rs
let mut values: [String; 5] = [
    String::new(),
    String::new(),
    String::new(),
    String::new(),
    String::new(),
];
```

Однако, мне кажется, что в примерах с сайта Stepik.org теряеся смысл ключевого случая использования tuple - возврат нескольких значений функции.

## Задание "Корректор"

Новым, в реализации данного задания, является использование массива для ввода значений:

```rs
use std::io;

fn main() {

    let mut values = Vec::new();

    for _ in 0..5 {
        let mut input = String::new();
        io::stdin().read_line(&mut input).expect("Failed to get input");
        let value: f64 = input.trim().parse().expect("Failure to parse");
        values.push(value);
    }

    let mut tup = (10.0, 5.0, -2.0, 100.0, 2000.0, 0.0);
    tup.0 = values[0].clone();
    tup.1 = values[1].clone();
    tup.2 = values[2].clone();
    tup.3 = values[3].clone();
    tup.4 = values[4].clone();
    print!("{}, {}, {}, {}, {}, {}", getint(tup.0), getint(tup.1), getint(tup.2), getint(tup.3), getint(tup.4), tup.5);
}

fn getint(value: f64) -> i64
{
    if value > 0.0 {
        return value.floor() as i64;
    }
    else 
    {
        return value.ceil() as i64;  
    }    
}
```

## Индексный прогноз

Особенность решение - используется инициализация всего массива указанным значением (`0_f64`). Определение массива осуществляется с выведением типа.

```rs
use std::io;

fn main() {
    let mut arr = [0_f64; 5];

    for index in 0..5 {
        let mut input = String::new();
        io::stdin().read_line(&mut input).expect("Failed to get input");
        arr[index] = input.trim().parse().expect("Failure to parse");
    }

    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    let index: usize = input.trim().parse().expect("Failure to parse");

    print!("{:.2}", arr[index]);
}
```

## Static Array Modifier

Особенности решения: при выводе результатов используется спецификатор формата `:?`, что позволяет вывести все элементы массива, или кортежа.

```rs
use std::io;

fn main()
{
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    let index: usize = input.trim().parse().expect("Failure to parse");

    input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    let value: i64 = input.trim().parse().expect("Failure to parse");

    let mut arr = [0; 10];
    arr[index] = value;
    print!("{:?}", arr);
}
```

## Задание "Индексный обмен"

Ввод одного значения выделен в отдельную функцию. Ключевой момент - тип возвращаемого значения указывается только в заголовке функции, т.е. для того, чтобы приспособить функцию для разных задач, достаточно поменять тип только в одной строке кода.

Также обмен двумя значениями реализован через tuple, хотя через временную переменную temp было бы более эффективно (скорее всего):

```rs
use std::io;

fn read_input() -> usize {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main()
{
    let mut arr = [-621.5, 11.1, 2.0, -7.123, 0.125, 0.0, 0.000051789];

    let first = read_input();
    let second = read_input();    

    (arr[first], arr[second]) = (arr[second], arr[first]);

    println!("{:.9?}", arr);
}
```

## Стражи города Четный и города Нечетный

Особенность решения - передача из функции типа String. Этот тип позволяет передать владение и может быть возвращён из функции. Тип str, который является типом с динамическим размером не может быть возвращён из функции напрямую.

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> i32 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn read_input_str() -> String {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().to_string()
}

fn main()
{
    let town = read_input_str();
    let first = read_input();
    let second = read_input();

    check_entrance(first, &town);
    check_entrance(second, &town);
}

fn check_entrance(number: i32, town: &str)
{
    let result = if town == "Четный" {0} else {1};

    if number % 2 == result {
        println!("{} в город {} вход разрешен", number, town)
    }
    else {
        println!("{} в город {} вход запрещен", number, town)
    }
}
```

## Разведчик

В задании разведчик нужно было разложить число на отдельные цифры, а затем сформировать из цифр строку, используя перекодировку.

Моё решение выглядит следующим образом:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> i64 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {
    let number = read_input();
    let first = number / 1000 % 10;
    let second = number / 100 % 10;
    let third = number / 10 % 10;
    let forth = number % 10;

    let mut s = String::new();
    s = s + &recode_num(first as i32);
    s = s + &recode_num(second as i32);
    s = s + &recode_num(third as i32);
    s = s + &recode_num(forth as i32);

    print!("{s}");
}

fn recode_num(num: i32) -> String
{
    return match num {
        1 => String::from("О"),
        2 => String::from("Н"),
        3 => String::from("Г"),
        _ => String::from("И"),
    }    
}
```

Моё решение не является эталоном компактности и эффективности - для разбора числа на цифры можно было бы использовать цикл. Также можно было бы использовать операцию replace(), например:

```rs
buffer.trim()
    .replace("1", "О")
    .replace("2", "Н")
    .replace("3", "Г")
    .replace("4", "И");
```

Ещё можно было бы ввести строку, а не целочисленное значение, создать итератор каждого символа строки и использовать в match уже его:

```rs
for i in number.trim().chars() {
```

Однако моим приоритетом было возвращать строку из функции, используя конструкцию match. Не оправдывая себя - написал, как написал. :-)

## Last Digit 9 ** x + 4 ** y

Задачи привлекательна тем, что позволяет дать очень лаконичное решение, отражающее минимализм конструкций Rust:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> u32 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {
    let power9 = read_input();
    let power4 = read_input();

    let result = (pow9(power9) + pow2(power4 * 2)) % 10;
    print!("Последняя цифра суммы равна {result}");
}

fn pow9(power: u32) -> u32
{
    if power % 2 == 0 {
        1
    } else {
        9
    }
}

fn pow2(power: u32) -> u32
{
    match power % 4 {
        0 => 6,
        1 => 2,
        2 => 4,
        _ => 8
    }
}
```

Что ещё понравилось в конкретной задаче - оптимизация некоторых математических операций по эвристическим формулам. По условиям задачи, функции pow2() и pow9() возвращают только последнюю цифру в числе.

## Лунная экспедиция

Решение задачи многословное, но в нём есть два важных момента:

- иллюстрируется необходимость выполнения trim() после ввода строки - в конце строки могут быть символы CR и LF, которые могут сломать логику алгоритма
- используется match для строковых значений

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> u32 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {

    let mut direction = String::new();
    io::stdin().read_line(&mut direction).expect("Failed to get input");
    let trimmed_direction = direction.trim();

    let command = read_input();

    let after_state = match trimmed_direction {
        "Север" => {
            match command {
                1 => "Запад",
                2 => "Восток",
                _ => "Север"
            }
        },
        "Восток" => {
            match command {
                1 => "Север",
                2 => "Юг",
                _ => "Восток"
            }
        },
        "Юг" => {
            match command {
                1 => "Восток",
                2 => "Запад",
                _ => "Юг"
            }
        },
        _ => {
            match command {
                1 => "Юг",
                2 => "Север",
                _ => "Запад"
            }
        }
    };

    print!("Направление лунохода после выполнения команды: {after_state}");
}
```

## Дата

Задача интересна возможностью использования оператора сопоставления (match) с использованием списка значений. Мой вариант решения:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> u8 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn read_input_year() -> u16 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {
    let day = read_input();
    let month = read_input();
    let year = read_input_year();

    let mut correct = false;

    if year >= 1 && year <= 2024 {

        match month {
            1 | 3 | 5 | 7 | 8 | 10 | 12  => {
                correct = if day >= 1 && day <= 31 {true} else {false};
            },
            2 => {
                if year % 4 == 0 && year % 100 != 0 {
                    correct = if day >= 1 && day <= 29 {true} else {false};
                } else {
                    correct = if day >= 1 && day <= 28 {true} else {false};
                }
            },
            4 | 6 | 9 | 11 => {
                correct = if day >= 1 && day <= 30 {true} else {false};
            },
            _ => {}
        }
    }

    if correct {
        print!("Дата корректна!");
    } else {
        print!("Дата некорректна!");
    }
}
```

Особенность задачи состоит в том, что требуется учитывать не только високосные года и года кратные 100, которые являются обычными, не високосными.

## Maximum and Minimum

Для использования максимального и минимального значения типов, их нужно корректно импортировать:

```rs
use std::{i32::MAX, i32::MIN, io};

fn main() {

    let mut max = MIN;
    let mut min = MAX;
    for _ in 0..n {
        // ...
    }
}
```

## Номерной валидатор

В решении задачи следует отметить использование макроса format!() и chaining methods, которые детают код читаемым и могут сильно сэкономить время. Цепочки методов - это, несомненно, та тема на изучение которой следует потратить время и развить навык использования этой техники.

Моё решение задачи, как часто бывает, слегка избыточное, но с дополнительными проверками кода:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> u64 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {
    let card_number = read_input();

    let mut acc = 0;
    for exp in 0..16 {
        acc += get_digit(card_number, exp as u32);
    }

    //println!("{acc}");

    let mut second_acc = 0;
    let mut count = 0;
    for exp in (1..16).step_by(2) {
        let digit = get_digit(card_number, exp as u32);
        second_acc += digit;

        if digit > 4 {
            count += 1;
        }
    }

    //println!("{second_acc}");
    //println!("{count}");

    let sum = acc + second_acc + count;
    //println!("{sum}");

    let formatted = get_formatted(card_number);

    if sum % 10 == 0 {
        println!("Карта с номером {} действительна", formatted);
    }
    else {
        println!("Карты с номером {} не существует", formatted);
    }
}

fn get_digit(card_number: u64, exponent: u32) -> u64
{
    let base: u64 = 10;
    let devider = base.pow(exponent);
    let reminder = card_number / devider;
    reminder % 10
}

fn get_formatted(card_number: u64) -> String
{
    let chunk_size = 4;

    let number = format!("{:0>16}",card_number);

    number
        .chars()
        .collect::<Vec<_>>() // Собираем символы в вектор
        .chunks(chunk_size) // Разбиваем на блоки
        .map(|chunk| chunk.iter().collect::<String>()) // Собираем блоки обратно в строки
        .collect::<Vec<_>>() // Собираем все блоки в вектор
        .join(" ") // Объединяем блоки с пробелом
}
```

## Stop

Моё решение:

```rs
use std::io;

fn main()
{
    let mut stop = String::new();
    io::stdin().read_line(&mut stop).expect("Failed to get input");

    let mut input = String::new();

    let mut acc = 0f32;

    loop {
        input.clear();
        std::io::stdin()
            .read_line(&mut input)
            .expect("Ошибка при чтении ввода.");

        if input == stop {
            break;
        }            

        let number: f32 = match input.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        acc += number;
    }

    let integer: i32 = (acc * 10.0) as i32;
    let corrected: f32 = (integer as f32) / 10.0;

    println!("{corrected:.1}");
}
```

В отличие от других решений, мне почему-то не удалось избежать получения в пятом тесте значения "-0.0", и пришлось решать проблему через округление.

Из интересных особенностей задачи:

- используется цикл **loop**
- при формитировании вывода я впервые использовал модификатор вывода `"{value:.1}"`

## Числовой палиндром

Моё решение задачи:

```rs
use std::io;

fn read_input() -> u32 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main()
{
    let num = read_input();

    let original_string = num.to_string();
    let reverse_string: String = original_string.chars().rev().collect();

    if original_string == reverse_string {
        print!("Число {num} является палиндромом");
    } else {
        print!("Число {num} не является палиндромом");
    }
}
```

Полезным в этом решении кажется использованием _chain method_ для формирования строки в обратном порядке: `num.to_string().chars().rev().collect()`

## Все на одно лицо

Решением задачи является:

```rs
use std::io;

fn read_input() -> u32 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main()
{
    let num = read_input();
    let num_str = num.to_string();

    let mut iter = num_str.chars();
    let first = iter.next().unwrap();
    if iter.all(|c| c == first) {
        println!("Все цифры числа {} равны", num);
    } else {
        println!("Цифры числа {} неодинаковые", num);
    }
}
```

К итератору на массив элементов типа char применяется метод **all**(), который сравнивает все элементы массива с его первым элементом.

## Ёлочка

В задаче используется функция repeat(), которая позволяет повторить строку n-раз, для вывода на экран ёлочки:

```rs
let n = read_input();

for i in 0..n {
    let spaces = " ".repeat(n - i - 1);
    let stars = "*".repeat(i * 2 + 1);
    println!("{spaces}{stars}");
}
```

## Треугольник Паскаля

Очень интересная задача, которая позволяет поэкспериментировать с работой с многомерными массивами, включая использование `Default::default()` для инициализации массива строк:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> usize {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {

    let n = read_input();

    // Создаём двух мерный массив и заполняем его нулями
    let mut array: [[i32; 10]; 10] = [[0; 10]; 10];
    for y in 0..n {
        array[y][0] = 1;
    }

    for y in 1..n {
        for x in 1..10 {
            array[y][x] = array[y-1][x-1] + array[y-1][x];
        }
    }

    // Далее формируем строки из символов
    let mut strings: [String; 10] = Default::default();
    for y in 0..n {
        let mut acc = String::new();
        for x in 0..y + 1 {
            if x > 0 {
                acc += " ";
            }

            acc += &array[y][x].to_string();
        }

        strings[y] = String::from(acc);
    }

    // Вычисляем смещение оси
    let max_length = strings[n - 1].len();

    for y in 0..n {
        let shift = (max_length - strings[y].len()) / 2;
        let spaces = " ".repeat(shift);
        println!("{}{}", spaces, strings[y]);
    }
}
```

ChatGPT 4o сгенерировал гораздо более интересный код:

```rs
fn main()
{
    let n = read_input();

    // Create a 2D vector and fill it with zeros
    let mut array = vec![vec![0; 10]; n];
    for y in 0..n {
        array[y][0] = 1;
    }

    for y in 1..n {
        for x in 1..10 {
            array[y][x] = array[y - 1][x - 1] + array[y - 1][x];
        }
    }

    // Create strings from the array
    let strings: Vec<String> = (0..n)
        .map(|y| {
            let row: String = (0..=y)
                .map(|x| array[y][x].to_string())
                .collect::<Vec<String>>()
                .join(" ");
            row
        })
        .collect();

    // Calculate the maximum length of the strings
    let max_length = strings.last().map_or(0, |s| s.len());

    for y in 0..n {
        let shift = (max_length - strings[y].len()) / 2;
        let spaces = " ".repeat(shift);
        println!("{}{}", spaces, strings[y]);
    }
}
```

По факту, Stepik ответ не принял - все варианты после шестого у него получились ошибочными. Как оказалось, это связано с тем, что "ёлочка" в варианте Stepik-а - это упрощённый вариант, в котором первый символ смещается на единицу, тогда как в моём решение - ёлочка красивая и сбалансированная вокруг "ствола дерева".

## Умный поезд

Ключевой особенностью задачи является потребность исключить из списка пассажиров тех, кто выходит на остановках. В первом варианте я использован метод remove() вектора `Vec<u32>`, но потом был найден более эффективный вариант - метод retain(), который не только обходит все элементы в списке, но и эффективно удаляет те, для которых лямбда-функция вернула значение false:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> u16 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn read_input_u32() -> u32 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {

    let count = read_input();

    // Заранее указываем количество элементов в массиве
    let mut passengers: Vec<u32> = Vec::with_capacity(count as usize);
    for _ in 0..count {
        let passenger = read_input_u32();
        passengers.push(passenger);
    }

    let mut station = 1;
    loop {

        println!("Поезд прибыл на Станцию № {station}!");

        let mut leave: Vec<u32> = Vec::new();
        passengers.retain(|&passenger| {
            let pass_id = passenger & 0xFFFF;
            let station_id = passenger >> 16;

            if station_id == station {
                leave.push(pass_id);
                false // Удаляем этого пассажира из списка
            } else {
                true // Сохрнаяем пассажира в списке
            }
        });
        
        if leave.len() > 0 {
            println!("Просим на выход пассажиров с номером(ами):");

            let result = leave.iter()
                .map(|num| num.to_string())
                .collect::<Vec<String>>()
                .join(", ");
            println!("{result}");
        }

        if passengers.len() == 0 {
            break;
        }

        station += 1;
    }
}
```

Также полезным является настройка размера массива посредством `Vec::with_capacity(count as usize);` в момент создания, т.к.мы заранее знаем размер этого массива.

## Палиндромер

Задача не была успешно решена из-за срабатывания лимита по времени. Тесты №10 и №11 не были выполнены. Похоже, что я реализовывал _brute force_ алгоритм, а в действительности, можно где-то выполнить _shortcut_.

Моё не оптимальное решение:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> u64 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {
    let value = read_input();
    let input = value.to_string();
    let digits: Vec<char> = input.chars().collect();

    let mut max_value = 0_u64;

    // Функция проверяет, что число является палиндромом и изменяет максимальное значение
    fn fun_name(perm: &[char], max_value: &mut u64) {
        let first: String = perm.iter().collect();
        let second: String = perm.iter().rev().collect();

        if first == second {
            let value: u64 = first.parse().unwrap();
            if *max_value < value {
                *max_value = value;
            }
        }
    }

    // Запускаем рекурсивную функцию, которая переберёт все возможные варианты строк
    permute(&digits, 0, &mut max_value, &fun_name);

    if max_value > 0 {
        println!("{max_value}");
    } else {
        println!("Число {value} не образует палиндром");
    }
}

// Рекурсивная функция, которая генерирует все возможные варианты чисел с указанными цифрами
fn permute(chars: &[char], start: usize, mvalue: &mut u64, callback: &dyn Fn(&[char], &mut u64)) {
    if start == chars.len() {
        callback(chars, mvalue);
        return;
    }

    for i in start..chars.len() {
        let mut chars = chars.to_vec();
        chars.swap(start, i);
        permute(&chars, start + 1, mvalue, callback);
    }
}
```

Более эффективным кажется алгоритм, в котором вычисляется количество пар цифр. Цифра без пары может быть только одна и она всегда размещается в середине числа. Соответственно, мы считаем общее количество используемых цифр в числе, для всех парных уменьшаем их количество в два раза и сортируем по убыванию. в центр вставляем единственное непарное число, если оно есть. Важно не забыть, что парные цифры могут встречаться два, четыре, шесть - и более раз.

Ниже добавил решение прошедшее Unit-тесты:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> u64 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {
    let value = read_input();
    let input = value.to_string();

    let mut counters = [0_u8; 10];

    let digits: Vec<char> = input.chars().collect();
    for ch in digits {
        let index = ((ch as u8) - ('0' as u8)) as usize;
        counters[index] = counters[index] + 1;
    }

    let mut in_the_middle = String::new();
    let mut even_counts = 0;
    for i in 0..10 {
        if counters[i] % 2 == 1 {
            even_counts += 1;
            in_the_middle = i.to_string();
        }
    }

    if even_counts > 1 {
        println!("Число {value} не образует палиндром");
        return;
    }

    let mut str = String::new();
    for i in (0..10).rev() {
        let count = counters[i] / 2;
        for _ in 0..count {
            str += &(i as u8).to_string();
        }
    }

    let reversed: String = str.chars().rev().collect();
    
    let count = str.len() + in_the_middle.len();
    if count > 0 {
        println!("{str}{in_the_middle}{reversed}");
    } else {
        println!("Число {value} не образует палиндром");
    }
}
```

Решение не оптимальное, но гораздо более производительное, чем _brute force_. Нет сил оптимизировать код прямо сейчас. :-(

## Ромбик

Задача интересна, по большей, части использованием функции repeat(), которая позволяет сформировать строку состоящую из повторяющихся n-раз подстроки:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> usize {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {
    let n = read_input();

    for i in 1..n {
        let count = i * 2 - 1;
        println!("{}{}", " ".repeat(n - i) , "*".repeat(count));
    }

    println!("{}", "*".repeat(n * 2 - 1));

    for i in (1..n).rev() {
        let count = i * 2 - 1;
        println!("{}{}", " ".repeat(n - i), "*".repeat(count));
    }
}
```

## Сложение без арифметики

Очень прикольная задача и алгоритм сложения двух чисел без использования арифметических операций, только с использованием побитовых (bitwise). Решение задачи следующее:

- на каждом шаге мы вычисляем переполнение для каждого бита первого и входного параметра. Например, для b1010 (10) и b1001 (9) переполнение будет b1000 со сдвигом на 1 бит, т.е. b10000 (16)
- используя операцию XOR мы получаем побитовую сумму без переполнения: b1010 ^ b1001 = b0011 (3)
- сумма без переполнения принимается как значение первого параметра
- переполнение принимается как значение второго параметра

В приведённом выше примере, на первой итерации, числе 10 и 9 будут заменены на b10000 (16) и b0011 (3). Переполнения на второй итерации уже не будет, т.е. нет пересекающихся бит, а b10000 ^ b0011 даст b10011 (19). Третьей итерации не будет, т.к. уже нет переполнения. В этом случае, потребуется всего два цикла для сложения двух чисел.

Решение:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> i32 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {
    let a = read_input();
    let b = read_input();
    let result = bitwise_sum(a, b);
    println!("{result}");
}

fn bitwise_sum(a: i32, b: i32) -> i32 {
    let mut x = a;
    let mut y = b;

    while y != 0 {
        // Вычисляем переполнение (carry)
        let carry = x & y;

        // Суммируем без переполнения
        x = x ^ y;

        // Переполнение переносим на следующую итерации, сдвинув его на один разряд
        y = carry << 1;
    }

    x
}
```

## Задача "Повторы"

Даны десять цифр, нужно вывести те из них, которые повторяются и вывести через пробел отсортированными в порядке убывания. 

Моё решение:

```rs
use std::io;
use std::collections::HashMap;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> i32 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {

    let mut counts: HashMap<i32, i32> = HashMap::new();

    // Подсчитываем повторы используя HashMap
    for _ in 0..10 {
        let value = read_input();
        *counts.entry(value).or_insert(0) += 1;
    }

    // Формируем выходной массив с повторами
    let mut numbers: Vec<&i32> = Vec::new();
    for (_name, _count) in &counts {
        if _count > &1 {
            numbers.push(_name);
        }
    }

    if numbers.is_empty() {
        println!("Повторяющихся чисел нет");
    }
    else {
        // Сортируем массив по убыванию
        numbers.sort_by(|a, b| b.cmp(a));
        let concatenated = numbers
            .iter() 
            .map(|&num| num.to_string()) 
            .collect::<Vec<String>>()
            .join(" ");

        println!("{concatenated}");
    }
}
```

Очень понравился API для работы в Rust с HashMap, а именно то, как можно добавлять значение в map, используя значение по умолчанию:

```rs
*counts.entry(value).or_insert(0) += 1;
```

Следует заметить, что идеоматический Rust-код действительно очень красив. В C\# потребовалось бы использовать условие и, как минимуми, две строки для изменения, или добавления кода.

## Задача "1 2 3"

Задача интересна идеоматической реализацией подсчёта количество символов в строке, с использованием лямбда-функций.

Реализация:

```rs
fn main() {

    for i in 123..=321 {

        let val = i.to_string();
        let count1 = val.chars().filter(|&c| c == '1').count();
        let count2 = val.chars().filter(|&c| c == '2').count();
        let count3 = val.chars().filter(|&c| c == '3').count();

        if count1 == 1 && count2 == 1 && count3 == 1 {
            println!("{i}");
        }
    }
}
```

Решение не является вычислительно эффективным, но зато читается достаточно легко.

## Задача "Навстречу ln 2"

Задача интересна использованием библиотеки **num-format**, которая повзоляет динамически изменять количество цифр после запятой.

Подключить библиотеку можно через "Cargo.toml":

```
[dependencies]
num-format = "0.4"
```

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> usize {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {

    let n = read_input();

    let mut acc = 1f64;
    for i in 2..=n {
        if i % 2 == 0 {
            acc -= 1f64 / (i as f64);
        }
        else {
            acc += 1f64 / (i as f64);
        }
    }

    let formatted_value = format!("{:.1$}", acc, n);
    println!("{formatted_value}");
}
```

Ключевая строка позволяет указать формат (`{:.1$}`), значение (acc) и количество цифр после запятой (n):

```rs
let formatted_value = format!("{:.1$}", acc, n);
```

В чужих решениях используется формат без указания на необходимость использования библиотеки num-format:

```rs
println!("{:.n$}", s);
```

## Решение задачи "Биатлон"

Для решения задачи "Биатлон" потребовалось теорию вероятности, которую я за последние 30 лет уже порядком забыл. Решение потребовало использования ChatGPT, который снова оказался на высоте и сгенерировал отличную инструкцию.

Для решения этой задачи можно использовать формулу вероятности для независимых событий. 

Пусть:
- \( N \) — общее количество действий,
- \( C \) — количество первых действий, которые должны завершиться успешно,
- \( P \) — вероятность успешного завершения одного действия,
- \( Q = 1 - P \) — вероятность неуспешного завершения одного действия.

Вероятность того, что первые \( C \) действий завершились успешно, равна \( P^C \).

Вероятность того, что оставшиеся \( N - C \) действий завершились неуспешно, равна \( Q^{(N - C)} \).

Таким образом, общая вероятность того, что первые \( C \) действий завершились успешно, а оставшиеся \( N - C \) действий завершились неуспешно, будет равна произведению этих двух вероятностей:

\[
P(C, N) = P^C \cdot Q^{(N - C)}
\]

Где:
- \( P(C, N) \) — искомая вероятность,
- \( P^C \) — вероятность успешного завершения первых \( C \) действий,
- \( Q^{(N - C)} \) — вероятность неуспешного завершения оставшихся \( N - C \) действий.

Следуя приведённой выше инструкции я написал код на Rust, который успешно прошел Unit-тесты:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> u8 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn read_input_f64() -> f64 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}


fn main() {

    let n = read_input();
    let p = read_input_f64();
    let c = read_input();

    let q = 1f64 - p;

    let component1 = p.powf(c as f64);
    let component2 = q.powf((n - c) as f64);
    let result = component1 * component2;

    println!("{result:.2}");
}
```

## Задача "Игральная кость"

Ещё одна задача из теории вероятности, хотя для решения задачи знать теорвер не обязательно.

Ключевое замечание по задаче - кубик является шестигранным, что следует из тестового набора данных.

Для использования теорвера необходимо вспомнить специальные символы, в частности - cимвол ∩, который обозначает пересечение двух событий. Когда мы пишем A∩B, это означает, что мы рассматриваем случаи, когда оба события A и B происходят одновременно.

Символ ∣ в теории вероятностей обозначает условие. Он используется для обозначения условной вероятности. Когда мы пишем P(A∣B), это означает вероятность события A при условии, что событие B произошло. То есть мы рассматриваем вероятность A с учетом того, что мы уже знаем, что B произошло.

Мой вариант решения, в котором используется трассировка возможных вариантов выпадения значений на кубиках:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> u8 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {

    let n = read_input();
    let m = read_input();

    // Вычисляем все возможные варианты выпадения N-очков
    let mut count = 0;
    let mut cases = 0;
    for i in 1..n {

        let reverse = n - i;
        //print!("({i}, {reverse}) = ");

        if i > 6 || reverse > 6 {
            //println!("ignore");
            continue;
        }

        cases += 1;

        if m == i || m == reverse {
            //println!("has {m}");
            count += 1;
        }
        else {
            //println!("not");
        }
    }

    //println!("Count = {count}");

    let probability = (count as f64) / (cases as f64);
    println!("{probability:.2}");
}
```

Важно заметить, что ChatGPT снова выдал грамотные советы по теории вероятности - это действительно очень полезный инструмент.

## Задача "iPOW"

Интересная часть задачи - использование алгоритма _exponentiation by squaring_ при возведении числа в целочисленную степень.

Не приводжу реализацию здесь, т.к. она весьма специфическая и в интернете доступно её описание.

## Задача "iSQRT"

Ещё одна задача интересная, но не как challange, но как ссылка на оптимизационные алгоритмы. Как и предыдущая задача - iPOW, требуется реализовать некоторый редко используемый и мало кому известный алгоритм, который вычисляет квадратный корень. Используется алгоритм Ньютона, также известный, как _Newton-Raphson method_.

Ниже приведено, условно моё решение (для уточнения постановки задачи пришлось использовать ChatGPT), с кучей упрощений и недостатков. Но, повторюсь, здесь интересна не реализация, а алгоритм:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> u32 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn sqrt(mut num: u32) -> f64 {

    if num == 0 {
        return 0.0;
    }

    let accur = 1e-10;

    // accur - это сокращение от accuracy, т.е. точность. Однако чаще используется
    // термин tolerance, точнее даже "a tolerance level"
    let mut guess = (num as f64) / 2.0; // Начальное приближение

    loop {
        let next_guess = 0.5 * (guess + (num as f64) / guess); // Формула Ньютона
        if (next_guess - guess).abs() < accur { // Проверяем достижение критерия точности
            return next_guess;
        }
        guess = next_guess; // Обновляем приближение
    }    
}

fn main() {

    let s = read_input();
    let result = sqrt(s);
    println!("{:.11}", result);
}

fn abs(num: f64) -> f64 {
    return 0.0;
}
```

Функция abs() в моём решении не используется - она требуется по условиям решения задачи, но особого смысла в её использовании нет.

## Задача "Нормализация массива"

Причина по которой этот тест был добавлен состоит в том, что в нём возвращается массив, в нём есть перечисление (enumeration) вместо доступа к элементам массива по индексу:

```rs
use std::io;

// Чтобы приспособить код к новой задаче, следует поменять тип возвращаемого значения
fn read_input() -> i32 {
    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to get input");
    input.trim().parse().expect("Failure to parse")
}

fn main() {

    let mut arr = [0i32; 20];
    for i in 0..arr.len() {
        arr[i] = read_input();
    }

    let result = norm_arr(arr);
    print!("[");
    for i in 0..result.len() - 1 {
        print!("{:.2}, ", result[i])
    }
    print!("{:.2}]", result[result.len() - 1]);
}

fn norm_arr(array: [i32; 20]) -> [f64; 20] {

    let mut output = [0f64; 20];
    if let (Some(min_val), Some(max_val)) = (array.iter().min(), array.iter().max()) {
        let diff = (max_val - min_val) as f64;

        if diff == 0.0 {
            return [0.0; 20];
        }

        for (i, &value) in array.iter().enumerate() {
            output[i] = (value as f64 - *min_val as f64) / diff;
        }

    } else {
        println!("The array is empty.");
    }   

    return output; 
}
```

ChatGPT o4-mini также порекомендовал добавить ссылку на массив, чтобы избежать ненужно копирования массива. Однако я этого не сделал, по условиям задачи на Stepik-е:

```rs
fn norm_arr(array: &[i32; 20]) -> [f64; 20] {
    // ...
    for (i, &value) in array.iter().enumerate() {
        output[i] = (value as f64 - min_val as f64) / diff;
    }
    // ...
}
```
