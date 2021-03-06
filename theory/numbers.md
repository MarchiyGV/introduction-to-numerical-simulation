# Особенности работы с машинной арифметикой

[Оглавление](../README.md)

Математическое моделирование почти всегда подразумевает работу
с вещественными числами, которые могут выражать скорость,
температуру, напряженность поля и т.п.
Так как вещественных чисел бесконечное количество, то точное
представление вещественного числа на компьютере невозможно,
поэтому при численном моделировании используются специально подобранные
конечные наборы чисел, позволяющие достаточно хорошо приближать
возникающие при моделировании значения.
Чаще всего используются числа с плавающей запятой, однако есть 
и другие варианты, которые также находят применение.
Рассмотрим несколько машинных представлений чисел часто используемых
на практике в порядке увеличения сложности.

## Целые числа

Для счета мы используем так называемые натуральные числа: 0, 1, 2, 3, 4...
Есть самое маленькое натуральное число: 0 (иногда берут 1),
и за каждым натуральным числом идет другое натуральное число,
на единицу большее.
Натуральные числа можно складывать и умножать по привычным школьным правилам.
Каждое натуральное число можно записать (и хранить в памяти) в двоичном виде:

$$
n=a_0+a_1\cdot 2+a_2\cdot  2^2+a_3\cdot  2^3+\ldots
$$

