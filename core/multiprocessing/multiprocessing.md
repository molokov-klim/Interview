# **Мультипроцессинг**

[Содержание](/CONTENTS.md#содержание)

## **Junior Level**

Мультипроцессинг в Python — это подход к параллельным вычислениям, использующий несколько независимых процессов вместо
потоков в рамках одной программы. Представьте, что вместо одного офиса с сотрудниками (потоками), работающими в общем
пространстве, вы создаёте несколько полностью отдельных офисов, каждый со своим собственным помещением, оборудованием и
набором сотрудников.

Каждый процесс работает в своём собственном пространстве памяти и имеет отдельный экземпляр интерпретатора Python с
собственным Global Interpreter Lock (GIL). Это позволяет полностью обойти ограничения GIL и задействовать все доступные
ядра процессора для выполнения задач, требующих интенсивных вычислений — таких как обработка изображений, сложные
математические расчёты или анализ больших данных.

Полная изоляция процессов обеспечивает большую стабильность: если один процесс завершится с ошибкой, остальные продолжат
работу. Однако эта же изоляция усложняет обмен данными между процессами — они не могут просто обращаться к общей памяти,
как это делают потоки. Для взаимодействия приходится использовать специальные механизмы межпроцессного взаимодействия (
IPC). Для работы с процессами в Python используется модуль `multiprocessing`, который предоставляет API, во многом
похожий на `threading`, но предназначенный для процессов.

[Содержание](/CONTENTS.md#содержание)

## **Middle Level**

Архитектурно каждый процесс в Python представляет собой отдельный экземпляр интерпретатора со своим адресным
пространством, кучей и стеком. Это обеспечивает настоящий параллелизм на уровне операционной системы — процессы могут
выполняться одновременно на разных ядрах процессора, что делает мультипроцессинг эффективным решением для
CPU-интенсивных задач.

Создание процессов может происходить тремя основными способами, каждый со своими особенностями. **Fork** (доступен в
Unix-системах) быстро копирует память родительского процесса, но при этом наследует все открытые файловые дескрипторы и
блокировки, что иногда приводит к неожиданным проблемам. **Spawn** (используется по умолчанию начиная с Python 3.8)
запускает новый интерпретатор Python, который импортирует модуль и выполняет целевую функцию — это безопаснее, но
требует больше времени. **Forkserver** предлагает компромисс: создаётся серверный процесс, от которого затем порождаются
все остальные процессы, что позволяет избежать многократного копирования ненужных ресурсов.

Для обмена данными между изолированными процессами модуль `multiprocessing` предоставляет несколько механизмов
межпроцессного взаимодействия (IPC).

- **Очереди (Queue)** представляют собой потокобезопасные FIFO-структуры,
  реализованные через системные каналы (pipes).
- **Каналы (Pipe)** обеспечивают двунаправленную связь между двумя
  процессами.
- **Разделяемая память (Shared Memory)** через `multiprocessing.Value` и `multiprocessing.Array` позволяет
  создавать переменные, доступные из разных процессов.
- **Менеджеры (Managers)** предлагают высокоуровневый API для
  создания разделяемых объектов (словарей, списков) через прокси-объекты, абстрагируя сложности IPC.

Синхронизация процессов осуществляется с помощью примитивов, аналогичных тем, что используются в многопоточности (Lock,
Semaphore, Event, Condition), но работающих через механизмы операционной системы или общую память. Для управления
множеством задач часто применяются **пулы процессов (Process Pool)** через `multiprocessing.Pool` или
`concurrent.futures.ProcessPoolExecutor`, которые автоматически распределяют задачи между фиксированным количеством
рабочих процессов, предоставляя удобные методы вроде `map()` и `apply_async()`.

Производительность мультипроцессинга определяется балансом между выгодами от истинного параллелизма и накладными
расходами на создание процессов и IPC. Процессы эффективны для задач, требующих значительных вычислений, но могут быть
избыточны для простых операций из-за высокой стоимости создания. Кроме того, при использовании fork необходимо
учитывать, что дочерний процесс получает копию всей памяти родителя, что может привести к неожиданному потреблению
памяти.

Мультипроцессинг в Python — это мощный инструмент для преодоления ограничений GIL и использования всех вычислительных
ресурсов системы, особенно когда задачи могут быть эффективно распараллелены и требуют минимального обмена данными между
процессами.

[Содержание](/CONTENTS.md#содержание)

## **Senior Level**

В CPython 3.9+ **мультипроцессинг** — **отдельные процессы** через `fork()`/`spawn()`/`forkserver()` в
`Lib/multiprocessing/spawn.py`, **IPC** через `Pipe`/`Queue` (`_multiprocessing.so`), **новый PyInterpreterState** в
каждом процессе, **semaphore/shm** для синхронизации. `Lib/multiprocessing/`,`Modules/_multiprocessing.c`

## 1. multiprocessing.set_start_method() - выбор метода (Lib/multiprocessing/context.py)

```python
def set_start_method(method, force=False):
    if method == 'fork':
        _fork_posix()
    elif method == 'spawn':
        _spawn_posix()
    elif method == 'forkserver':
        _forkserver_posix()
    else:
        raise ValueError(f"unknown start method {method}")

    _current_context._set_start_method(method, force)
```

Когда ты пишешь `mp.set_start_method('spawn')`, Python **выбирает способ** создания
нового процесса. `'fork'` — **быстрое копирование** текущего процесса (только Linux), `'spawn'` — **с нуля** (
Windows/macOS/Linux), `'forkserver'` — **сервер копий**. Это **критично** влияет на производительность и безопасность.

## 2. Process._bootstrap() - входная точка процесса (Lib/multiprocessing/process.py)

```python
def _bootstrap():
    # 1. Восстанавливаем аргументы из командной строки
    sys.argv = _args_from_interpreter_flags()

    # 2. Импортируем main модуль заново
    code, filename, main_path = _args_from_interpreter_flags()
    assert main_path is not None
    _run_module_as_main(main_path, code)
```

Новый процесс запускается с **аргументами**
`python -c "import main; main.worker()"`. **Главное** — `if __name__ == '__main__':` **обязательно**, иначе *
*бесконечный импорт** main модуля в дочернем процессе.

## 3. spawn._main() - spawn метод (Lib/multiprocessing/spawn.py)

```python
def _main(fd):
    # 1. Подключаемся к родительскому процессу через Unix pipe
    parent_r, parent_w = os.pipe()
    code, filename = _read_signed(fd)  # Читаем код main модуля

    # 2. Создаём новый Python интерпретатор
    interp = PyInterpreterState_New()
    tstate = PyThreadState_New(interp)
    PyThreadState_Swap(tstate)

    # 3. Выполняем код worker'а
    exec(code, {'__file__': filename})
```

**spawn** = **новый процесс** → **pipe** с родителем → **чтение** `main.py`
байткода → **PyInterpreterState_New()** → **отдельный интерпретатор** → `exec(main.worker())`. **Ничего** от родителя не
наследуется!

## 4. _multiprocessing.Connection - IPC Pipe (Lib/multiprocessing/connection.py)

```python
class Connection:
    def __init__(self, handle, readable=True):
        self._handle = handle  # file descriptor (Unix pipe/socket)
        self._readable = readable

    def send(self, obj):
        # Сериализуем объект pickle → пишем в pipe
        buf = pickle.dumps(obj)
        self._send_bytes(_header(buf) + buf)

    def recv(self):
        # Читаем из pipe → десериализуем
        buf = self._recv_bytes()
        return pickle.loads(buf)
```

`Queue.put(123)` → `pickle.dumps(123)` → **запись** в Unix pipe → другой процесс
`pipe.read()` → `pickle.loads()`. **Единственный способ** передачи данных между процессами.

## 5. _multiprocessing.SemLock - семафоры (Modules/_multiprocessing/semaphore.c)

```c
typedef struct {
    PyObject_HEAD
    volatile int state;            // 0=locked, 1=unlocked
    HANDLE sem;                    // Windows: HANDLE, Unix: sem_t*
    int wait_flag;                 // Для acquire()
    int release_flag;
} SemLockObject;

static PyObject *semlock_acquire(SemLockObject *self) {
    if (self->state == 1) {
        self->state = 0;           // Быстрое захватывание
        Py_RETURN_TRUE;
    }
    
    // Блокируемся на семафоре
    int success = WaitForSingleObject(self->sem, INFINITE);
    if (success == WAIT_OBJECT_0) {
        self->state = 0;
        Py_RETURN_TRUE;
    }
    Py_RETURN_FALSE;
}
```

`Lock.acquire()` → **system semaphore** (Unix `sem_t`, Windows `HANDLE`). **Один**
процесс захватывает, **другие ждут**. **Не Python lock** — **ОС-level** синхронизация.

## 6. multiprocessing.Pool - пул процессов (Lib/multiprocessing/pool.py)

```python
class Pool:
    def __init__(self, processes=None, initializer=None):
        self._processes = processes
        self._pool = []  # Список Process

        # Создаём worker процессы
        for i in range(self._processes):
            p = self.Process(target=worker)
            p.start()
            self._pool.append(p)

    def map(self, func, iterable):
        # Разбиваем задачу → отправляем в Queue
        # Ждём результаты из result_queue
        pass
```

`Pool(4)` → **4 постоянных** процесса-worker'ов → `map(f, lst)` → **разбивает**
`lst` на куски → **отправляет** в Queue каждому worker'у → **собирает** результаты.

## 7. fork() системный вызов (только Unix, spawn контекст)

```c
// В Lib/multiprocessing/forking.py (C вызов)
pid_t pid = fork();
if (pid == 0) {
    // Дочерний процесс
    close(parent_fd);
    _bootstrap_child();        // Входная точка
} else {
    // Родительский
    close(child_fd);
    return pid;
}
```

`fork()` — **клонирует** **весь** процесс **мгновенно** (копирует **page table**
памяти). **Оба** процесса видят **одинаковую** память до **первой записи** (Copy-on-Write). **Опасно** с GIL/threads!

## 8. PyInterpreterState_New() в дочернем процессе

```c
PyStatus PyInterpreterState_New(PyThreadState *tstate) {
    PyInterpreterState *interp;
    
    interp = PyMem_RawCalloc(1, sizeof(PyInterpreterState));
    if (interp == NULL) {
        return PyStatus_NoMemory();
    }
    
    // Инициализируем **новый** интерпретатор
    interp->tstate_head = NULL;
    interp->modules = NULL;
    interp->modules_reloading = 0;
    interp->sysdict = NULL;
    
    // Создаём PyThreadState для этого процесса
    PyThreadState *new_tstate = PyThreadState_New(interp);
    PyThreadState_Swap(new_tstate);
    
    // Инициализация интерпретатора
    if (_PyInterpreterState_Init(interp) < 0) {
        PyInterpreterState_Delete(interp);
        return PyStatus_Err();
    }
    
    return PyStatus_OK();
}
```

Каждый процесс имеет **свой** `PyInterpreterState` + `PyThreadState`. **Никаких**
общих globals/modules/builtins. **Полная изоляция**.

## 9. Shared Memory (multiprocessing.shared_memory)

```c
// Modules/_multiprocessing/shm_posix.c
int shm_open(const char *name, int oflag, mode_t mode) {
    // Создаёт /dev/shm/name сегмент памяти
    return syscall(SYS_shm_open, name, oflag, mode);
}

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset) {
    // Отображает shm в адресное пространство процесса
}
```

`SharedMemory('name', size=1e6)` → **/dev/shm/name** файл → `mmap()` → **общая
физическая память**. **Байтовое** копирование между процессами.

## 10. multiprocessing.Manager() - proxy объекты

```python
class Server:
    def serve_forever(self):
        # Запускает сокет сервер
        while True:
            conn = self.listener.accept()
            t = threading.Thread(target=Dispatcher(conn))
            t.start()
```

`Manager().dict()` → **отдельный процесс-сервер** → **Unix socket** → **pickle
запросы** → **Python RPC**. **Автоматическая** сериализация.

**Мультипроцессинг** в CPython 3.9+ — **spawn/fork/forkserver**, **новый PyInterpreterState** в каждом процессе, **Pipe
** (`pickle` + Unix sockets), **sem_t/HANDLE** семафоры, **Pool** worker'ы, **Copy-on-Write** (`fork`), **SharedMemory
** (`mmap`).

- [Содержание](/CONTENTS.md#содержание)