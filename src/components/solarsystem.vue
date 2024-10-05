<template>
  <div>
    <h1>Solar system and Near-Earth Objects system</h1>
    <!-- <h2>Slider control adjusts the simulation speed.</h2> -->
    <ul>
      <li>Right-click and drag to zoom in and out</li>
      <li>Left-click and drag to rotate model</li>
      <li>Middle-click and drag to move model</li>
      <li>Press the key 'A' to observe the entire system</li>
    </ul>

    <x3d width="100%" height="40%">
      <scene>
        <background DEF="bgnd" transparency="0" backUrl="/star.jpg"></background>
        <transform id="theSun" translation="0 0 0">
          <shape>
            <appearance>
              <material diffuseColor="1 0.1 0.15"></material>
            </appearance>
            <sphere radius="0.6"></sphere>
          </shape>
          <!-- 添加太阳的标签 -->
          <billboard>
            <transform translation="0 1.2 0">
              <shape>
                <text string='"Sun"' solid='false'>
                  <fontstyle size='0.2' justify='"MIDDLE" "MIDDLE"' />
                </text>
                <appearance>
                  <material diffuseColor="1 1 1"></material>
                </appearance>
              </shape>
            </transform>
          </billboard>
        </transform>
        <viewpoint fieldOfView="0.785398" position="3 3 3" orientation="1 -1 0 -0.785"></viewpoint>
      </scene>
    </x3d>

    <!-- <input type="range" id="myRange" min="1" max="100" value="100" step="1" onchange="showValue(this.value)" /><br> -->
    <!-- <input type="button" id="orbits" value="Toggle Orbits" @click="toggleOrbits" /> -->
    <!-- <input type="button" id="labels" value="Toggle Labels" @click="toggleLabels" /> -->

    <p id="modelDate"></p>
    <p class="case" align="center"><i>*Objects are not to scale</i></p>
  </div>
</template>

<script>
// 将 Trajectory 函数定义在 Vue 组件外部
function Trajectory(name, smA, oI, aP, oE, aN, mAe, Sidereal) {
  this.name = name;                       // 天体名称
  this.smA = smA * 7;                         // 半长轴（天体轨道的大小）
  this.oI = oI * 0.01745329;              // 倾角（从度转换为弧度）
  this.aP = aP * 0.01745329;              // 近地点参数（从度转换为弧度）
  this.oE = oE;                           // 轨道离心率（0 为圆形轨道，接近 1 为椭圆轨道）
  this.aN = aN * 0.01745329;              // 升交点经度（从度转换为弧度）
  this.period = Sidereal;                 // 轨道周期（以地球年为单位）
  this.epochMeanAnomaly = mAe * 0.01745329; // 平近点角（从度转换为弧度）
  this.trueAnomoly = 0;                   // 初始化真实近点角
  this.position = [0, 0, 0];              // 当前天体的三维位置
  this.time = 0;                          // 记录天体运动的时间
}
import Papa from 'papaparse'; // 引入 PapaParse 库

