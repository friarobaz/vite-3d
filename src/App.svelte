<script>
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import {OrbitControls} from './lib/OrbitControls'
  let el, renderer, scene, camera, controls, cube, mesh, ready, light, wing
  let inputs = {
    assiette: {value:6, min:5, max:7, step:0.01},
  }

  onMount(() => {
      init()
      animate()
    })

  const init = ()=>{

    // Renderer
    renderer = new THREE.WebGLRenderer({ antialias: true, canvas: el });
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFShadowMap;
    renderer.setSize( window.innerWidth, window.innerHeight );
    
    // Scene
    scene = new THREE.Scene();

    // Light
    //Create a DirectionalLight and turn on shadows for the light
    light = new THREE.DirectionalLight( 0xffffff, 1 );
    light.position.set( 5, 2, 8 ); //default; light shining from top
    light.castShadow = true; // default false
    scene.add( light );
    const ambiant = new THREE.AmbientLight( 0x808080 ); // soft white light
    scene.add( ambiant );

    // Camera
    camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
    controls = new OrbitControls( camera, renderer.domElement );
    camera.position.set( 0, 2, 8 );
    
    drawBackground({grid:true, fog:false, color:0xa0a0a0})
    //drawCube()
    drawWing()
    scene.add(drawLine({x:-100,y:5.9,z:0}, {x:100,y:5.9,z:0}, 'red'))
  
    renderer.render( scene, camera )
    ready = true
  }

  const drawBackground = ({grid = true,  fog= true, color=0xa0a0a0} = {})=>{
    scene.background = new THREE.Color( color );
    if (fog) {
      scene.fog = new THREE.Fog( color, 70, 100 );
    }

    // Grid
    if (grid) {
      const gridElement = new THREE.GridHelper( 500, 100, 0x000000, 0x000000 );
      gridElement.position.y = 0;
      gridElement.material.opacity = 0.2;
      gridElement.material.transparent = true;
      scene.add( gridElement );
    }
  }

  const drawCube = ()=>{
    scene.remove(cube)
    let dimensions = {x:1, y:inputs.height.value, z:1}
    const cubeGeometry = new THREE.BoxGeometry(dimensions.x,dimensions.y,dimensions.z);
    const cubeMaterial = new THREE.MeshPhongMaterial({ color: 'red'});
    cube = new THREE.Mesh(cubeGeometry, cubeMaterial);
    cube.castShadow = true; //default is false
    cube.receiveShadow = true; //default
    scene.add(cube);
    cube.position.set(0,dimensions.y/2,3)
    renderer.render( scene, camera );
  }

  const drawLine = (a,b, color = 'blue' )=>{
      const material = new THREE.LineBasicMaterial( { color } );
      const points = [];
      points.push( new THREE.Vector3( a.x, a.y, a.z ) );
      points.push( new THREE.Vector3( b.x, b.y, b.z ) );
      const geometry = new THREE.BufferGeometry().setFromPoints( points );
      return new THREE.Line( geometry, material );
    }

  
  const drawWing = ()=>{
    scene.remove(wing)
    wing = new THREE.Group();
    const drawHalfWing = (flip=false)=>{
      const halfWing = new THREE.Group();

      const drawTube = (x, z, y, length, radius, color)=>{
        const geometry = new THREE.TorusGeometry( x, radius, 100, 50, length );
        const material = new THREE.MeshPhongMaterial({ color: `rgb(${color},${color},${color})`, side:THREE.DoubleSide });
        const torus = new THREE.Mesh( geometry, material );
        torus.rotation.z = 1.3
        torus.position.set(0,y,z)
        halfWing.add( torus );
        const geometry2 = new THREE.CapsuleGeometry( 0.2-z/25, 0.2-z/25, 5, 20 );
        const capsule = new THREE.Mesh( geometry2, material );
        capsule.position.set(5.6-x*1.25,3.95+y*2,z)
        capsule.scale.y = 0.5
        halfWing.add( capsule );
      }
      for (let i = 0; i < 3; i+=0.05) {
        drawTube(5-i*i/10, i, 1-i*i/10, 0.4-i*i/40, 0.15-i/30, 255-(Math.floor(i*240/5)+15))
      }
      wing.add(halfWing)
      //allongement
      halfWing.scale.x = 1
      //voute
      halfWing.scale.z = 1.5
      if (flip) halfWing.scale.z = -halfWing.scale.z
    }
    
  drawHalfWing()
  drawHalfWing(true)
  scene.add(wing)
  wing.add(drawLine({x:-100, y:6.5, z:0}, {x:100, y:5.3, z:0}))
  }


  
  function animate() {
    requestAnimationFrame( animate );

    //mesh.rotation.y += 0.01;
    wing.rotation.z = inputs.assiette.value
    controls.update();

    renderer.render( scene, camera );
  };

  //$:if(inputs && ready) drawWing()
  
</script>

<div class="controls">
  {#each Object.keys(inputs) as key}
    {key} : {inputs[key].value}<br>
    <input type="range" bind:value={inputs[key].value} min={inputs[key].min} max={inputs[key].max} step={inputs[key].step}><br>
  {/each}
</div>
<canvas bind:this={el}></canvas>

<style>
  canvas{
    background-color: red;
  }
  .controls{
    position: absolute;
    padding: 20px;
    background-color: rgba(255, 255, 255, 0.4);
  }
</style>