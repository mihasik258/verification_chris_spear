## 1. Дан следующий фрагмент кода:

```
byte my_byte;
integer my_integer;
int my_int;
bit [15:0] my_bit;
shortint my_short_int1;
shortint my_short_int2;

my_integer = 32'b0000_1111_xxXX_ZZZZ;
my_int = my_integer;
my_bit = 16'h8000;
my_short_int1 = my_bit;
my_short_int2 = my_short_int1 - 1;
```

Ответьте на вопросы:

a. Какой диапазон значений может принимать переменная `my_byte`?

b. Какое значение будет у `my_int` в шестнадцатеричном формате?

c. Какое значение будет у `my_bit` в десятичной системе?

d. Какое значение будет у `my_short_int1` в десятичной системе?

e. Какое значение будет у `my_short_int2` в десятичной системе?

**a.** Тип `byte` - это 8-битное знаковое целое, поэтому диапазон значений составляет от -128 до 127.

**b.** При присваивании значения переменной типа `int` все `X` и `Z` заменяются на `0`, поэтому получится 32'h00000F00.

**c.** Тип `bit` беззнаковый => `16'h8000` = 32768.

**d.** После присваивания в `shortint` число интерпретируется как знаковое. Значение `16'h8000` соответствует **-32768**.

**e.** Переполнение знакового 16-битного числа => результат c другой стороны интервала - 32767.

## 2. Дан следующий фрагмент кода:

```
bit [7:0] my_mem [3];
logic [3:0] my_logicmem [4];
logic [3:0] my_logic;

my_mem = '{default:8'hA5};
my_logicmem = '{0,1,2,3};
my_logic = 4'hF;
```
Выполните приведённые ниже операции в указанном порядке и определите результат каждого присваивания.
```
a. my_mem[2] = my_logicmem[4];
b. my_logic = my_logicmem[4];
c. my_logicmem[3] = my_mem[3];
d. my_mem[3] = my_logic;
e. my_logic = my_logicmem[1];
f. my_logic = my_mem[1];
g. my_logic = my_logicmem[my_logicmem[4]];
```

**a.** Индекс `4` выходит за границы массива `my_logicmem`, поэтому при чтении получается `X`. При записи в тип `bit` значение `X` преобразуется в `0`. Итог: **8'h00**.

**b.** Чтение по несуществующему индексу массива типа `logic` возвращает 4'bX.

**c.** Индекс `3` выходит за границы массива `my_mem`, поэтому возвращается значение по умолчанию - 4'h0.

**d.** Запись за пределы массива не выполняется, содержимое массива не изменится.

**e.** Элемент `my_logicmem[1]` содержит значение 4'h1.

**f.** В `my_mem[1]` хранится `8'hA5`. При присваивании в 4-битную переменную остаются младшие четыре бита, поэтому получится 4'h5.

**g.** Выражение `my_logicmem[4]` возвращает `X`. Использование `X` в качестве индекса снова приводит к обращению вне массива, поэтому результат будет 4'bX.

## 3. Напишите код на SystemVerilog, который:

a. Объявляет 2-state массив `my_array`, состоящий из четырёх 12-битных элементов.

b. Инициализирует массив следующими значениями:

- `my_array[0] = 12'h012`
- `my_array[1] = 12'h345`
- `my_array[2] = 12'h678`
- `my_array[3] = 12'h9AB`

c. Проходит по массиву и выводит биты `[5:4]` каждого элемента:

- с помощью цикла `for`;
- с помощью цикла `foreach`.

```
module array_traversal;

  bit [11:0] my_array [4];

  initial begin
    my_array[0] = 12'h012;
    my_array[1] = 12'h345;
    my_array[2] = 12'h678;
    my_array[3] = 12'h9AB;

    for (int i = 0; i < $size(my_array); i++) begin
      $display("for: index = %0d, bits[5:4] = %b",
               i, my_array[i][5:4]);
    end

    foreach (my_array[i]) begin
      $display("foreach: index = %0d, bits[5:4] = %b",
               i, my_array[i][5:4]);
    end
  end

endmodule
```

## 4. Объявлен многомерный распакованный массив размером 5 на 31, my_array1. 
Каждый элемент распакованного массива содержит 4-состоятельное значение.

a. Какие из следующих операторов присваивания являются допустимыми и не выходят за пределы массива?
- my_array1 '[4]' [30] = 1'b1;  
- my_array1 '[29]' [4] = 1'b1;  
- my_array1 [4] = 32'b1;

b. Нарисуйте массив my_array1 после завершения всех допустимых присваиваний.

