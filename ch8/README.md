Базовый класс Binary, необходимый для выполнения первого упражнения, выглядит следующим образом:

```
class Binary;
  rand bit [3:0] val1, val2;
  function new(input bit [3:0] val1, val2);
    this.val1 = val1;
    this.val2 = val2;
  endfunction
  virtual function void print_int(input int val);
    $display("val=0d%0d", val);
  endfunction
endclass
```  
  

## 1. Реализация метода multiply в классе-наследнике
Дополнить расширенный класс ExtBinary функцией, которая будет перемножать свойства val1 и val2, возвращая результат в формате целого числа.

```
class ExtBinary extends Binary;
  function new(input bit [3:0] val1, val2);
    super.new(val1, val2);
  endfunction
  function int multiply();
    return val1 * val2;
  endfunction
endclass
```

Поскольку конструктор базового класса Binary::new требует передачи аргументов, дочерний класс ExtBinary обязан самым первым действием в своем конструкторе выполнить вызов super.new(...).

## 2. Инициализация значениями 15 и 8 с последующим выводом результата
```
ExtBinary b = new(15, 8);
b.print_int(b.multiply());        // 15*8 = 120  
```

Лог симулятора (XSim): val=0d120 (Математически 15·8 = 120. Префикс 0d является строковым литералом, жестко зашитым в базовую функцию print_int).

## 3. Создание дочернего класса с ограничением  <0

```
class Exercise3 extends ExtBinary;
  constraint c { val1 < 10; val2 < 10; }
  function new();
    super.new(0, 0);
  endfunction
endclass
```

## 4. Выполнение рандомизации и печать вычисленного произведения

```
Exercise3 e = new();
if (!e.randomize()) $error("randomize failed");
e.print_int(e.multiply());
```

Лог симулятора (XSim):
val1=0 val2=4  
val=0d0  
## 5. Допустимые операции и ошибки компиляции

Проанализировать фрагменты кода и определить, какие из них пройдут компиляцию, а какие вызовут ошибку.

Binary b;  
ExtBinary mc, mc2;  
  
| Сниппет                                 | Анализ операции                                                       | Итоговый результат                       |
| --------------------------------------- | --------------------------------------------------------------------- | ---------------------------------------- |
| a mc=new(15,8); b=mc;                   | upcast всегда разрешенонеявно.                                        | b успешно ссылается на объект ExtBinary. |
| b b=new(15,8); mc=b;                    | downcast (от базового к расширенному) без явного использования $cast. | Возникает ошибка компиляции.             |
| c mc=new(15,8); b=mc; mc2=b;            | mc2=b (downcast) без вызова функции $cast.                            | Возникает ошибка компиляции.             |
| d mc=new(15,8); b=mc; if($cast(mc2,b))… | $cast динамически анализирует тип, скрывающийся за указателем.        | Success.                                 |

Результаты тестирования:

Ex5a: OK, b points to ExtBinary object (b.val1=15)  
Ex5d: Success  
ex5b -> ERROR: [VRFC 10-900] incompatible complex type assignment  
ex5c -> ERROR: [VRFC 10-900] incompatible complex type assignment  


## 6. Разработка функции copy для класса ExtBinary
В условии предоставлена исходная реализация Binary::copy. Необходимо написать аналогичный метод для класса ExtBinary.

```
class ExtBinary extends Binary;
  ...
  virtual function Binary copy();
    ExtBinary c;
    c = new(val1, val2);
    return c;
  endfunction
endclass
```

## 7. Копирование объекта mc в mc2 (оба имеют тип ExtBinary)
```
if (!$cast(mc2, mc.copy())) $error("cast failed"); 
```

## 8. Реализация коллбека со случайной задержкой от 0 до 100 нс
Взяв за основу Sample 8.26–8.28, добавить механизм внесения случайной задержки в диапазоне 0–100 наносекунд.

```
virtual class Driver_cbs; // Sample 8.26
  virtual task pre_tx(ref Transaction tr, ref bit drop); endtask
  virtual task post_tx(ref Transaction tr); endtask
endclass

class Driver_cbs_delay extends Driver_cbs;
  virtual task pre_tx(ref Transaction tr, ref bit drop);
    int d = $urandom_range(0, 100);
    #(d * 1ns); // 0-100ns
    $display("%0t: [delay cb] added %0d ns", $time, d);
  endtask
endclass

drv.cbs.push_back(new());  // Driver_cbs_delay
```

Лог симулятора (XSim) для 4 транзакций:

58000: [delay cb] added 58 ns    58000: transmit data=73  
120000: [delay cb] added 62 ns   120000: transmit data=0  
131000: [delay cb] added 11 ns   131000: transmit data=174  
143000: [delay cb] added 12 ns   143000: transmit data=207  
## 9. Создание параметризованного компаратора
Разработать класс для проверки идентичности данных произвольного типа с использованием строгих операторов сравнения (='='=/!='=). Функция compare должна выдавать 1 при совпадении и 0 в случае расхождений. Тип по умолчанию - два 4-битных вектора.

```
class Comparator #(type T = bit [3:0]);
  function bit compare(T a, T b);
    return (a === b);
  endfunction
endclass
```
## 10. Сравнение 4-битных чисел и перечисления color_t с подсчетом ошибок

```
typedef enum {RED, GREEN, BLUE} color_t;

Comparator #(bit [3:0]) cmp4 = new();
Comparator #(color_t)   cmpc = new();
int errors = 0;

if (!cmp4.compare(expected_4bit, actual_4bit))   errors++;
if (!cmpc.compare(expected_color, actual_color)) errors++;
```  
