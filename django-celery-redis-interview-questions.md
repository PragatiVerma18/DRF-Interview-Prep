# Django + Celery + Redis + RabbitMQ Interview Questions

Here are some potential interview questions covering Django, Celery, Redis, and RabbitMQ, along with explanations and edge cases:

## Django

1. **How does Django handle asynchronous tasks?**

   Django itself is primarily synchronous, but it has support for asynchronous views, middleware, and database operations as of Django 3.1+. However, Django does not provide built-in background task execution. Instead, Celery is commonly used to handle async tasks like sending emails, processing images, or performing background computations.

   - https://dev.to/pragativerma18/unlocking-performance-a-guide-to-async-support-in-django-2jdj

2. **What are Django signals, and how do they compare to Celery tasks?**

   Django signals allow decoupled components of a Django application to communicate when certain events occur. For example, the `post_save` signal can be used to trigger an action when a model instance is saved.

   **Comparison with Celery:**

   - **Signals** are synchronous and executed within the request-response cycle.
   - **Celery tasks** are asynchronous and executed in the background, preventing delays in user-facing operations.

   **Use cases:**

   - Signals: Logging, cache invalidation, simple notifications.
   - Celery: Sending emails, generating reports, and time-consuming operations.

3. **How would you scale a Django application handling high traffic?**

   To scale a Django app for high traffic:

   - **Optimize Database Queries**: Use indexing, avoid N+1 queries, leverage caching.
   - **Use Load Balancing**: Deploy multiple application servers behind a load balancer.
   - **Enable Caching**: Use Redis or Memcached for frequently accessed data.
   - **Use a CDN**: Offload static files and media files to a CDN.
   - **Use Asynchronous Task Processing**: Offload heavy tasks to Celery workers.
   - **Deploy with WSGI or ASGI**: Use Gunicorn for WSGI, or Daphne/Uvicorn for ASGI (async support).

4. **How does Django's ORM interact with databases?**

   Django ORM (Object-Relational Mapper) allows interaction with databases using Python objects instead of raw SQL. It converts Python models into SQL queries and provides an abstraction layer.

   - **QuerySet API**: `Model.objects.filter(name="John")` translates to `SELECT * FROM model WHERE name='John'`.
   - **Transactions**: Uses ACID-compliant transactions.
   - **Connection Pooling**: Managed by Django’s database engine.
   - **Lazy Execution**: QuerySets are evaluated only when necessary, improving efficiency.

5. **What happens when a database transaction fails in Django?**

   If a transaction fails:

   - If **atomic blocks** (`@transaction.atomic`) are used, Django **rolls back** all changes within the block.
   - Without atomic transactions, partial changes may persist, leading to data inconsistency.

   Example:

   ```python
   from django.db import transaction

   try:
       with transaction.atomic():
           user = User.objects.create(username="test_user")
           Profile.objects.create(user=user)  # If this fails, user creation is also rolled back
   except Exception as e:
       print("Transaction failed:", e)
   ```

6. **How does Django handle caching, and how can Redis improve performance?**

   Django supports multiple caching backends:

   - **Database caching**: Stores cache data in a database table.
   - **File system caching**: Stores cache files on disk.
   - **Memory-based caching**: Uses Memcached or Redis for high-performance caching.

   **Redis Benefits:**

   - **In-memory storage**: Faster than disk-based caching.
   - **Persistence**: Supports data persistence (RDB, AOF).
   - **Distributed Cache**: Works across multiple servers.

   Example:

   ```python
   CACHES = {
       'default': {
           'BACKEND': 'django.core.cache.backends.redis.RedisCache',
           'LOCATION': 'redis://127.0.0.1:6379/1',
       }
   }
   ```

7. **What happens if the Django application loses connection to the database?**

   If the database connection is lost:

   - Django raises an `OperationalError`.
   - Default behavior: Retries connections but eventually fails if the database is unreachable.
   - Solution: Use **connection pooling** and **auto-reconnect** using Django’s `CONN_MAX_AGE` setting.

   Example:

   ```python
   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.postgresql',
           'NAME': 'mydb',
           'USER': 'myuser',
           'PASSWORD': 'mypassword',
           'HOST': 'db_host',
           'PORT': '5432',
           'CONN_MAX_AGE': 600,  # Persistent connections
       }
   }
   ```

