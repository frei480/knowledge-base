---
tags:
  - rust
Ссылка: https://www.youtube.com/watch?v=784JWR4oxOI
Тип: Пересказ видео
Название: All Rust features explained
---
# Объяснены Все Особенности Rust

00:00 Введение в Rust

• Rust заимствует функции из других языков.
• Понимание этих функций важно для эффективного использования Rust.
• Видео расскажет о популярных функциях Rust и их происхождении.

01:08 Принцип абстракций с нулевой стоимостью

• Абстракции высокого уровня не должны снижать производительность.
• Пример с вектором и итератором в C++ и Rust.
• Rust подчеркивает безопасность памяти и потоков.

03:04 Модель владения в Rust

• Модель владения основана на шаблоне [[RAII из C++]].
• Пример управления ресурсами с использованием RAII.
• Rust интегрировал правила владения и заимствования.

05:08 Правила владения и заимствования

• У каждого значения в Rust есть владелец.
• Одновременно может быть только один владелец.
• Когда владелец выходит за область видимости, значение удаляется
• Пример с базой данных в Rust.
```rust
struct DatabaseConnection {
	connection: Connection
}

impl DatabaseConnection {
	fn new(db_name: &str) -> Result<DatabaseConnection, Error>{
		let connection = Connection::open(db_name)?;
		Ok(DatabaseConnection { connection })
	}
}
fn main() -> Result<(), Error> {
	let connection = DatabseConnection::new("example.db")?;
	//Операции с БД
	Ok(())
}
```
• Правила заимствования предотвращают "гонки данных" и разыменование нулевого указателя.
1. В любой момент времени у вас может быть либо одна изменяемая ссылка, либо любое количество неизменяемых ссылок.
2. Ссылки всегда должны быть действительными.
```rust
struct DatabaseConnection {
	connection: Connection
}

impl DatabaseConnection {
	fn new(db_name: &str) -> Result<DatabaseConnection, Error>{
		let connection = Connection::open(db_name)?;
		Ok(DatabaseConnection { connection })
	}
}
fn main() -> Result<(), Error> {
	let connection = DatabseConnection::new("example.db")?;
	let conn1 = &connection;
	let conn2 = &connection;
	let conn3 = &connection;
	//Ошибка компиляции: используется неизменяемые и изменяемые ссылки одновременно
	let conn4 = &mut connection;
	
	//Операции с БД
	Ok(())
}
```

07:06 Алгебраические типы данных ADT

• ADT позволяют создавать составные типы.
• Пример реализации ADT в Haskell и Rust.
• Rust интегрирует ADT в императивный стиль программирования.
```rust
enum Employee {
	Manager {name: String, subordinates: Vec<Box<Employee>>},
	Worker { name:: String, manager: String },
}
```
08:26 Сопоставление шаблонов

• Сопоставление шаблонов позволяет обрабатывать каждый вариант по-разному.
• Пример использования выражения match в Rust.
• Сопоставление с образцом обеспечивает полноту и предотвращает ошибки во время выполнения.
```rust
enum Employee {
	Manager {name: String, subordinates: Vec<Box<Employee>>},
	Worker { name:: String, manager: String },
}
fn main() {
	let employee = Employee::worker {name: "Bob".to_string(), manager: "Alice".to_string()};
	match employee {
		Employee::Manager {name, subordinates } =>
			println!("{} is a manager with {} subordinates.", name, subordinates.len()),
		Employee::Worker {name, manager } =>
			println!("{} is a worker under the management of {}.", name, manager),
	}
}
```
09:24 Полиморфизм в Rust

• Полиморфизм позволяет объектам или функциям вести себя по-разному в зависимости от контекста.
• В Rust полиморфизм реализуется через [[Trait в Rust|признаки]] и обобщения.
• Признаки определяют набор функций и методов, которые могут реализовывать типы.
```rust
//признак
trait Shape {
	fn area(&self) -> f64;
}
struct Rectangle {
	width: f64,
	height: f64,
}
impl Shape for Rectangle {
	fn area(&self) -> f64 {
		self.width * self.height
	}
}
struct Circle {
	radius: f64,
}
impl Shape for Circle {
	fn area(&self) -> f64 {
		std::f64::consts::PI * self.radius * self.radius
	}
}
```
• Обобщения позволяют писать код, абстрагированный от типов, повышая эффективность.

