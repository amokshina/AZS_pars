# AZS_parcer
Проект предназначен для сбора информации об АЗС и отзывах о них. 
____
## Содержание
- [Описание](#описание)
- [Установка](#установка)
- [Использование](#использование)
  - [Примеры](#примеры)
  - [TOML файл](#toml-файл)
- [Основные функции](#основные-функции)
- [Структура БД](#структура-бд)

____
## Описание
Этот проект включает два основных парсера, конфигурационный файл и два класса, которые используются для записи полученных данных в БД:

1. **AZSinfo_parcer** - Парсер для получения информации об АЗС.
2. **AZSreviews_parcer** - Парсер для получения отзывов о АЗС.
3. **config.toml** - Конфигурационный файл, для поддержания актуальности API, изменения области поиска и явного указания файла, содержащего objectId для поиска отзывов
4. **AZS_info_loader** - Класс для записи основной информации в БД.
5. **AZS_loader** - Класс для записи отзывов в БД.
____
## Установка
1. Склонируйте репозиторий:
 ```
 git clone https://github.com/amokshina/AZS_pars.git
```
2. Установите необходимые зависимости:
____
## Использование

### Примеры
Запуск файла парсера общей информации:

```python
parser = AZSinfo_parcer('config.toml') #  формируем новую базу objectId      
out_file_name = parser.generate_outfile_name()
AZSinfo_parcer.load_to_file(parser, out_file_name)
```

```python
parser = AZSinfo_parcer('config.toml', ids_path='objectIds_2024-07-09_17-16.json')   # с имеющейся базой objectId       
out_file_name = parser.generate_outfile_name()
AZSinfo_parcer.load_to_file(parser, out_file_name)
```

Запуск файла парсера отзывов:

```python
parser = AZSreviews_parcer('config.toml') # база objectId берется из config.toml                        
a = parser.generate_outfile_name()
AZSreviews_parcer.load_to_file(parser, a)
```
```python
parser = AZSreviews_parcer('config.toml', ids_path='objectIds_2024-07-09_17-16.json') # берем желаемую базу objectId                      
a = parser.generate_outfile_name()
AZSreviews_parcer.load_to_file(parser, a)
```

Запуск файла записи в БД общей информации:
```python
filename_i = "info_2024-07-16_10-13-34.csv"
coord_file = 'coordinates.json'
azs_info_loader = AZS_info_loader(filename_i, coord_file)
asyncio.run(azs_info_loader.run_insert(parralel_task=10))
```

Запуск файла записи в БД отзывов:
```python
filename_r = "reviews_2024-07-16_17-27-42.csv"
azs_loader = AZS_loader(filename_r)
asyncio.run(azs_loader.run_insert(parralel_task=10))
```
### TOML файл
____
#### Структура TOML файла
**url_info** - действующая api, содержащая общую информацию об АЗС(objectId, coordinates, rating и т.д.)

**url_reviews** - действующая api, хранящая отзывы (максимум 1000) для текущего objectId

**coordinates** - координаты, необходимые для работы pars_azs. обозначают область поиска. 

**headers** - список user-agent, для предотвращения ошибки 500

**in_file** - входной файл для парсера отзывов, в нем хранится json следующего виде:
{objectId: reviewsCount}
имя файла автоматически обновляется при завершении парсинга АЗС
____
#### Обновление TOML файла
Для обновления данных в TOML файле можно использовать следующий код:
```python
import toml

config = toml.load('config.toml')
config['filename']['in_file'] = 'new_filename.json'

with open('config.toml', 'w', encoding='utf-8') as f:
    toml.dump(config, f)
```
## Основные функции
____
## pars_azs_new_api.py
Конструктор парсера, принимающий на вход путь до config файла, путь до файла результата (необязательный параметр, по умолчанию результат в той же директории, что и исполняемая программа), путь до файла со всеми objectId (необязательный параметр, указывается в случае, если хотим дополнить уже существующий файл, по умолчанию создается новый) 

```python
 def __init__(self, config_path, outfile_path = '', ids_path = None):
```

Функция, генерирующая список всех возможных координат в пределах значений, указанных в config файле.
```python
 def generate_coordinates(self):
```


Функция, генерирующая текущий url. Принимает на вход координаты и номер страницы.
```python
 def generate_url(self, min_lon, max_lat, max_lon, min_lat, page_num):
```


Генерация имени выходного файла по текущей дате и времени. 

Формат: info_%Y-%m-%d_%H-%M-%S.csv
```python
  def generate_outfile_name(self):
```


Основная функция, принимающая на вход объект парсера и имя выходного файла. Загружает в выходной файл основную информацию по АЗС в формате:
objectId (int), info(json) 
```python
  def load_to_file(parcer, out_file): 
```


## pars_reviews_new_api.py
Конструктор парсера, принимающий на вход путь до config файла, путь до файла со всеми objectId (необязательный параметр, по умолчанию берется тот, который указан в config файле), путь до файла результата (необязательный параметр, по умолчанию результат в той же директории, что и исполняемая программа) 
```python
 def __init__(self, config_path, ids_path = None, outfile_path = ''): 
```


Функция, генерирующая текущий url. Принимает на вход objectId и номер страницы. Также имеет необязательный параметр limit, отвечающий за максимальное количество результатов, показываемых на 1 странице.
```python
 def generate_url(self, objectId, page_num, limit=10):
```


Генерация имени выходного файла по текущей дате и времени. 

Формат: reviews_%Y-%m-%d_%H-%M-%S.csv
```python
  def generate_outfile_name(self):
```


Основная функция, принимающая на вход объект парсера и имя выходного файла. Загружает в выходной файл основную информацию по АЗС в формате:
objectId(int),reviews,categoryAspectsStats,pager
```python
  def load_to_file(parcer, out_file):
```


## async_info_to_db.py
Конструктор, принимающий на вход путь до csv файла с общей информацией, путь до json файла с информацией об уже известных координатах.
```python
 def __init__(self, info_file = '', coordinates_file=''): 
```


Функция, обращающаяся к ресурсу OpenStreetMap, для получения корректной информации о месте расположения АЗС.
```python
def get_place_by_coordinates(self, coordinates, retries=20, delay=2): 
```

Функция генерирующая асинхронный итератор. Принимает на вход два параметра: индекс начала и индекс конца. Берет строки из файла, лежащие в заданном промежутке.
```python
async def read_file_lines(self, start, end) -> AsyncIterator[str]:
```


Функция, используемая для обработки 1 конкретной строки. В ней происходит преобразование к модели Pydantic и вызывается функция записи в БД.
```python
async def process_line(self, line, data_to_insert_s_AZS_address, data_to_insert_AZS_info):
```


Функция, используемая для записи значений в БД. Внутри вызывается функция copy_records_to_table.
```python
async def load_data(self, schema, table, values):
```


Функция, используемая для создания списка задач. ``` lst_task.append(asyncio.create_task(self._process_lines(start, end))) ```
Обращается к функции обработки строки ``` process_line(line, data_to_insert_s_AZS_address, data_to_insert_AZS_info)``` .
```python
async def _process_lines(self, start, end):
```



Основная фунция. В ней создается пул соединений, создается список задач, а затем происходит вызов SQL функции, выполняющей вставку данных из буферных таблиц в основные. 
```python
async def run_insert(self, parralel_task: int = 10): 
```

## async_reviews_to_db.py
Конструктор, принимающий на вход путь до csv файла с отзывами.
```python
def __init__(self, rev_file = ''): 
```


Функция генерирующая асинхронный итератор. Принимает на вход два параметра: индекс начала и индекс конца. Берет строки из файла, лежащие в заданном промежутке.
```python
async def read_file_lines(self, start, end) -> AsyncIterator[str]:
```


Функция, используемая для обработки 1 конкретной строки. В ней происходит преобразование к модели Pydantic и вызывается функция записи в БД.
```python
async def process_line(self, line, data_to_insert_AZS_review, data_to_insert_s_AZS_rev_users, data_to_insert_AZS_rev_categ):
```


Функция, используемая для записи значений в БД. Внутри вызывается функция copy_records_to_table.
```python
async def load_data(self, schema, table, values):
```


Функция, используемая для создания списка задач. ``` lst_task.append(asyncio.create_task(self._process_lines(start, end))) ```
Обращается к функции обработки строки```  process_line(line, data_to_insert_AZS_review, data_to_insert_s_AZS_rev_users, data_to_insert_AZS_rev_categ)``` .
```python
async def _process_lines(self, start, end):
```


Основная фунция. В ней создается пул соединений, создается список задач, а затем происходит вызов SQL функции, выполняющей вставку данных из буферных таблиц в основные. 
```python
async def run_insert(self, parralel_task: int = 10): 
```

____
## Структура БД
![изображение](https://github.com/user-attachments/assets/12d1199b-fb2b-4778-802f-dcdcb9d62e3d)

