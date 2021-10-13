<template>
  <div id="container"></div>
</template>
<script>
import * as THREE from 'three'
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js'
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js'
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js'
import { BrightnessContrastShader } from 'three/examples/jsm/shaders/BrightnessContrastShader.js'
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js'
import { FXAAShader } from 'three/examples/jsm/shaders/FXAAShader.js'

export default {
  name: 'ThreeTest', // main component for Three Renders
  data() {
    return {
    }
  },
  methods: {
    /* 
    *  Init the scene, a perspective camera, the renderer with linear/srgb settings,
    *  Orbit controls to move the camera and an effect composer for post processing 
    */
    init: function() { 
      this.scene = new THREE.Scene()
      this.scene.background = new THREE.Color("rgb(0,0,0)").convertSRGBToLinear()

      this.camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      )
      this.camera.position.set(0, 5, -2)

      this.renderer = new THREE.WebGLRenderer({antialias: true, alpha:false})
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      this.renderer.shadowMap.enabled = true // cast shadows
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap // shadow type
      this.renderer.outputEncoding = THREE.sRGBEncoding // srgb colors

      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls.maxPolarAngle = Math.PI * 0.5
      this.controls.minDistance = 1
      this.controls.maxDistance = 10
      this.composer = new EffectComposer( this.renderer ) // to modify render passes
      document.body.appendChild(this.renderer.domElement) // rendering in 

      window.addEventListener('resize', this.onResize )
    },
    // when the web navigator is resized -> we resize the camera view
    onResize: function(){
      this.camera.aspect = window.innerWidth / window.innerHeight
      this.camera.updateProjectionMatrix()
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      this.renderer.render(this.scene, this.camera)
    },
    // main animate func
    animate: function() {
      requestAnimationFrame(this.animate)

      this.composer.render(this.scene, this.camera)
    },
    // Adding lights to the scene
    lights: function() {
      // main directional light
      let directionalLight = new THREE.DirectionalLight(new THREE.Color(251, 250, 245).convertSRGBToLinear() , 1) // soft white light converted to Linear
      directionalLight.castShadow = true
      this.scene.add( directionalLight )
    },
    // Post processing settings for the scene
    postProcessing: function()
    {
      // default render to render the scene
      this.composer.addPass(new RenderPass(this.scene, this.camera))
      // adding FXAA 
      this.composer.addPass(new ShaderPass( FXAAShader ))
      // bloom effect shader, can be modified with parameters
      this.composer.addPass(new UnrealBloomPass({x: 1024, y:1024}, 1.5, 0.4, 0.85))
      // shader for better contrast 
      this.composer.addPass(new ShaderPass(BrightnessContrastShader))
    }

  },
  mounted() {
    this.init()
    this.animate()
    this.lights()
    this.postProcessing()
  }
}
</script>