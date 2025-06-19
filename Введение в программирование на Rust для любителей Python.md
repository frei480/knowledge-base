---
tags:
  - python
  - rust
Автор: Arjan Egges
Ссылка: https://www.youtube.com/watch?v=MoqtsYLGCC4
Тип: Пересказ видео
Название: An Introduction to Coding In Rust for Pythonistas
---
# An Introduction to Coding In Rust for Pythonistas

00:00 Введение в Rust

• Rust набирает популярность как быстрый и удобный язык программирования.
• Rust уникален в подходе к безопасности типов, управлению памятью и обработке исключений.
• Синтаксис Rust и Python во многом похожи, и Rust можно привязать к модулям Python.

00:59 Программа "Привет, мир" в Rust и Python

• В Python программа "Привет, мир" проста: одна строка кода.
```python
print("Hello, World!")
```
• В Rust используется ключевое слово fn для создания основной функции.
```rust
fn main(){
	println!("Hello, World!");
}
```
• В Rust код нужно скомпилировать перед запуском, в отличие от Python.

02:53 Компиляция и запуск Rust

• Для компиляции Rust нужно установить Rust на компьютер.
• После компиляции создается исполняемый файл, который можно запустить.
```shell
rustc hello_world.rs
```
• В Rust вывод осуществляется с помощью функции println! с макросом.

03:47 Макросы и вариативные интерфейсы в Rust

• Макросы в Rust расширяются во время компиляции и полезны для избежания дублирования кода.
• Макросы позволяют использовать функции с переменным количеством аргументов.

04:45 Производительность и типизация

• Rust компилируется и оптимизируется для производительности.
• Python интерпретируется и менее оптимизирован, но более динамичен.
• Rust имеет строгую типизацию и больше ограничений, чем Python.

05:45 Управление памятью и параллелизм

• В Python есть автоматическая сборка мусора, в Rust используется [[владение и заимствование в rust|модель владения.]]
• В Python есть глобальная блокировка интерпретатора, в Rust ее нет.

05:45 Обработка ошибок и синтаксис

• В Python используются исключения, в Rust — типы результатов и опций.
• Синтаксис Python простой и лаконичный, Rust более строгий и точный.

06:43 Стили программирования

• Python поддерживает как функциональный, так и объектно-ориентированный стиль.
• Rust в основном функциональный, но имеет аспекты объектно-ориентированного программирования.

08:32 Объектно-ориентированное программирование в Rust
Пример на python:
```python
# Storing Data + Implementations

class User:
    def __init__(self, name, age):
        self.name = name
        self.email = name + "@arjancodes.com"

    def send_email(self, message):
        print(f"Sent {message} to {self.email}")
```
• В Rust нет классов, вместо них используются структуры.
• Пример: структура User с именем и адресом электронной почты.
```rust
struct User {
    name: String,
    email: String
}
impl User {
    fn new(name: &str) -> User {
        User {
            name: name.to_string(),
            email: format!("{}@arjancodes.com", name)
        }
    }
}
fn main() {
    let user = User::new("Arjan");
    println!("Hello, {}!", user.name);
    println!("Your email is: {}", user.email);
}
```
10:28 Функции и неизменяемость в Rust

• В Rust функции могут работать с структурами, возвращая значения.
• В Rust все по умолчанию константно и неизменяемо, в отличие от Python.

11:28 Неизменяемость в Rust

• В Rust переменные по умолчанию неизменяемы.
• Для изменения переменной нужно добавить слово `mut`.
• Пример: изменение имени пользователя после компиляции.

12:23 [[Обработка ошибок в Rust]]

• В Rust используется дорожное программирование, а не исключения.
• Нет типа `None`, вместо этого используется тип `Option`.
• Пример функции `get_user_option`, возвращающей `Option<User>`.
```rust
fn get_user_option(name: &str) -> Result<User, String> {
	if name == "Arjan" {
		Ok(User::new(name))
	} else {
		Err(String::from("User not found"))
	}
}
```
13:21 Сопоставление параметров

• В основной функции нужно сопоставлять параметры.
• Пример: сопоставление `Option<User>` и выполнение действий в зависимости от результата.
```rust
fn main() {
    let user = User::new("Arjan");
    println!("Hello, {}!", user.name);
    println!("Your email is: {}", user.email);
    let user_option = get_user_option("Arjan");
    match user_option {
	    Some(user) => println!("Hello, {}!", user.name),
	    None => println!("User not found")
    }
	
}
```

14:15 Типы `Option` и `Result`

• `Option` используется, когда значение может отсутствовать.
• `Result` используется для различения нормального состояния и ошибки.
• Пример использования сопоставления для обработки состояний.
```rust
let user_result = get_user_option("Arjan");
    match user_result {
	    Ok(user) => println!("Hello, {}!", user.name),
	    Err(err) => println!("{}",err)
    }
```
16:02 [[Trait в Rust|Трейты в Rust]]

