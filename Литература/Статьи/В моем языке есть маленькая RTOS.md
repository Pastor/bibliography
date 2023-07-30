#Статья #Перевод #Ada #Ravenscar #В-Работе 

| Наименование | Автор | Издание | Год | Ресурс |
|------|:---------|:-----------|:---------|:----------|
|В моем языке есть маленькая RTOS|[Фабьене Шуто](https://blog.adacore.com/author/chouteau)|Ada Core| 2017 |[web](https://blog.adacore.com/theres-a-mini-rtos-in-my-language)|
#Работа [WEB](https://blog.adacore.com/theres-a-mini-rtos-in-my-language)

Первое, что меня поразило, когда я начал изучать язык программирования Ada, была поддержка задач. В Ada создание задач, их синхронизация, совместное использование доступа к ресурсам являются частью языка

В этом блоге я сосредоточусь на встроенной стороне вещей. Во-первых, потому что это то, что мне нравится, а также потому, что это намного проще :)

Для приложений реального времени и встроенных приложений Ada определяет профиль под названиемРаvenсcар. Это подмножество языка, предназначенное для анализа планируемости, оно также более совместимо с такими платформами, как микроконтроллеры, которые имеют ограниченные ресурсы.

Так что это будет не полная лекция по задаче Ады. Я мог бы продолжить с некоторыми дополнительными функциями задач, если вы попросите об этом в комментариях ;)

# Задачи

Итак, первое - это создавать задачи, верно?

Есть два способа создания задач в Ada, сначала вы можете объявить и реализовать одну задачу:

```Ada
--  Task declaration
   task My_Task;
```

```Ada
--  Task implementation
   task body My_Task is
   begin
      --  Do something cool here...
   end My_Task;
```

Если у вас есть несколько задач, выполняя одну и ту же работу, или если вы пишете библиотеку, вы можете определить тип задачи:

```Ada
--  Task type declaration
   task type My_Task_Type;
```

```Ada
--  Task type implementation
   task body My_Task_Type is
   begin
      --  Do something really cool here...
   end My_Task_Type;
```

А затем создайте столько задач такого типа, сколько хотите:

```Ada
T1 : My_Task_Type;
   T2 : My_Task_Type;
```

Одно из ограничений Ravenscar по сравнению с полной Ada заключается в том, что количество задач должно быть известно во время компиляции.

# Время

Функции синхронизации Ravenscar предоставляются пакетом (вы догадались) **Ada.Real_Time**.

В этом пакете вы найдете:  

- определение типа **времени**, которое представляет время, прошедшее с момента запуска системы
- определение типа **Time_Span**, которое представляет собой период между двумя значениями **времени**
- функция **Clock**, которая возвращает текущее время (монотонное количество с момента запуска системы)
- Различные подпрограммы для манипулирования значениями **Time** и **Time_Span**

Язык Ada также содержит инструкцию приостановить задачу до определенного момента времени: **отложить до**.

Вот пример того, как создать циклическую задачу, используя функции синхронизации Ada.

```Ada
task body My_Task is
      Period       : constant Time_Span := Milliseconds (100);
      Next_Release : Time;
   begin
      --  Set Initial release time
      Next_Release := Clock + Period;

      loop
         --  Suspend My_Task until the Clock is greater than Next_Release
         delay until Next_Release;

         --  Compute the next release time
         Next_Release := Next_Release + Period;
         
         --  Do something really cool at 10Hz...
      end loop;

   end My_Task;
```

# Расписание

Ravenscar имеет приоритетное упреждающее планирование. Каждой задаче назначается приоритет, и планировщик позаботится о том, чтобы задача с наивысшим приоритетом - среди готовых задач - выполняется.

Задача может быть вытеснена, если высвобождается другая задача с более высоким приоритетом либо внешним событием (прерыванием), либо по истечении срока ее **задержки до** выписки (как показано выше).

Если две задачи имеют одинаковый приоритет, они будут выполнены в том порядке, в котором они были выпущены (FIFO в рамках приоритетов).

Приоритеты задач статичны, однако ниже мы увидим, что приоритет задачи может быть временно эскалирован.

Приоритет задачи - это целое значение от 1 до 256, более высокое значение означает более высокий приоритет. Это указывается с **приоритетным** аспектом:

```Ada
Task My_Low_Priority_Task
     with Priority => 1;

   Task My_High_Priority_Task
     with Priority => 2;
```

# Взаимная изоляция и общие ресурсы

В Аде взаимное исключение обеспечивается **охраняемыми** **объектами**.

Во время выполнения защищенные объекты предоставляют следующие свойства:

- В данный момент времени может быть только одна задача, выполняющая защищенную операцию (взаимное исключение)
- Не может быть [тупика](https://en.wikipedia.org/wiki/Deadlock)

В профиле Ravenscar это достигается с помощью [протокола приоритетного потолка](https://en.wikipedia.org/wiki/Priority_ceiling_protocol).

Каждому защищенному объекту назначается приоритет, любые задачи, вызывающие защищенную подпрограмму, должны иметь приоритет ниже или равный приоритету защищенного объекта.

Когда задача вызывает защищенную подпрограмму, ее приоритет временно повышается до приоритета защищаемого объекта. В результате эта задача не может быть вытеснена ни одной из других задач, которые потенциально используют этот защищенный объект, и поэтому обеспечивается взаимное исключение.

Протокол приоритетного потолка также обеспечивает решение классической проблемы планирования [инверсии приоритетов](https://en.wikipedia.org/wiki/Priority_inversion).

Вот пример защищенного объекта:

```Ada
--  Specification
   protected My_Protected_Object
     with Priority => 3
   is

      procedure Set_Data (Data : Integer);
      --  Protected procedues can read and/or modifiy the protected data
      
      function Data return Integer;
      --  Protected functions can only read the protected data

   private
   
      --  Protected data are declared in the private part
      PO_Data : Integer := 0;
   end;
```

```Ada
--  Implementation
   protected body My_Protected_Object is

      procedure Set_Data (Data : Interger) is
      begin
         PO_Data := Data;
      end Set_Data;

      function Data return Integer is
      begin
         return PO_Data;
      end Data;
   end My_Protected_Object;
```

# Синхронизация

Еще одной интересной особенностью **защищенных** **объектов** является синхронизация между задачами.

Это делается с помощью другого вида операции, называемой **вводом**.

Запись имеет те же свойства, что и защищенная процедура, за исключением того, что она будет выполнена только в том случае, если данное условие истинно. Задача, вызывающая запись, будет приостановлена до тех пор, пока условие не будет выполнено.

Эту функцию можно использовать для синхронизации задач. Вот пример:

```Ada
protected My_Protected_Object is
      procedure Send_Signal;
      entry Wait_For_Signal;
   private
      We_Have_A_Signal : Boolean := False;
   end My_Protected_Object;
```

```Ada
protected body My_Protected_Object is

      procedure Send_Signal is
      begin
          We_Have_A_Signal := True;
      end Send_Signal;    
      
      entry Wait_For_Signal when We_Have_A_Signal is
      begin
          We_Have_A_Signal := False;
      end Wait_For_Signal;
   end My_Protected_Object;
```

# Обработка прерываний

Защищенные объекты также используются для обработки прерываний. Частные процедуры защищенного объекта могут быть прикреплены к прерыванию с помощью аспекта **Attach_Handler**.

```Ada
protected My_Protected_Object
     with Interrupt_Priority => 255
   is
   
   private
   
      procedure UART_Interrupt_Handler
        with Attach_Handler => UART_Interrupt;
   
   end My_Protected_Object;
```

В сочетании с **записью** он обеспечивает элегантный способ обработки входящих данных на последовательном порту, например:

```Ada
protected My_Protected_Object
     with Interrupt_Priority => 255
   is
      entry Get_Next_Character (C : out Character);
      
   private
      procedure UART_Interrupt_Handler
              with Attach_Handler => UART_Interrupt;
      
      Received_Char  : Character := ASCII.NUL;
      We_Have_A_Char : Boolean := False;
   end
```

```Ada
protected body My_Protected_Object is

      entry Get_Next_Character (C : out Character) when We_Have_A_Char is
      begin
          C := Received_Char;
          We_Have_A_Char := False;
      end Get_Next_Character;
      
      procedure UART_Interrupt_Handler is
      begin
          Received_Char  := A_Character_From_UART_Device;
          We_Have_A_Char := True;
      end UART_Interrupt_Handler;      
   end
```

Задача, вызывающая запись **Get_Next_Character**, будет приостановлена до тех пор, пока не будет запущено прерывание и обработчик не прочитает символ с устройства UART. Тем временем другие задачи смогут выполняться на процессоре.

# Многоядерная поддержка

Ada поддерживает статическое и динамическое распределение задач по ядрам в многопроцессорных архитектурах. Профиль Ravenscar ограничивает эту поддержку полностью разделенным подходом, когда задачи статически распределяются между процессорами, и между процессорами нет миграции задач. Эти параллельные задачи, выполняемые на разных процессорах, могут взаимодействовать и синхронизироваться с использованием защищенных объектов.

Аспект ЦП определяет сходство задач:

```Ada
task Producer with CPU => 1;
   task Consumer with CPU => 2;
   --  Parallel tasks statically allocated to different cores
```

# Реализации

Это все для краткого обзора основных функций задач Ada Ravenscar.

Одним из преимуществ выполнения задач в рамках языкового стандарта является портативность, вы можете запустить одно и то же приложение Ravenscar на Windows, Linux, MacO или RTOS, такой как VxWorks. GNAT также предоставляет небольшое автономное время выполнения, которое реализует задачи Ravenscar на голом металле. Это время выполнения доступно, например, на микроконтроллерах ARM Cortex-M.

Это как иметь RTOS на вашем языке.