logic unpackedMultiArray '[5]'[31]
*a.*
- '[4]'[30] = 1'b1; - Индексы от 0 до 4 и от 0 до 30 допустимы.
- '[29]'[4] = 1'b1; -  Левая размерность ограничена от 0 до 4.
- '[4]' = 32'b1; - Присвоение упакованного 32-битного вектора напрямую распакованному массиву невозможно.

*b*. Это структура памяти, состоящая из 5 независимых одномерных массивов, каждый из которых содержит 31 элемент типа logic. Элемент с индексом '[4]'[30] будет содержать 1, все остальные элементы останутся со значением X для 4-значного типа.

## 5. Объявите многомерный упакованный массив размером 5 на 31, my_array2.
Каждый элемент упакованного массива содержит значение в двух состояниях.
a. Какие из следующих инструкций о присваивании являются допустимыми и не выходят за рамки допустимого?
- my_array2 '[4]' [30] = 1'b1;
- my_array2 '[29]' [4] = 1 'b1;
- my_array2 [3] = 32'b1;
b. Нарисуйте my_array2 после завершения выполнения инструкций присваивания.

bit '[4:0]'[30:0] packedMultiArray

*a*.
- '[4]'[30] = 1'b1; - ок.
- '[29]'[4] = 1'b1; - выход за границы 0-4.
- '[3]' = 32'b1; - Запись 32-битного значения в  срез размером 31 бит. Старший бит будет отсечен.

*b*. Массив размещается в памяти как единый вектор из 155 бит. Значения: старший срез с индексом 4 содержит 1 в старшем 30-м бите. Срез с индексом 3 содержит 31'b1 (младший бит  1, остальные 0). Все остальные биты равны 0.

## 6. Используя следующий код, определите, что будет отображаться.
module test;  
string street[$];  
initial begin  
street = {"Tejon", "Bijou", "Boulder"};  
$display("Street [0] = %s", street[0]);  
street.insert(2, "Platte");  
$display("Street [2] = %s", street [2]);  
street.push_front("St. Vrain");  
$display("Street [2] = %s", street [2]);  
$display("pop_back = %s", street.pop_back);  
$display("street.size = %d", street.size);  
end  
endmodule // test

answer 

Street [0] = Tejon
Street [2] = Platte
Street [2] = Bijou
pop_back = Boulder
street.size =           4

## 7. Напишите код для решения следующих задач.
a. Создайте память, используя ассоциативный массив для процессора с шириной слова в 24 бита и адресным пространством в 2^20 слов. Предположим, что компьютер запускается с адреса 0 при
сбросе. Пространство программы начинается с 0x400. Значение ISR соответствует максимальному адресу.
b. Заполните память следующими инструкциями:
- 24'hA50400; // Перейдите к местоположению 0x400 для основного кода.
- 24'h123456; // Инструкция 1 находится в ячейке 0x400
- 24 'h789ABC; // Инструкция 2 находится в ячейке 0x401
- 24 'h0F1E2D; // ISR Возвращается из прерывания
c. Выведите элементы и их количество в массиве.

```
module memory_model;  
  // a.  
  bit [23:0] processor_associative_memory [bit [19:0]];  
   
  initial begin  
    // b.  
    processor_associative_memory[20'h00000] = 24'hA50400;  
    processor_associative_memory[20'h00400] = 24'h123456;  
    processor_associative_memory[20'h00401] = 24'h789ABC;  
    processor_associative_memory[20'hFFFFF] = 24'h0F1E2D;  
  
    // c.  
    foreach (processor_associative_memory[memory-address]) begin  
      $display("Address 0x%0h: 0x%0h", memory_address, processor_associative_memory[memory_address]);  
    end  
    $display("Number of allocated elements: %0d", processor_associative_memory.num());  
  end  
endmodule
```
## 8. Создайте код SystemVerilog, соответствующий следующим требованиям
a. Создайте 3-байтовую очередь и инициализируйте ее значениями 2, -1 и 127
b. Выведите сумму в десятичной системе счисления для очереди
c. Выведите минимальное и максимальное значения в очереди
d. Отсортируйте все значения в очереди и распечатайте результирующую очередь
e. Распечатайте индекс всех отрицательных значений в очереди
f. Распечатайте положительные значения в очереди
g. Выполните обратную сортировку всех значений в очереди и распечатайте результирующую очередь