• Трейты похожи на ABC или протоколы в Python.
• Пример: абстрактный базовый класс `Device IoT` и его подкласс `Light`.
```python
import abc
import enum


class Connection:
    def __init__(self):
        self.name = None
        self.ip = None
        self.port = None

    @property
    def status(self):
        return (
            ConnectionStatus.CONNECTED
            if all([self.name, self.ip, self.port])
            else ConnectionStatus.DISCONNECTED
        )

    def connect(self, name, ip, port):
        self.name = name
        self.ip = ip
        self.port = port

    def disconnect(self):
        self.name = None
        self.ip = None
        self.port = None

    def send(self, message):
        print(f"Sent {message} to {self.name} at {self.ip}:{self.port}")

    def receive(self):
        return f"Received message from {self.name} at {self.ip}:{self.port}"


class IOTDevice(abc.ABC):
    @abc.abstractmethod
    def connect(self, name, ip, port):
        pass

    @abc.abstractmethod
    def disconnect(self):
        pass

    @abc.abstractmethod
    def send(self, message):
        pass

    @abc.abstractmethod
    def receive(self):
        pass


class Light(IOTDevice):
    def __init__(self, connection):
        self.connection = connection

    def connect(self, name, ip, port):
        self.connection.connect(name, ip, port)

    def disconnect(self):
        self.connection.disconnect()

    def send(self, message):
        self.connection.send(message)

    def receive(self):
        return self.connection.receive()


class ConnectionStatus(enum.Enum):
    CONNECTED = 1
    DISCONNECTED = 2


if __name__ == "__main__":
    connection = Connection()
    print(connection.status)
    connection.connect("Arjan", "localhost", 8080)
    print(connection.status)
    connection.disconnect()
    print(connection.status)
```
• Трейты не связаны с типом, их нужно явно реализовывать.
```rust
#[derive(Debug, Default)]
enum ConnectionStatus {
    Connected{ name: String, ip: String, port: u16 },
    #[default]
    Disconnected,
}

impl ConnectionStatus {
    fn new(name: String, ip: String, port: u16) -> ConnectionStatus {
        ConnectionStatus::Connected{ name, ip, port }
    }

    fn connect(&mut self, name: String, ip: String, port: u16) {
        *self = ConnectionStatus::Connected{ name, ip, port };
    }

    fn disconnect(&mut self) {
        *self = ConnectionStatus::Disconnected;
    }

    fn send(&self, message: &str) {
        match self {
            ConnectionStatus::Connected{ name, ip, port } => {
                println!("Sent {} to {} at {}:{}", message, name, ip, port);
            },
            ConnectionStatus::Disconnected => {
                println!("Cannot send message, not connected");
            },
        }
    }

    fn receive(&self) -> Option<String> {
        if let ConnectionStatus::Connected{ name, ip, port } = self {
            Some(format!("Received message from {} at {}:{}", name, ip, port))
        } else {
            None
        }
    }
}
trait IOTDevice {
    fn connect(&mut self);
    fn disconnect(&mut self);
    fn send(&mut self, message: &str);
    fn receive(&mut self) -> Option<String>;
}

struct Light {
    connection: ConnectionStatus,
}

impl IOTDevice for Light {
    fn connect(&mut self) {
        self.connection.connect("Arjan".to_string(), "localhost".to_string(), 8080);
    }

    fn disconnect(&mut self) {
        self.connection.disconnect();
    }

    fn send(&mut self, message: &str) {
        self.connection.send(message);
    }

    fn receive(&mut self) -> Option<String> {
        self.connection.receive()
    }
}

// impl_iot_device!(Light); // this macro can be used instead of the above impl IOTDevice for Light


impl Light {
    fn new() -> Light {
        Light {
            connection: ConnectionStatus::default(),
        }
    }
}


fn main() {
    let mut light = Light::new();
    light.connect();
    light.send("Hello");
    light.disconnect();
    println!("{:?}", light.receive());
}
```

17:55 [[Макросы в Rust|Макросы для трейтов]]

• Макросы позволяют автоматически реализовывать трейты для типов.
• Пример макроса для реализации `Device IoT` для различных типов устройств вместо `impl IOTDevice for Light`.
```rust
macro_rules! impl_iot_device {
    ($type:ty) => {
        impl IOTDevice for $type {
            fn connect(&mut self) {
                self.connection.connect("Arjan", "localhost", 8080);
            }

            fn disconnect(&mut self) {
                self.connection.disconnect();
            }

            fn send(&mut self, message: &str) {
                self.connection.send(message);
            }

            fn receive(&mut self) -> Option<String> {
                self.connection.receive()
            }
        }
    };
}
impl_iot_device!(Light);
```
18:52 Заключение

• Rust мощный, быстрый и высокопроизводительный язык.
• Python остается гибким и мощным языком с динамической типизацией.
• Оба языка имеют свои преимущества и экосистемы библиотек.
• Привязка для Rust позволяет [[Сочетание Rust и Python|вызывать код Rust из Python]].