8. **How do you secure Django applications from SQL injection, XSS, and CSRF attacks?**

   - **SQL Injection Prevention**: Django ORM escapes queries by default.
     ```python
     User.objects.get(username=username)  # Safe
     ```
   - **XSS (Cross-Site Scripting) Prevention**: Django auto-escapes templates.
     ```html
     {{ user_input }}
     <!-- Safe -->
     ```
   - **CSRF (Cross-Site Request Forgery) Prevention**: Django includes CSRF middleware.
     ```html
     <form method="post">{% csrf_token %}</form>
     ```

   Other best practices:

   - Use `SECURE_SSL_REDIRECT = True` (force HTTPS).
   - Use `HttpOnly` and `Secure` attributes on cookies.

9. **What is the difference between Django’s session storage in the database vs. in Redis?**

   Django allows session storage in:

   - **Database (`django.contrib.sessions.backends.db`)**: Stores session data in a table, leading to higher latency.
   - **Cache (Redis/Memcached)**: Stores session data in-memory, making it faster.

   **Why use Redis?**

   - **Faster retrieval** due to in-memory storage.
   - **Automatic expiration** of stale sessions.
   - **Scalability** for distributed applications.

10. **How do you manage background tasks in Django without Celery?**

    While Celery is the preferred choice for background tasks, alternatives include:

    1. **Django-cron**:
       - Runs periodic tasks based on system cron jobs.
       - Example:
         ```python
         from django_cron import CronJobBase, Schedule
         class MyCronJob(CronJobBase):
             RUN_EVERY_MINS = 60  # every hour
             schedule = Schedule(run_every_mins=RUN_EVERY_MINS)
             code = 'my_app.my_cron_job'
             def do(self):
                 print("Running background task")
         ```
    2. **Threading in Django Views** (not recommended for heavy tasks):

       ```python
       import threading
       def process_data():
           # Expensive operation
           pass

       t = threading.Thread(target=process_data)
       t.start()
       ```

    3. **Using Django’s `runserver` with Management Commands**:
       - Custom command:
         ```python
         from django.core.management.base import BaseCommand
         class Command(BaseCommand):
             def handle(self, *args, **kwargs):
                 print("Executing background task")
         ```
       - Run as:
         ```bash
         python manage.py my_custom_command
         ```
    4. **Database-backed queue**:
       - Use Django’s ORM to store tasks and process them with a simple worker script.

    While these alternatives work, Celery provides better reliability, scheduling, and retry mechanisms.

## Celery

1. **What is Celery, and why is it used?**

   Celery is an asynchronous task queue based on distributed message passing. It is used to offload long-running or background tasks from the main application, improving responsiveness and performance.

   **Common Use Cases:**

   - Sending emails asynchronously.
   - Processing large datasets.
   - Scheduling periodic tasks (e.g., reports, data backups).
   - Handling web scraping jobs.

2. **How does Celery execute tasks asynchronously?**

   Celery follows a producer-consumer architecture where:

   1. A **producer (Django app)** sends a task to a **message broker** (Redis/RabbitMQ).
   2. A **Celery worker** picks up the task from the broker and processes it.
   3. The result is optionally stored in a **results backend** (Redis, database, etc.).

   ![image.png](Django%20+%20Celery%20+%20Redis%20+%20RabbitMQ%20Interview%20Quest%201a0ac7d04d878032b379c840b387c722/image.png)

3. **What are the different Celery message brokers, and how do they compare?**

   Celery supports multiple message brokers:

   - **Redis** (fast in-memory store, supports pub/sub, but may lose tasks if not persistent).
   - **RabbitMQ** (persistent queues, better message durability).
   - **Amazon SQS** (fully managed, highly available, but higher latency).

   **Comparison:**

   | Feature     | Redis                  | RabbitMQ             | Amazon SQS        |
   | ----------- | ---------------------- | -------------------- | ----------------- |
   | Speed       | Faster (in-memory)     | Slower (disk-based)  | Slower            |
   | Persistence | Optional (RDB/AOF)     | Yes (durable queues) | Yes               |
   | Scalability | Horizontal scaling     | Clustering required  | Auto-scaled       |
   | Use Case    | Quick, transient tasks | Reliable messaging   | Cloud-native apps |

