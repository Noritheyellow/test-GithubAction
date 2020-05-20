# TEST GITHUB ACTIONS FOR ALL PLATFORM
* Objective : electron app을 mac, windows 각 platform에 맞게 release를
시키고 빌드 자동화를 github actions로 구현하자.

## Prerequisites
* 일렉트론 앱 build가 이미 구현되어 있어야 한다.

* electron-builder를 사용한다.

## [Getting started with Electron - electron-builder](https://www.youtube.com/watch?v=XEBcBEM9Zj4)
1) electron을 설치하고 기본적인 package.json config를 한다.

2) electron-builder를 설치한다.

3) "build" config를 만든다. 이를 통해 electron-builder에게 구체적으로 어떤 일을 시킬건지 설정할 수 있다.

* 참조:
    - https://www.electron.build/
    - https://www.electron.build/cli#targetconfiguration
    - https://www.electron.build/configuration/configuration#configuration
    - https://www.electron.build/multi-platform-build

#### Note
* [Create DMG installer from windows](https://github.com/electron-userland/electron-builder/issues/1591)
DMG tool은 오직 macOS에서만 작동한다.

* Windows에서 macOS의 빌드를 지원하지 않는건가...?
[electron-builder create .dmg on windows](https://stackoverflow.com/questions/57982770/electron-builder-create-dmg-on-windows)
~~~
⨯ Build for macOS is supported only on macOS, please see https://electron.build/multi-platform-build
~~~

## [Building an electron app on github actions! Windows and MacOS](https://medium.com/@johnjjung/building-an-electron-app-on-github-actions-windows-and-macos-53ab69703f7c)
