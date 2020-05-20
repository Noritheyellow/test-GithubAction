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
~~~yml
# name of your github action
name: CI
# this will help you specify where to run
on: push

# this is where the magic happens, each job happens in parallel btw
jobs:
  build_on_mac:
    runs-on: macOS-latest
    steps:
      - uses: actions/checkout@v2
        with:
          ref: electron
      - uses: actions/setup-node@v1
        with:
          node-version: 10.16
      - name: see directory in electron_dist
        run: ls
      - name: add key to single keychain
        # what's this? install security?
#        run: security import ./dist/june-ai-single2-certs-electron.p12 -P ${{ secrets.CSC_KEY_PASSWORD }}
        run: fastlane run create_keychain name:electron-macKey password:123456
      - name: electron mac os security identities
        run: security find-identity -v
      - name: Install dependencies
        run: yarn install
        # now building
#      - name: Build on MacOS
#        env:
#          ELECTRON: true
#          APP_VERSION_NUMBER: 1.0.0
#        # 'build'라는 명령은 default가 아니다. package.json의 script를 수행하는 것이다.
#        run: yarn build
      - name: Build Electron
        env:
          ELECTRON: true
          CSC_KEY_PASSWORD: ${{ secrets.CSC_KEY_PASSWORD }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          APP_VERSION_NUMBER: 1.0.0
        run: yarn electron:buildUnix
      - name: see directory
        run: ls
      - name: check env
        run: echo $ELECTRON $FEATHERS_URL
      - name: see directory in electron_dist
        run: ls ./dist
        # A Github Action that uploads a file to a new release.
#      - uses: lucyio/upload-to-release@master
#        with:
#          name: Noritheyellow/test-GithubAction
#          path: ./dist
#          action: unpublished
#          release_id: 1.0.0
#          release-repo: Noritheyellow/test-GithubAction
      - uses: JasonEtco/upload-to-release@master
        with:
          args:
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  build_on_win:
    # windows 2019 has too many dependency issues & github windows server is 2016
    runs-on: windows-2016
    steps:
      - uses: actions/checkout@v2
        with:
          ref: electron
      - uses: actions/setup-node@v1
        with:
          node-version: 10.16
      # 4.0.0 설치 이유: windows 10이 아니어서 걸린다.
      - name: install node tools
        run: npm install --global --production windows-build-tools@4.0.0
      - name: install node-gyp
        run: npm install --global node-gyp@latest
      # 몇몇 electron pkg는 이게 없으면 not build properly(comment it if you dont need)
      - name: Set node config to set msvs_version to 2015
        run: npm config set msvs_version 2015
      # windows의 node는 electron을 build할 때 vs lib를 사용하기도 한다.(필요없으면 comment)
#      - name: Work around for Windows Server 2019
#        run: set path=%path%;C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Current\Bin
      - name: Install dependencies
        run: yarn install
      # window에서 환경 변수 확인은 'set'으로 함.
#      - name: check env
#        env:
#          ELECTRON: true
#          APP_VERSION_NUMBER: 0.5.9
#        run: set
#      - name: Build on Windows
#        env:
#          ELECTRON: true
#          APP_VERSION_NUMBER: 0.5.9
#        run: yarn build
      - name: Build Electron
        env:
          ELECTRON: true
          APP_VERSION_NUMBER: 0.5.9
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: yarn electron:buildWin
      - name: see directory
        run: cd ./dist && dir
        # 잘 되긴 하는데 이건 작성자가 만든 action이므로 다른 더 안정적인 걸 추천하고 있음.
#      - uses: lucyio/upload-to-release@master
#        with:
#          name: Noritheyellow/test-GithubAction
#          path: ./dist/squirrel-windows
#          action: unpublished
#          release_id: 1.0.0
#          release-repo: Noritheyellow/test-GithubAction
      - uses: JasonEtco/upload-to-release@master
        with:
          args:
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
~~~

## [Electron Builder Action](https://github.com/marketplace/actions/electron-builder-action)

~~~
git commit -am v1.2.3 : 커밋 메세지 작성 + add commit을 한 번에 수행
~~~

* package.json
~~~json
{
  "name": "test-githubaction",
  "version": "1.0.1",
  "description": "",
  "main": "index.js",
  "scripts": {
    "postinstall": "electron-builder install-app-deps",
    "start": "electron .",
    "build": "electron-builder",
    "release": "electron-builder --mac --windows --linux",
    "electron:buildUnix": "electron-builder -m --publish always",
    "electron:buildWin": "electron-builder -w --publish always"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/Noritheyellow/test-GithubAction.git"
  },
  "author": "",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/Noritheyellow/test-GithubAction/issues"
  },
  "homepage": "https://github.com/Noritheyellow/test-GithubAction#readme",
  "devDependencies": {
    "electron": "^9.0.0",
    "electron-builder": "^22.6.1"
  },
  "dependencies": {
    "g": "^2.0.1",
    "yarn": "^1.22.4"
  },
  "build": {
    "appId": "nori",
    "win": {
      "target":"nsis",
      "icon": "build/icon.ico"
    },
    "mac": {
      "target": "dmg",
      "icon": "build/icon.icns"
    }
  }
}
~~~

* build.yml
~~~yml
name: Build/release

on: push

jobs:
  release:
    runs-on: ${{ matrix.os }}

    strategy:
      matrix:
        os: [macos-latest, windows-latest]

    steps:
      - name: Check out Git repository
        uses: actions/checkout@v1

      - name: Install Node.js, NPM and Yarn
        uses: actions/setup-node@v1
        with:
          node-version: 10

      - name: Build/release Electron app
        uses: samuelmeuli/action-electron-builder@v1
        with:
          # GitHub token, automatically provided to the action
          # (No need to define this secret in the repo settings)
          github_token: ${{ secrets.github_token }}

          # If the commit is tagged with a version (e.g. "v1.0.0"),
          # release the app after building
          release: ${{ startsWith(github.ref, 'refs/tags/v') }}
~~~