4. **What happens if a Celery task fails?**

   By default, failed tasks are logged, but they can be retried using `retry`.

   Example:

   ```python
   from celery import shared_task
   from celery.exceptions import MaxRetriesExceededError
   import requests

   @shared_task(bind=True, max_retries=3)
   def fetch_url(self, url):
       try:
           response = requests.get(url)
           return response.text
       except requests.RequestException as exc:
           raise self.retry(exc=exc, countdown=5)  # Retries after 5 seconds
   ```

   - The task retries up to **3 times** before failing permanently.
   - `countdown=5` adds a delay before retries.

5. **What happens if the Celery worker crashes?**

   - Any task in progress **may be lost** if it was running in memory.
   - If using **RabbitMQ** (durable queues), unprocessed tasks remain in the queue.
   - If using **Redis**, tasks might be lost unless `acks_late=True` is used.
   - Celery can recover tasks with **task acknowledgment** enabled.

   Solution: Enable **task persistence** using:

   ```python
   task_acks_late = True  # Ensures tasks are acknowledged after execution
   worker_prefetch_multiplier = 1  # Ensures fair task distribution
   ```

   ### Is the Task Lost If a Worker Crashes?

   | **When Worker Crashes?**                | **Is Task Lost?** | **Solution**                                   |
   | --------------------------------------- | ----------------- | ---------------------------------------------- |
   | **Before fetching task**                | ❌ No             | Task is still in queue                         |
   | **After fetching but before execution** | ✅ Yes (default)  | `acks_late=True`                               |
   | **During execution**                    | ✅ Yes (default)  | `acks_late=True`, `autoretry_for=(Exception,)` |
   | **After execution (completed task)**    | ❌ No             | Task is done                                   |

6. **How does Celery handle scheduled and periodic tasks?**

   Celery can schedule tasks using **Celery Beat**, a periodic task scheduler.

   Example:

   ```python
   from celery.schedules import crontab
   from celery import Celery

   app = Celery('tasks')

   app.conf.beat_schedule = {
       'every-day-task': {
           'task': 'tasks.daily_report',
           'schedule': crontab(hour=0, minute=0),
       },
   }
   ```

   This runs `daily_report` every day at midnight.

7. **What happens if Redis (or RabbitMQ) crashes while a task is in progress?**

   - **Redis as broker**: Tasks might be lost unless they are persistent.
   - **RabbitMQ as broker**: Messages remain in the queue due to durability settings.
   - **Result backend impact**: If Redis is the result backend, results might be lost.

   Solution:

   - Use **persistent queues** in RabbitMQ (`x-ha-policy: all` for HA queues).
   - Enable **AOF persistence** in Redis to reduce data loss.
   - Configure **retry policies** in Celery tasks.

8. **How do you handle task dependencies in Celery?**

   Celery provides **chaining**, **groups**, and **callbacks** for task dependencies.

   **Task Chain (Sequential Execution)**:

   ```python
   from celery import chain

   chain(task1.s(), task2.s(), task3.s())()
   ```

   **Task Group (Parallel Execution)**:

   ```python
   from celery import group

   group(task1.s(), task2.s(), task3.s())()
   ```

   **Chord (Parallel + Callback)**:

   ```python
   from celery import chord

   chord([task1.s(), task2.s()])(callback_task.s())
   ```

9. **What are Celery's different states (PENDING, STARTED, SUCCESS, FAILURE, etc.)?**

   Celery tasks have different states:

   - **PENDING**: Task is in the queue but not yet assigned.
   - **STARTED**: Task execution has begun (requires `task_track_started=True`).
   - **RETRY**: Task has failed but is retrying.
   - **SUCCESS**: Task executed successfully.
   - **FAILURE**: Task execution failed.

