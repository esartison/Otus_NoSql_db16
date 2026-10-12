
# Домашнее задание Сартисона Евгения N16 #

Qdrant и векторные базы данных

Цель:
настроить векторный поиск в Qdrant для извлечения семантически связанных обращений банковской техподдержки по смыслу запроса.


Описание/Пошаговая инструкция выполнения домашнего задания:
В крупном банке работает внутренняя служба технической поддержки сотрудников.


Каждый день сотрудники банка создают обращения:

• проблемы с доступом к корпоративным системам;

• ошибки при работе с CRM;

• сбои электронной почты;

• проблемы с электронным
документооборотом;

• ошибки печати документов;

• проблемы с бухгалтерскими приложениями;

• ошибки авторизации;

• зависания рабочих приложений.


За несколько лет в системе накопились тысячи обращений.


Сейчас поиск работает только по ключевым словам.


Например:Сотрудник вводит запрос:«не открывается клиент для работы с договорами».

Но нужное обращение содержит текст: «после обновления система оформления кредитных заявок завершается с ошибкой»
Из-за различия формулировок обычный поиск часто не находит нужные записи.


Задача:
Разработать прототип системы семантического поиска на основе Qdrant.
Система должна находить похожие обращения не по совпадению слов, а по смыслу текста.


Исходные данные
Необходимо создать коллекцию обращений сотрудников банка.


Каждое обращение содержит:
Поле Описание
id идентификатор
title краткое название
text описание проблемы
department подразделение
category тип проблемы


Тестовые данные

Необходимо загрузить в систему следующие обращения.


Обращение 1

{
"id": 1,
"title": "Ошибка CRM",
"text": "Система оформления кредитных
заявок закрывается после открытия
карточки клиента",
"department": "credit",
"category": "crm"
}


Обращение 2
{
"id": 2,
"title": "Не печатается документ",
"text": "После отправки договора на печать
устройство не отвечает",
"department": "operations",
"category": "printing"
}


Обращение 3
{
"id": 3,
"title": "Ошибка входа",
"text": "Сотрудник не может войти в
систему обработки платежей",
"department": "finance",
"category": "authentication"
}


Обращение 4
{
"id": 4,
"title": "Проблема Outlook",
"text": "Письма остаются в исходящих и не
отправляются получателям",
"department": "office",
"category": "mail"
}


Обращение 5
{
"id": 5,
"title": "Сбой документооборота",
"text": "Система электронного
согласования зависает при открытии
вложений",
"department": "legal",
"category": "edms"
}


Обращение 6
{
"id": 6,
"title": "Ошибка клиентского приложения",
"text": "Приложение для оформления
банковских продуктов аварийно
завершается после авторизации",
"department": "credit",
"category": "crm"
}


Задание

Необходимо:

(1) Установить Qdrant.

(2) Создать коллекцию bank_support.

(3) Настроить Python-клиент.

(4) Выбрать embedding-модель.

(5) Построить embeddings для всех обращений.

(6) Загрузить данные в Qdrant.

Необходимо выполнить поиск для следующих запросов:

Запрос пользователя Ожидаемый результат

«закрывается программа кредитов» обращения CRM

«не отправляется электронная почта» проблемы Outlook

«невозможно войти в платежную систему» authentication

«зависает система согласования документов» EDMS

Для каждого запроса необходимо:

Выписать найденные документы.

Объяснить:

почему семантический поиск нашел нужный документ?

какие слова отсутствовали в исходном тексте?

почему поиск по ключевым словам отработает хуже?








## (1) Установить Qdrant. ## 

Поднял docker контейнер
```
student:~/qdrant$ docker run -p 6333:6333 -p 6334:6334 -v /home/student/qdrant/qdrant_storage:/qdrant/storage qdrant/qdrant
           _                 _    
  __ _  __| |_ __ __ _ _ __ | |_  
 / _` |/ _` | '__/ _` | '_ \| __| 