где каждый двоичный разряд $a_k$ принимает одно из двух значений 0 или 1.
Для хранения произвольного натурального числа $n$ требуется конечное число бит
$[\log_2 n]$, однако так как для каждого числа можно найти большее, то 
для нельзя найти число бит, которое было бы достаточно для хранения любого
натурального числа.
Для работы с произвольно большими числами используются библиотеки 
[длинной арифметики](https://ru.wikipedia.org/wiki/%D0%94%D0%BB%D0%B8%D0%BD%D0%BD%D0%B0%D1%8F_%D0%B0%D1%80%D0%B8%D1%84%D0%BC%D0%B5%D1%82%D0%B8%D0%BA%D0%B0).
Сложность арифметических операций в таком представлении зависит от 
величины числа, например, сложность сложения $O(N)$, где $N$ - число разрядов.

Для аппаратной реализации натуральной арифметики количество используемых
разрядов ограничивается сверху, так получаются следующие типы:

Тип  | Число бит | Язык программирования
-----|-----------|-----------------------
unsigned long | >32 | C
uint32 | 32 | Matlab

При ограничении числа бит на одно число возникает ситуация, 
когда результат сложения чисел нельзя представить числом того же типа,
т.е. возникает переполнение.
В большинстве реализаций результат операции определен даже 
в случае переполнения и получается отбрасыванием старших разрядов, 
не помещающихся в хранимое число бит.
Такое поведение соответствует [модульной арифметике](https://ru.wikipedia.org/wiki/%D0%A1%D1%80%D0%B0%D0%B2%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5_%D0%BF%D0%BE_%D0%BC%D0%BE%D0%B4%D1%83%D0%BB%D1%8E),
в которой числа, имеющие одинаковые остаток от деления на модуль сравнения $m$
(в нашем случае $m=2^N$, где $N$ - число бит) считаются равными.
В модульной арифметике число $x$ равно $x+m$, что позволяет
работать с отрицательными числами не изобретая для них новых алгоритмов.
Например, число $-1$ мы можем заменить на $m-1$ и операции с $m-1$
дадут тот же результат, что и с $-1$, например,


$$-1+1=(N-1)+1=N=0\;(\mathrm{mod}\,m),$$
$$(-1)(-1)=(N-1)(N-1)=N^2-2N+1=1\;(\mathrm{mod}\,m).$$

Одному числу по модулю $m$ соответствует бесконечно много чисел,
отличающихся на число, кратное $m$.
Для беззнаковых типов, рассмотренных выше, числам сопоставляются 
только положительные значения из интервала $0..2^N-1$.
Типы чисел со знаком тоже отождествляются с числами по модулю $m$,
однако числа, старший бит которых равен 1, считаются отрицательными,
поэтому числа со знаком принимают значение из интервала
$-2^{N-1}..2^{N-1}-1$.
Такое представление отрицательных чисел называют 
[дополнительным кодом](https://ru.wikipedia.org/wiki/%D0%94%D0%BE%D0%BF%D0%BE%D0%BB%D0%BD%D0%B8%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D1%8B%D0%B9_%D0%BA%D0%BE%D0%B4_(%D0%BF%D1%80%D0%B5%D0%B4%D1%81%D1%82%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5_%D1%87%D0%B8%D1%81%D0%BB%D0%B0))
Так хранятся, в частности, следующие типы:

Тип  | Число бит | Язык программирования
-----|-----------|-----------------------
long | >32 | C
int | 32 | Java
int32 | 32 | Matlab
int | >32 | Python

Ограничив число бит на одно целое число мы приходим к арифметическим операциям,
имеющим фиксированную сложность $O(1)$, и как правило эффективно реализованным
на аппаратном уровне.
Целые числа используются в расчетах очень широко,
однако далеко не все можно выразить целым числом.

## Рациональные числа

Натуральные числа удобны для счета, целые числа позволяют выразить долг или нехватку чего-то.
Однако, чтобы выразить понятие доли требуются дробные числа, 
мы начнем обсуждение с рациональных чисел.
Любое рациональное число представляется в виде отношением $p/q$
двух целых чисел, называемых числителем $p$ и знаменателем $q$.
Чтобы представление было однозначным, нужно, чтобы $q>0$
и чтобы $p$ и $q$ были взаимно простыми.

Арифметика рациональных чисел проста и сводится к операциям с числителем
и знаменателем:


$$p/q+a/b=(pb+aq)/(qb)$$
$$(p/q)(a/b)=(pa)/(qb)$$

Рациональные числа представляются, например, следующими типами:

Тип  | Язык программирования
-----|-----------------------
[boost::rational](http://www.boost.org/doc/libs/1_55_0/libs/rational/rational.html) | C++
[org.jscience.mathematics.number.Rational](http://jscience.org/api/org/jscience/mathematics/number/Rational.html) | Java
fractions.Fraction | Python

С помощью дроби можно приблизить любое вещественное число с произвольной
точностью, рациональные числа позволяют абсолютно получать результат
всех арифметических операций, но тем не менее ниша применения рациональных чисел
узка.
Одной из проблем при операциях с рациональными числами является быстрый рост 
числителя и знаменателя, например, сложим аппроксимации для $e$ и $pi$:

$$355/113+193/71=47014/8023$$

Результат имеем почти в два раза больше знаков в числителе и знаменателе,
чем аргументы.
Это значит, что для работы с рациональными числами требуется длинная арифметика,
что увеличивает сложность вычислений и делает рациональные числа непригодными
для высокопроизводительных вычислений.

## Числа с фиксированной запятой

Любое рациональное число можно представит в виде двоичной дроби:

$$p/q=\ldots a_2 a_1 a_0. b_1 b_2\ldots=\ldots+a_22^2+a_12^1+a_02^0+b_12^{-1}+b_22^{-2}+\ldots$$
Здесь двоичные разряды $a_k$ И $b_k$ принимают одно из двух значений 0 или 1,
$a_k$ задает целую часть, $b_k$ задает дробную часть.
Только конечное число бит целой части отлично от ноля,
однако дробная часть может быть бесконечно длинной.
Часто нам не нужно знать число абсолютно точно, например,
число $\pi$ часто приближенно считают равным $3.14$,
в этом случае можно не хранить все биты дробной части,
а ограничиться только конечным число бит.
Более того, если мы работаем с большими числами,
например, считаем волосы на голове (приблизительно 100 000),
то для наших целей может быть достаточно знать несколько старших разрядов,
тогда можно отбросить даже несколько бит целой части.
Т.е. мы пришли к следующему представлению дробных чисел:

$$n2^e$$

где $n$ и $e$ - целые числа, и $e$ задает положение запятой в $n$.
Если мы работаем с числами с фиксированной запятой, то
число $e$ фиксируется на этапе разработки алгоритма и не меняется в процессе
вычислений.
В следующей таблице перечисляются несколько реализаций чисел с фиксированной
запятой:

Тип  | Язык программирования
-----|-----------------------
[_Fract, _Accum](https://gcc.gnu.org/onlinedocs/gcc/Fixed-Point.html)  | C (GCC extension)
[decimal.Decimal](https://docs.python.org/2/library/decimal.html) | Python

Рассотрим следующий широкоизвестный пример использования
чисел с фиксированной запятой.
При работе с суммами денег мы можем хранить только целую часть,
соответствующую числу рублей, и два десятичных знака дробной
части, соответствующих числу копеек.
Естественно, представление с фиксированной запятой не позволяет хранить
произвольные числа и приводит к ошибкам при вычислениях, например,
начислим один процент на сумму 100.01 руб.:

$$1.01\cdot 100.01 = 101.0101 \approx 101.01$$

Так как представления с фиксированным числом бит неизбежно приводят
к погрешностям при вычислениях, то нас не должна смущать возникающая
погрешность.
Однако мы должны уметь оценивать эту погрешность, чтобы знать,
насколько мы можем доверять результату.

Рассмотрим операцию $F(x)$ округления вещественного числа до ближайшего,
допускающего представление в виде числа с фиксированной запятой
($q$ фиксировано).
По определению числа с фиксированной запятой, для всех
вещественных чисел $x$ справедливо:

$$|F(x)-x|\leq 2^e/2.$$

Абсолютное значение разности между верным значением и приближенным
значением называется абсолютной погрешностью.
Мы видим, что абсолютная погрешность округления равна половине
шага $2^e$ между двумя соседними различающимися числами с фиксированной запятой.
Указанную величину шага $2^e$ называют машинной погрешностью, и мы будем 
обозначать ее через $\epsilon$.

Для определения погрешность часто удобно записывать число с погрешностью в виде
$x=x_0+\Delta x$, где $x_0$ верное значение, а $\Delta x$ - ошибка.
Для ошибки округления мы можем утверждать:

$$x=F(x)+\Delta F, где |\Delta F|<\epsilon/2.$$

Рассмотрим, что будет происходить с ошибками при других операциях.
Возьмем два числа изначально данных с ошибками:

$$x=x_0+\Delta x, y=y_0+\Delta y,\quad|\Delta x|, |\Delta y| < \Delta.$$

Выполним над ними арифметические операции и выделим в результате точное
значение операции, тогда оставшееся слагаемое представляет собой ошибку:


$$x+y=(x_0+y_0)+(\Delta x+\Delta y),$$
$$x-y=(x_0-y_0)+(\Delta x-\Delta y),$$
$$xy=(x_0+\Delta x)(y_0+\Delta y)=(x_0y_0)+(x_0\Delta y+y_0\Delta x+\Delta x\cdot\Delta y),$$
$$x/y=(x_0+\Delta x)/(y_0+\Delta y)=(x_0/y_0+\Delta x/y_0)/(1+\Delta y/y_0)=$$
$$=(x_0/y_0+\Delta x/y_0)(1-(\Delta y/y_0)+(\Delta y/y_0)^2-\ldots)=(x_0/y_0)+(\Delta x/y_0-\Delta y x_0/y_0^2+\ldots)$$

Так как на практике нас в основном интересует случай малой ошибки, 
то мы можем отбросить слагаемые меньших порядков, чем $\Delta x$ и $\Delta y$.
Пользуясь ограничениями на максимальное значение ошибок,
получаем:

$$
|\Delta x+\Delta y|\leq|\Delta x|+|\Delta y|<2\Delta,
$$

$$
|\Delta x-\Delta y|\leq|\Delta x|+|\Delta y|<2\Delta,
$$

$$
|x_0\Delta y+y_0\Delta x|\leq|x_0\Delta y|+|y_0\Delta x|<\Delta(|x_0|+|y_0|),
$$

$$
|\Delta x/y_0-\Delta y x_0/y_0^2|\leq|\Delta x/y_0|+|\Delta y x_0/y_0^2|<\Delta(1+|x_0/y_0|)/|y_0|.
$$

Мы видим, что абсолютная погрешность при сложении и вычитании равна абсолютной
погрешность аргументов.
Формулы для абсолютной погрешности умножения и деления сложнее,
однако мы можем заметить, что они отличаются качественно от сложения и 
вычитание тем, что погрешность результата зависит от операндов.
В частности, многократно умножая число, например, на два, мы
не можем гарантировать сохранение некоторой разумной точности,
напротив, погрешность быстро растет.
Это один из недостатков чисел с фиксированной запятой.

Другой недостаток связан с тем, что диапазон значений, которые может принимать
число с фиксированной запятой невелик.
Мы можем увеличить диапазон добавляя разряды в число, однако это приводит к 
другой проблеме.
Неважно насколько велико или мало число, погрешность его представления всегда
одинакова, однако если для маленьких чисел изменение на $\epsilon$ может 
соответствовать изменению старших разрядов, то для больших чисел изменение на 
$\epsilon$ может быть пренебрежимо мало.
Другими словами плотность расположения чисел с фиксированной запятой одинакова
для всего диапазона, на котором лежат эти числа,
но для малых чисел эта плотность может быть недостаточна, а для больших чисел
избыточна.
Например, пусть мы храним один знак после запятой и будем рассчитывать
скорость падения кирпича, первоначально находящегося в состоянии покоя.
Через одну секунду скорость равна $9.8$ м/c, а через 999 секунд
равна $9790.2$
для нашего приближенного значения ускорения свободного падения.
Все знаки в этих значениях верны, однако на практике нас часто
интересуют два старших разряда во втором числе, также как и в первом.
Это значит, что мы нерационально расходуем память для хранения чисел.
Вместо избыточной точности для больших чисел предпочтительнее было
бы иметь больший диапазон значений.
Такое усовершенствование действительно можно сделать, перейдя к числам с 
плавающей запятой.

## Числа с плавающей запятой.

числа с плавающей запятой на качественном уровне имеют очень похожее
представление на числа с фиксированной запятой:

$$n2^e,$$

однако $e$ в случае плавающей запятой хранится вместе с числом и может меняться
в процессе вычислений.
На практике чаще всего используется реализация чисел с плавающей запятой,
удовлетворяющая стандарту [IEEE 754](https://en.wikipedia.org/wiki/IEEE_floating_point),
рассмотрим его подробнее.
Вместо целого числа $n$ используется дробное число
$m=\pm 0. m_1 m_2 \ldots$, называемое *мантиссой*, тогда число с плавающей запятой представляется в виде:

$$m\cdot 2^e = \pm 0. m_1 m_2\ldots \cdot 2^e,$$

здесь $2$ является основанием экспоненты
(кроме $2$ иногда используется основание $10$),
а целое число $e$ называют *экспонентой*.

Так как запятую всегда можно переставить изменяя экспоненту,
то для всех чисел, кроме $0$, можно считать $m_1=1$,
это позволяет не хранить бит $m_1$ в памяти.
Для хранения нуля выделается специальное значение экспоненты,
когда все биты равны $0$.
Также особым является значение экспоненты со всеми битами установленными в $1$,
в частности, так представляются бесконечно большие значения.
Остальные значения экспоненты хранят величину показателя $e$ с некоторым 
сдвигом, например, для чисел одинарной точности,
чтобы получить значение показателя степени $e$,
нужно отнять от хранимого значения $127$.

К особенностям стандарта IEEE 754 можно отнести наличие нечисловых значений
среди чисел с плавающей запятой.
В частности, выделяют положительное нулевое значение $+0$ и отрицательное
нулевое значение $-0$, которые призваны выражать положительные и отрицательные
бесконечно малые величины.
Кроме бесконечно малых значений имеются также бесконечно
большие значения со знаком $+\infty$, $-\infty$.
Арифметические операции на них определены согласно известным из мат. анализа
утверждениям:

$$1/(+0)=+\infty,\quad 1/(-0)=-\infty,$$
$$1/(+\infty)=+0,\quad 1/(-\infty)=-0,$$
$$+\infty+\infty=+\infty,\quad-\infty-\infty=-\infty,$$
$$1+\infty=+\infty,\quad 1-\infty=-\infty,..$$

В ряде случаев ответ невозможно предъявить однозначно, в этом случае
возвращается специальное значение $NaN$, выражающее 
неопределенность, например:

$$\infty-\infty=NaN,\quad 0/0=NaN.$$

Наличие перечисленных нечисловых значений позволяет облегчить отладку кода
и отслеживать некорректные числовые значения во время выполнения программы.
При работе с этими значениями следует учитывать некоторые следующие особенности 
сравнения особых значений:

$$+0=-0,\quad NaN\neq NaN.$$

Обратимся к точности представления вещественных чисел числами
с плавающей запятой.
Рассмотрим два соседних числа с плавающей запятой.
Ясно, что чтобы получить следующее число, необходимо сделать приращение
для младшего бита мантиссы.
Если сделать такое приращение для $1$, то получим число
$1+\epsilon$, где $\epsilon=2^{-N}$ называется *машинной погрешностью*,
$N$ - число бит мантиссы (включая не хранимый в памяти бит $m_1$).
Однако следующее за $2$ число будет $2+2\epsilon$, так как приращение
мантиссы (снова `\epsilon`) необходимо умножить на экспоненту,
которая в этот раз больше.
Таким образом мы видим, что поставленная в предыдущей главе цель достигнута,
и плотность расположения чисел с плавающей запятой приблизительно 
обратно пропорциональна величине числа.

Рассмотрим функцию $F(x)$ округляющую вещественное число до ближайшего
числа с плавающей запятой.
Так как абсолютная погрешность округления в два раза меньше
расстояния между соседними числами, то для чисел $x$ близких к $1$
абсолютная погрешность округления на превосходит половины машинной точности:

$$|F(x)-x|\leq\epsilon/2.$$

Как мы отметили выше, расстояние между соседними вещественными числами примерно
пропорционально их величине, тогда для любого числа $x$ имеем:

$$|F(x)-x|\leq|x|\epsilon/2.$$

Если $x_0$ точное значение, а $x$ - приближенное, то
величина $|x-x_0|/|x|$ называется *относительной ошибкой*.
Т.е. мы можем утверждать, что относительная ошибка округления до числа 
с плавающей запятой не превосходит половины машинной точности для всех чисел
из диапазона, пробегаемого числами с плавающей запятой.

Обратимся теперь к точности арифметических операций.
Имея два числа точно представляемых в формате чисел с плавающей запятой,
мы можем выполнить над ними арифметическую операцию, результат которой
необязательно точно представляется числом с плавающей запятой,
например:

$$(1+10^{-n})^2=1+2\cdot 10^{-n}+10^{-2n},$$

где $n$ можно выбрать достаточно большим, чтобы последнее слагаемое попало в 
погрешность округления.
Это означает, что арифметические операции над числами с плавающей запятой 
обязательно привносят погрешности.
Идеальным ответом в данной ситуации было бы совпадения результата
операции с плавающей запятой с результатом округления вычислений в точной 
арифметике.
Стандарт IEEE 754 гарантирует, что погрешность будет ограничена сверху,
а именно, пусть $\star$ обозначает одну из четырех арифметических операций
в точной арифметике, а $\star$_F` обозначает реализацию этой операции над
числами с плавающей запятой, и пусть $x$ и $y$ числа,
имеющие точное представление с плавающей запятой, тогда:

$$|F(x\star y) - x \star_F y|/|F(x \star y)|\leq\epsilon.$$

Следует отметить, что если числа $x$ и (или) $y$ не имели точного представления
числом с плавающей запятой, то погрешность вычислений будет больше,
так как даже в точной арифметике операции могут увеличивать погрешность
аргументов.
Изучению этого вопроса будет посвящена следующая лекция.

## Вопросы для закрепления

* Когда лучше использовать числа с фиксированной запятой, а когда с плавающей?
* Какое число с плавающей запятой следует за 0 и отлично от нуля?
* Предъявите самое большое число с плавающей запятой, отличное от бесконечности. Какое число с плавающей запятой идет перед ним?
* Какие операции возвращают NaN?
* Из каких частей складывается погрешность операций над числами с плавающей запятой?
* Может ли результат арифметической операции быть точнее аргументов?
* Может ли результат вычислений с плавающей запятой зависеть от порядка вычислений, если в точной арифметике для тех же операций результат не зависит от их порядка? Как сильно результат может отличаться?

## Полезные ссылки

* [GMP](http://gmplib.org/) - реализация длинных чисел для C.


