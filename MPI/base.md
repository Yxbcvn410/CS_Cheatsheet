# MPI

## Начало работы

Всё необходимое в файле `mpi.h`.

Для начала работы необходимо вызвать функцию [MPI_Init](https://www.mpich.org/static/docs/v3.3/www3/MPI_Init.html):

```c++
int MPI_Init(int *argc, char ***argv)
```

Этой функции необходимо передать аргументы из командной строки (`argc` и `argv`), а возвращает она код ошибки - 0 в случае успеха, что-то ещё иначе.

В конце работы нужно вызвать функцию [MPI_Finalize](https://www.mpich.org/static/docs/v3.3/www3/MPI_Finalize.html):

```c++
void MPI_Finalize()
```

## Процессы в MPI

В MPI мы работаем с __множеством связанных процессов__, имеющим тип `MPI_Comm`. Также это называют __комммуникационное множество__.
Так, множество всех процессов - `MPI_COMM_WORLD`.

У каждого процесса есть сникальный для коммуникационного множества номер, или __ранг__.
Его можно узнать с помощью функции [MPI_Comm_rank](https://www.mpich.org/static/docs/v3.3/www3/MPI_Comm_rank.html):

```c++
int MPI_Comm_rank(MPI_Comm comm, int *rank)
```

__Важно__. Эта функция возвращает код ошибки. Чтобы узнать ранг процесса, нужно передать указатель на переменную в качестве параметра `rank`.

Также есть функция [MPI_Comm_size](https://www.mpich.org/static/docs/v3.3/www3/MPI_Comm_size.html),
позволяющая узнать размер коммуникационного множества:

```c++
int MPI_Comm_size(MPI_Comm comm, int *size)
```

__Важно__. Как и предыдущая функция, эта возвращает код ошибки. Ранг процесса будет записан в `*size`.

## Передача сообщений

Есть две функции - отправка и приём сообщения соответственно.

### Отправка

Отправка сообщения происходит с помощью функции [MPI_Send](https://www.mpich.org/static/docs/v3.3/www3/MPI_Send.html):

```c++
MPI_Send (void *buf, int count, MPI_Datatype datatype, int dest, int tag, MPI_Comm comm)
```

Параметры: 
* `buf` и `count` - указатель на данные сообщения и количество элементов (**а не байт!**)
* `datatype` - тип данных сообщения ( `MPI_CHAR`, `MPI_INT`, `MPI_DOUBLE`, ... )
* `dest` и `comm` - ранг потока-адресата и его коммуникационное множество
* `tag` - тег сообщения (произвольное число)

__Важно__. Эта функция приостанавливает выполнение до тех пор, пока сообщение не будет прочитано.


### Приём

Для приёма используем функцию [MPI_Recv](https://www.mpich.org/static/docs/v3.2/www3/MPI_Recv.html):

```c++
int MPI_Recv(void *buf, int count, MPI_Datatype datatype, int source, 
	int tag, MPI_Comm comm, MPI_Status *status)
```

Параметры: 
* `buf` и `count` - указатель на данные сообщения и количество элементов (**а не байт!**)
* `datatype` - тип данных сообщения ( `MPI_CHAR`, `MPI_INT`, `MPI_DOUBLE`, ... )
* `source` и `comm` - ранг потока-отправителя и его коммуникационное множество
* `tag` - тег сообщения (произвольное число)
* `status` - объект статуса - содержит информацию о потоке-отправителе, теге и пр.

__Важно__. Эта функция приостанавливает выполнение до тех пор, пока не будет принято сообщение с нужным тегом от нужного адресата.
Можно задать параметру `source` значение `MPI_ANY_SOURCE`, если поток-отправитель не важен.
То же самое делает значение `MPI_ANY_TAG` для тега и `MPI_STATUS_IGNORE` для статуса.

## Компиляция и запуск

Вызов компилятора: `mpicc` или `mpic++`, аргументы как у `gcc`.

Для того, чтобы запустить собранную программу, используем следующий скрипт:

```bash
#!/bin/bash

#PBS -l walltime=00:01:00,nodes=7:ppn=2
# Заказали время для задачи: 1 минута, 7 узлов по 2 ядра каждый

#PBS -N example_job
# Выбрали имя для задачи

#PBS -q batch

cd $PBS_O_WORKDIR

mpirun --hostfile $PBS_NODEFILE -np 14 ./a.out
#np - количество процессов	
```

Этот код надо сохранить в файл с расширением `*.sh` - скрипт. Например, `start.sh`.

Чтобы запустить, наконец-то, программу, нужно ввести `qsub start.sh`. Таким образом мы поставим программу в очередь на исполнение.
При этом на экран выведется имя, присвоенное задаче, начинающееся с 6 цифр.

Когда вычисления закончатся, вывод программы будет в файлах с именем типа `example_job.o******`, а ошибки - в файлах типа `example_job.e******`, где `******` - те самые 6 цифр.

----

[На главную](../Readme.md)