export default {
  name: 'SolarSystem',
  data() {
    return {
      heavenlyBodies: [], // 存储所有天体的轨道信息
      simSpeed: 1,        // 模拟速度
      solid: false,       // 控制轨道显示
      solidLabels: false, // 控制标签显示
      epoch: new Date('December 9, 2014'), // 起始时间
      earthPosition: [0, 0, 0], // 用于存储地球当前的三维位置
      alertDistance: 0.1, // 预警距离（可根据需要调整，单位：AU）
    };
  },

  mounted() {
    this.initializeSolarSystem(); // 初始化场景和天体
    this.addCometsFromJson('https://data.nasa.gov/resource/b67r-rgxc.json');
    this.addBodiesFromCsv('/sbdb_query_results.csv'); // 添加星体数据
    this.traceOrbits();           // 绘制轨道
    setInterval(this.updatePosition, 50); // 更新天体位置，每50ms更新一次
  },
  methods: {
    // 初始化天体和轨道参数
    initializeSolarSystem() {
      // 使用外部定义的 Trajectory 函数
      this.heavenlyBodies.push(new Trajectory("Mercury", 0.38709927, 7.00497902, 77.45779628, 0.20563593, 48.33076593, 174.79252722, 0.2408467));
      this.heavenlyBodies.push(new Trajectory("Venus", 0.72333566, 3.39467605, 131.60246718, 0.00677672, 76.67984255, 181.97909950, 0.615));
      this.heavenlyBodies.push(new Trajectory("theEarth", 1.00000261, -0.00001531, 102.93768193, 0.01671123, 0.0, 100.46457166, 1));
      this.heavenlyBodies.push(new Trajectory("Mars", 1.52371034, 1.84969142, -23.94362959, 0.09339410, 49.55953891, -4.55343205, 1.8808476));
      this.heavenlyBodies.push(new Trajectory("Jupiter", 5.20288700, 1.30439695, 14.72847983, 0.04838624, 100.47390909, 34.39644051, 11.862));
      this.heavenlyBodies.push(new Trajectory("Saturn", 9.53667594, 2.48599187, 92.59887831, 0.05386179, 113.66242448, 49.95424423, 29.4571));
      this.heavenlyBodies.push(new Trajectory("Uranus", 19.18916464, 0.77263783, 170.95427630, 0.04725744, 74.01692503, 313.23810451, 84.016846));
      this.heavenlyBodies.push(new Trajectory("Neptune", 30.06992276, 1.77004347, 44.96476227, 0.00859048, 131.78422574, -55.12002969, 164.79132));

      // 添加天体到X3D场景
      this.addNode("Mercury", 0.7, 0.2, 0.4, 0.024, "Mercury", true);
      this.addNode("Venus", 0.9, 0.9, 0.9, 0.061, "Venus", true);
      this.addNode("theEarth", 0.2, 0, 0.8, 0.064, "Earth", true);
      this.addNode("Mars", 1, 0.1, 0.15, 0.034, "Mars", true);
      this.addNode("Jupiter", 1, 0.5, 0, 0.699, "Jupiter", true);
      this.addNode("Saturn", 0.9, 0.7, 0.1, 0.582, "Saturn", true);
      this.addNode("Uranus", 0.6, 0.8, 1, 0.254, "Uranus", true);
      this.addNode("Neptune", 0.1, 0.2, 0.9, 0.246, "Neptune", true);
    },

    // 异步读取和解析 CSV 文件，并添加星体
    async addBodiesFromCsv(csvFilePath) {
      try {
        // 使用 fetch 加载 CSV 文件
        const response = await fetch(csvFilePath);
        const csvText = await response.text();

        // 使用 PapaParse 解析 CSV
        Papa.parse(csvText, {
          header: true, // 确保 CSV 文件中的第一行作为键
          skipEmptyLines: true, // 跳过空行
          complete: (results) => {
            const bodiesData = results.data;

            // 遍历每个星体数据，提取并创建 Trajectory 对象
            bodiesData.forEach(body => {
              const name = body.full_name; // 使用full_name 作为名称
              const smA = parseFloat(body.a);           // 半长轴
              const e = parseFloat(body.e);             // 离心率
              const i = parseFloat(body.i);             // 倾角
              const w = parseFloat(body.w);             // 近日点参数
              const node = parseFloat(body.om);         // 升交点经度
              const period = parseFloat(body.per) / 365.25;      // 轨道周期
              const tp = parseFloat(body.tp);           // 近日点通过时间
              const epoch = 2460600.5; // 当前纪元（儒略日格式）

              // 计算平近点角 M
              const T = period * 365.25;               // 轨道周期，转换为天
              const M = ((2 * Math.PI) / T) * (epoch - tp); // 计算平近点角
              const mae = M % (2 * Math.PI);           // 归一化平近点角

              // 使用 Trajectory 函数创建星体轨道
              const bodyTrajectory = new Trajectory(name, smA, i, w, e, node, mae, period);

              // 添加到 heavenlyBodies 列表中
              this.heavenlyBodies.push(bodyTrajectory);

              //设置颜色、大小，并添加到 X3D 场景中
              const colorR = 0.5, colorG = 0.5, colorB = 1;
              const radius = parseFloat(body.diameter) / 1000 || 0.05; // 如果有直径，缩放后使用；否则使用默认值
              this.addNode(name, colorR, colorG, colorB, radius, name, false);
            });
          },
          error: (error) => {
            console.error("解析 CSV 时发生错误: ", error);
          }
        });
      } catch (error) {
        console.error("获取或处理 CSV 文件时发生错误: ", error);
      }
    },
    // 异步获取彗星数据并添加到轨道
    async addCometsFromJson(url) {
      try {
        const response = await fetch(url);
        const cometsData = await response.json();

        cometsData.forEach(comet => {
          const name = comet.object_name;
          const smA = (parseFloat(comet.q_au_1) + parseFloat(comet.q_au_2)) / 2; // 估算半长轴，使用近日点和远日点平均值
          const e = parseFloat(comet.e);
          const i = parseFloat(comet.i_deg);
          const w = parseFloat(comet.w_deg);
          const node = parseFloat(comet.node_deg);
          const period = parseFloat(comet.p_yr);

          // 平近点角的计算（根据公式：M = 2π / T * (t - tp)）
          const t = parseFloat(comet.epoch_tdb);   // 纪元时间
          const tp = parseFloat(comet.tp_tdb);     // 近日点通过时间
          const T = period * 365.25;               // 轨道周期，转换为天
          const M = ((2 * Math.PI) / T) * (t - tp); // 计算平近点角
          const mae = M % (2 * Math.PI);           // 归一化平近点角

          const cometTrajectory = new Trajectory(name, smA, i, w, e, node, mae, period); // 使用计算的平近点角

          this.heavenlyBodies.push(cometTrajectory);

          const colorR = 0.5, colorG = 0.5, colorB = 1;
          const radius = 0.05;
          this.addNode(name, colorR, colorG, colorB, radius, name, false);
          // this.traceOrbits();  //绘制轨道电脑会很卡
        });
      } catch (error) {
        console.error("获取或处理彗星数据时发生错误: ", error);
      }
    },

    // 添加天体到X3D场景
    addNode(identifier, cR, cG, cB, radius, label, defaultVisible = true) {
      // 创建 Switch 节点用于控制显示和隐藏
      const switchNode = document.createElement('Switch');
      switchNode.setAttribute('id', identifier + '_Switch');
      switchNode.setAttribute('whichChoice', defaultVisible ? '0' : '-1');  // 默认显示行星，隐藏小行星

      // 创建 Transform 节点用于存放天体的空间变换
      const t = document.createElement('Transform');
      t.setAttribute('translation', '0 0 -5'); // 设置初始位置
      t.setAttribute('id', identifier);

      // 创建 Shape 节点表示天体的形状
      const s = document.createElement('Shape');
      const app = document.createElement('Appearance');
      const mat = document.createElement('Material');
      mat.setAttribute('id', identifier + 'Mat');
      mat.setAttribute('diffuseColor', `${cR} ${cG} ${cB}`); // 设置材质的漫反射颜色
      app.appendChild(mat);
      s.appendChild(app);

      const b = document.createElement('Sphere');
      b.setAttribute('radius', radius);  // 设置球体半径
      s.appendChild(b);
      t.appendChild(s);  // 将形状节点添加到 Transform 节点
      switchNode.appendChild(t);  // 添加 Transform 节点到 Switch 节点

      // 添加标签
      const billboard = document.createElement('Billboard');
      const labelTransform = document.createElement('Transform');
      labelTransform.setAttribute('translation', `0 ${radius + 0.2} 0`);
      const labelShape = document.createElement('Shape');
      const labelText = document.createElement('Text');
      labelText.setAttribute('string', `"${label}"`);
      const labelFont = document.createElement('FontStyle');
      labelFont.setAttribute('size', '0.2');
      labelFont.setAttribute('justify', '"MIDDLE" "MIDDLE"');
      labelText.appendChild(labelFont);
      labelShape.appendChild(labelText);

      const labelAppearance = document.createElement('Appearance');
      const labelMaterial = document.createElement('Material');
      labelMaterial.setAttribute('diffuseColor', '1 1 1');
      labelAppearance.appendChild(labelMaterial);
      labelShape.appendChild(labelAppearance);

      labelTransform.appendChild(labelShape);
      billboard.appendChild(labelTransform);
      t.appendChild(billboard);

      // 添加到场景中的太阳节点
      const ot = document.getElementById('theSun');
      if (ot) {
        ot.appendChild(switchNode);
        console.log(`Added node with ID: ${identifier} in switch`);
      }
    },

    // 计算和绘制轨道
    traceOrbits() {
      for (const hB of this.heavenlyBodies) {
        let orbPos = [];
        let orbitCoords = "";
        let orbIndices = "";
        let j = 0;
        let i = 0.0;

        // 生成轨道点
        while (i <= 6.28) {
          orbPos = this.propagate(hB, i);
          orbitCoords += `${orbPos[0].toFixed(2)} ${orbPos[1].toFixed(2)} ${orbPos[2].toFixed(2)} `;
          orbIndices += `${j} `;
          i += 0.0785;
          j++;
        }
        orbIndices += -1;

        const s = document.createElement('Shape');
        s.setAttribute('id', hB.name + "Orbit");

        const app = document.createElement('Appearance');
        const mat = document.createElement('Material');
        mat.setAttribute('id', hB.name + "OrbitMat");
        const omat = document.getElementById(hB.name + 'Mat');
        const oMatC = omat.getAttribute("diffuseColor");
        mat.setAttribute("emissiveColor", oMatC);
        app.appendChild(mat);
        s.appendChild(app);

        const line = document.createElement('IndexedLineSet');
        line.setAttribute("coordIndex", orbIndices);
        const coords = document.createElement('Coordinate');
        coords.setAttribute("point", orbitCoords);
        line.appendChild(coords);
        s.appendChild(line);

        const ot = document.getElementById('theSun');
        ot.appendChild(s);
      }
    },

    updatePosition() {
      if (!this.heavenlyBodies.length) {
        console.warn("Heavenly bodies not initialized");
        return;
      }

      for (const hB of this.heavenlyBodies) {
        // 更新真实近点角
        hB.trueAnomoly += (2 * Math.PI) * this.simSpeed / (hB.period * 365.25);
        if (hB.trueAnomoly > 2 * Math.PI) {
          hB.trueAnomoly -= 2 * Math.PI;
        }
        const currentPosition = this.propagate(hB, hB.trueAnomoly);

        // 更新地球位置
        if (hB.name === "theEarth") {
          this.earthPosition = currentPosition;
        }

        // 计算小行星与地球的距离
        if (hB.name.startsWith("NEO")) {
          const distance = Math.sqrt(
            Math.pow(currentPosition[0] - this.earthPosition[0], 2) +
            Math.pow(currentPosition[1] - this.earthPosition[1], 2) +
            Math.pow(currentPosition[2] - this.earthPosition[2], 2)
          );

          // 如果距离小于预警距离，则显示小行星和轨道
          if (distance < this.alertDistance) {
            this.showAsteroidAndOrbit(hB.name, true);  // 显示
          } else {
            this.showAsteroidAndOrbit(hB.name, false); // 隐藏
          }
        }

        // 更新天体位置
        const asteroid = document.getElementById(hB.name);
        if (asteroid) {
          asteroid.setAttribute('translation', currentPosition.join(' '));
        }
      }
    },



    // 更新日期
    updateTheDate() {
      this.epoch.setDate(this.epoch.getDate() + this.simSpeed);
      document.getElementById("modelDate").textContent = `${this.epoch.getMonth() + 1}-${this.epoch.getDate()}-${this.epoch.getFullYear()}`;
    },

    // 控制显示/隐藏轨道
    toggleOrbits() {
      this.solid = !this.solid;
      const opacity = this.solid ? 1 : 0;
      for (const hB of this.heavenlyBodies) {
        const omat = document.getElementById(hB.name + "OrbitMat");
        omat.setAttribute("transparency", opacity);
      }
    },

    // 控制显示/隐藏标签
    toggleLabels() {


      // 获取所有标签元素，假设标签的 id 格式为 `${planetName}_label`
      this.heavenlyBodies.forEach(body => {
        const labelElement = document.getElementById(`${body.name}_label`);
        if (labelElement) {
          labelElement.setAttribute('opacity', this.solidLabels ? '1' : '0'); // 设置透明度
        }
      });
      // 切换标签显示/隐藏的状态
      this.solidLabels = !this.solidLabels;
    },
    // 显示或隐藏小行星及其轨道
    showAsteroidAndOrbit(identifier, show) {
      const switchNode = document.getElementById(identifier + '_Switch');  // 获取 Switch 节点
      if (switchNode) {
        const whichChoice = show ? '0' : '-1';  // '0' 表示显示第一个子节点，'-1' 表示隐藏所有子节点
        switchNode.setAttribute('whichChoice', whichChoice);
      } else {
        console.warn(`Switch node for ${identifier} not found.`);
      }
    },



    // 滑动条改变速度
    showValue(newValue) {
      this.simSpeed = newValue * 0.01;
    },

    // 轨道传播函数，计算当前天体的位置
    propagate(trajectory, uA) {
      const pos = [];
      const smA = trajectory.smA;
      const oI = trajectory.oI;
      const aP = trajectory.aP;
      const oE = trajectory.oE;
      const aN = trajectory.aN;

      const sLR = smA * (1 - oE ** 2);
      const r = sLR / (1 + oE * Math.cos(uA));

      // 计算 X, Y, Z 坐标
      pos[0] = r * (Math.cos(aP + uA) * Math.cos(aN) - Math.cos(oI) * Math.sin(aP + uA) * Math.sin(aN));
      pos[1] = r * (Math.cos(aP + uA) * Math.sin(aN) + Math.cos(oI) * Math.sin(aP + uA) * Math.cos(aN));
      pos[2] = r * Math.sin(aP + uA) * Math.sin(oI);

      return pos;
    }
  }
};
</script>

<style scoped>
.case {
  clear: both;
  border-top: 0px solid #fff;
}
</style>
