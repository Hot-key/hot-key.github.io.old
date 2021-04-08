---
layout: post
title: "[Project Euler] - 프로젝트 오일러 4번 정답 및 풀이"
subtitle: "세자리 수를 곱해 만들 수 있는 가장 큰 대칭수"
categories: ps
tags: euler
---

## 세자리 수를 곱해 만들 수 있는 가장 큰 대칭수

> 앞에서부터 읽을 때나 뒤에서부터 읽을 때나 모양이 같은 수를 대칭수(palindrome)라고 부릅니다.
> 두 자리 수를 곱해 만들 수 있는 대칭수 중 가장 큰 수는 9009 (= 91 × 99) 입니다.
>
> 세 자리 수를 곱해 만들 수 있는 가장 큰 대칭수는 얼마입니까?

4번은 3자리 수를 곱하여 대칭수를 찾는 것 이다

일단 3자리 수의 곱을 만드는 시퀀스를 만들어 보자

```fsharp
let multipleSeq =
    (999,999)
    |> Seq.unfold (fun(a,b) ->
            if a = 99 then None
            elif b = 99 then Some(a * b,(a - 1, 999))
            else Some(a * b,(a, b - 1)))
```

코드 설명을 하자면 a와 b 모두 999부터 시작해서 b를 1씩 줄여나간다.  
b가 99 가되면 a - 1 을 하고 b는 다시 999로 변경한다.  
이걸 a 가 99 될 때까지 반복한다

이러면 999x999 에서 100x100 까지의 값이 들어있는 시퀀스를 만들 수 있다

이번에는 대칭수를 찾아보자.  
대칭수는 나온 수를 문자열로 변경하여 뒤집은 다음에 비교하면 알 수 있다.  
이걸 Seq.filter 를 이용하여 찾는다.

```fsharp
let solve =
    multipleSeq
    |> Seq.filter(fun x ->
        let stringData = x.ToString()
        stringData = new string(stringData.Reverse().ToArray()))
```

문제는 이중에 가장 큰 값을 찾는거 니까 Seq.max를 이용하여 찾아준다.

```fsharp
let solve =
    multipleSeq
    |> Seq.filter(fun x ->
        let stringData = x.ToString()
        stringData = new string(stringData.Reverse().ToArray()))
    |> Seq.max
```

마지막으로 solve를 호출하면 답이 나온다

```fsharp
solve
```

> 906609

[소스코드](https://tio.run/##XZDBSsQwEIbvfYphQUgkLXgs6y6IghfxYH2BtDvRQJp0k3Ttwr57ncbqtg6EMDP/n/kyKuSN8ziOrkML1TlEbLfZIikenTHYRO1sKJ7RotfNWvCi7XGbZQYjtL2JujNY4RF2GVCwsiwFHZ6yyx6oVfRWOXMApnrLpKg55PvU/g2tQMIOyhLiJ815dRZXfTSkqBeKyrXIJNxCLejK4U7ANJP/cwVcKwU9QmJOwoQfnDnhDL74yhJdaRPRT@QwEDb8TUj@6LX9eJJxoh@Kd1elArtyrBQWv@YCu9aLNzyhD8g4@R@8l2fG@Wp7rRzmbUttZ9qO7FHB5uawgfvLz0fG8Rs)
