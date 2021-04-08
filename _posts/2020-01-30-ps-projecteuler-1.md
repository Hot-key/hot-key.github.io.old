---
layout: post
title: "[Project Euler] - 프로젝트 오일러 1번 정답 및 풀이"
subtitle: "1000보다 작은 자연수 중에서 3 또는 5의 배수를 모두 더하면?"
categories: ps
tags: euler
---

## **1000보다 작은 자연수 중에서 3 또는 5의 배수를 모두 더하면?**

> 10보다 작은 자연수 중에서 3 또는 5의 배수는 3, 5, 6, 9 이고, 이것을 모두 더하면 23입니다.
>
> 1000보다 작은 자연수 중에서 3 또는 5의 배수를 모두 더하면 얼마일까요?

1번 문제는 간단하게 1부터 999까지 값을 올리면서 3과 5의 배수를 찾으면 된다.

필자는 약간 다른 방식을 이용하였다.

반복문을 이용하여 1000 까지 돌리는 것이 아닌 1 ~ 999 까지 들어있는 배열을 만들고  
`List.filter`를 이용하여 3과 5의 배수를 찾고 `List.sum` 을 이용하여 합을 구하였다.

```fsharp
let solve (startNum:int, endNum:int) : int =
    [startNum..(-) endNum 1]
    |> List.filter(fun(x)-> x % 3 = 0 || x % 5 = 0)
    |> List.sum
```

```fsharp
solve(1, 1000)
```

> 233168
