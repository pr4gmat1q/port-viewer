# Port Viewer
Web interface for [swtoolz-core](https://github.com/xcme/swtoolz-core/) | [Статья в blogspot (RU)](http://xcme.blogspot.ru/2016/05/portviewer.html)

## Архитектура приложения
- Приложение построено на PHP-фрэймворке [CodeIgniter](https://codeigniter.com/)
- В качестве СУБД используется MariaDB/MySQL
- Для визуального оформления используется css-фрэймворк Bootstrap 3 

## Основные возможности
### Визуализация данных

На данный момент приложение умеет отображать:
- Имя интерфейса (например 3/1 или gi0/1/2)
- Description интерфейса
- Тип среды передачи данных (выделяется цветом, а так же есть информация в title)
- Текущий и административный статусы интерфейса
- Текущую и административную скорость интерфейса (отображается в title)
- Наличие включенного административно flow-control на интерфейсе
- Описание модулей в шасси или в стеке
- В отдельный блок вынесена общая информация об устройстве, такая как: системное имя, uptime, модель устройства и т.д.

### Получение списка VLAN
- Можно вывести виланы в title
- Можно скачать виланы  устройства файлом в формате txt или scv (если включен api_debug, то в скачиваемый файл автоматически добавится вывод дебага - особенность архитектуры приложения)
- Для Foundry поддерживается получение l3-интерфейсов, которые маршрутизируют данные виланы. Логика построена на *vendor specific*, методом проб и ошибок выведен алгоритм. Возможно скоро появится поддержка Brocade SX.  

### Сохранение и просмотр состояний опрошенных устройств
Любое уникальное состояние опрошенного устройства сохраняется в базе.
Нажав на соответствующую кнопку можно посмотреть сохраненные состояния данного устройства в конкретный момент времени.

## Установка
### СУБД
Создаем в СУБД юзера и БД, заливаем в БД дамп **pv.sql**, после чего в таблице *config* настраиваем параметры взаимодействия с swtoolz-core. Если флаг *api_debug* имеет значение **1**, то в html код страницы будет выводится вся последовательность взаимодействия с swtoolz-core. 

Далее переходим к заполнению таблицы *com_indexes* в нее заносим все доступные индексы. Поле *str* заполняем для удобства восприятия, и для ориентирования в админке, которая находится в разработке. В конце выбираем тот индекс, с которым устройство будет опрашиваться по умолчанию, устанавливаем для этого индекса поле *default* в значение *1*

### Настройка WEB-сервера
Для работы приложения в PHP необходимо включить библиотеку **Curl**.
В качестве root-директории указываем папку **www**, это позволит повысить безопасность приложения.
Что бы исключить из всех ссылок index.php нужно соответствующим образом настроить параметры хоста.
#### Apache
Для работы должен быть включен mod_rewrite. Ниже приведенный код достаточно будет скопировать в файл **.htaccess**, и положить этот файла в папку **www**
```
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /  
    RewriteCond %{REQUEST_FILENAME} !-f   
    RewriteCond %{REQUEST_FILENAME} !-d  
    RewriteRule ^(.*)$ index.php/$1 [L]
</IfModule>
```
#### Nginx
Для базовой работы приложения в конфиге хоста будет достаточно изменить контекст *location* таким образом:
```
location / {
  try_files $uri $uri/ /index.php;
}
```
## Настройка приложения
Все файлы приложения находятся в папке **ci**. По умолчанию она должна находиться рядом с папкой **www**, но при желании можно изменить пути на нужные в файле **www/index.php** изменив соответствующим образом переменные **$system_path** и **$application_folder**, указывать желательно полный путь к папкам назначения. Почитать про эти настройки можно тут: **[EN](http://www.codeigniter.com/user_guide/installation/index.html)/[RU](http://codeigniter3.info/guide/installation/install)**.

Определившись с расположением файлов приступаем к настройке приложения, далее все действия будут происходить в папке **ci**.


1. Настраиваем подключение к СУБД в файле *application/config/database.php*
2. В файле *application/config/config.php* настраиваем параметр **base_url**
3. Приступаем к заполнению таблиц *node* и *device* в СУБД
