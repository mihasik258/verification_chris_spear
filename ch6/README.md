## 1. класс Exercise1 с constraint на address  
(a) Написать класс, включающий две `rand`-переменные: 8-битную `data` и 4-битную `address`; создать constraint, ограничивающий `address` значениями 3 или 4. (b) В блоке `initial` создать экземпляр объекта, вызвать метод рандомизации и проконтролировать код возврата.  
  
``` 
class Exercise1;  
  rand bit [7:0] data;                            // a  
  rand bit [3:0] address;                         // a  
  constraint c_addr { address inside {3, 4}; }    // address == 3 или 4  
endclass  
  
program automatic test;  
  Exercise1 ex;  
  initial begin  
    ex = new();  
    if (!ex.randomize())                          // b
      $error("randomize failed");  
    else  
      $display("data=%0d address=%0d", ex.data, ex.address);  
  end  
endprogram  
```

## 2. взвешенное распределение (dist)
Изменить код в Exercise2 следующим образом: (a) `data` всегда = 5; (b) P(`address`= =0)=10%; (c) P(`address`∈[1:14])=80%; (d) P(`address`= =15)=10%.

```
class Exercise2;  
  rand bit [7:0] data;  
  rand bit [3:0] address;  
  constraint c_data { data == 5; } // a
  constraint c_addr { address dist { 0 := 10, :/ 80, 15 := 10 };// b,c,d
endclass 
```

Использование [1:14] := 80 привело бы к тому, что каждому из 14 чисел достался бы вес 80.

## 3. 20 генераций с проверкой
На базе Exercise 1 или Exercise 2 продемонстрировать генерацию 20 пар data/address, проверяя успешность работы солвера на каждой итерации.

```
program automatic test;
  Exercise2 ex;
  initial begin
    ex = new();
    repeat (20) begin
      if (!ex.randomize()) $error("randomize failed");
      else $display("data=%0d address=%0d", ex.data, ex.address);
    end
  end
endprogram
```

## 4. 1000 генераций + гистограмма
Тестовое окружение должно рандомизировать Exercise2 тысячу раз. (a) Вычислить частоту выпадения каждого address и распечатать гистограмму; достигается ли точная пропорция 10/80/10? Объяснить причину. (b) Выполнить симуляцию с тремя различными seed-значениями и сопоставить результаты.

```
program automatic test;
  Exercise2 ex;
  int hist[16];
  initial begin
    ex = new();
    repeat (1000) begin
      if (!ex.randomize()) $error("randomize failed");
      hist[ex.address]++;
    end
    foreach (hist[i])
      $display("address %2d : %0d", i, hist[i]);
  end
endprogram
```
  

(a) Рандомизация с ограничениями выполняет выборку из распределения, поэтому тысяча попыток выдает приблизительную пропорцию 10/80/10 со статистической погрешностью порядка √N. Точные цифры достижимы только при бесконечном числе итераций. Кроме того, 80% делятся на 14 чисел на каждое приходится около 5.7%.

(b) Применение различных seed-значений генерирует отличающиеся наборы данных (разные ветки PRNG), однако соблюдают паттерн 10/80/10. Способы передачи seed: VCS simv +ntb_random_seed=42, IUS irun … -svseed 42, Questa  vsim -sv_seed 42.

## 5. описать constraints в Sample 6.4
```
class Stim;
  const bit [31:0] CONGEST_ADDR = 42;
  typedef enum {READ, WRITE, CONTROL} stim_e;
  randc stim_e kind;
  rand bit [31:0] len, src, dst;
  rand bit congestion_test;
  constraint c_stim {
    len < 1000;
    len > 0;
    if (congestion_test) {
      dst inside {[CONGEST_ADDR-10:CONGEST_ADDR+10]};
      src == CONGEST_ADDR;
    }
    else
      src inside {0, [2:10], [100:107]};
  }
endclass
```

