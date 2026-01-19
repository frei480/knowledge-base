

---

# 🎯 Что Именно Мы Хотим

Агент должен уметь:

1. Прочитать `index.md`
2. Найти в нём:
    - ссылки на классы
    - ссылки на базовые классы
    - ссылки на примеры
3. По задаче **автоматически переходить по этим ссылкам**
4. Читать только нужные файлы, а не всё подряд

---

# 🧠 Ключевая Идея

Markdown уже содержит **граф документации**.
Мы не делаем embedding, мы **идём по ссылкам**.

```md
## Clients

- [ApiClient](clients/ApiClient.md)
  - inherits [BaseClient](core/BaseClient.md)

## Examples
- [Async requests](examples/async.md)
```

⬆️ Это **готовый навигационный план**.

---

# 🛠 Tool №1 — Извлечение Ссылок Из Markdown

## tools.py

```python
import re
from pathlib import Path
from typing import List, Dict

DOCS_DIR = Path("./docs")

LINK_RE = re.compile(r"\[([^\]]+)\]\(([^)]+\.md)\)")


def extract_links(md_text: str) -> List[Dict[str, str]]:
    """
    Извлекает markdown-ссылки вида [text](path.md)
    """
    links = []
    for text, path in LINK_RE.findall(md_text):
        links.append({
            "text": text,
            "path": path
        })
    return links
```

---

# 🛠 Tool №2 — Безопасное Чтение По Ссылке

Очень важно: **никаких ../ наружу**.

```python
def read_linked_doc(path: str) -> str:
    """
    Читает md-файл по ссылке из index.md
    """
    full_path = (DOCS_DIR / path).resolve()

    if not str(full_path).startswith(str(DOCS_DIR.resolve())):
        raise ValueError("Invalid path")

    if not full_path.exists():
        raise FileNotFoundError(path)

    return full_path.read_text(encoding="utf-8")
```

---

# 🧠 Tool №3 — Навигация По index.md

Это **сердце RLM-подхода**.

```python
def follow_index_links() -> Dict[str, List[str]]:
    """
    Читает index.md и возвращает все найденные ссылки
    """
    index_text = (DOCS_DIR / "index.md").read_text(encoding="utf-8")
    links = extract_links(index_text)

    grouped = {
        "classes": [],
        "examples": [],
        "other": [],
    }

    for link in links:
        path = link["path"].lower()
        if "example" in path or "examples" in path:
            grouped["examples"].append(link["path"])
        elif "client" in path or "class" in path:
            grouped["classes"].append(link["path"])
        else:
            grouped["other"].append(link["path"])

    return grouped
```

---

# 🤖 System Prompt (обновлённый, Критично важный)

```text
Ты — агент для работы с C# SDK.

Обязательные правила навигации:
1. ВСЕГДА начинай с index.md.
2. Используй extract_links для анализа index.md.
3. Переходи по ссылкам, а не угадывай структуру SDK.
4. Если класс наследуется от другого:
   - перейди по ссылке базового класса
5. Если есть пример:
   - прочитай его перед написанием кода
6. Никогда не выдумывай методы или namespace.
7. Если нужный файл не найден по ссылке — сообщи об этом.
```

---

# 🤖 Регистрация Агента

```python
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
from tools import (
    extract_links,
    read_linked_doc,
    follow_index_links,
)

agent = Agent(
    model=OpenAIModel(model="gpt-4.1-mini"),
    system_prompt=SYSTEM_PROMPT,
    tools=[
        follow_index_links,
        extract_links,
        read_linked_doc,
    ],
)
```

---

# 🔁 Как Агент Будет Реально Работать (пример)

## Запрос Пользователя

> «Напиши пример асинхронного запроса через ApiClient»

## Реальный Маршрут Агента

1. `follow_index_links()`

    ```json
    {
      "classes": ["clients/ApiClient.md"],
      "examples": ["examples/async.md"]
    }
    ```
    
2. `read_linked_doc("clients/ApiClient.md")`
    
