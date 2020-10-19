<template>
  <div
    ref="three"
    style="width:100vw;height:100vh;"
  />
</template>

<script>
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader'

export default {
  mounted () {
    this.loaderHdr()
  },
  methods: {
    _initBase () {
      this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 10)
      this.camera.position.set(0, 0.8, 1.5)
      //   camera.up
      this.camera.lookAt(0, 0, 0)

      this.axes = new THREE.AxesHelper(3000)
      this.scene.add(this.axes)

      this.scene.add(new THREE.AmbientLight(0xFFFFFF, 1))
      this.pointLight1 = new THREE.PointLight(0x8C9DB7, 15, 1000)
      this.pointLight1.position.set(10, 10, 5)
      this.pointLight1.castShadow = true
      this.scene.add(this.pointLight1)

      this.pointLight2 = new THREE.PointLight(0x8C9DB7, 6, 1000)
      this.pointLight2.position.set(-12, 8, 5)
      this.pointLight2.castShadow = true
      this.scene.add(this.pointLight2)

      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls.maxPolarAngle = Math.PI * 0.5
      this.controls.enablePan = false
      this.controls.minDistance = 2
      this.controls.maxDistance = 8
      this.$refs.three.appendChild(this.renderer.domElement)

      window.addEventListener('resize', this.onWindowResize, false)

      this.render()

      const loader = new GLTFLoader().setPath('/static/demo5/soft02/')

      const textureLoader = new THREE.TextureLoader()
      // const textureCube = new THREE.CubeTextureLoader().load('/static/demo5/lapa_1k.hdr')
      // console.log(textureCube)
      loader.load('soft02.gltf', gltf => {
        gltf.scene.traverse(child => {
          console.log(child)
          if (child.isMesh) {
            const meshMeshStandardMaterial = new THREE.MeshStandardMaterial({
              // color: 0x000000,
              map: textureLoader.load('/static/demo5/soft02/0e7f1b34-a6a4-48c9-b858-7965ab16ccf6'),
              // bumpScale: textureLoader.load('/static/demo5/soft02/611aa6d3-2c10-4d39-ace4-012f7d2c9357.jpeg'),
              // 粗糙度属性
              roughness: 0.68,
              // 法线贴图
              // normalMap: textureLoader.load('/static/demo5/soft02/normalMap.jpg'),
              // 粗糙度贴图
              roughnessMap: textureLoader.load('/static/demo5/soft02/metalnessMap.jpg'),
              // 环境贴图
              // envMap: textureLoader.load('/static/demo5/soft02/4e53958c-1498-4118-b8a9-1e2b3c0908d5.png'),
              // 环境贴图影响程度
              // envMapIntensity: 0.9,
            })
            // const meshMeshStandardMaterial = new THREE.MeshStandardMaterial({
            //   color: 0xFF0000,
            //   metalness: 0,
            //   roughness: 0.2,
            //   // envMap: cubeTexture,
            //   envMapIntensity: 0.2,
            // })
            child.castShadow = true
            child.receiveShadow = true
            child.material = meshMeshStandardMaterial
          }
        })
        this.scene.add(gltf.scene)
      })
    },
    render () {
      requestAnimationFrame(this.render)
      this.renderer.render(this.scene, this.camera)
      this.controls.update()
    },
    onWindowResize () {
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      this.camera.aspect = window.innerWidth / window.innerHeight
      this.camera.updateProjectionMatrix()
    },
    loaderHdr () {
      this.renderer = new THREE.WebGLRenderer({ antialias: true })
      this.renderer.setClearColor(0xFFFFFF)// 设置渲染器背景色
      this.renderer.shadowMap.enabled = true
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      this.renderer.outputEncoding = THREE.sRGBEncoding
      this.renderer.physicallyCorrectLights = true

      this.scene = new THREE.Scene()

      const pmremGenerator = new THREE.PMREMGenerator(this.renderer)
      pmremGenerator.compileEquirectangularShader()
      new RGBELoader()
        .setDataType(THREE.UnsignedByteType)
        .load('/static/demo5/lapa_1k.hdr', texture => {
          const envMap = pmremGenerator.fromEquirectangular(texture).texture
          this.scene.environment = envMap
          this.scene.background = envMap

          pmremGenerator.dispose()
          texture.dispose()
          this._initBase()
        })
    },
    randomColor () {
      const colorArr = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f']
      let color = '#'
      for (let i = 0; i < 6; i++) {
        color += colorArr[this.random(0, 16)]
      }
      return color
    },
    random (min, max) {
      if (isNaN(min) || isNaN(max)) {
        return null
      }
      if (min > max) {
        min ^= max
        max ^= min
        min ^= max
      }
      return (Math.random() * (max - min) | 0) + min
    },
  },
}
</script>
