---
tags:
  - rust
---

видео https://www.youtube.com/watch?v=0y6RKiIk6cs
Ржавчина для чайников за 12 минут

00:00 Введение в Rust

• Rust предлагает уникальные возможности, которые делают его привлекательным для изучения.
• Видео предназначено для тех, кто хочет стать эффективным разработчиком на Rust.
• Программа Rust Live Accelerator поможет вам получить первую работу в Rust.

00:54 Почему код Rust не ломается

• Rust устраняет ошибки, которые могут возникнуть в процессе производства.
• Rust не использует сборщик мусора, что предотвращает снижение производительности.
• Модель владения в Rust предотвращает ошибки двойного освобождения и использования после освобождения.

02:09 Модель владения в Rust

• В Rust у каждого значения есть один владелец, который управляет его жизненным циклом.
```rust
fn main(){
	let user1 = User::new("Alice".to_owned());
	let user2 = user1;//ownership moved to user2
	print_name(user2);//ok
	print_name(user2);//Error:use of moved value
}
fn print_name(user3: User){
	println!("{}", user.name);
}// user3 is dropped and memory is freed
```

```rust
fn main(){
	let user1 = User::new("Alice".to_owned());
	let user2 = user1;//ownership moved to user2
	print_name(user1);//Error:use of moved value
}
```
• Передача права собственности на значение называется семантикой перемещения.
• Это устраняет ошибки памяти, такие как double free и use after free.

03:28 Модель заимствования в Rust
Правила модели заимствования:
1. Неизменяемые и изменяемые ссылки
2. Изменяемая ссылка не должна пересекаться с неизменяемыми
3. Ссылки должны указывать на действительную память

• Различаются неизменяемые и изменяемые ссылки, что предотвращает случайные мутации.
```rust
fn main(){
	let user1 = User::new("Alice".to_owned());
	let user2 = user1;//ownership moved to user2
	print_name(&user2);//ok
	print_name(&user2);//ok
}
fn print_name(user3: &User){
	println!("{}", user.name);
}// user3 is dropped and memory is freed
```
Код не скомпилируется потому что мы пытаемся передать значение по неизменяемой ссылке:
```rust
fn main(){
	let user1 = User::new("Alice".to_owned());
	let user2 = user1;//ownership moved to user2
	print_name(&user2);//ok
	update_name(&user2, "Abbey".to_owned());
	print_name(&user2);//ok
}
fn print_name(user3: &User){
	println!("{}", user.name);
}
fn update_name(user4: &User, name: String){
	user.name = name;//Error user.name is behind a & reference
}
```
Чтобы исправить это мы должны явно пометить значение `user2` как изменяемой и передать ссылку как изменяемую `mut`:
```rust
fn main(){
	let user1 = User::new("Alice".to_owned());
	let mut user2 = user1;//
	print_name(&user2);//ok
	update_name(&mut user2, "Abbey".to_owned());
	print_name(&user2);//ok
}
fn print_name(user3: &User){ //принимает неизменяемую ссылку
	println!("{}", user.name);
}
fn update_name(user4: &mut User, name: String){//принимает изменяемую ссылку
	user.name = name;//Error user.name is behind a & reference
}
```
• изменяемая ссылка не должна пересекаться с изменяемыми, `r1` и `r2` пересекаются:
```rust
fn main(){
	let mut user1 = User::new("Alice".to_owned());
	let r1 = &mut user1;
	let r2 = &user1;//Error: cannot borrow `user1` as immutable
	
	update_name(r1, "Abbey".to_owned());
	print_name(r2);
}
fn print_name(user3: &User){ //принимает неизменяемую ссылку
	println!("{}", user.name);
}
fn update_name(user4: &mut User, name: String){//принимает изменяемую ссылку
	user.name = name;
}
```
• Когда вызывается функция `update_name()` существует неизменяемая ссылка на `user1`, которая передается функции `print_name()` в следующей строке. Это проблематично, потому что неизменяемая ссылка не ожидает, что значение под неё изменится.
• Компилятор гарантирует, что если вы получаете неизменяемую ссылку на значение то оно не будет изменено пока вы владеете этой ссылкой.
Решение простое:
```rust
	let mut user1 = User::new("Alice".to_owned());
	let r1 = ;
	update_name(&mut user1, "Abbey".to_owned());
	let r2 = &user1;
	print_name(r2);
```
или
```rust
	let mut user1 = User::new("Alice".to_owned());
	update_name(&mut user1, "Abbey".to_owned());	
	print_name(&user1);
```
• Компилятор гарантирует, что ссылки указывают на действительную память. После удаления `user1` ссылки становятся недействительными и если мы попытаемся их использовать, то получим ошибку во время компиляции:
```rust
fn main(){
	let mut user1 = User::new("Alice".to_owned());
	update_name(&mut user1, "Abbey".to_owned());
	drop(useer1); //manualy drop value
	print_name(&user1); // Error: borrow of moved value
}
```
Такая система заимствований позволяет нам временно получать доступ к значениям без подводных камней, которые есть в других языках.
Таким образом, Rust предотвращает целый класс ошибок, связанный с безопасностью памяти с помощью модели владения и заимствования.

07:00 Система типов Rust

• Rust предотвращает логические ошибки с помощью системы типов.
• Обновление зависимостей не приводит к ошибкам, так как компилятор проверяет ограничения.
• Rust обеспечивает соблюдение ограничений на типы, что предотвращает неправильное использование классов.
• В Rust система типов по умолчанию безопасна и надежна.
• В Rust значение null не существует, вместо него используется обертка опция enum Option.
```rust
pub enum Option<T>{
	None,
	Some<T>
}
```
Что бы атрибут был необязательным, нужно обернуть его в опциональный enum:
```rust
pub struct User{
	name:Option<String>,
}
```
10:14 Неизменяемость и исключения в Rust

• Значения аргументов функций в Rust по умолчанию неизменяемы, изменяемые аргументы помечаются ключевым словом mut.
```rust
pub struct User{
name:String,
}
impl User{
	pub fn new(mut name: String) -> Self{
		User { name }
	}
	pub fn rename(&mut self, mut new_name: String){
		self.name = name;
	}
	pub fn from_user_profile(profile: &mut UserProfile) -> Self{
		let display_name = profile.get_display_name();
		User::new(display_name)
	}
}
```
• В Rust не нужно беспокоиться об исключениях, вместо этого используется enum-обертка для обработки ошибок.
```rust
	pub fn from_user_profile(profile: &UserProfile) -> Result<Self, String>{
	  let display_name = profile.get_display_name();
	  match display_name{
		  Some(name) => Ok(User::new(display_name)),
		  None => Err("No display name found".to_string()),
	  }	  
	}
```
• Сигнатуры функций в Rust сообщают все необходимые сведения о функции.

10:52 Преимущества Rust

• Rust-код, который компилируется, гарантированно работает.
• Для профессионального использования Rust необходимо понимать асинхронный Rust, лучшие практики и шаблоны.

11:17 Программа обучения Rust Live Accelerator

• Автор делится программой обучения Rust Live Accelerator, которая помогает освоить язык и найти работу в Rust.
• Программа включает трехэтапную дорожную карту и личную поддержку от автора.
• Программа доступна для 25 человек, запись на сайте let'sgetrusty.com.

