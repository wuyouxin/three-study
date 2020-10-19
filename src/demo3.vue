<template>
  <div class="plant-wrapper">
    <div
      ref="three"
      style="height:100%;"
    />
    <div class="selete-wrapper">
      <div class="title">厂区选择</div>
      <div class="contetn">
        <el-select
          v-model="value"
          style="width:100%;"
          placeholder="请选择"
        >
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
      </div>
    </div>
  </div>
</template>

<script>
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import TWEEN from 'three-tween'
export default {
  data () {
    return {
      options: [{
        value: '100001',
        label: '炼钢厂',
      }],
      value: '100001',
    }
  },
  mounted () {
    this._initBase()
  },
  methods: {

    _initBase () {
      this.camera = new THREE.PerspectiveCamera(43, this.$refs.three.clientWidth / this.$refs.three.clientHeight, 0.1, 30)

      this.camera.position.set(0, 5, 12)
      this.camera.lookAt(0, 0, 0)

      this.scene = new THREE.Scene()
      this.scene.background = new THREE.Color(0xDCDFE6)
      //   this.scene.add(new THREE.AmbientLight(0x8dbdf7, 0.5))

      const pointLight1 = new THREE.PointLight(0xdad6be, 3, 20)
      pointLight1.position.set(9, 11, 8)
      pointLight1.castShadow = true
      this.scene.add(pointLight1)

      const pointLight2 = new THREE.PointLight(0xEFEFEF, 2, 30)
      pointLight2.position.set(-8, 13, -8)
      pointLight2.castShadow = true
      this.scene.add(pointLight2)

      this.renderer = new THREE.WebGLRenderer({ antialias: true })
      this.renderer.setSize(this.$refs.three.clientWidth, this.$refs.three.clientHeight)
      this.renderer.shadowMap = true

      this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls.maxPolarAngle = Math.PI * 0.48
      this.controls.minPolarAngle = Math.PI * 0.18
      this.controls.enablePan = false
      this.controls.enableDamping = true
      this.controls.minDistance = 2
      this.controls.maxDistance = 20
      this.controls.autoRotate = true
      this.controls.autoRotateSpeed = 0.5
      this.renderer.domElement.removeAttribute('tabindex')
      this.$refs.three.appendChild(this.renderer.domElement)

      const loader = new GLTFLoader().setPath('/static/three-model/')

      loader.load('model.glb', gltf => {
        gltf.scene.traverse(child => {
          if (child.isMesh) {
            if (child.name === 'Dizuo') {
              const meshMeshStandardMaterial = new THREE.MeshStandardMaterial({
                side: THREE.FrontSide,
                color: 0x464646,
                // 粗糙度属性
                roughness: 0.9,
              })
              child.material = meshMeshStandardMaterial
              child.castShadow = true
            }
            if (child.name === 'Qita') {
              const meshMeshStandardMaterial = new THREE.MeshStandardMaterial({
                side: THREE.FrontSide,
                color: 0xc5c8ce,
                // 粗糙度属性
                roughness: 0.82,
              })
              child.material = meshMeshStandardMaterial
              child.receiveShadow = true
            }
            if (child.name === 'Diyitai') {
              const meshMeshStandardMaterial = new THREE.MeshStandardMaterial({
                side: THREE.FrontSide,
                color: 0x409EFF,
                // 粗糙度属性
                roughness: 0.65,
              })
              this.diyitai = child
              child.material = meshMeshStandardMaterial
              child.receiveShadow = true

              const random = (min, max) => {
                if (isNaN(min) || isNaN(max)) {
                  return null
                }
                if (min > max) {
                  min ^= max
                  max ^= min
                  min ^= max
                }
                return (Math.random() * (max - min) | 0) + min
              }

              setInterval(() => {
                const val = random(0, 10)
                const calcVal = val / 10
                const moveVal = 5 + (-10) * calcVal
                this.tween(child, moveVal)
              }, 5000)
            }
          }
        })

        window.addEventListener('resize', this.onWindowResize, false)
        document.body.addEventListener('click', this.onMouseClick, false)
        this.scene.add(gltf.scene)
      })

      const planeBufferGeometry = new THREE.PlaneBufferGeometry(200, 200)
      planeBufferGeometry.castShadow = true
      const plane = new THREE.Mesh(planeBufferGeometry, new THREE.MeshBasicMaterial({ color: 0xDCDFE6, side: true }))
      plane.rotation.x = -Math.PI / 2
      const gridHelper = new THREE.GridHelper(200, 200, 0xC0C4CC, 0xC0C4CC)
      this.scene.add(plane)
      this.scene.add(gridHelper)

      this.render()
    },

    tween (child, tweenVal) {
      new TWEEN.Tween(child.position).to({ x: tweenVal }, 5000).start()
    },

    onMouseClick (event) {
      if (!this.$refs.three) return
      event.preventDefault()
      const mouse = new THREE.Vector2()
      const raycaster = new THREE.Raycaster()
      const { x = 210, y = 91 } = this.$refs.three.getBoundingClientRect()
      mouse.x = ((event.clientX - x) / (this.$refs.three.clientWidth)) * 2 - 1
      mouse.y = -((event.clientY - y) / (this.$refs.three.clientHeight)) * 2 + 1
      raycaster.setFromCamera(mouse, this.camera)
      const intersects = raycaster.intersectObject(this.diyitai)
      if (intersects.length) {
        this.diyitai.material.color.setHex(0xff0000)
        setTimeout(() => {
          this.$router.push({
            path: '/RunningStatusMonitoring/views/RealTimeData/RealTimeData',
          })
        }, 500)
      } else {
        this.diyitai.material.color.setHex(0x409EFF)
      }
    },
    render () {
      TWEEN.update()
      requestAnimationFrame(this.render)
      this.renderer.render(this.scene, this.camera)
      this.controls.update()
    },
    onWindowResize () {
      if (!this.$refs.three) return
      this.renderer.setSize(this.$refs.three.clientWidth, this.$refs.three.clientHeight)
      this.camera.aspect = this.$refs.three.clientWidth / this.$refs.three.clientHeight
      this.camera.updateProjectionMatrix()
    },
  },
}
</script>

<style lang="stylus" scoped>
.plant-wrapper
  width 100%
  height 100%
  background #DCDFE6
  .selete-wrapper
    position absolute
    top 30px
    left 30px
    overflow hidden
    width 280px
    height 76px
    border-radius 6px
    background #ffffff
    .title
      width 100%
      height 34px
      border-bottom 1px solid #c3c3c3
      background #f8f8f9
      text-align center
      font-weight 500
      font-size 15px
      line-height 34px
    .contetn
      padding 4px
</style>
