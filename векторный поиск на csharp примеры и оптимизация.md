---
tags:
  - csharp
---

Организация векторного поиска на C# зависит от конкретной задачи. Чаще всего речь идет о **поиске в пространстве векторных представлений** (например, эмбеддингов текста или изображений). Вот пример реализации с использованием косинусного сходства:

---

### 1. Представление векторов
Используйте массивы `float[]` или библиотеку MathNet.Numerics для работы с векторами:
```csharp
// Установите пакет: MathNet.Numerics
using MathNet.Numerics.LinearAlgebra;

Vector<float> vector1 = Vector<float>.Build.Dense(new float[] {1, 2, 3});
Vector<float> vector2 = Vector<float>.Build.Dense(new float[] {4, 5, 6});
```

---

### 2. Метрика схожести
Реализуйте функцию для вычисления **косинусного сходства**:
```csharp
float CosineSimilarity(float[] a, float[] b)
{
    if (a.Length != b.Length) 
        throw new ArgumentException("Векторы разной длины");

    float dotProduct = 0;
    float magnitudeA = 0;
    float magnitudeB = 0;

    for (int i = 0; i < a.Length; i++)
    {
        dotProduct += a[i] * b[i];
        magnitudeA += a[i] * a[i];
        magnitudeB += b[i] * b[i];
    }

    magnitudeA = MathF.Sqrt(magnitudeA);
    magnitudeB = MathF.Sqrt(magnitudeB);

    return dotProduct / (magnitudeA * magnitudeB);
}
```

---

### 3. Поиск ближайших векторов
Прогоните целевой вектор через коллекцию и найдите наиболее похожие:
```csharp
List<float[]> vectors = ...; // Ваша коллекция векторов
float[] targetVector = ...;

var results = vectors
    .Select((v, index) => new 
    { 
        Index = index, 
        Similarity = CosineSimilarity(targetVector, v) 
    })
    .OrderByDescending(x => x.Similarity)
    .Take(5); // Топ-5 результатов
```

---

### 4. Оптимизация (для больших данных)
Для ускорения поиска используйте специализированные библиотеки или алгоритмы:
- **ML.NET**: Интеграция с алгоритмами машинного обучения.
- **FAISS** (через C++/CLI или REST API): Библиотека для эффективного векторного поиска.
- **Elasticsearch**: Версии 8.x+ поддерживают векторный поиск через `dense_vector`.

---

### Пример с Elasticsearch
```csharp
var settings = new ConnectionSettings(new Uri("http://localhost:9200"))
    .DefaultIndex("vectors");

var client = new ElasticClient(settings);

// Поиск по индексу
var response = client.Search<VectorDocument>(s => s
    .Query(q => q
        .ScriptScore(ss => ss
            .Query(qq => qq.MatchAll())
            .Script(sc => sc
                .Source("cosineSimilarity(params.queryVector, 'embedding') + 1.0")
                .Params(p => p.Add("queryVector", targetVector))
            )
        )
    )
);
```

---

### Полезные ссылки
1. **MathNet.Numerics**: [github.com/mathnet/mathnet-numerics](https://github.com/mathnet/mathnet-numerics)
2. **ML.NET**: [dot.net/ml](https://dot.net/ml)
3. **Elasticsearch Vector Search**: [elastic.co/guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/dense-vector.html)

Если нужно больше деталей по конкретному сценарию — уточните задачу! 😊