10. **What is the differences Between Celery Workers, Task Queues, Message Brokers, and Result Backends?**

    | **Component**       | **Definition**                          | **Purpose**                                                   | **Examples**                               |
    | ------------------- | --------------------------------------- | ------------------------------------------------------------- | ------------------------------------------ |
    | **Celery Workers**  | Processes that execute Celery tasks.    | Consume tasks from queues and execute them asynchronously.    | `celery worker -A myapp`                   |
    | **Task Queues**     | Queues that hold pending tasks.         | Stores tasks until they are picked up by a worker.            | `default`, `high-priority`, `low-priority` |
    | **Message Brokers** | Middleware that routes tasks to queues. | Transfers tasks from the producer (Django app) to the worker. | Redis, RabbitMQ, Amazon SQS                |
    | **Result Backends** | Stores task execution results.          | Allows retrieval of task status and results.                  | Redis, PostgreSQL, MongoDB, Memcached      |

    ***

    ### **Detailed Breakdown**

    ### **1. Celery Workers**

    - Workers are background processes that execute tasks asynchronously.
    - They listen for tasks in a queue and process them when available.
    - Multiple workers can run on different machines to scale task execution.

    **Example: Starting a worker**

    ```bash
    celery -A myapp worker --loglevel=info
    ```

    - `A myapp`: Specifies the Celery app.
    - `worker`: Starts a worker process.
    - `-loglevel=info`: Sets log verbosity.

    ***

    ### **2. Task Queues**

    - Task queues hold tasks that are waiting to be processed.
    - Workers consume tasks from these queues in a FIFO manner.
    - Tasks can be routed to different queues based on priority.

    **Example: Defining a queue in Celery**

    ```python
    from celery import Celery

    app = Celery('myapp', broker='redis://localhost:6379/0')

    app.conf.task_routes = {
        'tasks.high_priority': {'queue': 'high_priority'},
        'tasks.low_priority': {'queue': 'low_priority'},
    }
    ```

    - Tasks are routed based on the function name.

    **Sending a task to a specific queue**

    ```python
    add.apply_async(args=[4, 4], queue='high_priority')
    ```

    ***

    ### **3. Message Brokers**

    - A message broker acts as an intermediary between producers (Django app) and consumers (workers).
    - It ensures tasks are stored in queues until workers are ready to process them.

    **Popular Message Brokers:**

    | Broker         | Pros                                                     | Cons                                          |
    | -------------- | -------------------------------------------------------- | --------------------------------------------- |
    | **Redis**      | Fast, simple setup, supports Pub/Sub.                    | Volatile memory storage, potential data loss. |
    | **RabbitMQ**   | Persistent queues, message durability, advanced routing. | More complex setup.                           |
    | **Amazon SQS** | Fully managed, scalable.                                 | Higher latency, additional cost.              |

    ***

    ### **4. Result Backends**

    - The result backend stores task statuses and return values.
    - If a task returns a result, it can be retrieved later.
    - Some backends support expiration policies to delete old results.

    **Example: Storing results in Redis**

    ```python
    app.conf.result_backend = 'redis://localhost:6379/0'
    ```

    **Checking task status**

    ```python
    result = add.delay(4, 4)
    print(result.status)  # PENDING, SUCCESS, FAILURE
    print(result.get())   # Fetch result
    ```

    **Common result backends:**

    | Backend    | Pros                             | Cons                         |
    | ---------- | -------------------------------- | ---------------------------- |
    | Redis      | Fast, in-memory storage.         | Data loss if not persistent. |
    | PostgreSQL | Durable, SQL queries supported.  | Slower than Redis.           |
    | MongoDB    | Flexible, supports complex data. | Additional setup required.   |

