---
layout: post
title:  "[Project Euler] - 프로젝트 오일러 3번 정답 및 풀이"
subtitle:   "가장 큰 소인수 구하기"
categories: ps
tags: euler
---

## 가장 큰 소인수 구하기

> 어떤 수를 소수의 곱으로만 나타내는 것을 소인수분해라 하고, 이 소수들을 그 수의 소인수라고 합니다.
> 예를 들면 13195의 소인수는 5, 7, 13, 29 입니다.
>
> 600851475143의 소인수 중에서 가장 큰 수를 구하세요.

3번 문제는 특정수의 소인수 중 가장 큰 수를 찾는 문제이다.

먼저 필자는 seq 구문과 제귀를 이용하여 소인수를 구하는 코드를 만들어 보았다.  

```fsharp
let rec factorization (num: int64) (div: int64) =
    seq{
        if(num > 1L) then
            match num % div with
            | 0L -> yield! factorization ((/) num div) div
                    yield div
            | _  -> yield! factorization num (div + 1L)
    }
```

코드 설명을 하자면 num % div 를 하여 값을 확인한다.  
만약 값이 0 이 아니면 div 값을 1 높인후 자기 자신을 다시 호출한다.  
값이 0이라면 해당 값으로 나눌 수 있다는 것이다.  
그러므로 num 값을 num / div 의 값으로 바꾸어 자기 자신을 다시 호출 한다.

위 코드를 실행하면 이런 출력을 얻을 수 있다.

```fsharp
factorization 600851475143L 2L
```
| index | value |
|-------|-------|
| 0     | 6857  |
| 1     | 1471  |
| 2     | 839   |
| 3     | 71    |

문제는 소인수 중 가장 큰 값을 구하는 것이므로 Seq.max를 이용하여 최대 값을 구한다.  

```fsharp
let solve (num: int64) : int64 =
    factorization num 2L
    |> Seq.max
```

```fsharp
solve 600851475143L
```
6857

