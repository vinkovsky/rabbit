<template>
  <div class="loading" id="loading_text" :style="{ color: computedTextColor }">{{loading_text}}</div>
  <div class="createdBy" :style="{ color: computedTextColor }">CREATED BY TXSTC55 <br>CONTACT: TXSTC55@GMAIL.COM</div>
  <div class="canvWrapper">
    <canvas class="canvas" ref="myCanvas" />
  </div>
  <!-- <h1 class="creator"> test</h1> -->
</template>

<script>
import * as THREE from 'three';
import { TransformControls } from 'three/addons/controls/TransformControls.js';
import Stats from 'three/addons/libs/stats.module.js';
import { Flow } from 'three/addons/modifiers/CurveModifier.js';
import { FontLoader } from 'three/addons/loaders/FontLoader.js';
import { TextGeometry } from 'three/addons/geometries/TextGeometry.js';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import all_loops from '../assets/loops.json'
import colors from "../assets/colors.json"

var chosen_color = colors[Math.floor(Math.random() * colors.length)];
var choice = Math.floor(Math.random() * 2);
var bg_color = "#"+chosen_color[choice];
var line_color = "#"+chosen_color[(choice + 1 ) % 2];
// console.log(line_color);
var chosen_loop = Math.floor(Math.random() * all_loops.length);

const loops = all_loops[chosen_loop];

const curveHandles = [];
const flowList = [];
const flowSpeed = [];
let stats;
let scene,
  camera,
  renderer,
  rayCaster,
  controls,
  flow;

let text = " FOLLOW THE WHITE RABBIT "
let MIN_FONT_SIZE = 0.1;
let MAX_FONT_SIZE = 0.5;

export default {
  name: "Bunny",
  data() {
    return {
      loadingDone: false,
      dot_length: 0,
      line_size: 18,
      text_material: new THREE.MeshBasicMaterial({color: line_color}),
      flowNumber: 0,
      loading_text: "LOADING...",
    };
  },
  computed:{
    computedTextColor(){
      return line_color;
    }
  },
  methods:{
    computeLength(points){
      var d = 0;
      for (var i = 0; i < points.length-1; i++){
        d += points[i].distanceTo(points[i+1]);
      }
      return d;
    },

    async loadLoop(ind){
      const loader = new FontLoader();
      let me = this;
      loader.load( './Fira Code Light_Regular.json', function ( font ) {
      // load for each loop
        var loop_points = [];
        for (let j = 0; j < loops[ind].length; j++){
          var p = loops[ind][j];
          loop_points.push(new THREE.Vector3(p[0], p[1], p[2]));
        }
        // this is one loop
        var curve = new THREE.CatmullRomCurve3(loop_points);
        curve.curveType = 'centripetal';
        curve.closed = true;

        // compute font size and repeat times
        var l = me.computeLength(loop_points);
        var font_size = Math.random() * (MAX_FONT_SIZE - MIN_FONT_SIZE) + MIN_FONT_SIZE;
        var repeat_time = Math.floor(l / (me.line_size * font_size));
        
        var damping_repeat = Math.random() * 0.8 + 0.2;
        const geometry = new TextGeometry( text.repeat(Math.ceil(repeat_time * damping_repeat)), {
          font: font,
          size: font_size,
          height: 0.00000001,
          curveSegments: 12,
          bevelEnabled: false,
        } );

        geometry.rotateX( Math.random() * Math.PI);
        const objectToCurve = new THREE.Mesh(geometry, me.text_material );
        flow = new Flow( objectToCurve );
        flow.updateCurve( 0, curve );
        flowList.push(flow);
        flowSpeed.push(1/60/l);
        scene.add( flow.object3D );
        if (flowSpeed.length == loops.length){
          me.loadingDone = true;
          console.log("LOADING DONE");
          me.loading_text = "";
        }
      });
    },

    async init() {
      // global settings
      scene = new THREE.Scene();
      camera = new THREE.PerspectiveCamera(25, window.innerWidth / window.innerHeight, 1, 1000);
      camera.position.set( 0, 0, 50 );
      camera.lookAt( scene.position );

      // scene.add( line );
      

      // define the renderer first
      renderer = new THREE.WebGLRenderer({ canvas: this.$refs.myCanvas, antialias: true, powerPreference: "high-performance", });
      renderer.setPixelRatio( window.devicePixelRatio );
      renderer.setSize( window.innerWidth, window.innerHeight );
      renderer.setClearColor(bg_color, 1);
      // document.body.appendChild( renderer.domElement );


      controls = new OrbitControls( camera, renderer.domElement );
      controls.enableDamping = true; // an animation loop is required when either damping or auto-rotation are enabled
      controls.minDistance = 5;
      controls.maxDistance = 500;
      window.addEventListener( 'resize', this.onWindowResize );


      let me = this;
      for (let i = 0; i < loops.length; i++){
        await this.loadLoop(i);
      };
    },
    animate() {
      requestAnimationFrame(this.animate);
      controls.update();
      for (let i = 0; i < flowList.length; i++){
        flowList[i].moveAlongCurve(flowSpeed[i]);
      }
      this.render();
    },

    render() {
      renderer.render( scene, camera );
    },
    
    onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize( window.innerWidth, window.innerHeight );
    }
  },

  async mounted(){
    await this.init();
    await this.animate();
  }
}
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
  .loading{
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
    font-size:20px;
    z-index:100;
    -webkit-user-select: none; /* Safari */
    -ms-user-select: none; /* IE 10 and IE 11 */
    user-select: none; /* Standard syntax */
    font-weight: bold;
  }

  #loading_text{
  }

  .createdBy{
    position: absolute;
    bottom:1%;
    left: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
    font-size:10px;
    z-index:100;
    -webkit-user-select: none; /* Safari */
    -ms-user-select: none; /* IE 10 and IE 11 */
    user-select: none; /* Standard syntax */
    font-weight: bold;
  }

</style>
