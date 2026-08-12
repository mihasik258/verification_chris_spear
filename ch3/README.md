# SystemVerilog for Verification (3rd ed.) — Глава 3
## Procedural Statements and Routines — решения задач

---

## 1. Создать код SystemVerilog со следующими требованиями:
- a. Массив из 512 элементов типа `integer`.
- b. 9-битная переменная-адрес для индексации массива.
- c. Инициализировать последний элемент массива значением 5.
- d. Вызвать задачу `my_task()`, передав массив и адрес.
- e. Создать `my_task()` с двумя входами: константный массив из 512 элементов, переданный по ссылке, и 9-битный адрес. Задача вызывает функцию `print_int()`, передавая элемент массива по индексу-адресу с предекрементом адреса.
- f. Создать `print_int()`, печатающую время симуляции и значение входа. Функция без возвращаемого значения.

```
module array_task_passing;
  // a. 
  int memory_array[512];
  // b. 
  bit [8:0] array_address;
  // f.
  function void print_int(input int array_element);
    $display("@%0t: element value = %0d", $time, array_element);
  endfunction
  // e. 
  task automatic my_task(const ref int local_array[512],
                         input bit [8:0] local_address);
    print_int(local_array[--local_address]);
  endtask

  initial begin
    // c.
    memory_array[511] = 5;
    array_address = 0;
    // d.
    my_task(memory_array, array_address);
  end
endmodule
```
## 2. Что будет выведено, если задача `my_task2()` объявлена как `automatic`?

```
int new_address1, new_address2;
bit clk;

initial begin
  fork
    my_task2(21, new_address1);
    my_task2(20, new_address2);
  join
  $display("new_address1 = %0d", new_address1);
  $display("new_address2 = %0d", new_address2);
end

initial
  forever #50 clk = !clk;

task my_task2(input int address, output int new_address);
  @(clk);
  new_address = address;
endtask
```

При `automatic` симулятор размещает локальные переменные и аргументы на стеке, поэтому каждый параллельный вызов из `fork` получает независимую копию `address` и `new_address`. Потоки не мешают друг другу:

```
new_address1 = 21
new_address2 = 20
```

---

## 3. Тот же код, но задача `my_task2()` **не** является `automatic`.

По умолчанию используется статическая память, поэтому формальные аргументы `address` и `new_address` — общие для всех вызовов. Второй вызов `my_task2(20, new_address2)` перезаписывает статический `address` значением `20` ещё до того, как первый вызов пройдёт задержку `@(clk)`. На фронте `clk` оба потока присваивают выходам значение `20`:

```
new_address1 = 20
new_address2 = 20
```


## 4. Задать вывод времени в пикосекундах (ps), 2 знака после десятичной точки, минимальное число символов.

Системная функция `$timeformat` принимает четыре аргумента: масштабный коэффициент, число цифр после запятой, строку-суффикс и минимальную ширину поля. Для пикосекунд коэффициент равен `-12`, минимальная ширина `0` даёт «минимум символов»:

```
$timeformat(-12, 2, "ps", 0);
```

## 5.  Используя форматирование из задачи 4, что выведет код?

```
timeunit 1ns;
timeprecision 1ps;
parameter real t_real = 5.5;
parameter time t_time = 5ns;

initial begin
  #t_time $display("1 %t", $realtime);
  #t_real $display("1 %t", $realtime);
  #t_time $display("1 %t", $realtime);
  #t_real $display("1 %t", $realtime);
end

initial begin
  #t_time $display("2 %t", $time);
  #t_real $display("2 %t", $time);
  #t_time $display("2 %t", $time);
  #t_real $display("2 %t", $time);
end
```


`$realtime` возвращает вещественное число и сохраняет дробную часть;
`$time` возвращает целое, округляя к ближайшему (10.5 → 11, 15.5 → 16), а не отбрасывая дробь.

**Блок 1 (`$realtime`):**
```
1 5000.00ps
1 10500.00ps
1 15500.00ps
1 21000.00ps
```

**Блок 2 (`$time`):**
```
2 5000.00ps    // 5.0  -> 5
2 11000.00ps   // 10.5 -> 11
2 16000.00ps   // 15.5 -> 16
2 21000.00ps   // 21.0 -> 21
```
