---
tags:
  - xml
---

XML-схемы (или XSD — XML Schema Definition) используются для определения структуры и правил, которым должен соответствовать XML-документ. Они описывают, какие элементы и атрибуты могут содержаться в документе, их типы данных, а также ограничения на значения.

Вот основные аспекты XML-схем:

▎1. Основные элементы схемы:

• <xs:schema>: корневой элемент схемы, который определяет пространство имен и другие атрибуты.

• <xs:element>: определяет элемент в XML-документе.

• <xs:complexType>: определяет сложный тип, который может содержать другие элементы и атрибуты.

• <xs:simpleType>: определяет простой тип, который может содержать только текстовые значения.

• <xs:attribute>: определяет атрибут элемента.

▎2. Пример XML-схемы:

Вот простой пример XML-схемы для документа, представляющего список книг:
```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="library">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="book" maxOccurs="unbounded">
                    <xs:complexType>
                        <xs:sequence>
                            <xs:element name="title" type="xs:string"/>
                            <xs:element name="author" type="xs:string"/>
                            <xs:element name="year" type="xs:int"/>
                        </xs:sequence>
                        <xs:attribute name="id" type="xs:string" use="required"/>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
```
▎3. Объяснение примера:
• Корневой элемент library может содержать множество элементов book.
• Каждый элемент book имеет три дочерних элемента (title, author, year) и один атрибут (id).
• Элементы title и author имеют тип string, а элемент year имеет тип int.

▎4. Использование схем:
XML-схемы позволяют:
• Проверять корректность XML-документов.
• Обеспечивать совместимость данных между различными системами.
• Автоматизировать процесс обработки XML-документов.