```
module queue_operations;
  // a.
  byte signed_values_queue [$] = {2, -1, 127};

  int  calculated_sum;
  byte temporary_values_queue [$];
  int  temporary_indices_queue [$];

  initial begin
    // b. 
    calculated_sum = signed_values_queue.sum() with (int'(item));
    $display("Sum of queue: %0d", calculated_sum);

    // c.
    temporary_values_queue = signed_values_queue.min();
    $display("Min value: %0d", temporary_values_queue[0]);
    temporary_values_queue = signed_values_queue.max();
    $display("Max value: %0d", temporary_values_queue[0]);

    // d.
    signed_values_queue.sort();
    $display("Sorted queue: %p", signed_values_queue);

    // e.
    temporary_indices_queue = signed_values_queue.find_index() with (item < 0);
    $display("Indices of negative values: %p", temporary_indices_queue);

    // f.
    temporary_values_queue = signed_values_queue.find() with (item > 0);
    $display("Positive values: %p", temporary_values_queue);

    // g.
    signed_values_queue.rsort();
    $display("Reverse sorted queue: %p", signed_values_queue);

    $finish;
  end
endmodule
```

PS `min`/`max` в Verilator сравниваются как беззнаковые (поэтому Min=2, Max=−1),
## 9. Определите пользовательский 7-разрядный тип
и инкапсулируйте поля следующегопакета в структуру, используя свой новый тип. Наконец, присвойте заголовку значение 7 'h5A.
27 21 20 14 13 7 6 0
header  cmd     data   crc
```
module network_packet;
  typedef bit [6:0] field_t;

  typedef struct packed {
    field_t header;
    field_t cmd;
    field_t data;
    field_t crc;
  } packet_t;

  packet_t packet;

  initial begin
    packet.header = 7'h5A;
  end

endmodule
```

## 10. Напишите код на SystemVerilog, который:
a. Создаёт пользовательский 4-битный тип `nibble`.
b. Создаёт переменную типа `real` со значением `4.33`.
c. Создаёт переменную типа `shortint` с именем `i_pack`.
d. Создаёт распакованный массив из четырёх элементов типа `nibble` и инициализирует его значениями `4'h0`, `4'hF`, `4'hE` и `4'hD`.
e. Выводит содержимое массива.
f. Выполняет потоковую упаковку массива в `i_pack` справа налево побитно и выводит результат.
g. Выполняет потоковую упаковку справа налево по тетрадам и выводит результат.
h. Преобразует значение переменной `real` к типу `nibble`, записывает его в `k[0]` и выводит массив.
```
module streaming_operator_demo;
  // a.
  typedef bit [3:0] nibble;
  // b.
  real r = 4.33;
  // c.
  shortint i_pack;
  // d.
  nibble k [4] = '{ 4'h0, 4'hF, 4'hE, 4'hD };

  initial begin
    // e.
    $display("k = %p", k);
    // f.
    i_pack = {<<{k}};
    $display("Bit stream    = %h", i_pack);
    // g. 
    i_pack = {<<nibble{k}};
    $display("Nibble stream = %h", i_pack);
    // h.
    k[0] = nibble'(r);
    $display("k = %p", k);
  end
endmodule
```
## 11. ALU имеет коды операций, указанные в таблице 2.1.
Таблица 2.1 Коды операций ALU
Кодировка кода операции
ADD: A + B 2'b00
SUB: A - B 2 'b01
Bit-wise invert: A 2'b10
Reduction or: B 2 'b11
Напишите тестовый стенд, который выполняет следующие задачи.
a. Создайте перечисляемый тип кодов операций: opcode_e
b. Создайте переменную opcode типа opcode_e
c. Перебирайте все значения переменной opcode каждые 10 секунд.
d. Создайте экземпляр ALU с одним 2-разрядным входным кодом операции.

```
module alu (
  input  logic [1:0] opcode,
  input  logic [7:0] a,
  input  logic [7:0] b,
  output logic [7:0] result
);
  always_comb begin
    case (opcode)
      2'b00: result = a + b;        // Add
      2'b01: result = a - b;        // Sub
      2'b10: result = ~a;           // Bit-wise invert A
      2'b11: result = {7'b0, |b};   // Reduction OR of B
    endcase
  end
endmodule

module test;
  // a.
  typedef enum bit [1:0] { ADD = 2'b00, SUB = 2'b01,
                           INV = 2'b10, RED = 2'b11 } opcode_e;
  // b.
  opcode_e opcode;
  logic [7:0] a;
  logic [7:0] b;
  logic [7:0] result;
  logic [7:0] a_vals [3] = '{ 8'h0C, 8'hFF, 8'h55 };
  logic [7:0] b_vals [3] = '{ 8'h3C, 8'h00, 8'h0B };
  // d.
  alu dut (.opcode(opcode), .a(a), .b(b), .result(result));
  // c.
  initial begin
    foreach (a_vals[j]) begin
      a = a_vals[j];
      b = b_vals[j];
      opcode = opcode.first();
      repeat (opcode.num()) begin
        #10;
        $display("t=%0t  opcode=%b  a=%h b=%h -> result=%h",
                 $time, opcode, a, b, result);
        opcode = opcode.next();
      end
    end
    $finish;
  end
endmodule
```

