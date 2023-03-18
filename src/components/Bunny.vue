<template>
  <div class="canvWrapper">
    <canvas class="canvas" ref="myCanvas" />
  </div>
  <!-- <h1 class="creator"> test</h1> -->
</template>

<script>
const THREE = require("three")
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { Flow } from 'three/addons/modifiers/CurveModifier.js';
import { FontLoader } from 'three/addons/loaders/FontLoader.js';
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js'

export default {
  name: "Bunny",
  data() {
    return {
      mouseMoved:false,
    };
  },
  methods:{
    animate(){
      requestAnimationFrame(this.animate);
      this.controls.update();
      if (this.flow){
        // console.log("has flow");
        this.flow.moveAlongCurve(.01);
      }
      this.renderer.render(this.scene, this.camera);
    },
    init(){
      this.canvas = this.$refs.myCanvas;

      // define renderer
      this.renderer = new THREE.WebGLRenderer({ canvas: this.canvas, antialias: true, powerPreference: "high-performance", });
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.renderer.setClearColor(0x050d1a, 1);
      this.scene = new THREE.Scene();
      // define camera
      this.camera = new THREE.PerspectiveCamera(5, this.canvas.width / this.canvas.height, 1, 1000);
      this.camera.position.x = 0
      this.camera.position.y = 0
      this.camera.position.z = 200
      this.camera.up.set(0, 0, 1);

      // define orbital control
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      this.controls.minDistance = 20;
      this.controls.update();


      this.scene.add(this.camera);
      // console.log(this.camera);

      // Define an array of 3D points that define the path
      const points = [
         new THREE.Vector3( 1,0, - 1 ),
         new THREE.Vector3( 1,0, 1 ),
         new THREE.Vector3( - 1,0, 1 ),
         new THREE.Vector3( - 1,0, - 1 ),
      ];
      // Create a CatmullRomCurve3 object using the points
      const curve = new THREE.CatmullRomCurve3(points);
      curve.closed = true;

      // Define a geometry for the curve
      const curveGeometry = new THREE.BufferGeometry().setFromPoints(curve.getPoints(80));

      // Define a material for the curve
      const curveMaterial = new THREE.LineBasicMaterial({ color: 0xFFFFFF });

      // Create a mesh object to visualize the curve
      const curveObject = new THREE.Line(curveGeometry, curveMaterial);

      // Add the curve object to the scene
      this.scene.add(curveObject);

      // load text now
      const loader = new FontLoader();
      let me = this;
      loader.load("./helvetiker_regular.typeface.json",  function ( font ) {
        const geometry = new TextGeometry( 'Hello three.js!', {
          font: font,
          size: 0.2,
          height: 0.0000001,
          curveSegments: 12,
          bevelEnabled: true,
          bevelThickness: 0.000000001,
          bevelSize: 0.000000001,
          bevelOffset: 0,
          bevelSegments: 50,
        } );

        geometry.rotateX( Math.PI );

        const material = new THREE.MeshBasicMaterial( {
          color: "0xFFFFFF"
        } );

        const objectToCurve = new THREE.Mesh( geometry, material );

        me.flow = new Flow(objectToCurve);
        me.flow.updateCurve(0, curve);
        me.scene.add(me.flow.object3D);
      });
    }
  },
  mounted() {
    this.init();
    this.animate();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .canvWrapper{
    height:100vh;
    width:100vw;
    position:absolute;
    top:0px;
    left:0px;
  }
  .creator{
    position:absolute;
    color: red;
    bottom: 0px;
  }

</style>
