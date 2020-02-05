---
layout: post
title:  "[Project Euler] - 프로젝트 오일러 2번 정답 및 풀이"
subtitle:   "피보나치 수열에서 4백만 이하이면서 짝수인 항의 합"
categories: ps
tags: euler
---

## **피보나치 수열에서 4백만 이하이면서 짝수인 항의 합**

>피보나치 수열의 각 항은 바로 앞의 항 두 개를 더한 것이 됩니다. 1과 2로 시작하는 경우 이 수열은 아래와 같습니다.
>>1, 2, 3, 5, 8, 13, 21, 34, 55, 89, ...
>
>짝수이면서 4백만 이하인 모든 항을 더하면 얼마가 됩니까?

먼저 피보나치 수를 만들어주는 시퀀스를 만들어보자. 

피보나치 수열은 위에 나와있는것 처럼 바로 앞 2개의 향을 더한 것이다.  
이는 재귀함수를 통하여 만들 수 있다.

```fsharp
let rec fiboSeq(num1) (num2) (endNum) =
    seq{
        if(endNum > num2) then
            yield (num1 + num2)
            yield! fidoSeq1 num2 (num1 + num2) (endNum)
    }
```
위의 코드는 재귀를 이용하여 endNum까지 피보나치를 만드는 함수이다.  
저걸 그대로 사용해도 답을 구할 수 있지만 endNum 이라는 값이 큰 의미 없이 전달되는것 같다. 
한번 이걸 수정하여 보자.  

`Seq.unfold` 를 이용하여 계산 함수에서 시퀀스를 생성 하고 시퀀스의 후속 요소를 생성 하도록 만들 수 있다.

```fsharp
let fiboSeq = 
    Seq.unfold (fun(a,b) -> Some(a+b,(b, a+b))) (0,1)
```
위의 재귀를 이용한 방식보다 코드가 깔끔해졌다.  

`Seq.unfold` 함수는 Some을 이용하여 자기자신을 다시 계산하여 후속요소를 만드는 기능을 가지고 있다.  

Some의 첫번째 인자는 시퀀스를 통하여 반환할 값이다  
즉 위의 재귀코드에 있는 `yield (num1 + num2)` 부분과 동일하다.  

두번째 인자는 자신을 다시 호출하여 전달할 값이다.  
이는 재귀코드의 `yield! fidoSeq1 num2 (num1 + num2) (endNum)` 부분에서 호출을 위하여 사용하는 인자와 동일하다.

또한 None 값을 반환하여 시퀀스를 종료 할 수 있다.
`Seq.unfold` 의 뒤 `(1,1)`는 시퀀스를 처음 시작하기 위한 값이다.

피보나치 수열을 반환하는 무한 시퀀스를 만들었다.

다음으로는 무한 시퀀스에서 값을 추출해보자.

```fsharp
let solve endNum =
    fiboSeq
    |> Seq.takeWhile( fun x -> x <= endNum)
    |> Seq.filter(fun x -> x%2 = 0)
    |> Seq.sum
```
fiboSeq 는 무한 시퀀스이므로 그냥 사용 할 수는 없다.  
그러므로 `Seq.takeWhile`을 이용하여 특정 값까지 가져오는 작업을 한다.  
이후 `Seq.filter`로 위 문제의 조건으로 나와있는 짝수를 판별하여 `Seq.sum`에 넘겨준다.  
마지막으로 solve 함수를 호출하면 끝이다.

```fsharp
solve 4000000
```
4613732