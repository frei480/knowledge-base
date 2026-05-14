1. Файлы моделей находятся в репозитории, можно скачать:
- https://github.com/tesseract-ocr/tessdata_fast
- https://github.com/tesseract-ocr/tessdata_best
1. Нужно установить пакет поддержки русского языка
используя dnf
```bash
dnf install tesseract-langpack-rus
```
используя apt:
```
apt get tesseract-ocr-rus
```
пакет содержит один файл:
```bash
dnf repoquery -l tesseract-langpack-rus
/usr/share/tesseract/tessdata/rus.traineddata
```