11. **Why Do We Need Message Brokers If Queues Already Exist?**

    At first glance, it might seem like **task queues** should be enough to handle background tasks. However, **message brokers** provide essential functionality that simple queues alone cannot offer. Here’s why message brokers are necessary:

    ### **1. Message Brokers Manage Queues Efficiently**

    A **queue** is just a storage structure for tasks, but it does not have the intelligence to:

    - Route messages to the right workers.
    - Ensure message delivery reliability.
    - Handle multiple producers and consumers efficiently.

    A **message broker** (like Redis or RabbitMQ) **manages** queues by:

    - Storing tasks in memory or disk.
    - Ensuring tasks are delivered to the right queue.
    - Handling retries, acknowledgments, and routing.

    Without a broker, we would need to manually implement all of these features.

    ### **2. Message Brokers Ensure Reliability & Durability**

    - If a queue were just a simple data structure (e.g., a Python list or database table), **tasks could be lost** if the system crashes.
    - Brokers like **RabbitMQ** provide **persistent queues**, ensuring that tasks survive restarts.
    - **Redis** allows for **data persistence** using **Append-Only File (AOF) mode**.

    📌 **Example:**

    If a worker crashes while processing a task, RabbitMQ **re-delivers** the task to another worker.

    ***

    ### **3. Message Brokers Allow Asynchronous Communication**

    - The **producer (Django app)** doesn’t have to wait for the task to complete—it just **sends** it to the broker and moves on.
    - The **worker** picks up the task when it’s available, processes it, and sends the result back.
    - This enables **scalability** and **non-blocking execution**.

    📌 **Example:**

    A Django app sending 1,000 emails should **not** process them synchronously—it should push them to a broker like Redis, and multiple Celery workers can process them in parallel.

    ***

    ### **4. Message Brokers Support Multiple Consumers & Load Balancing**

    - A single queue can be **shared among multiple workers**, distributing tasks efficiently.
    - Brokers like RabbitMQ **load-balance** tasks among multiple workers using **round-robin scheduling**.

    📌 **Example:**

    If 5 workers are listening to a queue, the broker distributes tasks among them dynamically.

    ***

    ### **5. Message Brokers Support Advanced Task Routing**

    - Some tasks may be **high-priority**, while others can wait.
    - Brokers **route tasks to different queues** based on predefined rules.

    📌 **Example:**

    A web scraping job may go to a **low-priority queue**, while a payment processing job goes to a **high-priority queue**.

    ```python
    app.conf.task_routes = {
        'tasks.process_payment': {'queue': 'high_priority'},
        'tasks.web_scrape': {'queue': 'low_priority'},
    }
    ```

    ***

    ### **6. Message Brokers Handle Retries & Acknowledgments**

    - If a worker crashes while processing a task, a broker ensures the task is **retried**.
    - Celery allows enabling **acknowledgments**, meaning the broker considers a task **complete only when the worker confirms it**.

    📌 **Example:**

    If `task_acks_late=True` is enabled in Celery, a task will only be removed from the queue **after** successful execution.

    ```python
    app.conf.task_acks_late = True
    ```

    ***

    ### **7. Message Brokers Scale With the System**

    - As the number of tasks grows, brokers handle **thousands to millions of tasks** efficiently.
    - They support **distributed task execution**, meaning multiple workers across different servers can process tasks from a single broker.

    📌 **Example:**

    In a cloud-based architecture, **multiple workers on different servers** can pull tasks from a **single broker instance**.

    ***

    ### **Summary: Why Not Just Use Queues?**

    | Feature                            | Simple Queue (e.g., Python List, DB Table) | Message Broker (Redis, RabbitMQ)                 |
    | ---------------------------------- | ------------------------------------------ | ------------------------------------------------ |
    | **Persistence**                    | No (tasks lost on crash)                   | Yes (Redis AOF, RabbitMQ durable queues)         |
    | **Asynchronous Execution**         | No (blocking execution)                    | Yes (non-blocking)                               |
    | **Load Balancing**                 | No (one queue = one consumer)              | Yes (multiple workers consume tasks dynamically) |
    | **Task Retries & Acknowledgments** | No (manual handling)                       | Yes (automatic retries, acks)                    |
    | **Task Routing & Prioritization**  | No                                         | Yes (priority queues, routing rules)             |
    | **Scalability**                    | Low (single queue = single system)         | High (distributed processing)                    |

12. **Where Do Queues Exist in Celery?**

    The queues in Celery are **not stored inside Celery itself**—they exist **within the message broker** (like Redis or RabbitMQ). Celery **only interacts with these queues** to send and receive tasks.

    ### **How Celery Queues Work?**

    1. **Django (or any producer) sends a task** → The task is pushed into a queue inside the message broker.
    2. **The broker holds the queue** until a worker picks up the task.
    3. **Celery workers consume tasks** from these queues and execute them.

    ### **Where Do These Queues Physically Exist?**

    It depends on the message broker being used:

    | **Message Broker** | **Where Queues Exist?**                                                                                                   |
    | ------------------ | ------------------------------------------------------------------------------------------------------------------------- |
    | **Redis**          | Queues exist as **lists** inside Redis memory (e.g., `LPUSH` for adding tasks, `RPOP` for consuming).                     |
    | **RabbitMQ**       | Queues exist as **durable message queues** inside RabbitMQ (AMQP-based). Messages are stored temporarily until processed. |
    | **Amazon SQS**     | Queues exist inside Amazon’s managed Simple Queue Service (SQS), persisting messages until consumed.                      |
    | **Kafka**          | Queues exist as **partitions inside Kafka topics**, allowing scalable message streaming.                                  |