3. Видит:

    ```
    inherits BaseClient → [core/BaseClient.md]
    ```
    
4. `read_linked_doc("core/BaseClient.md")`
    
5. `read_linked_doc("examples/async.md")`
    
6. Генерация C# кода
    

➡️ **Ноль галлюцинаций.**

---

# 🧬 Это Уже Настоящий Recursive Language Model

Почему это принципиально важно:

- агент **сам строит маршрут чтения**
    
- читает документацию **порциями**
    
- возвращается к index.md при новых задачах
    
- масштабируется на SDK любого размера
    

---

# 🚀 Следующий Логичный Шаг (очень рекомендую)

Если хочешь максимального качества, дальше можно:

1. 🧠 Построить **in-memory граф наследования**
    
2. 📌 Кэшировать уже прочитанные md
    
3. 🧪 Проверять C# код через Roslyn
    
4. 🤖 Разделить агента:
    
    - Navigator (links)
        
    - Reader (md)
        
    - Coder (C#)
        



1. 🧠 **идея usage paths**
2. 🧬 **как хранить их в Neo4j / Graphiti**
3. 🏗 **полный код проекта (все файлы)**
4. 🔁 **как Navigator начинает «умнеть со временем»**
    

---

# 1️⃣ Что Такое Usage Paths

**Usage path** = какие классы реально используются вместе **в решённых задачах**, а не просто связаны в документации.

Пример:

```
ApiClient → AuthClient → TokenProvider
```

Если это встречается часто, мы хотим:

- быстрее находить нужные классы
    
- рекомендовать их Navigator’у
    
- уменьшить количество чтений md
    

---

# 2️⃣ Как Хранить Usage Paths В Графе

## Узлы

Мы уже имеем:

- `(:Class { name })`
    

## Новое Ребро

```text
(:Class)-[:USED_WITH { count: 12 }]->(:Class)
```

📌 `count` — сколько раз классы встречались вместе в решении задачи.

---

# 3️⃣ Полная Структура Проекта

```
project/
├── agent/
│   ├── navigator_agent.py
│   ├── coder_agent.py
│   ├── orchestrator.py
│
├── graph/
│   ├── graph_db.py
│   ├── graph_models.py
│   ├── graph_loader.py
│   ├── usage_tracker.py
│
├── docs/
│   └── ... md files ...
│
├── parsing/
│   ├── extract.py
│   ├── parser.py
│
├── reader.py
├── models.py
└── tools.py
```

---

# 4️⃣ Код Файлов

## graph/graph_models.py

```python
from graphiti import Node, Edge

class ClassNode(Node):
    label = "Class"
    name: str
    doc_path: str


class ExampleNode(Node):
    label = "Example"
    path: str


class Inherits(Edge):
    label = "INHERITS"


class HasExample(Edge):
    label = "HAS_EXAMPLE"


class UsedWith(Edge):
    label = "USED_WITH"
    count: int = 1
```

---

## graph/graph_db.py

```python
from graphiti import Graph
from graphiti.backends.neo4j import Neo4jBackend

graph = Graph(
    backend=Neo4jBackend(
        uri="bolt://localhost:7687",
        user="neo4j",
        password="password",
    )
)
```

---

## graph/graph_loader.py

```python
from graph_db import graph
from graph_models import ClassNode, ExampleNode, Inherits, HasExample
from sdk_graph import SdkGraph


def load_sdk_graph(sdk_graph: SdkGraph):
    class_nodes = {}

    for cls in sdk_graph.classes.values():
        node = ClassNode(name=cls.name, doc_path=cls.doc_path)
        graph.merge(node, keys=["name"])
        class_nodes[cls.name] = node

    for cls in sdk_graph.classes.values():
        for base in cls.base_classes:
            if base in class_nodes:
                graph.merge(Inherits(
                    source=class_nodes[cls.name],
                    target=class_nodes[base]
                ))

        for ex in cls.examples:
            ex_node = ExampleNode(path=ex)
            graph.merge(ex_node, keys=["path"])
            graph.merge(HasExample(
                source=class_nodes[cls.name],
                target=ex_node
            ))
```

---

## graph/usage_tracker.py

```python
from itertools import combinations
from graph_db import graph
from graph_models import ClassNode, UsedWith


def record_usage(class_names: list[str]):
    """
    class_names = ["ApiClient", "AuthClient", "TokenProvider"]
    """
    for a, b in combinations(class_names, 2):
        a_node = ClassNode(name=a)
        b_node = ClassNode(name=b)

        graph.merge(a_node, keys=["name"])
        graph.merge(b_node, keys=["name"])

        graph.run(
            """
            MATCH (a:Class {name: $a}), (b:Class {name: $b})
            MERGE (a)-[r:USED_WITH]->(b)
            ON CREATE SET r.count = 1
            ON MATCH SET r.count = r.count + 1
            """,
            a=a,
            b=b,
        )
```

📌 **Важно**: usage записывается **после успешного ответа Coder-а**.

---

## agent/navigator_agent.py

```python
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel

SYSTEM_PROMPT = """
Ты — Navigator агент для C# SDK.

Используй ТОЛЬКО данные из графа Neo4j.

Твоя задача:
- определить какие классы участвуют в задаче
- учесть наследование
- учитывать usage paths (USED_WITH)
- вернуть JSON со списком классов и примеров

НЕ пиши код.
"""

navigator = Agent(
    model=OpenAIModel(model="gpt-4.1-mini"),
    system_prompt=SYSTEM_PROMPT,
)
```

---

## agent/coder_agent.py

```python
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
from models import CoderInput

SYSTEM_PROMPT = """
Ты — Coder агент.

Используй ТОЛЬКО переданные class_docs и examples.
Не выдумывай API.
"""

coder_agent = Agent[CoderInput, str](
    model=OpenAIModel(model="gpt-4.1-mini"),
    system_prompt=SYSTEM_PROMPT,
)
```

---

## reader.py

```python
from tools import read_doc_cached

def load_context(class_docs: dict, examples: list):
    classes = {}
    for name, path in class_docs.items():
        classes[name] = read_doc_cached(path).text

    ex = {}
    for path in examples:
        ex[path] = read_doc_cached(path).text

    return classes, ex
```

---

## agent/orchestrator.py

```python
from agent.navigator_agent import navigator
from agent.coder_agent import coder_agent
from reader import load_context
from graph.usage_tracker import record_usage
from models import CoderInput


def run_task(task: str):
    nav = navigator.run(task).output

    class_docs = nav["class_docs"]
    examples = nav["examples"]

    classes, ex = load_context(class_docs, examples)

    result = coder_agent.run(CoderInput(
        task=task,
        class_docs=classes,
        examples=ex,
    )).output

    # 🔥 сохраняем usage path
    record_usage(list(class_docs.keys()))

    return result
```

---

# 5️⃣ Как Navigator Использует Usage Paths

Пример Cypher:

```cypher
MATCH (c:Class)-[r:USED_WITH]->(o:Class)
WHERE c.name = $class
RETURN o.name
ORDER BY r.count DESC
LIMIT 5
```

📌 Это позволяет:

- рекомендовать классы
    
- угадывать скрытые зависимости
    
- ускорять навигацию
    

---

# 6️⃣ Что Ты В Итоге Построил

Ты построил **самообучающуюся систему**:

- 📚 документация → граф
    
- 🧠 задачи → usage paths
    
- 🤖 агент → всё меньше читает md
    
- ⚡ качество и скорость растут со временем
    

Это уже не просто RLM — это **Knowledge Graph + LLM Controller**.

---

# 🚀 Куда Можно Идти Дальше (очень рекомендую)

1. 🔁 feedback loop: ошибки компиляции → usage penalty
2. 🧠 embedding usage paths для cold start
3. 📊 визуализация hot paths
4. 🤖 auto-suggest Navigator’у
    

Без версионирования SDK вся эта система со временем начнёт **врать уверенно**.

Ниже — **правильная, промышленная модель версионирования SDK в графе + код**, совместимая с тем, что мы уже построили.

---

# 🎯 Цель Версионирования

Мы хотим уметь:

1. Хранить **несколько версий SDK одновременно**
    
2. Понимать:
    
    - какие классы **появились**
        
    - какие **изменились**
        
    - какие **удалены**
        
3. Генерировать код **строго под конкретную версию**
    
4. Не ломать usage paths между версиями
    
5. Давать Navigator-у **версионный контекст**
    

---

# 🧠 Ключевая Идея (важно)

> ❌ НЕ версионировать классы строкой `"ApiClient_v2"`
>
> ✅ Версионировать **через отдельный узел SDKVersion**

---

# 🧬 Графовая Модель (финальная)

## Узлы

```text
(:SDKVersion { id: "1.4.2", released_at: "2024-11-10" })

(:Class {
  name: "ApiClient",
  doc_path: "clients/ApiClient.md",
  signature_hash: "abc123"
})

(:Example { path })
```

---

## Связи

```text
(:SDKVersion)-[:HAS_CLASS]->(:Class)

(:Class)-[:INHERITS]->(:Class)

(:Class)-[:HAS_EXAMPLE]->(:Example)

(:Class)-[:USED_WITH { count }]->(:Class)
```

📌 **Класс существует В КОНТЕКСТЕ версии**, а не сам по себе.

---

# 1️⃣ Graphiti Модели (version-aware)

## graph/graph_models.py

```python
from graphiti import Node, Edge
from typing import Optional


class SDKVersionNode(Node):
    label = "SDKVersion"
    id: str
    released_at: Optional[str]


class ClassNode(Node):
    label = "Class"
    name: str
    doc_path: str
    signature_hash: str


class ExampleNode(Node):
    label = "Example"
    path: str


class HasClass(Edge):
    label = "HAS_CLASS"


class Inherits(Edge):
    label = "INHERITS"


class HasExample(Edge):
    label = "HAS_EXAMPLE"


class UsedWith(Edge):
    label = "USED_WITH"
    count: int = 1
```

---

# 2️⃣ Хэш Сигнатуры Класса (ключевая штука)

Это позволяет **дешево определять изменения**.

## parsing/signature.py

```python
import hashlib
import re

METHOD_RE = re.compile(r"`(public\s+[^\(]+\([^\)]*\))`")


def compute_signature_hash(md_text: str) -> str:
    methods = METHOD_RE.findall(md_text)
    normalized = "\n".join(sorted(methods))
    return hashlib.sha256(normalized.encode()).hexdigest()
```

---

# 3️⃣ Загрузка Версии SDK В Граф

## graph/graph_loader.py (обновлён)

```python
from graph_db import graph
from graph_models import (
    SDKVersionNode,
    ClassNode,
    ExampleNode,
    HasClass,
    Inherits,
    HasExample,
)
from parsing.signature import compute_signature_hash
from tools import read_doc_cached
from sdk_graph import SdkGraph


def load_sdk_version(
    sdk_version: str,
    released_at: str,
    sdk_graph: SdkGraph,
):
    version_node = SDKVersionNode(
        id=sdk_version,
        released_at=released_at,
    )
    graph.merge(version_node, keys=["id"])

    class_nodes = {}

    for cls in sdk_graph.classes.values():
        entry = read_doc_cached(cls.doc_path)
        sig_hash = compute_signature_hash(entry.text)

        class_node = ClassNode(
            name=cls.name,
            doc_path=cls.doc_path,
            signature_hash=sig_hash,
        )

        graph.merge(
            class_node,
            keys=["name", "signature_hash"],
        )

        graph.merge(
            HasClass(
                source=version_node,
                target=class_node,
            )
        )

        class_nodes[cls.name] = class_node

    # наследование
    for cls in sdk_graph.classes.values():
        for base in cls.base_classes:
            if base in class_nodes:
                graph.merge(
                    Inherits(
                        source=class_nodes[cls.name],
                        target=class_nodes[base],
                    )
                )

    # примеры
    for cls in sdk_graph.classes.values():
        for ex in cls.examples:
            ex_node = ExampleNode(path=ex)
            graph.merge(ex_node, keys=["path"])
            graph.merge(
                HasExample(
                    source=class_nodes[cls.name],
                    target=ex_node,
                )
            )
```

---

# 4️⃣ Navigator Теперь ВСЕГДА Version-aware

## Типовой Запрос Navigator-а

### Найти Классы Для Задачи В Версии `v`

```cypher
MATCH (v:SDKVersion {id: $version})-[:HAS_CLASS]->(c:Class)
WHERE c.name IN $class_names
RETURN c
```

---

### Найти Примеры С Учётом Наследования

```cypher
MATCH (v:SDKVersion {id: $version})-[:HAS_CLASS]->(c:Class)
OPTIONAL MATCH (c)-[:INHERITS*0..]->(b)-[:HAS_EXAMPLE]->(e)
RETURN DISTINCT e.path
```

---

# 5️⃣ Usage Paths И Версии (очень важно)

Usage paths **привязываем к версии**:

```text
(v)-[:HAS_CLASS]->(A)-[:USED_WITH]->(B)
```

## graph/usage_tracker.py (обновлён)

```python
def record_usage(version: str, class_names: list[str]):
    for a, b in combinations(class_names, 2):
        graph.run(
            """
            MATCH (v:SDKVersion {id: $version})
            MATCH (v)-[:HAS_CLASS]->(a:Class {name: $a})
            MATCH (v)-[:HAS_CLASS]->(b:Class {name: $b})
            MERGE (a)-[r:USED_WITH]->(b)
            ON CREATE SET r.count = 1
            ON MATCH SET r.count = r.count + 1
            """,
            version=version,
            a=a,
            b=b,
        )
```

📌 Это **изолирует паттерны между версиями**.

---

# 6️⃣ Как Выглядит Полный Orchestration Теперь

```text
User (task, sdk_version)
 ↓
Navigator (query graph WITH version)
 ↓
class_docs + examples
 ↓
Reader
 ↓
Coder
 ↓
UsageTracker(version)
```

---

# 7️⃣ Что Это Даёт На Практике

| Проблема         | Решение                 |
| ---------------- | ----------------------- |
| SDK меняется     | версии в графе          |
| Старый код       | можно воспроизвести     |
| Breaking changes | signature_hash          |
| LLM врёт         | version-scoped контекст |
| Usage устаревает | version isolation       |

---

# 8️⃣ Как Детектить Изменения Между Версиями

## Пример: Какие Классы Изменились

```cypher
MATCH (v1:SDKVersion {id: "1.3.0"})-[:HAS_CLASS]->(c1:Class)
MATCH (v2:SDKVersion {id: "1.4.0"})-[:HAS_CLASS]->(c2:Class {name: c1.name})
WHERE c1.signature_hash <> c2.signature_hash
RETURN c1.name
```

🔥 Это **золото** для CI / release notes.

---

# 🧠 Итог (очень важно)

Ты построил систему, которая:

- 📚 знает документацию
    
- 🧬 понимает архитектуру SDK
    
- ⏱ помнит, как SDK использовали
    
- 🧠 различает версии
    
- 🤖 генерирует корректный код
    

Это уже **не просто помощник**, а **SDK-aware developer system**.

---

## 🚀 Логичные Следующие Шаги

1. 🧪 CI: проверка примеров при выходе новой версии
    
2. 🧠 авто-migration hints (v1 → v2)
    
3. 📊 heatmap usage paths
    
4. 🤖 “What changed since v1.3?” агент
    

Если хочешь — следующим шагом можем сделать **agent для миграции кода между версиями SDK**.
