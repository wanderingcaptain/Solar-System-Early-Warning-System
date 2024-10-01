<template>
  <div>
    <h1>Venus, Earth, Mars and Potentially Hazardous Asteroids (PHA)*</h1> 
    <h2>Slider control adjusts the simulation speed.</h2>
    <!-- <ul>
      <li>Right-click and drag to zoom in and out</li>
      <li>Left-click and drag to rotate model</li>
      <li>Middle-click and drag to move model</li>
    </ul> -->

    <x3d width="800px" height="600px">
      <scene>
        <background DEF="bgnd" transparency="0" skyColor="0.75 0.75 0.75"></background>
        <transform id="theSun" translation="0 0 0">
          <shape>
            <appearance>
              <material diffuseColor="1 1 0"></material>
            </appearance>
            <sphere radius="0.7"></sphere>
          </shape>
        </transform>
        <viewpoint fieldOfView="0.785398" position="6 5.5 6.5" orientation="1 -1 0 -0.785"></viewpoint>
      </scene>
    </x3d>

    <input type="range" id="myRange" min="1" max="100" value="100" step="1" @change="showValue" /><br>
    <input type="button" id="orbits" value="Toggle Orbits" @click="toggleOrbits" />
    <input type="button" id="labels" value="Toggle Labels" @click="toggleLabels" />

    <p id="modelDate"></p>
    <p class="case" align="center"><i>*Objects are not to scale</i></p>
  </div>
</template>

<script>
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
    this.traceOrbits();           // 绘制轨道
    setInterval(this.updatePosition, 50); // 更新天体位置
  },
  methods: {
    // 初始化天体和轨道参数
    initializeSolarSystem() {
      // 定义轨道对象
      function Trajectory(name, smA, oI, aP, oE, aN, mAe, Sidereal) {
        this.name = name;
        this.smA = smA;
        this.oI = oI * 0.01745329; // 度转为弧度
        this.aP = aP * 0.01745329;
        this.oE = oE;
        this.aN = aN * 0.01745329;
        this.period = Sidereal;
        this.epochMeanAnomaly = mAe * 0.01745329;
        this.trueAnomoly = 0;
        this.position = [0, 0, 0];
        this.time = 0;
      }

      // 创建天体轨道对象
      this.heavenlyBodies.push(new Trajectory("Venus", 0.72333199, 3.39471, 54.9, 0.00677323, 76.7, 181.98, 0.615));
      this.heavenlyBodies.push(new Trajectory("theEarth", 1, 0.00005, 102.94719, 0.01671022, 0, 100.47, 1));
      this.heavenlyBodies.push(new Trajectory("Mars", 1.52366231, 1.85061, 286.5, 0.09339, 49.57854, 355.43, 1.881));

      // 添加天体到X3D场景
      this.addNode("Venus", 0.9, 0.9, 0.9, 0.15);
      this.addNode("theEarth", 0.2, 0, 0.8, 0.15);
      this.addNode("Mars", 1, 0.1, 0.15, 0.1);
    },

    // 添加天体到X3D场景
    addNode(identifier, cR, cG, cB, radius) {
      const t = document.createElement('Transform');
      t.setAttribute('translation', '0 0 -5');
      t.setAttribute('id', identifier);

      const s = document.createElement('Shape');
      const app = document.createElement('Appearance');
      const mat = document.createElement('Material');
      mat.setAttribute('id', identifier + 'Mat');
      mat.setAttribute('diffuseColor', `${cR} ${cG} ${cB}`);
      app.appendChild(mat);
      s.appendChild(app);

      const b = document.createElement('Sphere');
      b.setAttribute('radius', radius);
      s.appendChild(b);
      t.appendChild(s);

      const ot = document.getElementById('theSun');
      ot.appendChild(t);
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

    // // 控制显示/隐藏标签
    // toggleLabels() {
    //   this.solidLabels = !this.solidLabels;
    //   const opacity = this.solidLabels ? 1 : 0;
    //   // 控制标签的显示/隐藏逻辑（可以根据需求添加具体标签）
    // },

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
