# Waves Control GUI system plugin

waves 환견을 제어하기 위한 GUI system plugin이다. Gazebo Sim 예제인 `gz-sim/examples/plugin/gui_system_plugin`을 기반으로 하며 이 예제 plugin에서 서버로부터 entity와 component 업데이트에 접근하는 방법을 보여준다.

추가 정보는 `WavesControl.hh`를 참조한다.

## Build

`asv_wave_sim` repo의 루트에서  아래 빌드 명령을 수행한다.:

```bash
$ cd gz-waves/src/gui/plugin/waves_control
$ mkdir build
$ cd build
$ cmake ..
$ make
```
`build` 아래에 `WavesControl` library를 생성한다.

## Run

`GZ_GUI_PLUGIN_PATH`에 library를 추가한다.:

```bash
$ cd gz-waves/src/gui/plugin/waves_control
$ export GZ_GUI_PLUGIN_PATH=$(pwd)/build
```

다음으로 아래와 같이 world를 실행한다. :

```
$ gz sim -v4 waves.sdf
```

오른쪽 위에 있는 GUI plugin 메뉴에서 "Waves Control"를 선택한다.