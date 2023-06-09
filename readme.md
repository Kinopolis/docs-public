<!-- TOC -->
  * [Описание](#описание)
    * [Версии](#версии)
    * [Стандартное содержимое кинотеатра](#стандартное-содержимое-кинотеатра)
    * [Дополнительные сущности](#дополнительные-сущности)
    * [rest](#rest)
  * [TODO](#todo)
  * [Потенциальный стэк](#потенциальный-стэк)
    * [`ffmpeg` - подготовка конечных файлов](#ffmpeg---подготовка-конечных-файлов)
    * [`video.js` - десктоп веб медиа плеер](#videojs---десктоп-веб-медиа-плеер)
    * [`transmission` - торрент клиент](#transmission---торрент-клиент)
  * [Инфраструктура](#инфраструктура)
    * [Хранилище медиа файлов для чтения на клиенте](#хранилище-медиа-файлов-для-чтения-на-клиенте)
    * [БД - postgresql](#бд---postgresql)
    * [Основные тачки - Москва](#основные-тачки---москва)
    * [Тачки вне РФ](#тачки-вне-рф)
    * [Процесс добавления контента](#процесс-добавления-контента)
  * [Заметки](#заметки)
<!-- TOC -->

## Описание
Онлайн кинотеатр, основной источник медиа - торрент. В админку встроен торрент клиент.

### Версии
- десктоп веб
- андроид приложение
- андроид ТВ приложение
- мобильная веб версия - хотя бы для админки

### Стандартное содержимое кинотеатра
- фильмы
- сериалы
- мультфильмы
- мультсериалы

### Дополнительные сущности
- вселенные (Марвел/Гарри Поттер)
- серии фильмов без названия (Сплит 2016 + 2 фильма 2000 и 2019)

### rest
- источник видео материалов - торрент
- монетизация - отсутствует. Ожидается приход идей.

## TODO
- узнать поддерживаемые в video.js кодеки аудио и видео
- освоить ffmpeg (перекодирование, нарезка, извлечение дорожек)
- дополнить доку по основному процессу (из тегеграма)
- дополнить доку по мониторингу (прометеус, спринг актуатор, [bucket4j](https://www.baeldung.com/spring-bucket4j))

## Потенциальный стэк
- `java 17`
- `vue 3`
- `minio`
- `docker compose`
- `nginx`
- `postgresql`
- `ffmpeg`
- `video.js` [videojs.com](https://videojs.com/) [github](https://github.com/videojs/video.js)
- `transmission`

### `ffmpeg` - подготовка конечных файлов
- преобразование медиа контейнеров
- перекодирование в целевые кодеки
- ограничение битрейта
- нарезка больших медиа файлов на маленькие

изначально ffmpeg консольный, есть [оболочка](https://github.com/bramp/ffmpeg-cli-wrapper) для java, но она пригодится только для слежения за системным процессом ffmpeg, основной функционал остается в виде консольных опций

еще есть какой-то [FFmpeg libav](https://habr.com/ru/company/edison/blog/495614/)

### `video.js` - десктоп веб медиа плеер
- умеет подгружать видео частями
  - даже если дать ссылку на один видео файл, его тоже будет грузить частями
    - возможно связано с [http-streaming](https://github.com/videojs/http-streaming)
- умеет переключать аудио и субтитры
- поддерживает кастомизацию
- использует базовые html элементы
- умеет в потоковое аудио???
### `transmission` - торрент клиент
- [консольная версия](https://cli-ck.io/transmission-cli-user-guide/)

## Инфраструктура
### Хранилище медиа файлов для чтения на клиенте
- возможно хватит производительности s3
- получение через апи 
  - с перенаправлением 301 на публичный s3 (непосредственно файлы качает сам клиент)
  - для регулирования расположения файлов (разные хосты/яндекс)

размещение:
- Minio + [hdd хостинг first vds](https://firstvds.ru/storage-vds) (32 ТБ трафика)
  - для каждой тачки есть ограничение цпу - если критично, то брать несколько тачек с объемом поменьше и на все раскатывать multi node Minio
- Object Storage в Яндекс Облаке - ледяное хранилище (платный трафик, хранение дешевле)
- даже с 1тб исходящего трафика яндекс облако выходит дороже (стоит учитывать, что входящий трафик в яндексе бесплатный, а на firstvds 32тб выделено под весь трафик)
- помимо minio также существуют
 - ceph
 - openstack swift
---
firstvds (указаны максимальные значения для одной системы)
- 2 ядра
- 4 гб озу
- 5 ТБ hdd
- 32 тб всего трафика
- **Итого 4199 руб**

yandex cloud s3
- 5 ТБ
- 1.5 тб всего трафика
- ледяное хранилище
- 200к get операций
- 10к post операций
- **Итого 4760 руб**

---

дополнительно:
- держать файлы сжатыми и производить декомпрессию на лету?
- через [gzip](https://www.baeldung.com/cs/zlib-vs-gzip-vs-zip#gzip)
- возможно ли разжимать файлы на клиенте???
### БД - postgresql

новый инстанс или новая бд в существующем?

### Основные тачки - Москва
- ruvds
  - один из цодов охраняется росгвардией (sic!)
  - но даже лендинг поломанный
  - возможно дудосят (из-за названия?)
  - дорогое дисковое пространство!
- firstvds 
  - дешевле
  - адекватнее категоризация тарифов
  - точно есть отдельный тариф под террабайтные hdd
  - в целом [безграничные мощности](https://firstvds.ru/products/vds_vps_cloud)
  - бекапы тачек (кроме террабайтных hdd)

### Тачки вне РФ
хостинги
- [pq.hosting](http://pq.hosting/)
- [ihor](https://ihor.ru)

WHY
- может пригодится для парсинга rutracker
- можно использовать как прокси сервер (HAProxy), а хостить в Москве

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

Выключаемые вм можно дешевле взять на https://clo.ru/. В 3 раза дешевле Яндекса

Тоже самое касается процесса загрузки торрентов. 
Если параллельно будет грузится много торрентов, то 
это нужен большой диск, но только на время процесса загрузки.
И не будет ли торрент упираться в цпу?


## Заметки
- сжатие видео ffmpeg https://unix.stackexchange.com/questions/28803/how-can-i-reduce-a-videos-size-with-ffmpeg
