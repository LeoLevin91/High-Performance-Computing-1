# High-Performance-Computing

## **Студент:** Петров Леонид
## **Группа:** 6133-010402D

***Pre.S. Из-за ошибки именования файла ноутбука он называется "Lab-1...", на самом деле он должен называться "Lab-0..."***

### Задание на Лабораторную работу №0:

***-> Задача:*** Реализовать алгоритм перемножения матриц

***-> Язык:*** C++ или Python

***-> Входные данные:*** 2 матрицы размером от 100х100 до 2000х2000 каждая.

***-> Выходные данные:*** проверка корректности перемножения + время вычисления

***-> Реализация должна содержать 2 функции перемножения матриц: на CPU и на GPU с применением CUDA.***

  Отчет о проделанной лабораторной работе - это git-репозиторий с исходным кодом реализации + описание проделанной работы там же в readme. Необходимо описать реализацию, объяснив, что конкретно было распараллелено и почему.

  Провести эксперименты: перемножить матрицы разных размеров, посчитать ускорение. Результаты привести в виде таблицы/графика.

## Оборудование для реализации

***Среда разработки:*** Google Colaboratory

***Язык реализации лабораторной:*** Python, C++

***Использованные библиотеки:***
  - pycuda
  - numpy
  - pandas
  - time

***Центральный процессор (CPU):*** На ресурсе Colab используются Intel(R) Xeon(R) CPU @ 2.30GHz. Данные об этом были получены после выполнения в отдельной ячейке следующего кода: ```!cat /proc/cpuinfo```

***Графический процессор (GPU):*** На ресурсе Colab используется Tesla K80. Данные об этом были получены после выполнения в отдельной ячейке следующей команды: ```!nvidia-smi```

## Описание выполнения лабораторной работы
1. Импорт библиотек указаных выше в данном README;
2. Создание двух массивов:
    - arrayGPUTime - запись времени выполениня на GPU;
    - arrayCPUTime - запись времени выполнения на CPU.
3. Создание python переменной с именем ***matrixMulFun*** в которой содержится строка, содержащая C++ код, который и будет выполняться на девайсе. Сама функция, выполняемая в ядре, носит название ***MatrixMulKernel***;
4. Функция python с именем ***GPU_matrix_mul*** которая осуществляет следующие действия:
    - Выделение памяти на девайсе для матриц ***A*** и ***B*** с помощью команды: ```cuda.mem_alloc(<matrix>.nbytes)```;
    - Выделение памяти для результирующей матрицы ***C*** и заполнение данной матрицы нулями;
    - Подсчёт количества нитей для выполнения на GPU. Данная операция выполняется путем точным расчета размерности матрицы и последующем покрытием её соответствующим количеством варпов (warp);
    - Начало замера времены выполнения с помощью команды: ```time.time()```. Стоит заметить, что в замер времени включен не только непосредственно сам расчет на девайсе, но также и пересылка данных с **хоста** на **девайс** и обратно;
    - Пересылка данных исходных матриц **A** и **B** на девайс осуществляется командой: ```memcpy_htod(<allocatedMemory>, <matrix>)```;
    - После чего происходит компиляция ядра командой: ```SourceModule(<NameVar>)```, где ```NameVar``` - имя строковой переменной, внутри которой лежит C++ код ядра (код выполняемый на девайсе);
    - Загружаем функцию, которая выполнит C++ код на девайсе с помощью команды: ```get_function(<NameCPPFunc>)```;
    - Вызываем C++ функцию и передаем ей необходимые параметры;
    - Выполняем пересылку данных с ***девайса*** на ***хост*** с помощью команды: ```memcpy_dtoh(<resultMatrix>,<allocatedMemoryResultMatrix>)```;
    - Останавливаем замер времени;
    - Добавляем в массив ***arrayGPUTime*** данные о замере времени (разность между концом замера времени и началом замера времени).
5. Функция python ***launch***. Синопсис данной функции в том, что она производит запуск программы, а именно она делает следующие:
    -  Создание основного цикла программы. Цикл for перебирает все значения из следуюего массива: ***[128, 256, 512, 1024]*** - где каждый элемент массива, это размерность матрицы;
    -  После чего создаются матрицы ***A*** и ***B*** и заполняются псевдослучайными значениями, с помощью следующей команды: ```np.random.randn(dim,dim).astype(np.float32)``` из библиотеки NumPy. Стоит заметить, что данные в таблице, для допустимости расчетов, должны иметь тип ***float32***;
    -  После чего запускается последовательно два цикла. Каждый цикл имеет 11 итераций. Результаты выполнения работы цикла беруться только за 10 итераций. В данном случае учитывается так называемый "холодный старт";
    -  В первом цикле происходит вызов метода ***GPU_matrix_mul*** для перемножения матриц на GPU (данная функция была описана в пункте 4);
    -  Во втором цикле вызывается функция ```np.dot(<matrixA>, <matrixB>)```. Данная функция из библиоткеи NumPy выполняет перемножение матриц;
    -  После выполнения двух циклов происходит проверка вычислений. Данная проверка производится с помощью функции ```np.allclose(resultGPU, resultCPU, atol=0.0001)``` - где, **resultGPU** - результат выполнения перемножения матриц на GPU, ***resultCPU*** - результат выполнения перемножения матриц на CPU, atol - точность сравнения (т.к. происходит сравнение типов float).
6. Графики были построенны с помощью библиотеки ***matplotlib***.

## Графики
![Comparison of working hours](https://github.com/LeoLevin91/High-Performance-Computing/blob/master/Images/1.PNG?raw=true)
![Comparison of working hours](https://github.com/LeoLevin91/High-Performance-Computing/blob/master/Images/2.PNG?raw=true)

## Таблица
|               | CPU | GPU | 
| ------------- | --- | --- |
| 124  | 0.000256	  | 0.001019  |
| 256  | 0.000502  | 0.001385  |
| 512  | 0.002012  | 0.002646  |
| 1024  | 0.010232  | 0.008826  |

## Результат
  Как видно из графиков и таблицы, на матрицах малой размерности CUDA уступает в скорости расчета CPU. Данный спад вызван следующим. При расчете времени работы GPU учитывалась так же пересылка данных с **хоста** на **девайс** и обратно с **девайса** на **хост**. При большой размернисти матрицы, вычисления на GPU смогли производить расчет значительно быстрее, не смотря на выше предсвленную пересылку матриц на **девайс** и обратно. Так же, на большей размерности матриц CUDA ускоряет вычисления, за счет использования большего количества нитей, что так же дает прирост скорости вычислений.
