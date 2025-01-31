## **Как устроена архитектура Airflow?**

Apache Airflow имеет **распределенную архитектуру**, которая включает несколько ключевых компонентов.  
Основные компоненты взаимодействуют друг с другом для выполнения, планирования и мониторинга задач.

### **Ключевые компоненты Airflow**:
* **Scheduler** – отвечает за планирование задач и добавление их в очередь.
* **Executor** – управляет выполнением задач (может быть локальным, Celery, Kubernetes и т. д.).
* **Worker** – исполняет задачи (при использовании Celery или Kubernetes).
* **Metadata Database** – хранит информацию о DAGs, задачах, статусах и зависимостях.
* **Web UI** – интерфейс для управления DAGs и их мониторинга.
* **DAGs (Directed Acyclic Graphs)** – файлы Python, описывающие последовательность выполнения задач.
* **TaskInstance** – конкретный запуск таска в DAG.
* **XCom** – механизм передачи данных между задачами.
* **Connections** – настройки подключения к базам данных, API, облачным сервисам.

### **Как работает Airflow?**
1. **Scheduler проверяет DAGs** и добавляет задачи в очередь.
2. **Executor получает задачи** из очереди и передает их Worker'ам.
3. **Worker выполняет задачи** и обновляет статус в базе данных.
4. **Web UI отображает статус DAGs** и их выполнение.

Архитектура Airflow позволяет **масштабировать систему** и обрабатывать большие объемы задач.

--------
## **Что такое Scheduler, Executor, Worker, DAG, Task, XCom, Connections?**

### **Scheduler (Планировщик)**
* Отвечает за **планирование задач**, добавление их в очередь и управление зависимостями.
* Работает **постоянно** и проверяет DAGs каждые **1-5 секунд**.

### **Executor (Исполнитель)**
* Отвечает за **запуск задач**, передавая их Worker'ам.
* Может быть **LocalExecutor, CeleryExecutor, KubernetesExecutor**.

### **Worker (Исполнитель задач)**
* Выполняет задачи DAG'ов (если используется Celery/Kubernetes).
* Может быть **отдельным сервером или контейнером**.

### **DAG (Directed Acyclic Graph)**
* Python-файл, который описывает **зависимости и порядок выполнения задач**.

### **Task**
* Отдельная операция в DAG (например, SQL-запрос, HTTP-запрос, обработка данных).

### **XCom (Cross-communication)**
* Механизм **обмена данными между задачами**.
* Позволяет передавать **переменные и результаты выполнения**.

### **Connections**
* Хранят **подключения к базам данных, API, облачным сервисам**.
* Используются в **Hooks и Operators**.

Эти компоненты вместе обеспечивают **гибкость и масштабируемость Apache Airflow**.

---------
## **Как работает Scheduler и как часто он запускает задачи?**

