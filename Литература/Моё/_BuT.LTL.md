#Языки #LTL #Автоматное-программирование #Верификация #В-Работе #Грамматики #BNF 

### LTL формулы

В качестве базового представления формулы темпоральной логики, будет использовать синтаксис и семантика используемая в наборе утилит #SPIN. В [[SPIN.LTL|документации]] к набору утилит есть описание синтаксиса. Запишем ее в расширенной нотации #BNF( #EBNF ).

```ebnf
ltl = operand | "(", ltl, ")" | ltl, binop, ltl | unop, ltl;
operand = "true" | "false" | expression;
unop = "[]" | "<>" | "!";
binop = "U" | "W" | "V" | "&&" | "||" | "->" | "<->";
user_defined_name = letter, { letter | digit | "_" };
digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9";
nat_number = digit, { digit };
number = ["+" | "-"], nat_number, [ ".", { nat_number } ];
letter = "A" | "B" | "C" | "D" | "E" | "F" | "G"
       | "H" | "I" | "J" | "K" | "L" | "M" | "N"
       | "O" | "P" | "Q" | "R" | "S" | "T" | "U"
       | "V" | "W" | "X" | "Y" | "Z" | "a" | "b"
       | "c" | "d" | "e" | "f" | "g" | "h" | "i"
       | "j" | "k" | "l" | "m" | "n" | "o" | "p"
       | "q" | "r" | "s" | "t" | "u" | "v" | "w"
       | "x" | "y" | "z" ;
expression = conjunction, { "||", conjunction };
conjunction = relation, { "&&", relation };
relation = addition, { ("<" | "<=" | ">" | ">=" | "==" | "!="), addition };
addition = term, { ("+" | "-"), term };
term = factor, { ("*" | "/"), factor };
factor = user_definited_name | number | "(", expression, ")";
```

При помощи [утилиты](https://matthijsgroen.github.io/ebnf2railroad/try-yourself.html) разложим грамматику на составляющие.

#### Полное представление грамматики
![[LTL.Grammar.FULL.PNG]]

#### Операнд
```ebnf
operand = "true" | "false" | expression;
```

![[LTL.Grammar.OPERAND.PNG|Операнд]]
#### Унарная операция
```ebnf
unop = "[]" | "<>" | "!";
```

![[LTL.Grammar.UNOP.PNG]]
#### Бинарные операции
```ebnf
binop = "U" | "W" | "V" | "&&" | "||" | "->" | "<->";
```

![[LTL.Grammar.BINOP.PNG]]
#### Пользовательские переменные
```ebnf
user_defined_name = letter, { letter | digit | "_" };
```

![[LTL.Grammar.USER_DEFINED_NAME.PNG]]
#### Цифры
```ebnf
digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9";
```

![[LTL.Grammar.DIGIT.PNG]]
#### Натуральные числа
```ebnf
nat_number = digit, { digit };
```

![[LTL.Grammar.NAT_NUMBER.PNG]]
#### Числа
```ebnf
number = ["+" | "-"], nat_number, [ ".", nat_number ];
```

![[LTL.Grammar.NUMBER.PNG]]
#### Буквы
```ebnf
letter = "A" | "B" | "C" | "D" | "E" | "F" | "G"
       | "H" | "I" | "J" | "K" | "L" | "M" | "N"
       | "O" | "P" | "Q" | "R" | "S" | "T" | "U"
       | "V" | "W" | "X" | "Y" | "Z" | "a" | "b"
       | "c" | "d" | "e" | "f" | "g" | "h" | "i"
       | "j" | "k" | "l" | "m" | "n" | "o" | "p"
       | "q" | "r" | "s" | "t" | "u" | "v" | "w"
       | "x" | "y" | "z" ;
```

![[LTL.Grammar.LETTER.PNG]]
#### Выражение
```ebnf
expression = conjunction, { "||", conjunction };
```

![[LTL.Grammar.EXPRESSION.PNG]]
#### Коньюнкция
```ebnf
conjunction = relation, { "&&", relation };
```

![[LTL.Grammar.CONJUNCTION.PNG]]
#### Дизьюнкция
```ebnf
relation = addition, { ("<" | "<=" | ">" | ">=" | "==" | "!="), addition };
```

![[LTL.Grammar.RELATION.PNG]]
#### Сложение и вычитание
```ebnf
addition = term, { ("+" | "-"), term };
```

![[LTL.Grammar.ADDITION.PNG]]
#### Умножение и деление
```ebnf
term = factor, { ("*" | "/"), factor };
```

![[LTL.Grammar.TERM.PNG]]
#### Компоненты операций
```ebnf
factor = user_definited_name | number | "(", expression, ")";
```

![[LTL.Grammar.FACTOR.PNG]]
