# 1. Уточнение По Pydantic AI (Implementation details)
Фреймворк `pydantic-ai` сильно полагается на типизацию и Dependency Injection.
*   **Добавить:** Требование использовать `RunContext` для доступа к зависимостям (например, к драйверу Neo4j/Graphiti и к Reader-у) внутри инструментов (tools).
*   **Добавить:** Требование строгой типизации возвращаемых значений агентов (`result_type=SomePydanticModel`). Агенты должны общаться не строками, а Pydantic-моделями.
*   **Добавить:** Явное указание на использование `system_prompt` как динамической функции, зависящей от контекста (например, текущей версии SDK).

# 2. Схема Графа (Ontology)
Чтобы `Navigator` мог эффективно работать, ему нужна четкая схема данных.
*   **Добавить:** Описание узлов и связей для Neo4j.
    *   *Nodes:* `SDKVersion`, `Class`, `Interface`, `Method`, `Example`, `File`.
    *   *Relationships:*
        *   `(:Class)-[:INHERITS_FROM]->(:Class)`
        *   `(:Class)-[:IMPLEMENTS]->(:Interface)`
        *   `(:Class)-[:DEFINED_IN]->(:File)`
        *   `(:Example)-[:DEMONSTRATES]->(:Class)`
        *   `(:SDKVersion)-[:CONTAINS]->(:File)`
*   **Уточнить:** Индексация. Попроси добавить создание индексов в Neo4j (по имени класса, по версии) для быстрого поиска.

# 3. Логика Парсинга (Parsing Nuances)
C# — язык со спецификой. Markdown тоже бывает разным.
*   **Добавить:** Обработка **Generics** (обобщений). В документации класс может называться `List<T>`, а ссылка вести на `List_T.md`. Парсер должен это понимать.
*   **Добавить:** Обработка **Extension Methods**. Часто в SDK функционал расширяется методами, описанными в других файлах.
*   **Добавить:** Формат ссылок. Указать, являются ли ссылки в md относительными (`../Classes/Foo.md`) или абсолютными. Агент должен уметь нормализовать пути.

# 4. Контракт Между Navigator И Coder
*   **Добавить:** Структура `CoderContext`:
    ```json
    {
      "task_description": "...",
      "sdk_version": "1.0.0",
      "classes": [
        {
          "name": "MyClient",
          "signature": "class MyClient : BaseClient",
          "methods": ["connect(string url)", "send(Data d)"],
          "documentation": "Main entry point..."
        }
      ],
      "relevant_examples": [
        {
          "code": "var client = new MyClient();...",
          "description": "Basic usage"
        }
      ]
    }
    ```
    Это гарантирует, что Coder не будет галлюцинировать методы.



# Итоговый Промпт
**Роль:** Senior Python Developer / AI Architect.
**Задача:** Сгенерировать production-ready код мульти-агентной системы на Python с использованием фреймворка **pydantic-ai** и графовой базы данных (Neo4j/Graphiti).

**Контекст проекта:**
Мы создаем AI-ассистента для написания кода на основе C# SDK. Документация SDK представлена в виде Markdown файлов, где `index.md` — точка входа с графом ссылок.
**Ограничения:** Мы НЕ используем векторные эмбеддинги. Вся навигация строится на детерминированном графе ссылок и структуре наследования C#.

**Технологический стек:**
- **Framework:** `pydantic-ai` (использовать `Agent`, `RunContext`, `tool`, dependency injection).
- **Database:** Neo4j (через `neo4j` драйвер) или Graphiti сервис.
- **Language:** Python 3.10+.
- **Parsing:** `marko` или `beautifulsoup` для парсинга MD/HTML.


# Архитектура И Требования

## 1. Модель Данных (Graph Schema)
Реализовать создание схемы в БД со следующими узлами и связями:
- **Nodes:** `SDKVersion`, `File` (с хэшем контента), `Class`, `Interface`, `Example`.
- **Edges:**
  - `(:SDKVersion)-[:INCLUDES]->(:File)`
  - `(:File)-[:DEFINES]->(:Class)`
  - `(:Class)-[:INHERITS]->(:Class)` (учитывать глубину наследования)
  - `(:Example)-[:REFERENCES]->(:Class)`
  - `(:File)-[:LINKS_TO]->(:File)`

