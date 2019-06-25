# Поиск строки в строке

Рассмотрим задачу, которая возникает каждый раз, когда вы делаете `ctrl+f`:

> Есть большой текст $t$. Нужно найти все вхождения строки $s$ в него.

Наивное решение со сравнением всех подстрок $t$ длины $|s|$ со строкой $s$ работает за $O(|t| \cdot |s|)$. Если текст большой, то длинные слова в нем искать становится очень долго.

Однако существует множество способов решить эту задачу за $O(|s| + |t|)$, два самых распространённых и простых из них: через *префикс-функцию* и через *z-функцию* (*примечание: не «зи», а «зет»*).

## Префикс-функция

**Определение**. Префикс-функцией от строки $s$ называется массив $p$, где $p_i$ равно длине самого большого префикса строки $s_0 s_1 s_2 \ldots s_i$, который также является и суффиксом $i$-того префика (не считая весь $i$-й префикс).

Например, самый большой префикс, который равен суффиксу для строки «aataataa» — это «aataa»; префикс-функция для этой строки равна $[0, 1, 0, 1, 2, 3, 4, 5]$.

```c++
vector<int> slow_prefix_function(string s) {
    int n = (int) s.size();
    vector<int> p(n, 0);
    for (int i = 1; i < n; i++)
        for (int len = 1; len <= i; len++)
            // если префикс длины len равен суффиксу длины len
            if (s.substr(0, len) == s.substr(i - len + 1, len))
                p[i] = len;
    return p;
}
```

Этот алгоритм пока что работает за $O(n^3)$, но позже мы его ускорми.

### Как это поможет решить исходную задачу?

Давайте пока поверим, что мы умеем считать префикс-функцию за линейное от размера строки, и научимся с помощью нее искать подстроку в строке.

Соединим подстроки $s$ и $t$ каким-нибудь символом, который не встречается ни там, ни там — обозначим пусть этот символ #. Посмотрим на префикс-функцию получившейся строки $s\#t$.

```c++
string s = "choose";
string t =
    "choose life. choose a job. choose a career. choose a family. choose a fu...";

cout << s + "#" + t << endl;
cout << slow_prefix_function(s + "#" + t) << endl;
```

```bash
choose#choose life. choose a job. choose a career. choose a family. choose a fu...
0000000123456000000012345600000000123456000100000001234560000000000012345600000000
```

Видно, что все места, где значения равны 6 (длине $s$) — это концы вхождений $s$ в текст $t$.

