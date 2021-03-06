# Эквивалентные преобразования и "обход" истинных зависимостей

__Принцип__. Если истинную зависимость сделаем loop-independent, мы её обойдём.

__Определение. Эквивалентное преобразование__ - изменение динамического порядка операторов без изменения графа алгоритмов.

__Фундаментальная теорема.__ Если после изменения динамического порядка операторов не меняются зависимости по данным - граф тоже не меняется.

## Перестановка циклов

```fortran
do i=1,a
	do j=1,b
		do k=1,c
			a(i,j,k) = a(i,j-1,k+1) * 2
		end do
	end do
end do
```

По индексу i распараллеливание без ограничений, по k или обоим - с барьерными синхронизациями после цикла по k.

**Что делаем:**  

Иногда бывает важно, по какой переменной распараллеливать - например, если для каких-то конкретных значений оператор выполняется сильно дольше.

Если нам ну смерть как надо параллелить именно по k, потребуется **a * b** барьерных синхронизаций.
Можем перенести цикл по **i** вовнутрь - и тогда **b** барьерных синхронизаций. PROFIT!!

## Замена переменной

```fortran
do i=1,a
	do j=1,b
		a(i,j) = a(i-1,j-1) * 2
	end do
end do
```

Можно сделать замену - например, `p = i + j, q = i - j`. Тогда вызов оператора будет примерно таким:

`b(p,q) == a(i,j) = a(i-1,j-1) * 2 = b(p-2,q) * 2`

## Разделение цикла

[ДАННЫЕ УДАЛЕНЫ]

## Выравнивание цикла

```fortran
do i=1,n
	a(i) = d(i) + 5 * i
	c(i) = a(i - 1) * 2
end do
```

Здесь есть истинная зависимость. Можно сделать такое преобразование:

```fortran
c(1) = a(0) * 2
a(n) = d(n) + 5*n

do i=1,n-1
	a(i) = d(i) + 5 * i
	c(i + 1) = a(i) * 2
end do
```

Или такое:

```fortran
do i=0,n
	if (i != 0)
		a(i) = d(i) + 5 * i
	if (i != n)
		c(i + 1) = a(i) * 2
end do
```

Истинная зависимость стала loop-independent за счёт увеличения накладных расходов.

## Репликация кода

```fortran
do i=1.n
	a(i + 1) = d(i) + 5 * i
	c(i) = a(i + 1) * 2 + a(i)
end do
```

Преобразуем:

```fortran
do i=1.n
	a(i + 1) = d(i) + 5 * i
	if (i == 1)
		tmp(i) = a(1)
	else
		tmp(i) = d(i - 1) + 5 * i
	c(i) = a(i + 1) * 2 + tmp(i)
end do
```

Важно понимать, что формально мы преобразуем алгоритм, но на абсолютно ему эквивалентный.

__Основная теорема.__
Всё это работает, если расстояние зависимости не зависит от индекса, а рекурсивные зависимости отсутствуют.

## Приватизация

```fortran
var tmp
do i=1,n
	tmp = b(i)
	b(i) = a(i)
	a(i) = tmp
end do
```

Напрямую не распараллеливается, поскольку `tmp` одно на все потоки.
Решение очевидно:

```fortran
do i=1,n
	private tmp
	tmp = b(i)
	b(i) = a(i)
	a(i) = tmp
end do
```

Иными словами, завели своё `tmp` на каждого исполнителя.

----

[На главную](../Readme.md)