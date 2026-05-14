---
tags:
  - OCR
---

RapidOCR обычно состоит из трех компонентов:
1. **Det** (Detection) — обнаружение текста.
2. **Rec** (Recognition) — распознавание символов (нужен .onnx файл + .txt файл словаря).
3. **Cls** (Classification) — определение ориентации текста (необязательно)
   
Простой путь, скачать из репозитория Modelscope:
https://www.modelscope.cn/models/RapidAI/RapidOCR/tree/master/onnx/PP-OCRv5/
файлов много,
```
RapidOCR
	onnx
	paddle
	resources
	torch
```
Поэтому клонируем репозиторий:
```bash
git lfs install
git clone https://www.modelscope.cn/RapidAI/RapidOCR.git
```

Сложный путь,
# 1. Взять Модели От PaddleOCR cyrillic_PP-OCRv3_xx:
https://github.com/PaddlePaddle/PaddleOCR/blob/main/docs/index/index.ru.md
Модель обнаружения:
- [модель вывода](https://paddleocr.bj.bcebos.com/PP-OCRv3/multilingual/Multilingual_PP-OCRv3_det_infer.tar)
- [обученный модель](https://paddleocr.bj.bcebos.com/PP-OCRv3/multilingual/Multilingual_PP-OCRv3_det_distill_train.tar)
Модель распознавания:
- [модель вывода](https://paddleocr.bj.bcebos.com/PP-OCRv3/multilingual/cyrillic_PP-OCRv3_rec_infer.tar)
- [обученный модель](https://paddleocr.bj.bcebos.com/PP-OCRv3/multilingual/cyrillic_PP-OCRv3_rec_train.tar)

- **Character Dictionary** (файл .txt со списком символов, обычно лежит в папке ppocr/utils/dict/ репозитория PaddleOCR).
# 2. Конвертация В ONNX

Можно использовать стандартный paddle2onnx, для RapidOCR лучше всего подходит инструмент **paddleocr_convert**, созданный автором RapidOCR. Он автоматически прописывает нужные метаданные и словарь символов внутрь ONNX-файла.

**Установка конвертера:**
```bash
pip install paddleocr_convert
```

**Пример конвертации (для модели распознавания):**

```bash
paddleocr_convert -p https://paddleocr.bj.bcebos.com/PP-OCRv3/chinese/ch_PP-OCRv3_rec_infer.tar \
                  -o models \
                  -txt_path https://raw.githubusercontent.com/PaddlePaddle/PaddleOCR/main/ppocr/utils/ppocr_keys_v1.txt
```
- -p: Ссылка на .tar архив или путь к локальной папке с inference.pdmodel и inference.pdiparams.
- -o: Папка, куда сохранится готовый .onnx.
- -txt_path: Путь к словарю (обязательно для моделей распознавания).