10:22 Привязка дженериков к признакам

• Дженерики в Rust могут быть привязаны к признакам.
• Пример: функция print_area принимает аргумент типа, реализующего признак shape.
```rust
trait Shape {
	fn area(&self) -> f64;
}
//...
fn print_area<T: Shape>(shape: &T) {
	println!("Area: {}", shape.area());
}

fn main() {

	let rectangle = Rectangle {
		width: 5.0,
		height: 3.0,
	};
	print_area(&rectangle);
	let circle = Circle { radius: 2.5 };
	print_area(&circle);
	let shapes: Vec<Box<dyn Shape>> =  vec![Box::new(rectangle), Box::new(circle)];
}
```
• Rust опирается на концепцию классов типов Haskell, включая права собственности и сроки жизни.

11:01 Динамическая диспетчеризация и преимущества признаков

• Rust поддерживает динамическую диспетчеризацию объектов-признаков.
• Признаки позволяют гибко проектировать и компоновать код без строгих иерархий наследования.
• Признаки не инвазивны и избегают проблемы хрупкости базового класса.
• Система признаков по умолчанию использует статическую диспетчеризацию что обеспечивает эффективную генерацию кода.

11:52 Асинхронное программирование в Rust

• Rust позаимствовал синтаксис async/await из JavaScript.
• Асинхронное программирование позволяет задачам выполняться независимо.
• В Rust используются фьючерсы (Future), аналогичные обещаниям (promise) в JavaScript.

14:24 Различия между асинхронным программированием в Rust и JavaScript

• В Rust фьючерсы ленивы и не начинают выполняться до явного вызова.
• В JavaScript promises нетерпеливы и начинают выполняться сразу после создания.
• Rust предлагает настоящий параллелизм, используя несколько потоков.

15:38 Макросы в Rust

• Макросы позволяют определять пользовательский синтаксис и генерировать код во время компиляции.
• Пример декларативного макроса map для создания хэш-карт.
```rust
macro_rules! map{
   ($key:ty, $val:ty) =>{
	   {
		   let map: HashMap<$key, $val> = HashMap::new();
		   map
	   }
   };
   ($($key:expr => $val:expr),*) =>{
	   {
	      let mut map = HashMap::new();
	      $(map.insert($key, $val);)*
	      map
	   }
   };
}
```
пример использования макроса в коде:
```rust
fn main(){
	let scores: HashMap<String, i32> = HashMap::new();//вручную
	let scores = map!(String,i32);//через макрос
	//вручную
	let mut scores2= HashMap::new();
	scores2.insert("Red team".to_owned(), 3);
	scores2.insert("Blue team".to_owned(), 5);
	scores2.insert("Green team".to_owned(), 2);
	//через макрос
	let scores2 = map!(
	"Red team".to_owned() => 3,
	"Blue team".to_owned() => 5,
	"Green team".to_owned() => 2
	);
}
```
• Макросы Rust гигиеничны, предотвращая конфликты переменных.
```rust
macro_rules! greet {
   ($name:expr) => {
      let mut name = $name;
      println!("Hello, {}!", name);
	};
}
fn main() {
	let name = "John";
	greet!(name);
	println!("{name}");
}
```
Переменная `name` внутри макроса не конфликтует с переменной в окружающем коде.

18:48 Экосистема Rust

• Rust позаимствовал идею управления зависимостями из экосистемы JavaScript.
• Созданы Cargo и Crates для управления зависимостями и совместного использования кода.

19:27 Cargo - система сборки Rust

• Cargo - официальная система сборки Rust и менеджер пакетов.
• Выполняет задачи: создание кода, загрузка и компиляция библиотек, управление версиями и конфигурациями.
• Команды: cargo build, cargo test, cargo run, аналогичные npm install, npm test, npm start.

19:57 Управление зависимостями в Rust

• В Node.js зависимости перечислены в package.json, в Rust - в cargo.toml.
• В Node.js общедоступные зависимости из npmjs.com, в Rust - из crates.io.
• Cargo и crates.io упрощают создание проектов, тестирование и распространение пакетов.

20:45 Rust Developer Bootcamp

• Rust Developer Bootcamp - комплексная программа обучения Rust.
• Включает более 100 видеороликов, упражнения по программированию, экзамены и реальные проекты.
• Программа стартует скоро, ранний доступ доступен на let's get rusty.com/bootcamp.

