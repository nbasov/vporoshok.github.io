---
title: Калькулятор 2000
description: Пишем простой калькулятор на Go, а заодно разбираемся с тем как строить архитектуру программы
date: 2019-01-27T08:14:35Z
draft: true
categories:
- develop
tags:
- go
- architect
- AST
- DSL
- functional programming
mermaid: true
---

Солнечный понедельник. Хорошо выспавшись в выходные, вы пришли на работу пораньше и, налив себе чашку чая, листаете свежие новости. ПО прогнозам неделя будет тёплой, и ничего не предвещает беды. Внезапно вас вызывает начальник. Отставив чашку и вооружившись блокнотом и ручкой, вы идёте к нему в кабинет.

## Надо сделать калькулятор

Примерно так звучит ТЗ. Но вы, уже наученные горьким опытом того, что ТЗ к задачам надо делать самому, аккуратно начинаете вытягивать из руководителя что же именно он подразумевает под калькулятором. Итак, вот что вам удаётся выяснить:
1. Калькулятор должен принимать на вход выражение записанное в человеко-понятном виде.
2. Должен поддерживать операции сложения, вычитания, умножения, деления, взятие процента и возведение в степень.

## Определяемся с интерфейсом

Итак, нам необходимо сделать пакет, в котором будет публичная функция:
```go
func Eval(expression string) (float64, error)
```

Ну, а в качестве `expression` будут выступать строчки вида:
- `8 / 12`;
- `22 + 6`;
- `2 * 30 - 12`;
- `14 ^ 2`;

Что-то вас гложет при виде этих примеров, но вы пока не можете понять что. Впрочем, некогда думать, надо делать. Но, помня про поговорку про «торопыжку», вы решаете применить лучшие практики и…

## Применяем TDD

Опишем пустую функцию `Eval`, чтобы код хоть как-то компилировался:
```go
// calc.go
package calc

func Eval(expression string) (float64, error) {
    _ = expression
    return 0, nil
}
```
и начнём писать тесты:
```go
// calc_test.go
package calc_test

import (
    "testing"

    "calc"
)

func TestCalc(t *testing.T) {
    cases := []struct{
        exp string
        res float64
        err string
    }{
        {"8 / 12", 8.0 / 12, ""},
        {"22 + 6", 28, ""},
        {"2 * 30 - 12", 48, ""},
        {"14 ^ 2", 196, ""},
        {
            "3 * foo 8", 0,
            `"3 * foo 8": unexpected symbol 'f' on position 4: syntax error`,
        },
    }

    for _, c := range cases {
        t.Run(c.exp, func(t *texting.T) {
            res, err := calc.Eval(c.exp)
            checkError(t, c.err, err)
            if res != c.res {
                t.Errorf(
                    "result of %q should be %f, but got %f",
                    c.exp, c.res, res)
            }
        })
    }
}

func checkError(t *testing.T, expected string, actual error) {
    t.Helper()
    if len(expected) > 0 {
        if actual == nil || err.Error() != c.err {
            t.Errorf("expected error %q but got %v", expected, actual)
        }
        return
    }
    if actual != nil {
        t.Errorf("unexpected error %v", actual)
    }
}
```

## Делим слона

Итак, нам на вход подаётся выражение в виде строки. Надо сначала его распарсить в AST, а затем вычислить. Итак, у нас появляется 2 отдельные задачи: parse и calculate. При этом задачи взаимосвязаны, потому что только зная что именно умеет считать calculate мы сможем корректно составить AST. Для начала давайте определимся с тем: что у нас будет храниться в узлах синтаксического дерева? В листьях операнды, то есть числа, а в остальных узлах операции. Например, дерево выражения `2 * 3` будет выглядеть:
{{<mermaid>}}
graph TD
    A{"&times;"}
    A-->x((2))
    A-->y((3))
{{</mermaid>}}

Но при таком подходе становится проблематичным универсальность этого дерева, ведь у операции сложения в качестве операнда может быть как число, так и результат другой операции, например, умножения. Значит нам необходим

### Универсальный интерфейс операции

Любая операция должна быть вычислима

## Приоритет операций. Добавляем стек

Кадр стека хранит пару из состояния и операции.
Вычитываем число — это текущее состояние. Если выражение вычитано до конца и стек пуст, возвращаем текущее состояние как результат.
Вычитываем операцию. Если её приоритет больше, чем у головы стека, то положить текущее состояние и операцию как новый кадр стека, иначе, возвращаемся по стеку до тех пор пока он не пуст и приоритет операции в голове стека больше или равен приоритету текущей операции.

## От стека к AST

## И тут появляются скобочки. Подвыражения

## Отрицательные числа и начальное состояние

## Унарные операции

## Ячейки памяти

## Расширяем список операций

<!-- А применив новейшие методы пыток с использованием отсылок к бизнес-ценности и юзер-стори, вы узнаёте также следующее:
<ol start="3">
<li>Должна быть ячейка памяти результата.
</ol>


1. Понадобится расширять функциональность, например, вводить произвольные степени, тригонометрические функции и логарифмы. -->

