# Разработка Desktop-приложений на Tauri

[Tauri](https://tauri.app/) - Framework на Rust, обеспечивающий мульти-платформенную разработку Desktop-приложений. Tauri создаёт окно web-брайзера, без элементов пользовательского интерфейса браузера и запускает в нём web-приложение. Существует возможность вызвать из web-приложений нативный код, скомпилированный из исходников на Rust, что позволяет, в частности, выполнять взаимодействие с узлами компьютера на низком уровне.

Ключевые достоинства Tauri:

- очень маленький размер package - в десятки раз меньше, чем прилолжения Electron, или Avalonia
- сокрость запуска приложения очень высокая (меньше секунды)
- возможность выполнять вычислительно эффективный нативый код (скомпилированный из Rust с использованием LLVM)
- пользовательский интерфейс можно разрабатывать удобным для web-программиста образом, используя: HTML/CSS/JS, SvelteKit, Next.js, Vite, и т.д.

Недостатки: 

- время сборки проекта может быть значимым (минуты)
- инструментальные средства для работы с Rust пока ещё не такие зрелые, как для других языков программирования (C#, JavaScript)
- обмен данными между web-приложением и нативным кодом скомпилированным из Rust осуществляется через IPC, т.е. расходы на коммуникацию достаточно существенные

## Начало работы с Rust

Рекомендуется установить генератор шаблонного кода:

``` shell
cargo install create-tauri-app
```

Эта команда загрузит исходные тексты утилиты и скомпилирует её под текущую платформу. Процесс может занять несколько минут.

Генерация каркаса приложения осуществляется командой:

``` shell
cargo create-tauri-app
```

В процессе работы приложения требуется ответить на ряд вопросов генератора кода, в частности, выбрать web-платформу. В сгенерированном проекте будет две части: web-приложение (в папке src) и Rust-приложение (в папке src-tauri). Для сборки приложения следует перейти в папку `src-tauri` и выполнить команду:

``` shell
cargo build --release
```

Для сборки Tauri-приложения требуется загрузить 200+ crates (вспомогательных библиотек). Сборка приложения может занять больше минуты. В результате компиляции, на платформе Windows создаётся исполняемый файл размером ~5 мегабайт, который можно запустить на исполнение.

Для сборки приложения в Ubuntu 22.04 необходимо предварительно разрешить в Software & Updates, в разделе "Ubuntu Software" следующие источники данных: "Canonical-supported free and open-source software (main)" и "Source code". После этого следует установить [pre-requisites](https://tauri.app/v1/guides/getting-started/prerequisites):

``` shell
sudo apt update
sudo apt install libwebkit2gtk-4.0-dev \
    build-essential \
    curl \
    wget \
    libssl-dev \
    libgtk-3-dev \
    libayatana-appindicator3-dev \
    librsvg2-dev
```

Исполняемый файл занимает около 11 Мбайт и запускается менее, чем за секунду.

## Вызов команд Rust из web-приложения

В главном файле Rust-приложения ("main.rs") может быть определена команда, вызываемая из кода web-приложения. В демонстрационном приложении такая команда выглядит так:

``` rs
#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}! You've been greeted from Rust!", name)
}
```

Входной параметр функции - строковый срез (&str), а возвращаемое значение имеет тип "строка" (String). В соответствии с синтаксисом Rust, возвращено будет строковое значение, являющееся результатом вызова format!().

Регистрация команды выглядит следюущим образом:

``` rs
fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![greet])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

В приведённом выше коде создаётся контейнер комманд, в который добавляется обработчик - ранее определённая функция greet(). Если web-приложение вызовет эту команду, то Tauri создаст контекст выполнения команды и вызовет функцию greet(). В случае возникновения ошибки, в текстовую консоль (только для Debug-режима) будет выведена информация об ошибке.

JavaScript код (main.js) выполняет _destructuring_ и получает из `window.__TAURI__.tauri` функциональный объект _invoke_, который можно использовать для вызова функции на Rust:

``` js
const { invoke } = window.__TAURI__.tauri;

let greetInputEl;
let greetMsgEl;

async function greet() {
    greetMsgEl.textContent = await invoke("greet", { name: greetInputEl.value });
}

window.addEventListener("DOMContentLoaded", () => {
    greetInputEl = document.querySelector("#greet-input");
    greetMsgEl = document.querySelector("#greet-msg");
    document
        .querySelector("#greet-button")
        .addEventListener("click", () => greet());
});
```

В приведённом выше примере, при нажатии на кнопку с идентификатором greet-button, из поля ввода с идентификатором greet-input будет извлечено значение и добавлено в текстовую строку с идентификатором greet-msg.

Почитать больше о командах можно в [официальной документации по Tauri](https://tauri.app/v1/guides/features/command).

Более углублённая статья от японского разработчика - [Trying to the Tauri GUI on Rust : 3. Communicate with js (^-^](https://medium.com/@marm.nakamura/trying-to-tauri-on-rust-3-communicate-with-js-d56390116e1f) by mar-m. nakamura.

## Вызвать JavaScript-функцию из Rust-кода

Отправная точка для поиска: "tauri call js from rust". Пример кода, найденный в интернет, но который я ещё не проверял:

``` rs
.on_menu_event(|event| {
    let window = event.window();
    match event.menu_item_id() {
        "trigger" => {
            window.emit();
        }
        _ => {}
    }
})
```

## Создание приложения Tauri + React

Статья о создании приложения Tauri на React: [How To Create Tauri Desktop Applications Using React](https://medium.com/geekculture/how-to-create-tauri-desktop-applications-using-react-8541e42b1f22) by Mahbub Zaman.

Предварительные действия:

``` shell
cargo install tauri-cli
cargo tauri --help
```

Создать приложение можно используя `yarn`, альтернативный менеджер пакетов, считающийся более эффективным, чем npm. Сначала необходимо создать папку для решения, а потом уже запустить команду генерации шаблонного кода:

``` shell
yarn create tauri-app
```

Команда загружает генератор шаблонов из репозитария и запускает его. Когда приложение запрашивает какой менджер проектов использовать, выбор cargo позволит лишь добавить VanillaJS-проекты. В случае выбора npm/yarn предоставляются варианты с использованием различных библиотек и frameworks. Один из вариантов: react-ts.

После генерации кода, предлагается выполнить следующие команды:

``` shell
cd tauri-app
yarn
yarn tauri dev
```

Команда `yarn` загружает зависимости проекта. На этом этапе можно "поймать" ошибки, связанные с инструментальными средствами. Например, на моей машине была установлена устаревшая версия Node.js (14.17.5). Команда `yarn tauri dev` выполняет сборку проекта.

Используемый Front-End - действительно React с использованием TypeScript и [Vite](https://vitejs.dev/).

Чтобы выполнить сборку для выпуска продукта следует использовать команду `yarn tauri build`, однако, эта команда не будет выполнена если не поменять идентификатор приложения в файле "tauri.config.json", в ветке tauri -> bundle -> identifier со значения "com.tauri.dev" на "com.tauri.app". Размер приложения "по умолчанию" - 7,5 Мб. Запускается мгновенно. Кроме исполняемого файла скрипт собрал и msi-файл (для Windows), используя систему сборки WiX.