---
tags:
  - xml
  - csharp
---

Давайте усложним пример, добавив атрибуты и параметры в XML-документ. Мы создадим XML-структуру с книгами, где каждая книга будет иметь атрибуты, такие как ISBN и PublishedYear, а также вложенные элементы для описания жанра и рецензий.


▎Пример XML
```xml
<Library>
    <Book ISBN="978-3-16-148410-0" PublishedYear="2020">
        <Title>Book Title 1</Title>
        <Author>Author 1</Author>
        <Genre>Fiction</Genre>
        <Reviews>
            <Review rating="5">Excellent read!</Review>
            <Review rating="4">Very good.</Review>
        </Reviews>
    </Book>
    <Book ISBN="978-1-23-456789-7" PublishedYear="2019">
        <Title>Book Title 2</Title>
        <Author>Author 2</Author>
        <Genre>Science Fiction</Genre>
        <Reviews>
            <Review rating="3">Interesting concepts.</Review>
            <Review rating="4">Good, but slow in parts.</Review>
        </Reviews>
    </Book>
</Library>
```

▎Обработка XML с использованием LINQ to XML
Теперь давайте обработаем этот XML-документ, извлекая информацию о книгах, их атрибутах и рецензиях.
```cs
using System;
using System.Linq;
using System.Xml.Linq;
  
class Program
{
    static void Main()
    {
        // Загрузка XML-документа
        XDocument xdoc = XDocument.Load("path/to/your/library.xml");
  
        // Получение всех книг
        var books = xdoc.Descendants("Book")
            .Select(b => new
            {
                ISBN = b.Attribute("ISBN")?.Value,
                PublishedYear = b.Attribute("PublishedYear")?.Value,
                Title = b.Element("Title")?.Value,
                Author = b.Element("Author")?.Value,
                Genre = b.Element("Genre")?.Value,
                Reviews = b.Element("Reviews")?.Elements("Review")
                            .Select(r => new
                            {
                                Rating = r.Attribute("rating")?.Value,
                                Comment = r.Value
                            })
            });
  
        // Вывод информации о книгах
        foreach (var book in books)
        {
            Console.WriteLine($"Title: {book.Title}");
            Console.WriteLine($"Author: {book.Author}");
            Console.WriteLine($"ISBN: {book.ISBN}");
            Console.WriteLine($"Published Year: {book.PublishedYear}");
            Console.WriteLine($"Genre: {book.Genre}");
  
            // Вывод рецензий
            Console.WriteLine("Reviews:");
            foreach (var review in book.Reviews)
            {
                Console.WriteLine($" - Rating: {review.Rating}, Comment: {review.Comment}");
            }
            Console.WriteLine();
        }
    }
}
```

▎Обработка XML с использованием XmlDocument

Теперь давайте рассмотрим, как можно сделать то же самое с использованием XmlDocument.
```cs
using System;
using System.Xml;
  
class Program
{
    static void Main()
    {
        // Загрузка XML-документа
        XmlDocument xmlDoc = new XmlDocument();
        xmlDoc.Load("path/to/your/library.xml");
  
        // Получение всех книг
        XmlNodeList bookNodes = xmlDoc.SelectNodes("//Book");
  
        foreach (XmlNode bookNode in bookNodes)
        {
            string isbn = bookNode.Attributes["ISBN"]?.Value;
            string publishedYear = bookNode.Attributes["PublishedYear"]?.Value;
            string title = bookNode["Title"]?.InnerText;
            string author = bookNode["Author"]?.InnerText;
            string genre = bookNode["Genre"]?.InnerText;
  
            Console.WriteLine($"Title: {title}");
            Console.WriteLine($"Author: {author}");
            Console.WriteLine($"ISBN: {isbn}");
            Console.WriteLine($"Published Year: {publishedYear}");
            Console.WriteLine($"Genre: {genre}");
  
            // Вывод рецензий
            XmlNodeList reviewNodes = bookNode.SelectNodes("Reviews/Review");
            Console.WriteLine("Reviews:");
            foreach (XmlNode reviewNode in reviewNodes)
            {
                string rating = reviewNode.Attributes["rating"]?.Value;
				string comment = reviewNode.InnerText;
				Console.WriteLine($" - Rating: {rating}, Comment: {comment}");
            }
            Console.WriteLine();
        }
    }
}
```
▎Заключение

В этом примере мы добавили атрибуты к элементам книги и вложенные элементы для рецензий. Мы также продемонстрировали, как извлекать информацию из таких структур с помощью двух различных подходов — LINQ to XML и XmlDocument. Оба подхода позволяют легко работать с атрибутами и вложенными элементами, но LINQ to XML предлагает более лаконичный и удобочитаемый синтаксис.