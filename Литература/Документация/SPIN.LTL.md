#LTL #SPIN #Верификация #BNF

| Наименование | Автор | Издательство | Год | Дата начала |
|------|:---------|:-----------|:---------|:----------|
|Документация SPIN. LTL раздел. (версия 6.5.1)|  | https://spinroot.com/ | 2020-07 |2023-07-07|

#Работа [web](https://spinroot.com/spin/Man/ltl.html)

Синтаксис LTL формул приведенный в документации:
```
Grammar:
	ltl ::= **opd** | ( ltl ) | ltl **binop** ltl | **unop** ltl

Operands (**opd**):
	true, false, user-defined names starting with a lower-case letter,
	or embedded expressions inside curly braces, e.g.,: { a+b>n }.

Unary Operators (**unop**):
	[]	(the temporal operator _always_)
	<>	(the temporal operator _eventually_)
	! 	(the boolean operator for _negation_)

Binary Operators (**binop**):
	U 	(the temporal operator _strong until_)
	W	(the temporal operator _weak until_ (only when used in inline formula)
	V 	(the dual of U): (p V q) means !(!p U !q))
	&&	(the boolean operator for _logical and_)
	||	(the boolean operator for _logical or_)
	/\	(alternative form of &&)
	\/	(alternative form of ||)
	->	(the boolean operator for _logical implication_)
	<->	(the boolean operator for _logical equivalence_)
```

Формула задается глобально (вне всех объявлений `proctype` или `init`) следующим образом:

```
ltl [name] '{' formula '}'
```

Имя необязательно, но более наглядно если формул несколько. (Каждая формула имеет один и тот же формат.) Формула имеет описанную выше грамматику с некоторыми расширениями. Во-первых, пробелы (переводы строк, знаки табуляции) можно использовать где угодно для разделения операндов и операторов. Во-вторых, имена операторов можно либо сокращать с помощью символов, показанных выше, либо указывать полностью (`always`, `eventually`, `until`, `implies`, и `equivalent` ). Другие операторы `weakuntil`, `stronguntil`, и `release` также поддерживаются. Это означает, что две последующие формулы эквивалентны:

```
ltl p1 { []<> p }
ltl p2 { always eventually p }
```

Сформулированные таким образом свойства принимаются как положительные свойства, которым должна удовлетворять модель. Средство проверки модели выполнит автоматическое отрицание формулы, чтобы найти контр примеры. (Отрицание не происходит автоматически, когда вы используете альтернативный и более старый метода, описанный ниже). 

Еще одно улучшение в спецификации формулы #LTL в #SPIN версии 6 и более поздних версиях состоит в том, что встроенная формула #LTL может содержать любую формулу пропозиционального состояния, т. е. она не ограничивается строчными пропозициональными символами из предыдущего, где каждый пропозициональный символ должен быть определены в определениях макросов. Это означает, что мы можем написать:

```
ltl p3 { eventually (a > b) implies len(q) == 0 }
```

Есть только одно ограничение: вы не можете использовать предопределенные операторы `empty` , `nempty` , `full`, `nfull` во встроенной формуле #LTL (и не должны использовать их в альтернативном методе, хотя там это не обязательно). Причина техническая: в соответствии с конкретными правилами частичного сокращения порядка, используемыми в средстве проверки моделей, четыре перечисленных оператора канала не могут быть инвертированы. В процессе преобразования из формулы #LTL в формулу "никогда не утверждает" (автомата #Бюхи) будут иметь место отрицания, которые могут закончиться в конечном автомате, что может привести к тому, что сокращения частичного порядка будут недействительными (логически необоснованными).

Когда указано несколько встроенных формул #LTL, вы можете выбрать, какая из них будет применяться во время прогона проверки с новым параметром времени выполнения `-N` для верификаторов панорамирования . По умолчанию это будет первая формула #LTL, которая появляется в спецификации. Чтобы отключить все формулы #LTL во время проверки, вы можете скомпилировать `pan.c` с директивой `-DNOCLAIM`.
