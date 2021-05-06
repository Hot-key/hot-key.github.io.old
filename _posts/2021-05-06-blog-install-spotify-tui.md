---
layout: post
title: "cli로 스포티파이 사용하기"
subtitle: "Spotifyd 와 spotify-tui 설치하기"
categories: blog
tags: blog
---

# cli로 스포티파이 사용하기

## 설치한 이유

---

요즘은 16기가 메모리도 부족한 세상이 되어간다.  
가상메모리 등의 방법으로 사용할 메모리를 늘릴 수도 있지만 그런다고 근본적인 문제가 해결되는 건 아니다.

![fuck electron](/assets/img/blog/Install-spotify-tui/memory.png)

이 모든 일의 원흉은 Electron 때문이다.  
Electron 앱은 엄청난 양의 메모리를 점유하면서 전염병처럼 퍼져 나가고 있다.  
하지만 Electron 앱을 사용 안 할 수는 없으니 이런 상황에서 사용되는 메모리를 일부라도 절약해보기로 하였다.

## Spotifyd 설치하기

---

먼저 Spotifyd를 설치해서 Electron을 사용하지 않고 Spotify에서 노래를 들을 수 있도록 만들어 보겠다.

Spotifyd 는 UNIX 데몬으로 실행되는 오픈 소스 Spotify 클라이언트로 메모리 절약이 가능하다.  
참고로 Spotifyd를 사용하기 위해서는 Spotify Premium 계정이 필요하다.

처음은 Spotifyd를 설치하는 것 부터 시작해보자.  
[Spotifyd 깃허브](https://github.com/Spotifyd/spotifyd)
Spotifyd 위키의 설치 지침에서는 클론 후 빌드하는 것이지만 필자는 깃허브에서 받아서 사용했다.  
https://github.com/Spotifyd/spotifyd/releases

위 페이지에서 일치하는 것을 받아서 사용하면 된다.  
필자는 `spotifyd-linux-full.tar.gz` 을 다운 받았다.

이후 위 파일의 압축을 적당한 곳에 풀면 된다.

다음으로 설정을 해보겠다.

`~/.config/spotifyd/spotifyd.conf`위치에 파일을 만든다.

[구성 파일](https://spotifyd.github.io/spotifyd/config/File.html)
에서 모든 설정을 볼 수 있지만 필자의 구성을 보여주겠다.

```
[global]
# Your Spotify account name.
username = "{이메일 주소}"

# Your Spotify account password.
password = "********"

# How this machine shows up in Spotify Connect.
device_name = "spotifyd"
device_type = "computer"

#device = "hw:CARD=DeviceEEPROM,DEV=0"
#control = "hw:CARD=DeviceEEPROM,DEV=0"
# This is the default location of Spotify's cache, so just replace LOGIN_NAME
# with your macOS login name (type `whoami` at a Terminal window).
cache_path = "/home/hotkey/.Temp/Spotifyd"
no_audio_cache = false

# Various playback options. Tweak these if Spotify is too quiet.
bitrate = 320
volume_normalisation = true
normalisation_pregain = -5

# These need to be set, but don't need to be changed.
backend = "alsa"
mixer = "PCM"
volume_controller = "softvol"
zeroconf_port = 1234
```

위 구성에서 username 과 password 만 수정해도 바로 사용 가능 할 것이다.  
그러면 잘 돌아가는지 확인해보자.

```
➜  ~ spotifyd --no-daemon
Loading config from "/home/hotkey/.config/spotifyd/spotifyd.conf"
No proxy specified
Using software volume controller.
Connecting to AP "gae2-accesspoint-e-2jhh.ap.spotify.com:443"
Authenticated as
Using alsa sink
Country: "KR"
```

구성이 정상적으로 성정되었으면 위와 같은 출력을 얻을 수 있다.  
![spotify-select-device](/assets/img/blog/Install-spotify-tui/spotify-select-device.png)

또한 이처럼 기존 Spotify 클라이언트에서 볼 수 있다.  
여기까지 진행했으면 스마트폰이나 다른 기기로도 곡을 선택 할 수 있으므로 이걸로 충분하다면
아래 내용은 진행 할 핋요 없다.

## spotify-tui 설치하기

---

다음으로 spotify-tui를 설치해보겠다.  
spotify-tui는 터미널에서 사용 가능한 Spotify 클라이언트다.

![spotify-tui](https://user-images.githubusercontent.com/12150276/75177190-91d4ab00-572d-11ea-80bd-c5e28c7b17ad.gif)

설치 방법은 여러가지가 있지만 필자는 Snap 을 사용하여 설치 하였다.

```
➜  ~ snap install spt

...

➜  ~ spt

   _________  ____  / /_(_) __/_  __      / /___  __(_)
  / ___/ __ \/ __ \/ __/ / /_/ / / /_____/ __/ / / / /
 (__  ) /_/ / /_/ / /_/ / __/ /_/ /_____/ /_/ /_/ / /
/____/ .___/\____/\__/_/_/  \__, /      \__/\__,_/_/
    /_/                    /____/

Config will be saved to /home/hotkey/snap/spt/current/.config/spotify-tui/client.yml

How to get setup:

  1. Go to the Spotify dashboard - https://developer.spotify.com/dashboard/applications
  2. Click `Create a Client ID` and create an app
  3. Now click `Edit Settings`
  4. Add `http://localhost:8888/callback` to the Redirect URIs
  5. You are now ready to authenticate with Spotify!
```

이후는 위에 나온 setup을 따라 하면 된다.  
Spotify 대시 보드에서 앱을 만들고 리디렉션 URI를 설정하면 끝이다.

설정이 끝나면 이와 같은 화면을 볼 수 있다.  
![spotify-tui](/assets/img/blog/Install-spotify-tui/spotify-tui.png)

UI가 못 볼 정도는 아니지만 상당히 끔찍하다...  
이를 한번 수정 해 보겠다.

필자와 똑같이 Snap 을 이용하여 설치 하였으면 해당 위치에 파일을 만들어서 설정 가능하다.  
`~/snap/spt/current/.config/spotify-tui/config.yml`

```
theme:
  active: "29, 185, 84" # current playing song in list
  banner: "29, 185, 84" # the "spotify-tui" banner on launch
  error_border: Red # error dialog border
  error_text: LightRed # error message text (e.g. "Spotify API reported error 404")
  hint: Yellow # hint text in errors
  hovered: White # hovered pane border
  inactive: "160, 160, 160" # borders of inactive panes
  playbar_background: Black # background of progress bar
  playbar_progress: "29, 185, 84" # filled-in part of the progress bar
  playbar_progress_text: "29, 185, 84" # song length and time played/left indicator in the progress bar
  playbar_text: "210, 210, 210" # artist name in player pane
  selected: "102, 213, 140" # a) selected pane border, b) hovered item in list, & c) track title in player
  text: "150, 150, 150" # text in panes
  header: "230, 230, 230" # header text in panes (e.g. 'Title', 'Artist', etc.)
```

![spotify-tui2](/assets/img/blog/Install-spotify-tui/spotify-tui2.png)

확실히 이전보다는 볼만 한것 같다.

[spotify-tui 깃허브](https://github.com/Rigellute/spotify-tui#configuration)
에 들어가면 theme 말고도 다른 여러가지 설정을 볼 수 있다.

그러면 널널해진 메모리로 다른 여러가지 작업을 해보자!

## 참고한 사이트

---

https://jonathanchang.org/blog/setting-up-spotifyd-on-macos/