## Redis

1. **How does Redis differ from traditional databases?**

   | Feature              | Redis                                        | Traditional Databases (PostgreSQL, MySQL, etc.) |
   | -------------------- | -------------------------------------------- | ----------------------------------------------- |
   | **Storage**          | In-memory                                    | Disk-based                                      |
   | **Speed**            | Extremely fast                               | Slower (disk I/O involved)                      |
   | **Data Persistence** | Optional (AOF, RDB)                          | Persistent by default                           |
   | **Data Model**       | Key-value                                    | Relational (tables, joins)                      |
   | **Transactions**     | Supports transactions, but limited           | Full ACID compliance                            |
   | **Scalability**      | Scales horizontally (sharding, clustering)   | Can be scaled but needs optimizations           |
   | **Use Cases**        | Caching, message queues, real-time analytics | OLTP, relational data storage                   |

   Redis is **not a replacement** for traditional databases but is used for caching, real-time operations, and message brokering.

2. **What happens if Redis crashes while it is storing Celery task states?**

   - If Redis is being used as a **Celery result backend**, all task states **will be lost** unless persistence is enabled.
   - Tasks that are **queued but not yet fetched** may also be lost.
   - Running tasks **continue** if they have already been picked up by workers.

   ✅ **Solution:**

   - Enable Redis **AOF (Append-Only File) or RDB snapshots** for persistence.
   - Use an **alternative result backend** (e.g., PostgreSQL, S3, or RabbitMQ) to avoid losing task states.

3. **How can Redis persistence be enabled, and what are the trade-offs?**

   Redis provides **two persistence mechanisms**:

   1. **RDB (Redis Database Backup) – Periodic Snapshots**

      - Saves a snapshot of the dataset **at intervals** (e.g., every 5 minutes).
      - Less disk I/O but **risk of data loss** between snapshots.
      - ✅ **Good for caching, not for critical data.**
      - 🔴 **Trade-off:** Data loss possible if Redis crashes between snapshots.

      **Enable RDB:**

      ```
      save 900 1   # Save every 900 seconds if at least 1 change
      save 300 10  # Save every 300 seconds if at least 10 changes
      ```

   2. **AOF (Append-Only File) – Continuous Logging**

      - Logs every write operation **to disk** for full recovery.
      - ✅ **Best for Celery queues and critical tasks.**
      - 🔴 **Trade-off:** More disk usage and I/O overhead.

      **Enable AOF:**

      ```
      appendonly yes
      appendfsync everysec  # Sync to disk every second
      ```

   ✅ **Best Practice:** Use **both AOF and RDB** for better recovery.

4. **What happens if Redis runs out of memory?**

   - Redis **stops accepting writes** once it reaches the `maxmemory` limit.
   - If `maxmemory-policy` is set, it starts **evicting old keys** (depending on policy).
   - If no eviction policy is set, Redis **throws errors** for new writes.

   ✅ **Solutions:**

   - Increase `maxmemory`:
     ```
     maxmemory 2gb
     ```
   - Set an eviction policy (`allkeys-lru`, `volatile-lru`, etc.):
     ```
     maxmemory-policy allkeys-lru
     ```

5. **How does Redis handle concurrency?**

   - Redis is **single-threaded** but **uses an event loop** to handle multiple requests.
   - It executes commands **one at a time** in sequence (Atomic operations).
   - **Pipelining** allows sending multiple commands at once, improving performance.
   - **Transactions (`MULTI/EXEC`)** ensure multiple operations are executed atomically.

   ✅ **Best Practices:**

   - Use **pipelining** for batch operations.
   - Use **Lua scripts** for atomic multi-step operations.
   - Scale Redis with **sharding and clustering**.