Интерпретация ограничений:
- len > 0 и len < 1000,.
- dst ограничивается в случае congestion_test == 1:  dst  [32, 52]. 
- src привязан к флагу congestion_test:
- congestion_test == 1: src == 42 (CONGEST_ADDR);
- congestion_test == 0: src ∈ {0} ∪ [2:10] ∪ [100:107].
## 6. заполнить Table 6.9 (solve … before)

```
class MemTrans;
  rand bit x;
  rand bit [1:0] y;
  constraint c_xy {
    y inside {[x:3]};
    solve x before y;
  }
endclass
```
  

solve x before: в первую очередь выбрать равновероятное значение x, затем подобрать равновероятный y из разрешенного диапазона для зафиксированного x.

- Переменная x принимает значения 0 или 1 P(x=0) = P(x=1) = 1/2
- x=0: множество y {0,1,2,3} содержит 4 элемента P=1/4
-  x=1: множество y  {1,2,3} содержит 3 элемента P=1/3

| Вариант | x   | y   | Вычисленная вероятность |
| ------- | --- | --- | ----------------------- |
| A       | 0   | 0   | 1/2 * 1/4 = 12.5%       |
| B       | 0   | 1   | 12.5%                   |
| C       | 0   | 2   | 12.5%                   |
| D       | 0   | 3   | 12.5%                   |
| E       | 1   | 0   | 0%                      |
| F       | 1   | 1   | 1/2  * 1/4 = 16.67%     |
| G       | 1   | 2   | = 16.67%                |
| H       | 1   | 3   | = 16.67%                |

## 7. implication + отключение constraint + in-line

```
class MemTrans;
  rand bit rw; // read rw=0, write rw=1
  rand bit [7:0] data_in;
  rand bit [3:0] address;
endclass
```
  
(a) Внедрить ограничение, фиксирующее адреса read-операций в рамках [0:7]. (b) Написать управляющий код: деактивировать встроенный constraint, создать объект и выполнить рандомизацию, применив in-line constraint, который расширяет адрес read-операций до диапазона [0:8]; верифицировать результат.

(a)

```
class MemTrans;
  rand bit rw;
  rand bit [7:0] data_in;
  rand bit [3:0] address;
  constraint read_addr { rw == 0 -> address inside {[0:7]}; }  // read -> адрес в [0:7]
endclass
```
  

(b)

```
program automatic test;
  MemTrans mt;
  initial begin
    mt = new();
    mt.read_addr.constraint_mode(0);   // отключаем зашитый constraint
    repeat (20) begin
      if (!mt.randomize() with { rw == 0 -> address inside {[0:8]}; })  // in-line
        $error("randomize failed");
      if (mt.rw == 0)
        assert (mt.address <= 8)
          else $error("read address %0d out of range", mt.address);
    end
  end
endprogram
```  
  

Логическая импликация rw == 0 -> address inside {…} тождественна условию !(rw= =0) || address inside {…} во время операций write ограничение на адрес снимается. Метод constraint_mode(0) отключает правило read_addr, а in-line блок with {…} накладывает новое условие исключительно на текущий вызов randomize().

## 8. картинка 10×10, ~20% белого
Создать класс, представляющий картинку размером 10×10 пикселей, где каждый элемент окрашен в черный или белый цвет. Сгенерировать матрицу, содержащую в среднем 20% белых точек, вывести ее в консоль и подсчитать итоговое количество пикселей обоих типов.

```
class Image;
  rand bit pixel[10][10];   // 1 white, 0 black
  constraint c_white { foreach (pixel[i,j]) pixel[i][j] dist { 1 := 20, 0 := 80 }; }
endclass

program automatic test;
  Image img;
  int white = 0;
  initial begin
    img = new();
    if (!img.randomize()) $error("randomize failed");
    foreach (img.pixel[i,j]) begin
      $write("%s", img.pixel[i][j] ? "#" : ".");
      if (img.pixel[i][j]) white++;
      if (j == 9) $display("");
    end
    $display("white=%0d black=%0d", white, 100 - white);
  end
endprogram
``` 
  

