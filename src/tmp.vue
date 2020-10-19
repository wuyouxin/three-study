<template>
  <div id="three" />
</template>
<script>
// 转动像机
import * as THREE from 'three'
import TrackballControls from 'three-trackballcontrols'
export default {
  created () {
    this._initBase()
    this.render()
  },
  methods: {
    _initBase () {
      // 透视像机
      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 1, 5000)
      // 像机位置
      this.camera.position.set(600, 400, 1500)
      // 像机看的地方
      this.camera.lookAt(0, 0, 0)
      // 生成场景
      this.scene = new THREE.Scene()
      // 背景色
      this.scene.background = new THREE.Color('rgba(20,40,70,0.2)')
      // 渲染器
      this.renderer = new THREE.WebGLRenderer({ antialias: true })
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      // 轨迹球控件
      this.controls = new TrackballControls(this.camera, this.renderer.document)

      document.body.appendChild(this.renderer.domElement)
      window.addEventListener('resize', this.onWindowResize, false)
    },
    render () {
      requestAnimationFrame(this.render)
      this.controls.update()
      this.renderer.render(this.scene, this.camera)
    },
    onWindowResize () {
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      this.camera.aspect = window.innerWidth / window.innerHeight
      this.camera.updateProjectionMatrix()
    },

  },
}
</script>

<style lang="stylus" scoped></style>
