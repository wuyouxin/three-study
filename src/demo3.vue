<template>
  <div id="three"></div>
</template>
<script>
// 转动像机
import * as THREE from 'three'
// import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass'
// import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass'
// import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
// import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js'
// import { GUI } from 'three/examples/jsm/libs/dat.gui.module.js'

const BLOOM_SCENE = 1
// const vertexShader = `
//     varying vec2 vUv;
//     void main() {
//       vUv = uv;
//       gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );
//     }`
// const fragmentShader = `
//     uniform sampler2D baseTexture;
//     uniform sampler2D bloomTexture;
//     varying vec2 vUv;

//     vec4 getTexture( sampler2D texelToLinearTexture ) {
//       return mapTexelToLinear( texture2D( texelToLinearTexture , vUv ) );
//     }
//     void main() {
//       gl_FragColor = ( getTexture( baseTexture ) + vec4( 1.0 ) * getTexture( bloomTexture ) );
//     }
// `
export default {
  mounted () {
    this._initBase()
    this.render()
    this.createScene()
  },
  methods: {
    _initBase () {
      // 透视像机
      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 1, 5000)
      // 像机位置
      this.camera.position.set(0, 0, 20)
      // 像机看的地方
      this.camera.lookAt(0, 0, 0)

      const helper = new THREE.CameraHelper(this.camera)
      // 生成场景
      this.scene = new THREE.Scene()
      // 背景色
      this.scene.background = new THREE.Color('rgb(200,200,200)')
      this.scene.add(helper)
      // 添加灯光
      this.scene.add(new THREE.AmbientLight(0x404040))
      // const params = {
      //   exposure: 1,
      //   bloomStrength: 5,
      //   bloomThreshold: 0,
      //   bloomRadius: 0,
      //   scene: 'Scene with Glow'
      // }
      // const renderScene = new RenderPass(this.scene, this.camera)

      // this.bloomPass = new UnrealBloomPass(new THREE.Vector2(window.innerWidth, window.innerHeight), 1.5, 0.4, 0.85)
      // this.bloomPass.threshold = params.bloomThreshold
      // this.bloomPass.strength = params.bloomStrength
      // this.bloomPass.radius = params.bloomRadius

      // // 渲染器
      this.renderer = new THREE.WebGLRenderer({ antialias: true })
      // // 高动态 ?
      // this.renderer.toneMapping = THREE.ReinhardToneMapping
      this.renderer.setSize(window.innerWidth, window.innerHeight)

      // const bloomComposer = new EffectComposer(this.renderer)
      // bloomComposer.renderToScreen = false
      // bloomComposer.addPass(renderScene)
      // bloomComposer.addPass(this.bloomPass)

      // const finalPass = new ShaderPass(
      //   new THREE.ShaderMaterial({
      //     uniforms: {
      //       baseTexture: { value: null },
      //       bloomTexture: { value: bloomComposer.renderTarget2.texture }
      //     },
      //     vertexShader: vertexShader,
      //     fragmentShader: fragmentShader,
      //     defines: {}
      //   }), 'baseTexture'
      // )
      // finalPass.needsSwap = true

      // const finalComposer = new EffectComposer(this.renderer)
      // finalComposer.addPass(renderScene)
      // finalComposer.addPass(finalPass)

      // this.raycaster = new THREE.Raycaster()

      // this.mouse = new THREE.Vector2()

      // window.addEventListener('click', this.onDocumentMouseClick, false)

      // // // 轨迹球控件
      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls.maxPolarAngle = Math.PI * 0.5
      this.controls.minDistance = 1
      this.controls.maxDistance = 100
      // this.controls.addEventListener('change', this.render)

      // const gui = new GUI()
      // gui.add(params, 'scene', ['Scene with Glow', 'Glow only', 'Scene only']).onChange(function (value) {
      //   switch (value) {
      //     case 'Scene with Glow':
      //       bloomComposer.renderToScreen = false
      //       break
      //     case 'Glow only':
      //       bloomComposer.renderToScreen = true
      //       break
      //     case 'Scene only':
      //       // nothing to do
      //       break
      //   }
      //   this.render()
      // })

      // const folder = gui.addFolder('Bloom Parameters')

      // folder.add(params, 'exposure', 0.1, 2).onChange(function (value) {
      //   this.renderer.toneMappingExposure = Math.pow(value, 4.0)
      //   this.render()
      // })

      // folder.add(params, 'bloomThreshold', 0.0, 1.0).onChange(function (value) {
      //   this.bloomPass.threshold = Number(value)
      //   this.render()
      // })

      // folder.add(params, 'bloomStrength', 0.0, 10.0).onChange(function (value) {
      //   this.bloomPass.strength = Number(value)
      //   this.render()
      // })

      // folder.add(params, 'bloomRadius', 0.0, 1.0).step(0.01).onChange(function (value) {
      //   this.bloomPass.radius = Number(value)
      //   this.render()
      // })

      document.body.appendChild(this.renderer.domElement)
      window.addEventListener('resize', this.onWindowResize, false)
    },
    disposeMaterial (obj) {
      if (obj.material) {
        obj.material.dispose()
      }
    },
    createScene () {
      this.scene.traverse(this.disposeMaterial)
      this.scene.children.length = 0
      const geometry = new THREE.IcosahedronBufferGeometry(1, 2)
      for (let i = 0; i < 50; i++) {
        const color = new THREE.Color()
        color.setHSL(Math.random(), 0.7, Math.random() * 0.2 + 0.05)
        const material = new THREE.MeshBasicMaterial({ color: color })
        const cubeEdges = new THREE.EdgesGeometry(geometry, 1)
        // { color: 0xffffff, depthTest: false } 深度测试，若开启则是边框透明的效果
        const edgesMtl = new THREE.LineBasicMaterial({ color: 0xffffff })
        const cubeLine = new THREE.LineSegments(cubeEdges, edgesMtl)
        const sphere = new THREE.Mesh(geometry, material)
        sphere.add(cubeLine)
        sphere.position.x = Math.random() * 14 - 5
        sphere.position.y = Math.random() * 14 - 5
        sphere.position.z = Math.random() * 14 - 5
        this.scene.add(sphere)
      }
      this.render()
    },
    onDocumentMouseClick (event) {
      event.preventDefault()
      this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1
      this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1

      this.raycaster.setFromCamera(this.mouse, this.camera)
      const intersects = this.raycaster.intersectObjects(this.scene.children)
      if (intersects.length > 0) {
        var object = intersects[0].object
        object.layers.toggle(BLOOM_SCENE)
        this.render()
      }
    },
    render () {
      requestAnimationFrame(this.render)
      // this.controls.update()
      this.renderer.render(this.scene, this.camera)
    },
    onWindowResize () {
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      this.camera.aspect = window.innerWidth / window.innerHeight
      this.camera.updateProjectionMatrix()
    }

  }
}
</script>

<style lang="stylus" scoped></style>
