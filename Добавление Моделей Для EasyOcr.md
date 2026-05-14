---
tags:
  - OCR
---

В файле https://github.com/JaidedAI/EasyOCR/blob/master/easyocr/config.py содержится информация о кириллической модели:
```json
'cyrillic_g2':{
            'filename': 'cyrillic_g2.pth',
            'model_script': 'cyrillic',
            'url': 'https://github.com/JaidedAI/EasyOCR/releases/download/v1.6.1/cyrillic_g2.zip',
            ...
```
Качаем файл по ссылке и устанавливаем в папку `models\EasyOcr`