Исполняемая модель на базе $urandom 
  ...#......  
  .........#  
  ....#.#...  
  ..........  
  #.......#.  
  .......#..  
  ..#.......  
  .#........  
  .......#..  
  .#....#.#.  
white=13  black=87  (of 100)  
  

На выборке из 100 элементов статистический разброс довольно велик (√(100·0.2·0.8)≈4). Усреднение по множеству итераций даст искомые 20% белых точек.

## 9. StimData с массивом переменного размера
Спроектировать класс StimData, содержащий целочисленный массив сэмплов. Настроить рандомизацию длины и содержимого массива, зафиксировав размер в пределах от 1 до 1000. Провести тестирование, сгенерировав 20 пакетов и напечатав их итоговые размеры.

```
class StimData;
  rand int samples[];
  constraint c_size { samples.size() inside {[1:1000]}; }
endclass

program automatic test;
  StimData sd;
  initial begin
    sd = new();
    repeat (20) begin
      if (!sd.randomize()) $error("randomize failed");
      $display("size = %0d", sd.samples.size());
    end
  end
endprogram
```
  

## 10. back-to-back одного типа с разными адресами (одиночная транзакция)
Доработать класс Transaction таким образом, чтобы две подряд идущие транзакции одинакового типа не использовали идентичный адрес. Прогнать симуляцию на 20 итерациях.

```
package my_package;
  typedef enum {READ, WRITE} rw_e;
  class Transaction;
    rw_e       old_rw;
    bit [31:0] old_addr;
    rand rw_e  rw;
    rand bit [31:0] addr, data;

    constraint rw_c   { if (old_rw == WRITE) rw != WRITE; }
    constraint addr_c { if (rw == old_rw) addr != old_addr; }

    function void post_randomize;
      old_rw   = rw;
      old_addr = addr;
    endfunction

    function void print_all;
      $display("addr = %0d, data = %0d, rw = %s", addr, data, rw.name());
    endfunction
  endclass
endpackage

program automatic test;
  import my_package::*;
  Transaction t;
  initial begin
    t = new();
    repeat (20) begin
      if (!t.randomize()) $error("randomize failed");
      t.print_all();
    end
  end
endprogram
```


Логика addr_c: текущее значение rw (rand) сравнивается с old_rw. Если типы идентичны, применяется ограничение addr != old_addr.

## 11. то же для RandTransaction (массив объектов, решаемый целиком)
Добавить в класс RandTransaction правило, исключающее совпадение адресов у последовательных транзакций одного типа. Протестировать на массиве из 20 записей.

```
typedef enum {READ, WRITE} rw_e;
parameter int TESTS = 20;

class Transaction;
  rand rw_e rw;
  rand bit [31:0] addr, data;
endclass

class RandTransaction;
  rand Transaction trans_array[];

  constraint rw_c {
    foreach (trans_array[i])
      if ((i > 0) && (trans_array[i-1].rw == WRITE))
        trans_array[i].rw != WRITE;
  }

  constraint addr_c {
    foreach (trans_array[i])
      if ((i > 0) && (trans_array[i].rw == trans_array[i-1].rw))
        trans_array[i].addr != trans_array[i-1].addr;
  }

  function new();
    trans_array = new[TESTS];
    foreach (trans_array[i]) trans_array[i] = new();
  endfunction
endclass

program automatic test;
  RandTransaction rt;
  initial begin
    rt = new();
    if (!rt.randomize()) $error("randomize failed");
    foreach (rt.trans_array[i])
      $display("[%0d] rw=%s addr=%0d", i,
               rt.trans_array[i].rw.name(), rt.trans_array[i].addr);
  end
endprogram
```

группа TESTS транзакций вычисляется в рамках единого обращения к randomize(). Благодаря этому constraint способен анализировать смежные объекты trans_array[i-1] и trans_array[i] синхронно. 

цепочечное правило rw_c игнорируется при запуске в Vivado XSim 2019.1
