<template>
  <div>
    <h1>Venus, Earth, Mars and Potentially Hazardous Asteroids (PHA)*</h1>
    <h2>Slider control adjusts the simulation speed.</h2>
    <ul>
      <li>Right-click and drag to zoom in and out</li>
      <li>Left-click and drag to rotate model</li>
      <li>Middle-click and drag to move model</li>
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

    <input type="range" id="myRange" min="1" max="100" value="100" step="1" onchange="showValue(this.value)" /><br>
    <input type="button" id="orbits" value="Toggle Orbits" @click="toggleOrbits" />
    <input type="button" id="labels" value="Toggle Labels" @click="toggleLabels" />

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

export default {
  name: 'SolarSystem',
  data() {
    return {
      heavenlyBodies: [], // 存储所有天体的轨道信息
      simSpeed: 1,        // 模拟速度
      solid: false,       // 控制轨道显示
      solidLabels: false, // 控制标签显示
      epoch: new Date('December 9, 2014'), // 起始时间
    };
  },
  mounted() {
    this.initializeSolarSystem(); // 初始化场景和天体
    this.addCometsFromJson('https://data.nasa.gov/resource/b67r-rgxc.json');
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
      this.addNode("Mercury", 0.7, 0.2, 0.4, 0.024, "Mercury");
      this.addNode("Venus", 0.9, 0.9, 0.9, 0.061, "Venus");
      this.addNode("theEarth", 0.2, 0, 0.8, 0.064, "Earth");
      this.addNode("Mars", 1, 0.1, 0.15, 0.034, "Mars");
      this.addNode("Jupiter", 1, 0.5, 0, 0.699, "Jupiter");
      this.addNode("Saturn", 0.9, 0.7, 0.1, 0.582, "Saturn");
      this.addNode("Uranus", 0.6, 0.8, 1, 0.254, "Uranus");
      this.addNode("Neptune", 0.1, 0.2, 0.9, 0.246, "Neptune");
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
          this.addNode(name, colorR, colorG, colorB, radius, name);
          // this.traceOrbits();  //绘制轨道电脑会很卡
        });
      } catch (error) {
        console.error("获取或处理彗星数据时发生错误: ", error);
      }
    },

    // 添加天体到X3D场景
    addNode(identifier, cR, cG, cB, radius, label) {
      // 创建 Transform 节点用于存放天体的空间变换
      const t = document.createElement('Transform');
      t.setAttribute('translation', '0 0 -5'); // 设置初始位置
      t.setAttribute('id', identifier);        // 设置节点的 id，使用天体标识符

      // 创建 Shape 节点表示天体的形状
      const s = document.createElement('Shape');
      const app = document.createElement('Appearance');   // 创建 Appearance 节点用于定义外观
      const mat = document.createElement('Material');     // 创建 Material 节点用于定义材质颜色
      mat.setAttribute('id', identifier + 'Mat');
      mat.setAttribute('diffuseColor', `${cR} ${cG} ${cB}`); // 设置材质的漫反射颜色
      app.appendChild(mat);                               // 将材质添加到外观节点
      s.appendChild(app);                                 // 将外观添加到形状节点

      // 创建 Sphere 节点表示天体的几何形状
      const b = document.createElement('Sphere');
      b.setAttribute('radius', radius);                   // 设置球体半径
      s.appendChild(b);                                   // 将球体添加到形状节点
      t.appendChild(s);                                   // 将形状节点添加到 Transform 节点

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
      // 找到场景中的太阳节点，并将天体添加到该节点下
      const ot = document.getElementById('theSun');
      ot.appendChild(t);                                  // 将新天体添加到场景
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

    // 更新天体位置
    updatePosition() {
      for (const hB of this.heavenlyBodies) {
        // 更新真实近点角
        hB.trueAnomoly += (2 * Math.PI) * this.simSpeed / (hB.period * 365.25);
        if (hB.trueAnomoly > 2 * Math.PI) {
          hB.trueAnomoly -= 2 * Math.PI; // 确保角度不会超出 2π
        }
        const currentPosition = this.propagate(hB, hB.trueAnomoly);
        document.getElementById(hB.name).setAttribute('translation', currentPosition.join(' '));
      }
      this.updateTheDate();
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
