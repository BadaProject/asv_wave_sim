# Wave Sim

[![Ubuntu Jammy CI](https://github.com/srmainwaring/asv_wave_sim/actions/workflows/ubuntu-jammy-ci.yml/badge.svg)](https://github.com/srmainwaring/asv_wave_sim/actions/workflows/ubuntu-jammy-ci.yml)
[![macOS Monterey CI](https://github.com/srmainwaring/asv_wave_sim/actions/workflows/macos-monterey-ci.yml/badge.svg)](https://github.com/srmainwaring/asv_wave_sim/actions/workflows/macos-monterey-ci.yml)
[![Cpplint](https://github.com/srmainwaring/asv_wave_sim/actions/workflows/ccplint.yml/badge.svg)](https://github.com/srmainwaring/asv_wave_sim/actions/workflows/ccplint.yml)
[![Cppcheck](https://github.com/srmainwaring/asv_wave_sim/actions/workflows/ccpcheck.yml/badge.svg)](https://github.com/srmainwaring/asv_wave_sim/actions/workflows/ccpcheck.yml)

This package contains plugins that support the simulation of waves and surface vessels in [Gazebo](https://gazebosim.org/home).

![rs750_ardupilot_v3_upwind](https://user-images.githubusercontent.com/24916364/228044489-b434b1ae-c30f-4676-9415-1719ee75479b.gif)


여기서 main 브랜치는 [Gazebo Garden](https://gazebosim.org/docs/garden)을 타겟으로 하며 ROS에 대한 의존성이 더 이상 없다.

새로운 기능으로 FFT wave 생성 방법, ocean tiling, 랜더링 엔진으로 [Ogre2](https://github.com/OGRECave/ogre-next)을 지원한다. wave 파라미터 설정하는 방식이 약간 변경되었지만 이전 버전과의 호환성을 최대한 유지한다.

## Dependencies

- A working installation of [Gazebo Garden](https://gazebosim.org/docs/garden) or later including development symbols.

- The simulation uses the [CGAL](https://www.cgal.org/) library for mesh manipulation and [FFTW](http://www.fftw.org/) to compute Fourier transforms. Both libraries are licensed GPL-3.0.

## Ubuntu

- Ubuntu 22.04 (Jammy)
- Gazebo Sim, version 7.1.0 (Garden)

Install CGAL and FFTW:

```zsh
sudo apt-get update
sudo apt-get install libcgal-dev libfftw3-dev
```


## 설치

### workspace 생성

```bash
mkdir -p gz_ws/src
```

### Clone 및 build

`asv_wave_sim` repo clone 하기:

```bash
cd ~/gz_ws/src
git clone https://github.com/srmainwaring/asv_wave_sim.git
```

package 컴파일:

#### Ubuntu

```bash
colcon build --symlink-install --merge-install --cmake-args \
-DCMAKE_BUILD_TYPE=RelWithDebInfo \
-DBUILD_TESTING=ON \
-DCMAKE_CXX_STANDARD=17
```

Source the workspace:

```bash
source ./install/setup.bash
```
### GUI plugin 빌드하기 (옵션) 

wave 파라미터를 제어하는 GUI plugin

```bash
cd ~/gz_ws/src/asv_wave_sim/gz-waves/src/gui/plugins/waves_control 
mkdir build && cd build
cmake .. && make
```

## 사용법

### 환경변수 설정

```bash
# for future use - to support multiple Gazebo versions
export GZ_VERSION=garden

# not usually required as should default to localhost address
export GZ_IP=127.0.0.1

# ensure the model and world files are found
export GZ_SIM_RESOURCE_PATH=\
$GZ_SIM_RESOURCE_PATH:\
$HOME/gz_ws/src/asv_wave_sim/gz-waves-models/models:\
$HOME/gz_ws/src/asv_wave_sim/gz-waves-models/world_models:\
$HOME/gz_ws/src/asv_wave_sim/gz-waves-models/worlds

# ensure the system plugins are found
export GZ_SIM_SYSTEM_PLUGIN_PATH=\
$GZ_SIM_SYSTEM_PLUGIN_PATH:\
$HOME/gz_ws/install/lib

# ensure the gui plugin is found
export GZ_GUI_PLUGIN_PATH=\
$GZ_GUI_PLUGIN_PATH:\
$HOME/gz_ws/src/asv_wave_sim/gz-waves/src/gui/plugins/waves_control/build
```
## 예제

Server:

```bash
gz sim -v4 -s -r waves.sdf
```

Client:

```bash
gz sim -v4 -g
```

### Waves model과 visual plugins

- The `filename` and `name` attributes for the wave model and visal plugins have changed.
- The `<size>` element has been renamed to `<tile_size>` and moved into `<waves>`
- The `<cell_count>` element has been moved into `<waves>`
- Add new element `<algorithm>` to specify the wave generation algorithm. Valid options are: `sinusoid`, `trochoid` and `fft`.
- Add new element `<wind_velocity>` for use with the `fft` algorithm.
- Add new element `<wind_speed>` for use with the `fft` algorithm.
- Add new element `<wind_angle_deg>` for use with the `fft` algorithm.

```xml
<plugin
    filename="gz-waves1-waves-model-system"
    name="gz::sim::systems::WavesModel">
    <static>0</static>
    <update_rate>30</update_rate>
    <wave>
      <!-- Grid dimensions
        - The tile_size and cell_count may be a single value
          for square grids, or a 2d vector if different resolution
          is desired along the x and y axis.
        - The cell_count must be a power of 2 for fft waves
      -->
      <!-- Either: single value for square grids -->
      <tile_size>256.0</tile_size>
      <cell_count>128</cell_count>

      <!-- Or: 2d vectors for different resolution in each axis -->
      <tile_size>256.0 64.0</tile_size>
      <cell_count>128 32</cell_count>

      <!-- Wave algorithms
        - These elements specify the wave generation method
          and wave spectrum parameters.
      -->

      <!-- Either: `fft` waves parameters -->
      <algorithm>fft</algorithm>
      <wind_speed>5.0</wind_speed>
      <wind_angle_deg>135</wind_angle_deg>
      <steepness>2</steepness>

      <!-- Or: `trochoid` waves parameters -->
      <algorithm>trochoid</algorithm>
      <number>3</number>
      <scale>1.5</scale>
      <angle>0.4</angle>
      <amplitude>0.4</amplitude>
      <period>8.0</period>
      <phase>0.0</phase>
      <steepness>1.0</steepness>
      <direction>1 0</direction>
    </wave>
</plugin>
```

The waves visual plugin has the same algorithm elements as the model plugin and extra elements to control the shading algorithm. Two approaches are available:

  - `DYNAMIC_GEOMETRY` uses PBS shaders and is suitable for small areas.
  - `DYNAMIC_TEXTURE` uses a custom shader and is suitable for tiled areas.

```xml
<plugin
    filename="gz-waves1-waves-visual-system"
    name="gz::sim::systems::WavesVisual">
  <static>0</static>

  <!-- set the mesh deformation method  -->
  <mesh_deformation_method>DYNAMIC_GEOMETRY</mesh_deformation_method>

  <!-- number of additional tiles along each axis -->
  <tiles_x>-1 1</tiles_x>
  <tiles_y>-1 1</tiles_y>
  <wave>
    <!-- `fft` wave parameters -->
    <algorithm>fft</algorithm>
    <tile_size>256.0</tile_size>
    <cell_count>128</cell_count>
    <wind_speed>5.0</wind_speed>
    <wind_angle_deg>135</wind_angle_deg>
    <steepness>2</steepness>
  </wave>

  <!--
    Shader parameters only apply when using DYNAMIC_TEXTURE
  -->

  <!-- shader program -->
  <shader language="glsl">
    <vertex>materials/waves_vs.glsl</vertex>
    <fragment>materials/waves_fs.glsl</fragment>
  </shader>
  <shader language="metal">
    <vertex>materials/waves_vs.metal</vertex>
    <fragment>materials/waves_fs.metal</fragment>
  </shader>

  <!-- vertex shader params -->
  <param>
    <shader>vertex</shader>
    <name>world_matrix</name>
  </param>
  <param>
    <shader>vertex</shader>
    <name>worldviewproj_matrix</name>
  </param>
  <param>
    <shader>vertex</shader>
    <name>camera_position</name>
  </param>
  <param>
    <shader>vertex</shader>
    <name>rescale</name>
    <value>0.5</value>
    <type>float</type>
  </param>
  <param>
    <shader>vertex</shader>
    <name>bumpScale</name>
    <value>64 64</value>
    <type>float_array</type>
  </param>
  <param>
    <shader>vertex</shader>
    <name>bumpSpeed</name>
    <value>0.01 0.01</value>
    <type>float_array</type>
  </param>
  <param>
    <shader>vertex</shader>
    <name>t</name>
    <value>TIME</value>
  </param>

  <!-- pixel shader params -->
  <param>
    <shader>fragment</shader>
    <name>deepColor</name>
    <value>0.0 0.05 0.2 1.0</value>
    <type>float_array</type>
  </param>
  <param>
    <shader>fragment</shader>
    <name>shallowColor</name>
    <value>0.0 0.1 0.3 1.0</value>
    <type>float_array</type>
  </param>
  <param>
    <shader>fragment</shader>
    <name>fresnelPower</name>
    <value>5.0</value>
    <type>float</type>
  </param>
  <param>
    <shader>fragment</shader>
    <name>hdrMultiplier</name>
    <value>0.4</value>
    <type>float</type>
  </param>
  <param>
    <shader>fragment</shader>
    <name>bumpMap</name>
    <value>materials/wave_normals.dds</value>
    <type>texture</type>
    <arg>0</arg>
  </param>
  <param>
    <shader>fragment</shader>
    <name>cubeMap</name>
    <value>materials/skybox_lowres.dds</value>
    <type>texture_cube</type>
    <arg>1</arg>
  </param>

</plugin>
```

### Hydrodynamics plugin

- The `filename` and `name` attributes for the hydrodynamics plugin have changed.
- The hydrodynamics parameters are now scoped in an additional `<hydrodynamics>` element.
- The buoyancy and hydrodynamics forces can be applied to specific entities
in a model using the `<enable>` element. The parameter should be a fully
scoped model entity (model, link or collision name).
- The `<wave_model>` element is not used.

```xml
<plugin
  filename="gz-waves1-hydrodynamics-system"
  name="gz::sim::systems::Hydrodynamics">

  <!-- Apply hydrodynamics to the entire model (default) -->
  <enable>model_name</enable>

  <!-- Or apply hydrodynamics to named links -->
  <enable>model_name::link1</enable>
  <enable>model_name::link2</enable>

  <!-- Or apply hydrodynamics to named collisions -->
  <enable>model_name::link1::collision1</enable>
  <enable>model_name::link1::collision2</enable>

  <!-- Hydrodynamics -->
  <hydrodynamics>
    <damping_on>1</damping_on>
    <viscous_drag_on>1</viscous_drag_on>
    <pressure_drag_on>1</pressure_drag_on>

    <!-- Linear and Angular Damping -->  
    <cDampL1>1.0E-6</cDampL1>
    <cDampL2>1.0E-6</cDampL2>
    <cDampR1>1.0E-6</cDampR1>
    <cDampR2>1.0E-6</cDampR2>

    <!-- 'Pressure' Drag -->
    <cPDrag1>1.0E+2</cPDrag1>
    <cPDrag2>1.0E+2</cPDrag2>
    <fPDrag>0.4</fPDrag>
    <cSDrag1>1.0E+2</cSDrag1>
    <cSDrag2>1.0E+2</cSDrag2>
    <fSDrag>0.4</fSDrag>
    <vRDrag>1.0</vRDrag>
  </hydrodynamics>

  <!-- Control visibility of markers -->
  <markers>
    <update_rate>10</update_rate>
    <water_patch>1</water_patch>
    <waterline>1</waterline>
    <underwater_surface>1</underwater_surface>
  </markers>
</plugin>
```

## Tests

```bash
# build with tests
$ colcon build --merge-install --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_MACOSX_RPATH=FALSE -DCMAKE_INSTALL_NAME_DIR=$(pwd)/install/lib -DBUILD_TESTING=ON --packages-select gz-waves1

# run tests
colcon test --merge-install 

# check results
colcon test-result --all --verbose 
```

project 빌드 디렉토리내에서 테스트

```bash
$ cd ~/gz_ws/src/asv_wave_sim/gz-waves
$ mkdir build && cd build
$ cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo -DBUILD_TESTING=ON
$ make && make test
```

## Plots

Plots may be generated for some of the wave spectra and wave simulation methods:

```bash
./install/bin/PLOT_WaveSpectrum
```

## License

This is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This software is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the [GNU General Public License](LICENSE) for more details.

This project makes use of other open source software, for full details see the file [LICENSE_THIRDPARTY](LICENSE_THIRDPARTY).

## Acknowledgments

- Jacques Kerner's two part blog describing boat physics for games: [Water interaction model for boats in video games](https://www.gamasutra.com/view/news/237528/Water_interaction_model_for_boats_in_video_games.php) and [Water interaction model for boats in video games: Part 2](https://www.gamasutra.com/view/news/263237/Water_interaction_model_for_boats_in_video_games_Part_2.php).
- The [CGAL](https://doc.cgal.org) libraries are used for the wave field and model meshes.
- The [UUV Simulator](https://github.com/uuvsimulator/uuv_simulator) package for the orginal vertex shaders used in the wave field visuals.
- The [VMRC](https://bitbucket.org/osrf/vmrc) package for textures and meshes used in the wave field visuals.
- Jerry Tessendorf's paper on	[Simulating Ocean Water](https://people.cs.clemson.edu/~jtessen/reports/papers_files/coursenotes2004.pdf)
- Curtis Mobley's web book [Ocean Optics](https://www.oceanopticsbook.info/) in particular the section on [Modeling Sea Surfaces](https://www.oceanopticsbook.info/view/surfaces/level-2/modeling-sea-surfaces) and [example IDL code](https://www.oceanopticsbook.info/packages/iws_l2h/conversion/files/IDL-SurfaceGenerationCode.zip)  