## 2. Агенты (Pydantic AI Agents)

**A. Navigator Agent**
*   **Ответственность:** "Мозг" навигации. Не пишет код.
*   **Tools:**
    1.  `scan_documentation(version)`: Рекурсивный обход ссылок от `index.md`. Парсинг C# сигнатур, выявление наследования. Обновление графа (только если hash файла изменился).
    2.  `query_graph(query)`: Поиск классов/примеров в Neo4j по задаче пользователя.
*   **Input:** Пользовательский запрос + версия SDK.
*   **Output (Strict Type):** Объект `CodingContext` (JSON-схема), содержащий только отфильтрованную информацию: полные сигнатуры нужных классов, предков и релевантные примеры кода.

**B. Coder Agent**
*   **Ответственность:** Генерация кода.
*   **Ограничения:** Изолирован от файловой системы и сети. Не имеет tools для чтения файлов.
*   **Input:** Объект `CodingContext` от Navigator.
*   **Output:** Готовый код на Python/C# (в зависимости от задачи).

**C. Reader (Infrastructure Layer)**
*   Простой класс/сервис (не агент), который физически читает `.md` файлы и вычисляет SHA-256 хэши. Внедряется в Navigator через DI.

## 3. Детали Реализации
1.  **Кэширование:** При старте сканирования проверять хэши файлов. Если хэш в БД совпадает с хэшем на диске — пропускать парсинг (incremental update). git-репозиторий документации находится в подкаталоге ./docs.
2.  **C# Specifics:** Парсер должен корректно обрабатывать C# Generics (`List<T>`), наследование (`:`) и интерфейсы.
3.  **Versioning:** Граф должен поддерживать одновременное хранение нескольких версий SDK (узел `SDKVersion` как корень подграфа).
4.  **Pydantic AI Injection:** Использовать `deps_type` для передачи соединения с БД и Reader-сервиса внутрь агентов.
 5. Специфика обработки T-FLEX API (Critical parsing rules) :
    1. **Generic Types Mapping:** Учесть, что классы `Name<T>` в документации отображаются в файлы `Name_1.md` (где 1 — арность). Парсер должен уметь сопоставлять `List<T>` -> `List_1`.
    2.  **Examples Handling:** Файлы примеров часто имеют имена-GUID (напр. `6208cb0a-….md`). Их нельзя игнорировать. При построении графа извлекать семантику примера из заголовка `#` внутри файла или из текста ссылки, ведущей на этот файл.
    3.  **Member Lists Navigation:** Документация класса часто разбита на отдельные файлы списков (`Methods_T_…`, `Properties_T_…`). Navigator должен понимать, что `Methods_T_Class.md` является продолжением `T_Class.md`.
    4. **Usage Patterns:** В примерах кода (см. `FragmentVariables`) используются обязательные блоки `BeginChanges`/`EndChanges`. Coder Agent должен получать инструкцию "Always check for transaction scopes in examples" и применять их в генерируемом коде.
    5. **Delegates & Events:** Для событий (Events) парсер должен находить связанный `Delegate` тип, чтобы Coder знал сигнатуру callback-функции.
# Ожидаемый Результат
Сгенерируй структуру проекта и код для следующих файлов:

1.  `models.py`: Pydantic модели для узлов графа и контракта общения между агентами (`CodingContext`).
2.  `graph/graph_loader.py`: Логика обхода `index.md`, парсинга ссылок/наследования и записи в Neo4j (Cypher запросы).
3.  `agent/navigator.py`: Код агента на `pydantic-ai` с определением tools.
4.  `agent/coder.py`: Код агента-генератора.
5.  `orchestrator.py`: Скрипт запуска, связывающий User -> Navigator -> Coder.

**Важно:** Код должен быть модульным, с обработкой ошибок (например, битые ссылки в md) и готовым к масштабированию на тысячи файлов.
