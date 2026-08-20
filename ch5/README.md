## 1. класс MemTrans с полями и print  
Написать класс `MemTrans`, содержащий набор полей, и создать объект внутри блока `initial`: (a) 8-битная переменная `data_in` с типом `logic`; (b) 4-битная переменная `address` с типом `logic`; (c) функция `print` (тип void) для вывода значений `data_in` и `address` в консоль.  
  
```  
class MemTrans;  
  logic [7:0] data_in;   // a  
  logic [3:0] address;   // b  
  
  function void print;   // c  
    $display("\%0t: data_in = \%h, address = \%h", $time, data_in, address);  
  endfunction  
endclass  
  
program automatic test;  
  MemTrans mt;  
  initial begin  
    mt = new();  
    mt.print();
  end  
endprogram  
```


## 2. свой конструктор, инициализация в 0
Дополнить код конструктором (new), который принудительно обнуляет значения переменных data_in и address.
```
function new();  
  data_in = 0;  
  address = 0;  
endfunction  
```  

## 3. конструктор с аргументами (по умолчанию 0)
Конструктор должен обнулять обе переменные, однако позволять перезаписывать их через входящие аргументы. Написать скрипт: (a) создать два объекта класса; (b) в первом экземпляре присвоить address=2, передав параметр по имени; (c) во втором экземпляре передать data_in=3 и address=4, также используя именованные аргументы.


```
function new(logic [7:0] data_in = 0, logic [3:0] address = 0);
  this.data_in = data_in;
  this.address = address;
endfunction
```
  

```
program automatic test;
  MemTrans m1, m2;
  initial begin
    m1 = new(.address(4'd2));  // b
    m2 = new(.data_in(8'd3), .address(4'd4));  // c
  end
endprogram
```
  

## 4. доработать программу из Ex3
(a) Сразу после конструирования изменить параметр address первого объекта на 4'hF; (b) с помощью функции print отобразить статусы обоих экземпляров; (c) явно удалить второй объект из памяти.
    

```
program automatic test;
  MemTrans m1, m2;
  initial begin
    m1 = new(.address(4'd2));
    m2 = new(.data_in(8'd3), .address(4'd4));
    m1.address = 4'hF;   // a
    m1.print();          // b
    m2.print();          // b
    m2 = null;           // c
  end
endprogram
```
  

handle = null удаляет последнюю ссылку, после чего сборщик мусора очищает объект.

## 5. статическая переменная last_address
Интегрировать статическую переменную last_address, предназначенную для сохранения стартового значения address самого свежего объекта (переданного в процессе конструирования). Сразу после выделения памяти (из задания 4) распечатать текущий статус last_address.

```
class MemTrans;
  logic [7:0] data_in;
  logic [3:0] address;
  static logic [3:0] last_address;
  
  function new(logic [7:0] data_in = 0, logic [3:0] address = 0);
    this.data_in = data_in;
    this.address = address;
    last_address = address;
  endfunction
  // ... print()
endclass
```
  

```
$display("last_address = %h", MemTrans::last_address);  // -> 4
``` 

## 6. статический метод print_last_address
Разработать статический метод print_last_address для вывода переменной last_address. Активировать его сразу после выделения памяти.


```
static function void print_last_address();
  $display("%0t: last_address = %h", $time, last_address);
endfunction
```

```
MemTrans::print_last_address();
```

Финальный объединенный класс (включает код Ex1-6):

```
class MemTrans;
  logic [7:0] data_in;
  logic [3:0] address;
  static logic [3:0] last_address;

  function new(logic [7:0] data_in = 0, logic [3:0] address = 0);
    this.data_in = data_in;
    this.address = address;
    last_address = address;
  endfunction

  function void print();
    $display("%0t: data_in = %h, address = %h", $time, data_in, address);
  endfunction

  static function void print_last_address();
    $display("%0t: last_address = %h", $time, last_address);
  endfunction
endclass
```

## 7. заполнить print_all через PrintUtilities
Расширить функционал метода print_all внутри MemTrans, переключив вывод значений data_in и address на использование внешнего класса PrintUtilities.

```
function void print_all;
  print.print_8("data_in", data_in);
  print.print_4("address", address);
endfunction
```
  
Практический пример:

```
program automatic test;
  MemTrans mt;
  initial begin
    mt = new();
    mt.data_in = 8'hA5;
    mt.address = 4'hC;
    mt.print_all();
  end
endprogram
```

## 8. массив хэндлов, генератор, передача
Вставить необходимый код в местах, помеченных символами //.

```
program automatic test;
  import my_package::*;
  
  initial begin
    Transaction tr[5];
    generator(tr);
  end

  task generator(ref Transaction tr[5]);
    foreach (tr[i]) begin
      tr[i] = new();
      transmit(tr[i]);
    end
  endtask

  task transmit(Transaction tr);
    .......
  endtask : transmit
endprogram
```
  
(1) массив требуется передавать через ref иначе сгенерированные сущности не вернутся в точку вызова; (2) метод new() необходимо поместить внутрь цикла
## 9. функция глубокого копирования copy
Написать реализацию функции copy для предложенного класса и показать ее работу в коде. Вспомогательный класс Statistics уже снабжен встроенным методом copy.

```
package automatic my_package;
  class MemTrans;
    bit [7:0] data_in;
    bit [3:0] address;
    Statistics stats;
    function new();
      data_in = 3;
      address = 5;
      stats = new();
    endfunction

    function MemTrans copy();
      copy = new();
      copy.data_in = data_in;
      copy.address = address;
      copy.stats   = stats.copy();
    endfunction
  endclass
endpackage
```
  
Проверка автономности скопированного объекта:

```
MemTrans src, dst;
src = new();
src.stats.start_time = 100;
dst = src.copy();
dst.stats.start_time = 999;
// src.stats.start_time остаётся 100 => stats независимы
```
