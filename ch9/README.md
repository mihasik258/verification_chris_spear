Базовый класс

```
typedef enum {ADD, SUB, MULT, DIV} opcode_e;
class Transaction;
  rand opcode_e opcode;
  rand byte operand1;
  rand byte operand2;
endclass
Transaction tr;
```

## 1. Создание группы покрытия для тестирования всех опкодов АЛУ
Разработать `covergroup`, обеспечивающую выполнение требования тестирования всех кодов операций АЛУ). Семпл валидных опкодов должен осуществляться по фронту `clk`.

```
covergroup Covcode @(posedge clk);
  opcode_cp: coverpoint tr.opcode;
endgroup
```

для переменных типа enum система автоматически генерирует индивидуальную корзину под каждое возможное состояние.
## 2. Настройка бинов для граничных значений operand1
 Обеспечить сбор покрытия для сценариев, при которых `operand1` принимает критические значения: максимальное отрицательное (−128), нулевое и максимальное положительное (127). Требуется выделить персональный бин под каждое состояние, а также добавить корзину default. Имя точки покрытия - operand1_cp.

```
operand1_cp: coverpoint tr.operand1 {
  bins neg    = {-128};      // max negative
  bins zero   = {0};
  bins pos    = {127};       // max positive
  bins others = default;
}
```
## 3. Организация бинов для опкодов: объединение и переходы
 Собрать статистику для следующих ситуаций: a) выпадение операций `ADD` либо `SUB` (одна общая корзина); b) регистрация последовательности, где за `ADD` сразу следует `SUB` (вторая корзина). Назначить идентификатор `opcode_cp`.


```
opcode_cp: coverpoint tr.opcode {
  bins add_or_sub   = {ADD, SUB}; // a
  bins add_then_sub = (ADD => SUB); // b
}
```

 `{ADD, SUB}` формирует единый бин, который будет считаться пройденным при выпадении любой из этих двух команд. 
 `(ADD => SUB)`- бин перехода - только тогда, когда на текущем такте зафиксирован `ADD`, а ровно на следующем - `SUB`.

## 4. Использование illegal_bins для блокировки инструкции DIV
 Внедрить правило «опкод не должен принимать значение `DIV`». При возникновении подобного сценария система должна сигнализировать об ошибке через механизм `illegal_bins`.

```
opcode_cp: coverpoint tr.opcode {
  bins add_or_sub    = {ADD, SUB};
  bins add_then_sub  = (ADD => SUB);
  illegal_bins no_div = {DIV};        // семпл DIV -> ошибка симуляции
}
```

## 5. Настройка кросс-покрытия (cross) с весовым коэффициентом 5
 Отследить комбинации, при которых операция `ADD` либо `SUB` выпадает синхронно с присвоением `operand1` максимальных отрицательных или положительных значений. Задать для этого пересечения весовой множитель, равный 5.


```
cross_cp: cross opcode_cp, operand1_cp {
  option.weight = 5;
}
```

## 6. Извлечение данных о покрытии через экземпляры и типы
 Учитывая, что группа покрытия называется `Covcode`, а её экземпляр в тесте - `ck`, необходимо: a) извлечь покрытие для `operand1_cp` через указатель на инстанс; b) запросить статистику по `opcode_cp`, обратившись напрямую к имени типа `covergroup`.

```
Covcode ck;
...
// a
$display("operand1_cp = %0.2f%%", ck.operand1_cp.get_coverage());
// b
$display("opcode_cp   = %0.2f%%", Covcode::opcode_cp.get_coverage());
```

## Итоговый код

```
typedef enum {ADD, SUB, MULT, DIV} opcode_e;

class Transaction;
  rand opcode_e opcode;
  rand byte operand1, operand2;
  constraint c_no_div { opcode != DIV; }
  constraint c_op1 { operand1 dist { -128 := 3, 0 := 3, 127 := 3,
                                     [-127:-1] := 1, [1:126] := 1 }; }
endclass

module top;
  bit clk = 0; always #5 clk = ~clk;
  Transaction tr;

  covergroup Covcode @(posedge clk);
    opcode_cp: coverpoint tr.opcode {
      bins add_or_sub    = {ADD, SUB}; // Ex3a
      bins add_then_sub  = (ADD => SUB); // Ex3b
      illegal_bins no_div = {DIV}; // Ex4
    }
    operand1_cp: coverpoint tr.operand1 { // Ex2
      bins neg = {-128}; bins zero = {0}; bins pos = {127};
      bins others = default;
    }
    cross_cp: cross opcode_cp, operand1_cp { option.weight = 5; } // Ex5
  endgroup

  Covcode ck; // Ex6
  initial begin
    tr = new(); ck = new();
    repeat (300) begin
      @(negedge clk);
      assert (tr.randomize());
    end
    @(posedge clk);
    $display("operand1_cp (ck) = %0.2f%%", ck.operand1_cp.get_coverage()); // Ex6a
    $display("opcode_cp (Covcode) = %0.2f%%", Covcode::opcode_cp.get_coverage()); // Ex6b
    $display("overall (ck) = %0.2f%%", ck.get_coverage());
    $finish;
  end
endmodule
```
