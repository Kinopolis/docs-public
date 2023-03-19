# Кинополис - онлайн кинотеатр

### Версии:
- десктоп веб
- андроид приложение
- андроид ТВ приложение
- мобильная веб версия - хотя бы для админки

### Стандартное содержимое кинотеатра:
- фильмы
- сериалы
- мультфильмы
- мультсериалы

### Дополнительные сущности:
- вселенные (Марвел/Гарри Поттер)
- серии фильмов без названия (Сплит 2016 + 2 фильма 2000 и 2019)

### rest
- источник видео материалов - торрент
- монетизация - отсутствует. Ожидается приход идей.

---

## Потенциальный стэк
### подготовка конечных файлов - `ffmpeg`:
- преобразование медиа контейнеров
- перекодирование в целевые кодеки
- ограничение битрейта
- нарезка больших медиа файлов на маленькие

изначально ffmpeg консольный, есть [оболочка](https://github.com/bramp/ffmpeg-cli-wrapper) для java, но она пригодится только для слежения за системным процессом ffmpeg, основной функционал остается в виде консольных опций

### Десктоп веб медиа плеер - [video.js](https://videojs.com/):
- умеет подгружать видео частями
  - даже если дать ссылку на один видео файл, его тоже будет грузить частями (что за магия?!)
- умеет переключать аудио и субтитры
- поддерживает кастомизацию
- использует базовые html элементы
- умеет в потоковое аудио???
### Торрент клиент - `transmission`
- для автоматической работы [консольная версия](https://cli-ck.io/transmission-cli-user-guide/)

## Инфраструктура
### Хранилище медиа файлов для чтения на клиенте

Скорее всего через апи

возможно хватит производительности s3

держать файлы сжатыми и производить декомпрессию на лету?

через [gzip](https://www.baeldung.com/cs/zlib-vs-gzip-vs-zip#gzip)

возможно ли разжимать файлы на клиенте???

- Minio + [hdd хостинг first vds](https://firstvds.ru/storage-vds) (32 ТБ трафика)
  - для каждой тачки есть ограничение цпу - если критично, то брать несколько тачек с объемом поменьше и на все раскатывать multi node Minio
- Object Storage в Яндекс Облаке - ледяное хранилище (платный трафик, хранение дешевле)

### БД - postgresql

новый инстанс или новая бд в существующем?

### Основные тачки - Москва
- [ruvds](https://ruvds.com/ru-rub)
  - почему? надежность?
- дешевле firstvds (если там будет хранилище)

### Тачки вне РФ - Финляндия - [pq.hosting](http://pq.hosting/)
- может пригодится для парсинга rutracker
- можно использовать поднятый там прокси сервер

### Процесс добавления контента
- скачивание торрента
- преобразование файлов
- загрузка в хранилище

если процесс преобразования файлов через 
ffmpeg будет упираться в цпу 
(а скорее всего так и будет), то 
при старте процесса запускать отдельную тачку 
на Яндексе с кучей цпу, 
выполнять на ней расчёт, выгружать и выключать.

Тоже самое касается процесса загрузки торрентов. 
Если параллельно будет грузится много торрентов, то 
это нужен большой диск, но только на время процесса загрузки.
И не будет ли торрент упираться в цпу?


## Заметки
- сжатие видео ffmpeg https://unix.stackexchange.com/questions/28803/how-can-i-reduce-a-videos-size-with-ffmpeg