6. **What are Redis pub/sub and its use cases?**
   - **Publish/Subscribe (Pub/Sub)** allows message broadcasting.
   - **Publishers** send messages to channels, and **subscribers** receive them.
   - **Use Cases:**
     - **Real-time notifications** (e.g., chat apps, live updates).
     - **Event-driven architecture** (decoupling microservices).
     - **Streaming data processing** (e.g., logs, monitoring).
7. **How does Redis replication work, and how do you handle failover?**
   - Redis **replicates** data from a **primary (master) node** to **one or more replicas (slaves)**.
   - Replicas sync **asynchronously**.
   - Failover is handled using **Redis Sentinel** or **Redis Cluster**.
8. **What are the differences between Redis and RabbitMQ as message brokers?**

   | Feature           | Redis                           | RabbitMQ                                   |
   | ----------------- | ------------------------------- | ------------------------------------------ |
   | **Message Model** | Pub/Sub & Streams               | AMQP Queues                                |
   | **Persistence**   | Optional (AOF/RDB)              | Persistent by default                      |
   | **Reliability**   | Less reliable (default)         | Highly reliable                            |
   | **Ordering**      | FIFO not guaranteed             | FIFO & priority queues available           |
   | **Scalability**   | Horizontally scalable           | Needs clustering for scale                 |
   | **Best For**      | Fast, real-time data processing | Reliable task queues, event-driven systems |

   ✅ **When to Use Redis?**

   - Low-latency tasks (real-time updates, analytics).
   - Lightweight pub/sub messaging.

   ✅ **When to Use RabbitMQ?**

   - Reliable, **persistent** message queues.
   - Ensuring **no task loss** in Celery.

9. **What happens if Redis gets overloaded with too many Celery tasks?**

   - High CPU and memory usage.
   - Redis may **reject new tasks** if `maxmemory` is reached.
   - Celery workers may **timeout waiting for tasks**.

   ✅ **Solutions:**

   1. **Increase Redis memory limit** or **enable eviction policy**.
   2. **Use multiple Redis instances (sharding).**
   3. **Switch to RabbitMQ** for a more **scalable** message broker.
   4. **Use Celery rate limiting** to prevent task floods

10. **How do you secure Redis from unauthorized access and attacks?**

    1. **Bind Redis to localhost (prevent external access)**

       ```
       bind 127.0.0.1
       ```

    2. **Set a strong Redis password (`requirepass`)**

       ```
       requirepass MySecurePass
       ```

    3. **Disable dangerous commands (flushall, config, shutdown)**

       ```
       rename-command FLUSHALL ""
       rename-command CONFIG ""
       ```

    4. **Enable TLS Encryption**

       ```
       tls-cert-file /etc/ssl/redis.crt
       tls-key-file /etc/ssl/redis.key
       ```

    5. **Use a firewall (`ufw` or `iptables`)** to restrict access.

       ```
       sudo ufw allow from 192.168.1.100 to any port 6379
       ```

    ✅ **Best Practice:** Deploy Redis **behind a VPN** or **inside a private network**.

## RabbitMQ

1. **How does RabbitMQ work as a message broker for Celery?**

   RabbitMQ is a **reliable** message broker that Celery uses to queue tasks and ensure delivery. It supports **durable queues, message acknowledgments, and retries**, making it a better choice for **persistent and fault-tolerant** task queues.

   1. **Producer (Django/Celery Task) publishes a task** → Sent to an **Exchange**
   2. **Exchange routes the task** → Pushed into a **Queue** (based on bindings)
   3. **Consumer (Celery Worker) fetches the task** → Acknowledges completion after processing

   ✅ **Why RabbitMQ?**

   - Supports **persistent queues** (tasks survive restarts).
   - Ensures **only one worker picks a task** (FIFO processing).
   - Built-in **acknowledgment & retry mechanisms** prevent task loss.