### **Как работает Scheduler?**
1. **Сканирует DAGs в папке dags/** (по умолчанию каждые 5 секунд).
2. **Определяет, какие задачи готовы к запуску** (учитывая зависимости).
3. **Добавляет задачи в очередь** на выполнение.
4. **Обновляет статусы задач** в базе данных.

### **Как часто Scheduler запускает задачи?**
* По умолчанию **каждые 1-5 секунд**.
* Можно изменить интервал с помощью `min_file_process_interval` в `airflow.cfg`.
* DAGs запускаются **по расписанию (cron, timedelta)** или **вручную**.

### **Как улучшить производительность Scheduler?**
* Уменьшить `min_file_process_interval` (по умолчанию 30 секунд).
* Использовать **CeleryExecutor** или **KubernetesExecutor**.
* Разделять DAGs по разным Worker'ам.

Scheduler – **критически важный компонент Airflow**, который управляет выполнением задач.


------
## **Что такое TaskInstance и как он управляется?**

### **TaskInstance (Экземпляр задачи)**
* Это **конкретный запуск задачи в DAG**.
* Содержит **уникальный идентификатор** (dag_id, task_id, execution_date).

### **Основные состояния TaskInstance**
| **Статус**    | **Описание** |
|--------------|------------|
| `queued` | Ожидает выполнения |
| `running` | Выполняется |
| `success` | Выполнен успешно |
| `failed` | Ошибка выполнения |
| `up_for_retry` | Повторный запуск после ошибки |

### **Как управлять TaskInstance?**
* **Перезапуск задачи вручную** (через UI или CLI).
* **Использование ретраев** (`retries=3`, `retry_delay=timedelta(minutes=5)`).
* **Изменение зависимостей** (Trigger Rules).

TaskInstance – это **основная единица выполнения в Airflow**, которая управляет статусами задач.

----------
## **Какие есть методы хранения метаданных в Airflow?**

### **База данных метаданных**
Airflow использует **реляционную базу данных** для хранения:
* DAGs и их состояния.
* Информации о TaskInstance.
* XCom (межзадачные данные).
* Connections (подключения к сервисам).

### **Какие базы данных поддерживаются?**
* **SQLite** (только для локального тестирования).
* **PostgreSQL** (рекомендуется для продакшена).
* **MySQL** (используется, но менее стабилен, чем PostgreSQL).

### **Как настроить базу данных в Airflow?**
* Указываем `sql_alchemy_conn` в `airflow.cfg`:

> sql_alchemy_conn = postgresql+psycopg2://user:password@host:5432/airflow

-------

## **Как работает backfilling (запуск DAG задним числом)?**

### **Что такое backfilling?**
Backfilling – это **запуск DAG задним числом** (например, если DAG изменился, и нужно пересчитать старые данные).

### **Как запустить backfill?**
Используем команду:

> airflow dags backfill -s 2023-01-01 -e 2023-01-10 my_dag

------------
## **Что такое DAG в Airflow?**

**DAG (Directed Acyclic Graph)** — это **набор задач (Tasks), объединенных зависимостями** и выполняемых по заданному расписанию.  
DAG определяет **порядок выполнения задач** и гарантирует, что **не будет циклов** (Acyclic – «ацикличный»).  

### **Основные характеристики DAG:**
* **Содержит задачи (Tasks)** – каждая задача выполняет одну операцию.
* **Имеет зависимости** – порядок выполнения определяет их.
* **Запускается по расписанию** – через `schedule_interval` (cron или timedelta).
* **Не должен содержать циклов** – нельзя создать бесконечный цикл задач.

### **Пример DAG**
```python
from airflow import DAG
from airflow.operators.dummy import DummyOperator
from datetime import datetime

dag = DAG(
    'my_first_dag',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily'
)

task_1 = DummyOperator(task_id='start', dag=dag)
task_2 = DummyOperator(task_id='end', dag=dag)

task_1 >> task_2  # Определяем зависимость
```

DAG – это основа Airflow, которая определяет какие задачи и в каком порядке выполняются.

-----

## **Как создать простой DAG?**

### **Шаги создания DAG:**
1. **Импортируем модули** (`airflow`, `operators`).
2. **Создаем объект DAG** (с `start_date` и `schedule_interval`).
3. **Определяем задачи (Tasks)**.
4. **Настраиваем зависимости между задачами**.

### **Простой пример DAG**
```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'airflow',
    'retries': 3,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'simple_dag',
    default_args=default_args,
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily'
)

task_1 = BashOperator(
    task_id='print_hello',
    bash_command='echo "Hello, Airflow!"',
    dag=dag
)

task_1
```

--------

## **Зависимости, правила триггеров, ретраи и таймауты**

### **Как настроить зависимости между задачами (Task Dependencies)?**

В Airflow задачи могут выполняться **последовательно** или **параллельно**.  
Зависимости между задачами **задаются через операторы `>>` и `<<`**.

### **Пример зависимостей**
```python
task_1 >> task_2  # task_2 запустится после task_1
task_3 << task_2  # task_3 запустится после task_2
```

```python
task_1 >> [task_2, task_3]  # task_2 и task_3 выполняются параллельно
task_2 >> task_4
task_3 >> task_4  # task_4 запустится только после task_2 и task_3

```

-----
## **Как работают Trigger Rules?**

**Trigger Rules** определяют, **когда можно запускать задачу**, учитывая статусы предыдущих задач.

### **Основные Trigger Rules**
| **Trigger Rule** | **Описание** |
|-----------------|-------------|
| `all_success` (по умолчанию) | Запускается, если все предыдущие таски **успешны** |
| `all_failed` | Запускается, если **все предыдущие задачи упали** |
| `all_done` | Запускается, если **все предыдущие таски завершены** (успех или ошибка) |
| `one_failed` | Запускается, если **хотя бы одна задача упала** |
| `one_success` | Запускается, если **хотя бы одна задача успешна** |
| `none_failed` | Запускается, если **нет ни одной ошибки** |

### **Пример Trigger Rules**
```python
task_1 = DummyOperator(task_id='task_1', dag=dag)
task_2 = DummyOperator(task_id='task_2', dag=dag, trigger_rule='one_failed')

task_1 >> task_2  # task_2 выполнится, если task_1 упадет
```

-------------

# **Task и Operator в Airflow**

## **Что такое Task в Airflow?**

**Task (Задача)** — это **единица работы в Airflow**, которая выполняется в рамках DAG.  
Каждый Task выполняет **конкретную операцию**: запуск скрипта, SQL-запроса, API-вызова и т. д.

### **Ключевые характеристики Task**
* **Запускается в рамках DAG**.
* **Может иметь зависимости** от других Tasks.
* **Имеет параметры (retries, timeout, execution_date и т. д.)**.
* **Хранит состояние** (queued, running, success, failed и др.).

### **Пример Task в Airflow**
```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime

dag = DAG(
    'example_dag',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily'
)

task = BashOperator(
    task_id='print_hello',
    bash_command='echo "Hello, Airflow!"',
    dag=dag
)
```
----------

## **Что такое Operator в Airflow?**

**Operator** — это **шаблон (класс) для создания Tasks**.  
Airflow предоставляет **различные операторы** для выполнения команд, SQL-запросов, API-запросов и других действий.

### **Виды операторов**
| **Тип оператора** | **Описание** |
|-------------------|-------------|
| **BashOperator** | Выполняет Bash-команды |
| **PythonOperator** | Запускает Python-функции |
| **DummyOperator** | Пустая задача (для логики DAG) |
| **BranchPythonOperator** | Позволяет ветвить выполнение DAG |
| **PostgresOperator** | Запускает SQL-запрос в PostgreSQL |
| **MySqlOperator** | Запускает SQL-запрос в MySQL |
| **HttpOperator** | Делает HTTP-запрос |
| **S3Operator** | Загружает/скачивает файлы из S3 |
| **EmailOperator** | Отправляет email-уведомления |
| **TriggerDagRunOperator** | Запускает другой DAG |

### **Пример использования PythonOperator**
```python
from airflow.operators.python import PythonOperator

def print_message():
    print("Hello from PythonOperator!")

task = PythonOperator(
    task_id='print_message',
    python_callable=print_message,
    dag=dag
)
```

-------

## **Как передавать данные между Tasks (XCom)?**

**XCom (Cross-communication)** — это механизм **обмена данными между задачами**.  
XCom позволяет **сохранять и извлекать** данные во время выполнения DAG.

### **Как отправить данные в XCom (push)?**
```python
def push_data(**kwargs):
    kwargs['ti'].xcom_push(key='message', value='Hello, XCom!')

push_task = PythonOperator(
    task_id='push_task',
    python_callable=push_data,
    provide_context=True,
    dag=dag
)
```

Как получить данные из XCom:
```python
def pull_data(**kwargs):
    message = kwargs['ti'].xcom_pull(task_ids='push_task', key='message')
    print(f"Received: {message}")

pull_task = PythonOperator(
    task_id='pull_task',
    python_callable=pull_data,
    provide_context=True,
    dag=dag
)

push_task >> pull_task
```

---------
## **Как передавать данные между DAGs (TriggerDagRunOperator)?**

Иногда нужно **запускать один DAG из другого DAG**.  
Для этого используется **TriggerDagRunOperator**.

### **Пример запуска другого DAG**
```python
from airflow.operators.trigger_dagrun import TriggerDagRunOperator

trigger_task = TriggerDagRunOperator(
    task_id='trigger_another_dag',
    trigger_dag_id='target_dag',  # Название DAG, который нужно запустить
    dag=dag
)
```

----
## **Что такое Airflow Connections?**

**Connections (Подключения)** в Airflow – это механизм **хранения и управления учетными данными** для подключения к внешним системам (базы данных, облачные сервисы, API).  
Airflow использует Connections для аутентификации в **PostgreSQL, MySQL, AWS, GCP, Snowflake, API и других сервисах**.

### **Где хранятся Connections?**
1. **В базе данных Airflow** (по умолчанию).
2. **Через переменные окружения** (`AIRFLOW_CONN_<имя_коннекта>`).
3. **Секретные хранилища** (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager).

### **Как создать Connection через Web UI?**
1. Перейти в **Admin → Connections**.
2. Нажать **"Create"**.
3. Указать **Conn ID**, **Conn Type**, **Host**, **Login**, **Password**.
4. Сохранить.

### **Пример добавления Connection через Python**
```python
from airflow.models import Connection
from airflow.settings import Session

conn = Connection(
    conn_id="my_postgres",
    conn_type="postgres",
    host="mydb.example.com",
    login="user",
    password="password",
    schema="mydatabase",
    port=5432
)

session = Session()
session.add(conn)
session.commit()
session.close()
```

-----
## **Какие есть встроенные Hooks в Airflow?**

**Hooks** – это API-интерфейсы для взаимодействия с внешними сервисами через Connections.  
Hooks используются **внутри Operators** для работы с базами данных, облачными сервисами, API.

### **Основные Hooks в Airflow**
| **Hook** | **Описание** |
|---------|-------------|
| `PostgresHook` | Подключение к PostgreSQL |
| `MySqlHook` | Подключение к MySQL |
| `S3Hook` | Работа с AWS S3 |
| `GoogleCloudStorageHook` | Работа с GCS |
| `SnowflakeHook` | Подключение к Snowflake |
| `HttpHook` | Запросы к REST API |
| `SlackWebhookHook` | Отправка сообщений в Slack |
| `RedisHook` | Подключение к Redis |
| `MongoHook` | Работа с MongoDB |

Hooks **абстрагируют логику подключения** и позволяют взаимодействовать с сервисами через Python-код.

---
## **Как использовать Hooks в DAG?**

### **Использование PostgresHook**
```python
from airflow.providers.postgres.hooks.postgres import PostgresHook

def fetch_data():
    hook = PostgresHook(postgres_conn_id="my_postgres")
    conn = hook.get_conn()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users;")
    return cursor.fetchall()
```

----------
## **Как создать кастомный Hook в Airflow?**

Если встроенного Hook нет, можно создать **кастомный Hook**.

### **Пример кастомного Hook для API**
```python
from airflow.hooks.base import BaseHook
import requests

class MyCustomHook(BaseHook):
    def __init__(self, conn_id):
        super().__init__()
        self.conn = self.get_connection(conn_id)

    def get_data(self, endpoint):
        url = f"{self.conn.host}/{endpoint}"
        response = requests.get(url, auth=(self.conn.login, self.conn.password))
        return response.json()
```

----

## **Какие есть типы Executors в Airflow?**

**Executor** – это компонент Airflow, который управляет выполнением задач (Tasks).  
Выбор Executor'а влияет на **производительность и масштабируемость** Airflow.

### **Основные типы Executors**
| **Executor**        | **Описание** | **Когда использовать?** |
|---------------------|-------------|------------------------|
| `SequentialExecutor` | Запускает задачи **по одной** | Локальная разработка |
| `LocalExecutor` | Запускает **несколько задач параллельно** | Маленькие нагрузки |
| `CeleryExecutor` | Распределяет задачи на **несколько Worker'ов** | Продакшен с высокой нагрузкой |
| `KubernetesExecutor` | Запускает задачи **в отдельных контейнерах** | Облачные среды и Kubernetes |
| `DaskExecutor` | Использует **кластер Dask** | Аналитика и машинное обучение |

Выбор Executor'а зависит от **нагрузки** и **инфраструктуры**.

### **Как работает SequentialExecutor?**
* Запускает **только одну задачу одновременно**.
* Не поддерживает **параллельное выполнение задач**.
* Используется **только с SQLite** (не поддерживает PostgreSQL/MySQL).

Когда использовать SequentialExecutor?

✅ Для локальной отладки DAGs.

✅ Если параллельное выполнение не нужно.

❌ Не подходит для продакшена – слишком медленный.

---
## **LocalExecutor: Как работает и когда использовать?**

`LocalExecutor` – это **улучшенная версия SequentialExecutor**, поддерживающая **многопоточное выполнение**.

### **Как работает LocalExecutor?**
* Запускает **несколько задач одновременно**.
* Работает **только на одной машине** (без распределенного выполнения).
* Использует **многопоточность и multiprocessing**.

### **Настройка LocalExecutor**
Файл `airflow.cfg`:
```ini
executor = LocalExecutor
parallelism = 4  # Максимальное количество параллельных задач
```
Когда использовать LocalExecutor?

✅ Для небольших проектов с умеренной нагрузкой.

✅ Если нет необходимости в распределенных Worker'ах.

❌ Не масштабируется на несколько серверов.


---
## **CeleryExecutor: Как работает и когда использовать?**

`CeleryExecutor` – это **распределенный Executor**, который позволяет запускать задачи **на нескольких Worker'ах**.

### **Как работает CeleryExecutor?**
* **Scheduler отправляет задачи** в очередь (Redis, RabbitMQ).
* **Worker'ы берут задачи** из очереди и выполняют их.
* **Результаты выполнения хранятся** в базе данных Airflow.


**Когда использовать CeleryExecutor?**

✅ Если нужно распределенное выполнение DAGs.

✅ Если нагрузка высокая и требует масштабирования.

❌ Требует настройки очередей сообщений (Redis, RabbitMQ).

❌ Может быть сложнее в отладке из-за распределенной архитектуры.

---
## **KubernetesExecutor: Как работает и когда использовать?**

`KubernetesExecutor` – это **контейнеризированный Executor**, который создает **отдельный Pod для каждой задачи**.

### **Как работает KubernetesExecutor?**
* **Scheduler отправляет задачу** в Kubernetes API.
* **Kubernetes создает Pod**, который выполняет задачу.
* **После выполнения Pod удаляется**.

Когда использовать KubernetesExecutor?

✅ Если Airflow работает в Kubernetes.

✅ Если задачи требуют разных ресурсов (CPU/GPU).

✅ Если нужно авто-масштабирование Worker'ов.

❌ Требует настройки Kubernetes и усложняет деплой.

---
## **DaskExecutor: Как работает и когда использовать?**

`DaskExecutor` использует **кластер Dask** для параллельного выполнения задач.

### **Как работает DaskExecutor?**
* **Airflow отправляет задачи** в кластер Dask.
* **Dask распределяет выполнение** между Worker'ами.
* **Подходит для анализа данных** и машинного обучения.

Когда использовать DaskExecutor?

✅ Если нужно распределенное выполнение для ML и анализа данных.

✅ Если уже используется кластер Dask.

❌ Не самый универсальный Executor – больше подходит для аналитики.

--------
## **Как выбрать Executor для Airflow?**

### **Выбор Executor'а зависит от нагрузки и инфраструктуры**
| **Executor**         | **Когда использовать?** |
|----------------------|------------------------|
| **SequentialExecutor** | Для **локальной разработки**, тестирования DAGs |
| **LocalExecutor** | Если **не нужно распределенное выполнение** |
| **CeleryExecutor** | Для **распределенной обработки задач** |
| **KubernetesExecutor** | Если **нужна гибкость и масштабируемость в облаке** |
| **DaskExecutor** | Для **анализа данных и машинного обучения** |

### **Общий совет**
* Для **локальной разработки** – `LocalExecutor` или `SequentialExecutor`.
* Для **нагруженного продакшена** – `CeleryExecutor` или `KubernetesExecutor`.
* Для **облачных решений** – `KubernetesExecutor` (если Airflow работает в Kubernetes).

Выбор правильного Executor'а **определяет производительность и масштабируемость Airflow**.

---------
## **Как работает Scheduler в Airflow?**

**Scheduler (Планировщик)** — это компонент Airflow, который отвечает за **управление расписанием задач** и их распределением на исполнение.

### **Как работает Scheduler?**
1. **Сканирует DAGs в директории `dags/`** (по умолчанию каждые 5 секунд).
2. **Определяет, какие задачи должны выполняться** согласно их `schedule_interval` и зависимостям.
3. **Добавляет задачи в очередь** (Task Queue).
4. **Передает задачи Executor'у**, который распределяет их между Worker'ами.
5. **Обновляет статус задач** в базе метаданных.

### **Основные параметры Scheduler**
| **Параметр** | **Описание** |
|-------------|-------------|
| `scheduler_interval` | Интервал обновления DAGs (по умолчанию 5 сек) |
| `min_file_process_interval` | Минимальный интервал обработки файлов DAG |
| `dag_dir_list_interval` | Интервал сканирования каталога DAGs |
| `max_active_runs_per_dag` | Ограничение на число одновременно выполняющихся DAGs |

### **Как запустить Scheduler?**
```bash
airflow scheduler
```

-----
## **Как часто Scheduler проверяет DAGs?**

Scheduler **проверяет DAGs в нескольких случаях**:
1. **При запуске Scheduler** (все DAGs загружаются в память).
2. **Через `dag_dir_list_interval`** (по умолчанию 5 секунд).
3. **При ручном запуске DAG через UI/API**.
4. **При изменении DAG-файла** (если включено автоматическое обнаружение).

### **Основные параметры, влияющие на частоту обновления DAGs**
| **Параметр** | **Описание** | **Значение по умолчанию** |
|-------------|-------------|------------------|
| `min_file_process_interval` | Минимальный интервал обработки файлов DAG | 30 сек |
| `dag_dir_list_interval` | Интервал сканирования каталога DAGs | 5 сек |
| `scheduler_heartbeat_sec` | Как часто Scheduler отправляет "пульс" | 5 сек |

### **Как изменить частоту проверки DAGs?**
Изменить параметр `dag_dir_list_interval` в `airflow.cfg`:
```ini
[scheduler]
dag_dir_list_interval = 10
```
**Чем чаще Scheduler проверяет DAGs, тем выше нагрузка на CPU. Нужно балансировать между скоростью обновления и производительностью.**

----
## **Как избежать нагрузки на Scheduler?**

Если Scheduler перегружен, DAGs могут **запускаться с задержками или вовсе не стартовать**.  
Чтобы этого избежать, нужно **оптимизировать конфигурацию и распределить нагрузку**.

### **Основные причины перегрузки Scheduler**
1. **Слишком частые проверки DAGs** (низкое значение `dag_dir_list_interval`).
2. **Слишком много одновременно выполняемых DAGs** (`max_active_runs` не ограничен).
3. **Нехватка ресурсов (CPU/RAM) для Scheduler**.
4. **Слишком сложные DAGs с динамическими тасками**.
5. **Проблемы с базой метаданных (медленные запросы к БД)**.

Для решения этих проблем можно использовать **оптимизацию конфигурации, ограничение DAGs и масштабирование системы**.

## **Ограничение частоты сканирования DAGs**

Scheduler **по умолчанию проверяет директорию `dags/` каждые 5 секунд**, что может перегружать CPU.  
Чтобы уменьшить нагрузку, можно **увеличить интервал сканирования**.

### **Как уменьшить частоту сканирования DAGs?**
Изменить параметр `dag_dir_list_interval` в `airflow.cfg`:
```ini
[scheduler]
dag_dir_list_interval = 30  # Проверять DAGs раз в 30 секунд
```
----
## **Ограничение количества активных DAG-запусков**

Если одновременно выполняется **слишком много DAGs**, это **перегружает Scheduler**.  
Решение — **ограничить число параллельных DAG-запусков**.

### **Как ограничить количество активных DAGs?**
В `airflow.cfg`:
```ini
[scheduler]
max_active_runs_per_dag = 2  # Не более 2 одновременных запусков DAG
```
---
## **Что такое Sensors (сенсоры) в Airflow?**

**Sensors (сенсоры)** – это специальные задачи в Airflow, которые **ожидают наступления события** перед тем, как продолжить выполнение DAG.  
Сенсоры проверяют наличие файлов, выполнение других задач, поступление данных в базу и другие внешние события.

### **Как работают сенсоры?**
* Сенсор выполняет **периодические проверки** (polling).
* Если **условие выполняется** – сенсор завершает работу, и DAG продолжается.
* Если **условие не выполнено** – сенсор ждет заданное время и повторяет проверку.

### **Пример FileSensor: ждет файл в S3**
```python
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

wait_for_file = S3KeySensor(
    task_id='wait_for_s3_file',
    bucket_name='my-bucket',
    bucket_key='path/to/file.csv',
    aws_conn_id='aws_default',
    poke_interval=60,  # Проверять каждые 60 секунд
    timeout=3600,  # Завершить, если файл не появился через 1 час
    dag=dag
)
```
----

## **Что такое переменные (Variables) в Airflow?**

Переменные (**Variables**) в Airflow – это глобальные настройки, которые можно использовать в DAGs.  
Они хранятся в **метаданных Airflow** и доступны в Python-коде и шаблонах Jinja.

### **Как создать переменную через UI?**
1. Перейти в **Admin → Variables**.
2. Нажать **"Create"**.
3. Ввести **ключ (Key) и значение (Value)**.
4. Сохранить.

### **Как создать переменную через CLI?**
```bash
airflow variables set my_key my_value
```

**Переменные позволяют управлять конфигурацией DAGs без изменения кода.**


## **Как передавать переменные в DAG и между тасками?**

### **1. Использование `Variable.get()` в тасках**
```python
from airflow.models import Variable

api_key = Variable.get("api_key")

task = BashOperator(
    task_id="use_variable",
    bash_command=f"echo {api_key}",
    dag=dag
)
```

----
## **Что такое отложенные операторы (Deferrable Operators) в Airflow?**

**Deferrable Operators (отложенные операторы)** – это операторы, которые **приостанавливают** выполнение таска до наступления события, **не занимая Worker'ов**.

### **Почему отложенные операторы важны?**
* **Обычные сенсоры** постоянно проверяют условия и занимают Worker'ов.
* **Отложенные операторы** используют **асинхронный Event Loop**, который ждет событие **без блокировки ресурсов**.

### **Как работает Deferrable Operator?**
1. **Запускается таск** с отложенным оператором.
2. **Airflow освобождает Worker** и ждет наступления события.
3. **Когда событие наступает**, таск продолжает выполнение.

Отложенные операторы **значительно улучшают производительность Airflow**, особенно при ожидании файлов, API-запросов и событий в облаке.

## **Как использовать отложенные операторы в Airflow?**

### **Использование Deferrable Sensor для ожидания файла в S3**
```python
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensorAsync

wait_for_file = S3KeySensorAsync(
    task_id='wait_for_s3_file',
    bucket_name='my-bucket',
    bucket_key='path/to/file.csv',
    aws_conn_id='aws_default',
    deferrable=True,  # Включаем отложенный режим
    dag=dag
)
```