| (_| | (_| | | | (_| | | | | |_  
 \__, |\__,_|_|  \__,_|_| |_|\__| 
    |_|                           

Version: 1.19.2, build: 016542aa
Access web UI at http://localhost:6333/dashboard
Agentic Skills: https://skills.qdrant.tech/

2026-10-11T17:16:37.757850Z  INFO storage::content_manager::consensus::persistent: Initializing new raft state at ./storage/raft_state.json
2026-10-11T17:16:37.792834Z  INFO qdrant: Distributed mode disabled
2026-10-11T17:16:37.793407Z  INFO qdrant: Telemetry reporting enabled, id: 42778bfa-12c8-4e6a-8a4e-00c5287277a9
2026-10-11T17:16:37.852458Z  INFO qdrant::actix: REST transport settings: keep_alive=5s, client_request_timeout=5s, client_disconnect_timeout=5s
2026-10-11T17:16:37.858520Z  INFO qdrant::actix: TLS disabled for REST API
2026-10-11T17:16:37.858713Z  INFO qdrant::actix: Qdrant HTTP listening on 6333
2026-10-11T17:16:37.858729Z  INFO actix_server::builder: starting 1 workers
2026-10-11T17:16:37.858742Z  INFO actix_server::server: Actix runtime found; starting in Actix runtime
2026-10-11T17:16:37.858745Z  INFO actix_server::server: starting service: "actix-web-service-0.0.0.0:6333", workers: 1, listening on: 0.0.0.0:6333
2026-10-11T17:16:37.928780Z  INFO qdrant::tonic: Qdrant gRPC listening on 6334
2026-10-11T17:16:37.930729Z  INFO qdrant::tonic: TLS disabled for gRPC API
2026-10-11T17:16:46.666424Z ERROR rustls_platform_verifier::verification::others: failed to verify TLS certificate: invalid peer certificate: UnknownIssuer
2026-10-11T17:16:46.738860Z ERROR qdrant::common::telemetry_reporting: Failed to report telemetry Service internal error: Http request error: error sending request for url (https://telemetry.qdrant.io/)

student:~$ docker ps
CONTAINER ID   IMAGE           COMMAND             CREATED              STATUS              PORTS                                                             NAMES
33539ab1abdc   qdrant/qdrant   "./entrypoint.sh"   About a minute ago   Up About a minute   0.0.0.0:6333-6334->6333-6334/tcp, [::]:6333-6334->6333-6334/tcp   intelligent_blackburn
```

Консоль доступна по http://localhost:6333/dashboard#/console
<img width="821" height="272" alt="image" src="https://github.com/user-attachments/assets/13474eea-067e-406a-a828-7ab104976477" />


## (2) Создать коллекцию bank_support. ## 

Создать коллекцию
```
>> client.create_collection(
...     collection_name='bank_support',
...     vectors_config=models.VectorParams(
...         size=384, distance=models.Distance.COSINE),
... )
True
```


## (3) Настроить Python-клиент. ##

Устанавливаем расширение  qdrant-client
```
tudent:~$ pip install qdrant-client --break-system-packages
Defaulting to user installation because normal site-packages is not writeable
Collecting qdrant-client
  Using cached qdrant_client-1.19.1-py3-none-any.whl.metadata (11 kB)
Requirement already satisfied: grpcio>=1.41.0 in ./.local/lib/python3.12/site-packages (from qdrant-client) (1.83.0)
Collecting httpx>=0.20.0 (from httpx[http2]>=0.20.0->qdrant-client)
  Using cached httpx-0.28.1-py3-none-any.whl.metadata (7.1 kB)
Collecting numpy>=1.26 (from qdrant-client)
  Using cached numpy-2.5.4-cp312-cp312-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl.metadata (6.6 kB)
Collecting portalocker<4.0,>=2.7.0 (from qdrant-client)
  Using cached portalocker-3.2.0-py3-none-any.whl.metadata (8.7 kB)
Requirement already satisfied: protobuf>=3.20.0 in ./.local/lib/python3.12/site-packages (from qdrant-client) (7.35.1)
Collecting pydantic!=2.0.*,!=2.1.*,!=2.2.0,>=1.10.8 (from qdrant-client)
  Using cached pydantic-2.14.0-py3-none-any.whl.metadata (131 kB)
Collecting urllib3<3,>=1.26.14 (from qdrant-client)
  Using cached urllib3-2.8.0-py3-none-any.whl.metadata (7.4 kB)
Requirement already satisfied: typing-extensions~=4.12 in ./.local/lib/python3.12/site-packages (from grpcio>=1.41.0->qdrant-client) (4.16.0)
Collecting anyio (from httpx>=0.20.0->httpx[http2]>=0.20.0->qdrant-client)
  Using cached anyio-4.15.1-py3-none-any.whl.metadata (4.7 kB)
Collecting certifi (from httpx>=0.20.0->httpx[http2]>=0.20.0->qdrant-client)
  Using cached certifi-2026.7.22-py3-none-any.whl.metadata (2.5 kB)
Collecting httpcore==1.* (from httpx>=0.20.0->httpx[http2]>=0.20.0->qdrant-client)
  Using cached httpcore-1.0.9-py3-none-any.whl.metadata (21 kB)
Collecting idna (from httpx>=0.20.0->httpx[http2]>=0.20.0->qdrant-client)
  Using cached idna-3.20-py3-none-any.whl.metadata (7.2 kB)
Collecting h11>=0.16 (from httpcore==1.*->httpx>=0.20.0->httpx[http2]>=0.20.0->qdrant-client)
  Using cached h11-0.16.0-py3-none-any.whl.metadata (8.3 kB)
Collecting h2<5,>=3 (from httpx[http2]>=0.20.0->qdrant-client)
  Using cached h2-4.4.1-py3-none-any.whl.metadata (3.8 kB)
Collecting annotated-types>=0.6.0 (from pydantic!=2.0.*,!=2.1.*,!=2.2.0,>=1.10.8->qdrant-client)
  Using cached annotated_types-0.8.0-py3-none-any.whl.metadata (15 kB)
Collecting pydantic-core==2.50.0 (from pydantic!=2.0.*,!=2.1.*,!=2.2.0,>=1.10.8->qdrant-client)
  Using cached pydantic_core-2.50.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (6.5 kB)
Collecting typing-inspection>=0.4.4 (from pydantic!=2.0.*,!=2.1.*,!=2.2.0,>=1.10.8->qdrant-client)
  Using cached typing_inspection-0.4.4-py3-none-any.whl.metadata (2.6 kB)
Collecting hyperframe<7,>=6.1 (from h2<5,>=3->httpx[http2]>=0.20.0->qdrant-client)
  Using cached hyperframe-6.1.0-py3-none-any.whl.metadata (4.3 kB)
Collecting hpack<5,>=4.2 (from h2<5,>=3->httpx[http2]>=0.20.0->qdrant-client)
  Using cached hpack-4.2.0-py3-none-any.whl.metadata (3.3 kB)
Using cached qdrant_client-1.19.1-py3-none-any.whl (406 kB)
Using cached httpx-0.28.1-py3-none-any.whl (73 kB)
Using cached httpcore-1.0.9-py3-none-any.whl (78 kB)
Using cached numpy-2.5.4-cp312-cp312-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl (16.7 MB)
Using cached portalocker-3.2.0-py3-none-any.whl (22 kB)
Using cached pydantic-2.14.0-py3-none-any.whl (487 kB)
Using cached pydantic_core-2.50.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (2.2 MB)
Using cached urllib3-2.8.0-py3-none-any.whl (135 kB)
Using cached annotated_types-0.8.0-py3-none-any.whl (13 kB)
Using cached h2-4.4.1-py3-none-any.whl (62 kB)
Using cached typing_inspection-0.4.4-py3-none-any.whl (14 kB)
Using cached anyio-4.15.1-py3-none-any.whl (132 kB)
Using cached idna-3.20-py3-none-any.whl (69 kB)
Using cached certifi-2026.7.22-py3-none-any.whl (136 kB)
Using cached h11-0.16.0-py3-none-any.whl (37 kB)
Using cached hpack-4.2.0-py3-none-any.whl (34 kB)
Using cached hyperframe-6.1.0-py3-none-any.whl (13 kB)
Installing collected packages: urllib3, typing-inspection, pydantic-core, portalocker, numpy, idna, hyperframe, hpack, h11, certifi, annotated-types, pydantic, httpcore, h2, anyio, httpx, qdrant-client

Successfully installed annotated-types-0.8.0 anyio-4.15.1 certifi-2026.7.22 h11-0.16.0 h2-4.4.1 hpack-4.2.0 httpcore-1.0.9 httpx-0.28.1 hyperframe-6.1.0 idna-3.20 numpy-2.5.4 portalocker-3.2.0 pydantic-2.14.0 pydantic-core-2.50.0 qdrant-client-1.19.1 typing-inspection-0.4.4 urllib3-2.8.0
```
Создание тестовой коллекции
```
student:~$ python3
Python 3.12.3 (main, Aug 31 2026, 10:18:26) [GCC 13.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from qdrant_client import QdrantClient, models
>>> client = QdrantClient(url='http://localhost:6333')
>>> 
>>> 
>>> 
>>> client.create_collection(
...     collection_name="test_collection",
...     vectors_config=models.VectorParams(
...         size=384,  # Match the vector dimension of your embedding model
...         distance=models.Distance.COSINE,  # Choose COSINE, EUCLID, or DOT
...     ),
... )
True
>>> response = client.get_collections()
>>> print(response)
collections=[CollectionDescription(name='test_collection')]
>>> 
```

## (4) Выбрать embedding-модель. ##
Выбираю embedding-модель all-MiniLM-L6-v2 и при первом обращении она скачивается
```
>>> from qdrant_client import QdrantClient
>>> from qdrant_client.models import Distance, VectorParams, PointStruct
>>> from sentence_transformers import SentenceTransformer
>>> 
>>> client = QdrantClient(url='http://localhost:6333') 
>>> 
>>> model = SentenceTransformer('all-MiniLM-L6-v2')

modules.json: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 349/349 [00:00<00:00, 1.17MB/s]
Warning: You are sending unauthenticated requests to the HF Hub. Please set a HF_TOKEN to enable higher rate limits and faster downloads.
config_sentence_transformers.json: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 116/116 [00:00<00:00, 382kB/s]
README.md: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 10.5k/10.5k [00:00<00:00, 27.6MB/s]
sentence_bert_config.json: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 53.0/53.0 [00:00<00:00, 204kB/s]
config.json: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 612/612 [00:00<00:00, 1.80MB/s]
model.safetensors: downloading bytes: ███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 85.0MB, 5.02MB/s  
model.safetensors: reconstructing file: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 90.9MB / 90.9MB, 7.29MB/s  
Loading weights: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 103/103 [00:00<00:00, 6508.58it/s]
tokenizer_config.json: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 350/350 [00:00<00:00, 1.73MB/s]
vocab.txt: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 232k/232k [00:00<00:00, 2.13MB/s]
tokenizer.json: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 466k/466k [00:00<00:00, 4.32MB/s]
special_tokens_map.json: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 112/112 [00:00<00:00, 122kB/s]
config.json: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 190/190 [00:00<00:00, 325kB/s]
>>> vector_size = model.get_sentence_embedding_dimension()
<stdin>:1: FutureWarning: The `get_sentence_embedding_dimension` method has been renamed to `get_embedding_dimension`.
```


## (5) Построить embeddings для всех обращений. ##

```
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct
from sentence_transformers import SentenceTransformer

client = QdrantClient(url='http://localhost:6333') 

model = SentenceTransformer('all-MiniLM-L6-v2')
vector_size = model.get_sentence_embedding_dimension()

collection_name = "bank_support"


# 4. Define your raw documents and metadata
raw_documents = [
{"id": 1, "title": "Ошибка CRM", "text": "Система оформления кредитных заявок закрывается после открытия карточки клиента", "department": "credit","category": "crm"},
{"id": 2, "title": "Не печатается документ", "text": "После отправки договора на печать устройство не отвечает", "department": "operations", "category": "printing"},
{"id": 3, "title": "Ошибка входа", "text": "Сотрудник не может войти в систему обработки платежей", "department": "finance", "category": "authentication"},
{"id": 4, "title": "Проблема Outlook", "text": "Письма остаются в исходящих и не отправляются получателям", "department": "office", "category": "mail"},
{"id": 5, "title": "Сбой документооборота", "text": "Система электронного согласования зависает при открытии вложений", "department": "legal","category": "edms"},
{"id": 6, "title": "Ошибка клиентского приложения", "text": "Приложение для оформления банковских продуктов аварийно завершается после авторизации", "department": "credit", "category": "crm"}
]

points = []
for doc in raw_documents:
    # Generate the vector embedding for the text content
    embedding = model.encode(doc["text"]).tolist()
    
    # Construct the point
    point = PointStruct(
        id=doc["id"],
        vector=embedding,
        payload={
            "text": doc["text"],        # Store the original text inside the payload
            "category": doc["category"]   # Add any metadata fields
        }
    )
    points.append(point)
```


## (6) Загрузить данные в Qdrant. ##

```
operation_info = client.upsert(
    collection_name=collection_name,
    wait=True,
    points=points
)

print(f"Upsert status: {operation_info.status}")
Upsert status: completed
```

проверить данные в коллекции bank_support, все загрузилось
```
>>> points, next_page_offset = client.scroll(
...     collection_name="bank_support",
...     limit=10,  # Number of points to return
...     with_payload=True,
...     with_vectors=False,  # Set to True if you also want vector arrays
... )
>>> 
>>> for point in points:
...   print(point.id, point.payload)
... 
1 {'text': 'Система оформления кредитных заявок закрывается после открытия карточки клиента', 'category': 'crm'}
2 {'text': 'После отправки договора на печать устройство не отвечает', 'category': 'printing'}
3 {'text': 'Сотрудник не может войти в систему обработки платежей', 'category': 'authentication'}
4 {'text': 'Письма остаются в исходящих и не отправляются получателям', 'category': 'mail'}
5 {'text': 'Система электронного согласования зависает при открытии вложений', 'category': 'edms'}
6 {'text': 'Приложение для оформления банковских продуктов аварийно завершается после авторизации', 'category': 'crm'}
```




## (7) Для каждого запроса необходимо сделать ##

Необходимо выполнить поиск для следующих запросов:

Запрос пользователя Ожидаемый результат

# «закрывается программа кредитов» обращения CRM
```

q = model.encode(['query: закрывается программа кредитов'],
                normalize_embeddings=True)[0]

client.query_points(
   collection_name='bank_support',
     query=q.tolist(),
     limit=5,
)
QueryResponse(points=[ScoredPoint(id=1, version=1, score=0.6310812, payload={'text': 'Система оформления кредитных заявок закрывается после открытия карточки клиента', 'category': 'crm'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=4, version=1, score=0.5867803, payload={'text': 'Письма остаются в исходящих и не отправляются получателям', 'category': 'mail'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=6, version=1, score=0.5767096, payload={'text': 'Приложение для оформления банковских продуктов аварийно завершается после авторизации', 'category': 'crm'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=5, version=1, score=0.5669989, payload={'text': 'Система электронного согласования зависает при открытии вложений', 'category': 'edms'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=2, version=1, score=0.5307403, payload={'text': 'После отправки договора на печать устройство не отвечает', 'category': 'printing'}, vector=None, shard_key=None, order_value=None)])

```
почему семантический поиск нашел нужный документ?
- потому-что совпали слова закрывается и кредитов. Поиск отработал как надо. 
какие слова отсутствовали в исходном тексте?
- не было слова программа
почему поиск по ключевым словам отработает хуже?
- потому-что не все ключевые слова есть в тексте


# «невозможно войти в платежную систему» authentication
```
q = model.encode(['query: невозможно войти в платежную систему'],
                normalize_embeddings=True)[0]

client.query_points(
     collection_name='bank_support',
     query=q.tolist(),
     limit=5,
)

QueryResponse(points=[ScoredPoint(id=3, version=1, score=0.74127036, payload={'text': 'Сотрудник не может войти в систему обработки платежей', 'category': 'authentication'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=4, version=1, score=0.5703194, payload={'text': 'Письма остаются в исходящих и не отправляются получателям', 'category': 'mail'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=5, version=1, score=0.5457332, payload={'text': 'Система электронного согласования зависает при открытии вложений', 'category': 'edms'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=2, version=1, score=0.5326968, payload={'text': 'После отправки договора на печать устройство не отвечает', 'category': 'printing'}, vector=None, shard_key=None, order_value=None),
 ScoredPoint(id=6, version=1, score=0.47339642, payload={'text': 'Приложение для оформления банковских продуктов аварийно завершается после авторизации', 'category': 'crm'}, vector=None, shard_key=None, order_value=None)])
```
почему семантический поиск нашел нужный документ?
- потому-что совпали слова  войти,платежную,систему. Поиск отработал как надо. 
какие слова отсутствовали в исходном тексте?
- не было слова невозможно
почему поиск по ключевым словам отработает хуже?
- потому-что не все ключевые слова есть в тексте




# «зависает система согласования документов» EDMS

```
q = model.encode(['query: зависает система согласования документов'],
                normalize_embeddings=True)[0]

client.query_points(
     collection_name='bank_support',
     query=q.tolist(),
     limit=5,
)
QueryResponse(points=[ScoredPoint(id=5, version=1, score=0.7246635, payload={'text': 'Система электронного согласования зависает при открытии вложений', 'category': 'edms'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=1, version=1, score=0.71989876, payload={'text': 'Система оформления кредитных заявок закрывается после открытия карточки клиента', 'category': 'crm'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=6, version=1, score=0.7076012, payload={'text': 'Приложение для оформления банковских продуктов аварийно завершается после авторизации', 'category': 'crm'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=2, version=1, score=0.56134963, payload={'text': 'После отправки договора на печать устройство не отвечает', 'category': 'printing'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=4, version=1, score=0.5231869, payload={'text': 'Письма остаются в исходящих и не отправляются получателям', 'category': 'mail'}, vector=None, shard_key=None, order_value=None)])

```
почему семантический поиск нашел нужный документ?
- потому-что совпали слова  зависает, система, согласования. Поиск отработал как надо. 
какие слова отсутствовали в исходном тексте?
- не было слова документов
почему поиск по ключевым словам отработает хуже?
- потому-что не все ключевые слова есть в тексте


# «не отправляется электронная почта» проблемы Outlook
```
q = model.encode(['query: не отправляется электронная почта'],
                normalize_embeddings=True)[0]

client.query_points(
     collection_name='bank_support',
     query=q.tolist(),
     limit=5,
)

QueryResponse(points=[ScoredPoint(id=2, version=1, score=0.70173323, payload={'text': 'После отправки договора на печать устройство не отвечает', 'category': 'printing'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=4, version=1, score=0.6910798, payload={'text': 'Письма остаются в исходящих и не отправляются получателям', 'category': 'mail'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=3, version=1, score=0.5391722, payload={'text': 'Сотрудник не может войти в систему обработки платежей', 'category': 'authentication'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=5, version=1, score=0.51696676, payload={'text': 'Система электронного согласования зависает при открытии вложений', 'category': 'edms'}, vector=None, shard_key=None, order_value=None),
ScoredPoint(id=1, version=1, score=0.49991882, payload={'text': 'Система оформления кредитных заявок закрывается после открытия карточки клиента', 'category': 'crm'}, vector=None, shard_key=None, order_value=None)])
```
почему семантический поиск нашел нужный документ?
- Поиск НЕ отработал как надо. Нужный документ только на втором месте. Возможно связано с моделью эмбеддингов. 
какие слова отсутствовали в исходном тексте?
- не было слова электронная почта
почему поиск по ключевым словам отработает хуже?
- потому-что не все ключевые слова есть в тексте





