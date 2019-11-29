---
layout: post
title:  "DLL 과 리플렉션을 이용한 플러그인 만들기"
subtitle:   "플러그인 만들기"
categories: develop
tags: csharp
---

윈폼을 이용하여 프로그램을 만들다 프로그래밍을 할 수 있는 사람들이 자신만의 컨트롤러를 만들어서 확장 가능하게 하면 어떨까?

라는 생각을 가지고 찾아본 방법을 정리한 글이다.

일단 c#에서는 ``System.Reflection.Assembly``을 이용하여 다른 Assembly의 내용을 불러 올 수 있다.

즉 ``System.Reflection.Assembly``을 이용하면 DLL 파일과 클래스 이름으로 객체를 만들 수 있다는 것이다.

```csharp
public static T LoadItem<T>(string fileName, string name, params object[] initData)
{
    Type interfaceType = typeof(T);

    foreach (var type in System.Reflection.Assembly.LoadFile(fileName).GetTypes())
    {
         if (interfaceType.IsAssignableFrom(type) && interfaceType != type && type.Name == name)
        {
            T instance = (T) Activator.CreateInstance(type, initData);

            return instance;
        }
    }
    return default;
}
```

코드를 한번 설명해 보자면 특정 타입을 상속받는 타입을 찿아서 생성하는 코드이다.

3번 줄을 보면 제네릭을 통하여 받은 타입을 이용하여 Type 객체를 만들었다.  
해당 객체는 해당타입을 상속받는지 확인하는 용도로 사용한다.

다음으로 5번 줄을 보면 위에서 말한 ``System.Reflection.Assembly``의 ``LoadFile()`` 을 이용하여 Assembly를 로딩하고 모든 타입을 가져온다.

7번 줄을 보면 위에서 만든 ``interfaceType``와 ``Type.IsAssignableFrom`` 을 이용하여 지정된 형식의 인스턴스를 해당 형식 인스턴스에 할당 가능한지 확인한다.  

9번 줄 에서는 ``Activator.CreateInstance``를 이용해서 해당 타입의 객체를 만들어준다.

14번 줄에서는 ``default`` 를 이용하여 해당 타입의 기본 생성자를 호출한다.   
이는 제네릭을 이용하여 받은 타입이 NULL 을 허용하지 않는 값일수 있기 때문이다.

여기까지 만들었으면 플러그인을 로드하는 부분은 완성이다.

위의 코드를 이용하여 만든 [간단한 플러그인 시스텡 예제](https://github.com/Hot-key/PluginSystem)이다.  
관심이 있는 사람은 한번 코드를 확인해 보는 것도 좋을 것 같다.