2. **What is the difference between Redis and RabbitMQ as Celery brokers?**

   | Feature         | Redis                          | RabbitMQ                      |
   | --------------- | ------------------------------ | ----------------------------- |
   | **Data Model**  | Key-Value Store                | AMQP-based Message Queue      |
   | **Persistence** | Optional (AOF, RDB)            | Persistent by default         |
   | **Ordering**    | FIFO not guaranteed            | FIFO guaranteed               |
   | **Reliability** | Less reliable (may drop tasks) | Highly reliable               |
   | **Scalability** | Sharding, clustering           | Clustering, high availability |
   | **Use Case**    | Fast, real-time jobs           | Guaranteed delivery of tasks  |

   **When to use Redis?**

   - **High-speed** tasks (caching, low-priority jobs).
   - If **task loss is acceptable**.

   **When to use RabbitMQ?**

   - If **task persistence is required**.
   - **Ensuring at-least-once task execution**.

3. **How does RabbitMQ handle message acknowledgments and retries?**

   - When a worker **fetches a task**, RabbitMQ marks it as **unacknowledged**.
   - The task is **removed from the queue only after acknowledgment** (`ack`).
   - If the worker **crashes before ack**, RabbitMQ **requeues** the task automatically.
   - RabbitMQ supports **automatic retries** if a task fails.

   ✅ **Explicit Acknowledgment in Celery:**

   ```python
   @app.task(acks_late=True)  # Ensures message is acknowledged only after completion
   def process_task():
       ...
   ```

4. **What happens if RabbitMQ crashes while tasks are in the queue?**

   - If queues are **non-durable**, all tasks are **lost**.
   - If queues are **durable** and messages are **persistent**, tasks survive a crash.

   ✅ **Solution:**

   - **Enable durable queues**:
     ```python
     app.conf.task_queues = Queue('default', durable=True)
     ```
   - **Use persistent messages**:
     ```python
     app.conf.task_serializer = 'json'
     app.conf.result_persistent = True
     ```

5. **What is the purpose of RabbitMQ’s exchange, queue, and binding?**

   RabbitMQ routes messages using an **Exchange**, which determines how tasks reach queues.

   - **Exchange**: Routes messages based on rules (`direct`, `fanout`, `topic`).
   - **Queue**: Stores messages until a worker consumes them.
   - **Binding**: Connects exchanges to queues based on rules.

   ✅ **Example:**

   ```python
   # Direct exchange
   app.conf.task_queues = (
       Queue('tasks', Exchange('default'), routing_key='task_queue'),
   )
   ```

6. **How can RabbitMQ ensure message durability?**
   - **Declare durable queues:**
     ```python
     Queue('default', durable=True)
     ```
   - **Enable persistent messages:**
     ```python
     app.conf.task_serializer = 'json'
     app.conf.result_persistent = True
     ```
   - **Use mirrored queues (HA Mode)** in clustered RabbitMQ.
7. **What happens if a consumer crashes after consuming a message?**

   - If the worker **did not acknowledge** the message, RabbitMQ **requeues it**.
   - If **acknowledgment was sent**, the message is **lost** unless a result backend is used.

   ✅ **Solution:** Use `acks_late=True` to avoid losing tasks.

   ```python
   @app.task(acks_late=True)
   def process_task():
       ...
   ```

8. **How do you scale RabbitMQ for high availability?**
   - **Use Clustering** – Deploy multiple RabbitMQ nodes.
   - **Enable High-Availability Queues** – Mirror queues across nodes:
     ```
     rabbitmqctl set_policy ha-all ".*" '{"ha-mode":"all"}'
     ```
   - **Load balancing** – Use **HAProxy** or **NGINX**.
9. **What happens if RabbitMQ queues get overloaded?**
   - If RabbitMQ gets overloaded, it **rejects new tasks** or **slows down consumers**.
   - **Enable flow control** to prevent overload:
     ```
     rabbitmqctl set_vm_memory_high_watermark 0.6
     ```
   - **Increase prefetch limit** to improve throughput:
     ```python
     app.conf.worker_prefetch_multiplier = 10
     ```
10. **How does RabbitMQ handle delayed message delivery?**

    RabbitMQ does not natively support delayed messages but can use **Dead Letter Exchanges (DLX)**.

    ✅ **Example: Delay Message Processing by 10s**

    ```python
    app.conf.task_routes = {'tasks.slow_task': {'queue': 'delayed'}}
    ```

---
