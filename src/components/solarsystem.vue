<template>
    <div>
      <h1>交互式太阳系仪</h1>
      <x3d width="800px" height="600px">
  <scene>
    <!-- 背景设置为黑色 -->
    <background skyColor="0 0 0" />
    <NavigationInfo type='"EXAMINE"' />

    <!-- 视角设置 -->
    <viewpoint position="0 0 10" orientation="0 1 0 0" fieldOfView="0.785398" />

    <!-- 椭圆轨道 -->
    <transform rotation="0 0 1 0.5">
      <shape>
        <appearance>
          <material diffuseColor="0.8 0.8 0.8" emissiveColor="1 1 1" /> <!-- 灰色轨道，发光颜色为白色 -->
        </appearance>
        <indexedlineset coordindex="0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 -1">
          <coordinate point="
            1 0 0,
            0.923 0 0.382,
            0.707 0 0.707,
            0.382 0 0.923,
            0 0 1,
            -0.382 0 0.923,
            -0.707 0 0.707,
            -0.923 0 0.382,
            -1 0 0,
            -0.923 0 -0.382,
            -0.707 0 -0.707,
            -0.382 0 -0.923,
            0 0 -1,
            0.382 0 -0.923,
            0.707 0 -0.707,
            0.923 0 -0.382,
            1 0 0
          "/>
        </indexedlineset>
      </shape>
    </transform>

    <!-- 动态行星 -->
    <transform DEF="planetTransform">
      <shape>
        <appearance>
          <material diffuseColor="0 0 1" /> <!-- 蓝色行星 -->
        </appearance>
        <sphere radius="0.1" />
      </shape>
    </transform>

    <!-- 时间传感器 -->
    <TimeSensor DEF="orbitTimer" cycleInterval="10" loop="true"></TimeSensor>

    <!-- 位置插值器：行星轨道 -->
    <PositionInterpolator DEF="planetOrbit" key="0 0.25 0.5 0.75 1" 
      keyValue="
        1 0 0,
        0.707 0 0.707,
        0 0 1,
        -0.707 0 0.707,
        -1 0 0
      ">
    </PositionInterpolator>

    <!-- 路由：连接时间传感器和位置插值器 -->
    <ROUTE fromNode="orbitTimer" fromField="fraction_changed" toNode="planetOrbit" toField="set_fraction"></ROUTE>
    <!-- 路由：连接位置插值器和平移 -->
    <ROUTE fromNode="planetOrbit" fromField="value_changed" toNode="planetTransform" toField="set_translation"></ROUTE>
  </scene>
</x3d>

    </div>
  </template>
  
  <script>
  export default {
    name: 'SolarSystem',
  };
  </script>
  
  <style>
  /* 设置 X3DOM 线条宽度 */
  x3d {
    shape-rendering: crispEdges;
  }
  line {
    stroke-width: 2px; /* 设置线条宽度为 2px */
  }
</style>

  