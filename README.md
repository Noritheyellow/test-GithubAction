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

* simple sample:
    간단하게 특정 branch의 list를 출력하는 action
~~~yml
# name of your github action
name: CI
# this will help you specify where to run
on:
  push:
    branches:
      - electron

# this is where the magic happens, each job happens in parallel btw
jobs:
  build_on_mac:
    # github에서는 macOS, Windows, Linux를 전부 지원한다고 한다.
    # github가 제공해주는 platform 중 무엇 위에서 돌릴 것인지다.(runs-on)
    runs-on: macOS-latest
    steps:
      # 'uses'가 actions/checkout@v2 라는 프로젝트를 가져와서 실행한다.
      # 이것은 어떤 github action's team이 제작한 것으로
      # 내 저장소를 '$GITHUB_WORKSPACE'라는 곳으로 checkout 시켜서 내 workflow가
      # access 가능하도록 돕는다.
      - uses: actions/checkout@v2
        with:
          # 'electron' branch로 checkout 시킨다.
          ref: electron
      # node env를 setting하는 action이다.
      - uses: actions/setup-node@v1
        with:
          node-version: 10.16
      - name: see directory
        # ls -al을 실행한다.
        run: ls -al

  build_on_win:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v2
        with:
          ref: electron
      - uses: actions/setup-node@v1
        with:
          node-version: 10.16
      - name: see directory
        # UNIX 기반이 아니므로 dir를 실행한다.
        run: dir
~~~

* main action

## [Electron Builder Action](https://github.com/marketplace/actions/electron-builder-action)

~~~
git commit -am v1.2.3 : 커밋 메세지 작성 + add commit을 한 번에 수행
~~~
