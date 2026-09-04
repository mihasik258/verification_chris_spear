## 1. Хронология и очередность для конструкций join, join_any, join_none
Выяснить, в какие моменты времени и в какой последовательности будут выполняться операторы при использовании разных вариантов join.

```
initial begin
  $display("@%0d: start fork...join example", $time);
  fork
    begin
      #20 $display("@%0d: sequential A after #20", $time);
      #20 $display("@%0d: sequential B after #20", $time);
    end  
    $display("@%0d: parallel start", $time);
    #50 $display("@%0d: parallel after #50", $time);
    begin  
      #30 $display("@%0d: sequential after #30", $time);
      #10 $display("@%0d: sequential after #10", $time);
    end  
  join
  $display("@%0d: after join", $time);
  #80 $display("@%0d: finish after #80", $time);
end  
```

Поскольку содержимое блока fork неизменно во всех сценариях, параллельное выполнение веток происходит следующим образом:

|Временная отметка|Вывод в консоль|
|---|---|
|@0|start; parallel start (ветка 2)|
|@20|sequential A (ветка 1)|
|@30|sequential after #30 (ветка 4)|
|@40|sequential B (ветка 1); sequential after #10 (ветка 4)|
|@50|parallel after #50 (ветка 3)|

| Тип       | Возобновление родительского процесса          | after join | finish after #80 |
| --------- | --------------------------------------------- | ---------- | ---------------- |
| join      | дожидается всех веток  (@50)                  | @50        | @130             |
| join_any  | дожидается первой ветки (ветка 2 готова в @0) | @0         | @80              |
| join_none | идет дальше сразу же (@0)                     | @0         | @80              |

## 2. Поведение программы с конструкцией wait fork и без неё
Спрогнозировать консольный вывод для двух вариантов: с закомментированным и раскомментированным wait fork в указанном месте.

```
fork transmit(1); transmit(2); join_none // transmit: #10ns
fork: receive_fork receive(1); receive(2); join_none // receive: #(index*10ns)
// wait fork
#15ns disable receive_fork;
$display("%0d: Done", $time);
```
  
`transmit(i)` печатает в @10; `receive(1)`@10, `receive(2)`@20.

Без wait fork 

10: Receive is done for index = 1  
10: Transmit is done for index = 1  
10: Transmit is done for index = 2  
15: Done  

С wait fork

10: Receive is done for index = 1  
10: Transmit is done for index = 1  
10: Transmit is done for index = 2  
20: Receive is done for index = 2  
35: Done  

## 3. Передача событий в качестве хэндлов
Определить результат выполнения. Объявления событий и задачи trigger находятся в блоке program automatic.

```
task trigger(event local_event, input time wait_time);
  #wait_time;
  ->local_event;
endtask

initial begin
  fork
    trigger(e1, 10ns);
    begin wait(e1.triggered()); $display("%0d: e1 triggered", $time); end
  join
end
// аналогично e2 с 20ns
```

10: e1 triggered  
20: e2 triggered  
  
тип event работает как передаваемая ссылка. Использование wait(e1.triggered()) обеспечивает уровневый контроль, который гарантированно фиксирует триггер, даже при его возникновении в текущем временном слоте. Обычная конструкция @e могла бы упустить событие из-за race condition.

## 4. Использование семафора в задаче wait10
Реализовать задачу wait10, делающую 10 попыток. Каждая попытка включает задержку в 10 нс и проверку доступности одного ключа из семафора. При успешном получении ключа цикл должен прерваться, а текущее время — вывестись на экран.

```
task automatic wait10();
  for (int i = 0; i < 10; i++) begin
    #10ns;
    if (sem.try_get(1)) begin
      $display("%0d: got key (try i=%0d)", $time, i);
      return;
    end
  end
  $display("%0d: no key after 10 tries", $time);
endtask
```

try_get(1). Он возвращает 1 при успешном захвате ключа и 0 в случае его отсутствия. Это позволяет исключительно проверять статус семафора без риска "зависания" (блокировки) родительского потока.

## 5. Результат работы задачи wait10
Проанализировать поведение программы при следующем вызове:

```
fork
  begin
    sem = new(1);
    sem.get(1);
    #45ns;
    sem.put(2);
  end
  wait10();
join
```
  

вызов wait10 осуществляет проверки на отметках @10, @20, @30 и @40, но ключи на этот момент отсутствуют. В момент времени @45 выполняется put(2), что увеличивает количество доступных ключей до двух. проверка на отметке @50 (i=4) успех.

## 6. Работа с методами почтового ящика (mailbox)
Спрогнозировать консольный вывод следующего фрагмента.

```
mailbox #(int) mbx;
int value;
mbx = new(1);
$display("mbx.num()=%0d", mbx.num());
$display("mbx.try_get= %0d", mbx.try_get(value));
mbx.put(2);
$display("mbx.try_put= %0d", mbx.try_put(value));
$display("mbx.num()=%0d", mbx.num());
mbx.peek(value);
$display("value=%0d", value); 
``` 

mbx.num()=0        // пусто  
mbx.try_get= 0     // try_get на пустом -> 0 ; value не меняется (=0)  
mbx.try_put= 0     // после put(2) ящик ПОЛОН (размер 1) -> try_put -> 0 (неудача)  
mbx.num()=1        // в ящике 1 элемент (2)  
value=2            // peek читает голову  -> 2  
## 7. Разработка класса Monitor
Опираясь на рисунок 7.8, спроектировать класс Monitor. Он должен взаимодействовать с классом OutputTrans (имеющим поля out1, out2) и подключаться к тестируемому устройству (DUT) посредством интерфейса my_bus с блоком clocking cb. Монитор обязан на каждом рабочем фронте тактового сигнала захватывать значения DUT, помещать их в экземпляр OutputTrans и отправлять в почтовый ящик.

```
class Monitor;
  virtual my_bus vif;
  mailbox #(OutputTrans) mon2chk;

  function new(virtual my_bus vif, mailbox #(OutputTrans) mon2chk);
    this.vif = vif;
    this.mon2chk = mon2chk;
  endfunction

  task run();
    forever begin
      OutputTrans tr;
      @(vif.cb);
      tr = new();
      tr.out1 = vif.cb.out1;
      tr.out2 = vif.cb.out2;
      mon2chk.put(tr);
    end
  endtask
endclass
```
  

Результат тестирования в XSim (использовалась модель-заглушка DUT, где out1 инкрементируется, а out2 увеличивается на 2 каждый такт, плюс модуль проверки, читающий данные из мейлбокса):

MON: out1=00 out2=a0  
MON: out1=01 out2=a2  
MON: out1=02 out2=a4  
MON: out1=03 out2=a6  
MON: out1=04 out2=a8
