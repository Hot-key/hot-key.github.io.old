---
layout: post
title:  "[WPF]에어로 글래스(Aero glass) 효과 만들기"
subtitle:   "Aero glass 효과 만들기"
categories: develop
tags: wpf
---

## Aero glass 효과란
윈도우 7 을 사용한 사람을 자주 보았을 효과이다.  
![폼](/assets/img/dev/wpf/aeroglass/ex1.png)  

위 그림처럼 뒤 화면이 가려지지 않고 유리를 올려놓은것 처럼 번져서보이는 효과이다. 

기존에는 [해당](https://docs.microsoft.com/ko-kr/dotnet/framework/wpf/graphics-multimedia/extend-glass-frame-into-a-wpf-application) 코드로 위 효과의 적용이 가능하였다.  

하지만 윈도우 10에서는 위의 코드로 작동하지 않는다. 

분명 해당 기능은 윈도우 10에서도 볼 수 있던 기능이다.  

그러면 누군가는 사용 할 수 있도록 만들지 않았을까 생각하고 구글링을 통하여 방법을 찾았다.  

이 글은 [여기서](https://withinrafael.com/2015/07/08/adding-the-aero-glass-blur-to-your-windows-10-apps/) 본 글을 정리 한 것이다.

## Aero glass 효과 적용하기

```csharp
class TransparencyConverter
{
    private readonly Window _window;

    public TransparencyConverter(Window window)
    {
        _window = window;
    }

    internal void EnableBlur()
    {
        var windowHelper = new WindowInteropHelper(_window);

        var accent = new AccentPolicy();
        var accentStructSize = Marshal.SizeOf(accent);
        accent.AccentState = AccentState.ACCENT_ENABLE_BLURBEHIND;

        var accentPtr = Marshal.AllocHGlobal(accentStructSize);
        Marshal.StructureToPtr(accent, accentPtr, false);

        var data = new WindowCompositionAttributeData();
        data.Attribute = WindowCompositionAttribute.WCA_ACCENT_POLICY;
        data.SizeOfData = accentStructSize;
        data.Data = accentPtr;

        SetWindowCompositionAttribute(windowHelper.Handle, ref data);

        Marshal.FreeHGlobal(accentPtr);
    }

    [DllImport("user32.dll")]
    internal static extern int SetWindowCompositionAttribute(IntPtr hwnd, ref WindowCompositionAttributeData data);

    [StructLayout(LayoutKind.Sequential)]
    internal struct WindowCompositionAttributeData
    {
        public WindowCompositionAttribute Attribute;
        public IntPtr Data;
        public int SizeOfData;
    }

    internal enum WindowCompositionAttribute
    {
        // ...
        WCA_ACCENT_POLICY = 19
        // ...
    }

    internal enum AccentState
    {
        ACCENT_DISABLED = 0,
        ACCENT_ENABLE_GRADIENT = 1,
        ACCENT_ENABLE_TRANSPARENTGRADIENT = 2,
        ACCENT_ENABLE_BLURBEHIND = 3,
        ACCENT_INVALID_STATE = 4
    }

    [StructLayout(LayoutKind.Sequential)]
    internal struct AccentPolicy
    {
        public AccentState AccentState;
        public int AccentFlags;
        public int GradientColor;
        public int AnimationId;
    }
}
```
위의 글에서 본 코드를 하나의 클래스로 만든 것 이다.

사용법은 단순하다. 

위 클래스를 만들고 폼이 로드될때 해당 코드를 적용시키면 된다.

```csharp
private void Window_Loaded(object sender, RoutedEventArgs e)
{
    var transparencyConverter = new TransparencyConverter(this);
    transparencyConverter.EnableBlur();
}
```

위 코드를 실행하고 나면 ```Background="Transparent"``` 인 부분에 Aero 효과가 적용된다. 

![폼](/assets/img/dev/wpf/aeroglass/test.png)  

위의 사진처럼 말이다. 

질문이 있으면 댓글로 남겨주면 아는범위 내에서 답변해 줄 수 있다.