Такой алгоритм (посчитать префикс-функцию от $s\#t$ и посмотреть, в каких позициях она равна $|s|$) называется **алгоритмом Кнута-Морриса-Пратта**.

### Как её быстро считать

Рассмотрим ещё несколько примеров префикс-функций и попытаемся найти закономерности:

```bash
aaaaa
01234

abcdef
000000

abacabadava
00101230101
```

Можно заметить следующую особенность: $p_{i+1}$ максимум на единицу превосходит $p_i$.

**Доказательство.** Если есть префикс, равный суффиксу строки $s_{:i+1}$, длины $p_{i+1}$, то, отбросив последний символ, можно получить правильный суффикс для строки $s_{:i}$, длина которого будет ровно на единицу меньше.

Попытаемся решить задачу с помощью динамики: найдём формулу для $p_i$ через предыдущие значения.

Заметим, что $p_{i+1} = p_i + 1$ в том и только том случае, когда $s_{p_i} =s_{i+1}$. В этом случае мы можем просто обновить $p_{i+1}$ и пойти дальше.

Например, в строке $\underbrace{aabaa}t\overbrace{aabaa}$ выделен максимальный префикс, равный суффиксу: $p_{10} = 5$. Если следующий символ равен будет равен $t$, то $p_{11} = p_{10} + 1 = 6$.

Но что происходит, когда $s_{p_i}\neq s_{i+1}$? Пусть следующий символ в этом же примере равен не $t$, а $b$.

* $\implies$ Длина префикса, равного суффиксу новой строки, будет точно меньше 5.
* $\implies$ Помимо того, что искомый новый супрефикс является суффиксом «aabaa**b**», он ещё является префиксом подстроки «aabaa».
* $\implies$ Значит, следующий кандидат на проверку — это значение префикс-функции от «aabaa», то есть $p_4 = 2$, которое мы уже посчитали.
* $\implies$ Если $s_2 = s_{11}$ (т. е. новый символ совпадает с идущим после префикса-кандидата), то $p_{11} = p_2 + 1 = 2 + 1 = 3$.

В данном случае это действительно так (нужный префикс — «aab»). Но что делать, если, в общем случае, $p_{i+1} \neq p_{p_i+1}$? Тогда мы проводим такое же рассуждение и получаем нового кандидата, меньшей длины — $p_{p_{p_i}}$. Если и этот не подошел — аналогично проверяем меньшего, пока этот индекс не станет нулевым.

```c++
vector<int> prefix_function(string s) {
    int n = (int) s.size();
    vector<int> p(n, 0);
    for (int i = 1; i < n; i++) {
        // префикс функция точно не больше этого значения + 1
        int cur = p[i - 1];
        // уменьшаем cur значение, пока новый символ не сматчится
        while (s[i] != s[cur] && cur > 0)
            cur = p[cur - 1];
        // здесь либо s[i] == s[cur], либо cur == 0
        if (s[i] == s[cur])
            p[i] = cur + 1;
    }
    return p;
}
```

**Асимптотика.** В худшем случае этот `while` может работать $O(n)$ раз за одну итерацию, но *в среднем* каждый `while` работает за $O(1)$.

Префикс-функция каждый шаг возрастает максимум на единицу и после каждой итерации `while` уменьшается хотя бы на единицу. Значит, суммарно операций будет не более $O(n)$.

## Z-функция

Немногого более простая для понимания альтернатива префикс-функции — z-функция.

Z-функция от строки $s$ определяется как массив $z$, такой что $z_i$ равно длине максимальной подстроки, **начинающейся** с $i$-й позиции, которая равна префиксу $s$.

$$
\underbrace{aba}c\overbrace{aba}daba \hspace{1em} (z_4 = 3)
$$

```c++
vector<int> slow_z_function (string s) {
    int n = (int) s.size();
    vector<int> z(n, 0); // z[0] считается не определенным
    for (int i = 1; i < n; i++)
        // если мы не вышли за границу и следующие символы совпадают
        while (i + z[i] < n && s[z[i]] == s[i + z[i]])
            z[i]++;
    return z;
}
```

```bash
aaaaa
04321

abcdef
000000

abacabadava
00103010101
```

Z-функцию можно использовать вместо префикс-функции в алгоритме Кнута-Морриса-Пратта — только теперь нужные позиции будут начинаться c $|s|$, а не заканчиваться. Осталось только научиться её искать за $O(n)$.

### Как её быстро считать

Будем идти слева направо и хранить *z-блок* — самую правую подстроку, равную префиксу, которую мы успели обнаружить. Будем обозначать его границы как $l$ и $r$ включительно.

Пусть мы сейчас хотим найти $z_i$, а все предыдущие уже нашли. Новый $i$-й символ может лежать либо правее z-блока, либо внутри него:

* Если правее, то мы просто наивно перебором найдем $z_i$ (максимальный отрезок, начинающийся с $s_i$ и равный префиксу), и объявим его новым z-блоком.
* Если $i$-й элемент лежит внутри z-блока, то мы можем посмотреть на значение $z_{i-l}$ и использовать его, чтобы инициализировать $z_i$ чем-то, возможно, отличным от нуля. Если $z_{i-l}$ левее правой границы $z$-блока, то $z_i = z_{i-l}$ — больше $z_i$ быть не может. Если он упирается в границу, то «обрежем» его до неё и будем увеличивать на единичку.

```c++
vector<int> z_function (string s) {
    int n = (int) s.size();
    vector<int> z(n, 0);
    int l = 0, r = 0;
    for (int i = 1; i < n; i++) {
        // если мы уже видели этот символ
        if (i <= r)
            // то мы можем попробовать его инициализировать z[i - l],
            // но не дальше правой границы: там мы уже ничего не знаем
            z[i] = min(r - i + 1, z[i - l]);
        // дальше каждое успешное увеличение z[i] сдвинет z-блок на единицу
        while (i + z[i] < n && s[z[i]] == s[i + z[i]])
            z[i]++;
        // проверим, правее ли мы текущего z-блока
        if (i + z[i] - 1 > r) {
            r = i + z[i] - 1;
            l = i;
        }
    }
    return z;
}
```

**Асимптотика**. В алгоритме мы делаем столько же действий, сколько раз сдвигается правая граница z-блока — а это $O(n)$.

### Сравнение

В целом они зет- и префикс-функции очень похожи, но алгоритм Кнута-Морриса-Пратта есть во всех классических учебниках по программированию, а про z-функцию почему-то мало кто знает кроме олимпиадных программистов.

Про префикс-функцию важно ещё знать, что она онлайновая — достаточно считать следующий символ, и сразу можно узнать значение.

**Упражнение 1.** Дан массив префикс-функции. Исходная строка не дана. Вычислите за $O(n)$ зет-функцию этой строки.

**Упражнение 2.** Дан массив зет-функции. Исходная строка не дана. Вычислите за $O(n)$ префикс-функцию этой строки.