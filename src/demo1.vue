<template>
  <div id="three" />
</template>

<script>
// 转动像机
import * as THREE from 'three'
export default {
  created () {
    this.lon = 0
    this.lat = 0
    this.scene = new THREE.Scene()
    this.scene.background = new THREE.Color('rgba(20,40,70,0.2)')
    this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
    // 添加烟雾
    this.scene.fog = new THREE.Fog(0xf7d9aa, 100, 950)
    // 添加坐标线
    this.axes = new THREE.AxisHelper(500)
    this.renderer = new THREE.WebGLRenderer({ antialias: true })
    this.renderer.setSize(window.innerWidth, window.innerHeight)
    this.renderer.setPixelRatio(window.devicePixelRatio)

    // 添加全景图
    const textureLoader = new THREE.TextureLoader()
    textureLoader.load('/static/bg.jpg', texture => {
      texture.mapping = THREE.UVMapping
      const options = {
        generateMipmaps: true,
        minFilter: THREE.LinearMipmapLinearFilter,
        magFilter: THREE.LinearFilter,
      }
      this.scene.background = new THREE.WebGLCubeRenderTarget(1024, options).fromEquirectangularTexture(this.renderer, texture)
    })

    document.body.appendChild(this.renderer.domElement)
    window.addEventListener('resize', () => this.onWindowResize())

    const boxGeometry = new THREE.BoxGeometry(50, 50, 40)
    const meshBasicMaterial = new THREE.MeshPhongMaterial({ color: '#fa5241' })

    this.mesh = new THREE.Mesh(boxGeometry, meshBasicMaterial)
    this.mesh.name = 'box'
    this.light = new THREE.HemisphereLight(0xbbbbff, 0x444422, 1.2)
    this.light.position.set(0, 1, 0)

    this.mesh.rotation.y = 30
    this.mesh.rotation.x = 50

    this.scene.add(this.light)
    this.scene.add(this.mesh)
    this.scene.add(this.axes)

    this.camera.position.set(0, 0, 0)

    document.addEventListener('mousedown', this.onDocumentMouseDown, false)
    document.addEventListener('wheel', this.onDocumentMouseWheel, false)
    this.render()
  },
  methods: {
    render () {
      window.requestAnimationFrame(() => this.render())
      const skyboxMesh = this.scene.getObjectByName('box')
      // 转起来
      skyboxMesh.rotation.y += 0.01
      skyboxMesh.rotation.x += 0.01
      this.lat = Math.max(-85, Math.min(85, this.lat))
      const phi = THREE.MathUtils.degToRad(90 - this.lat)
      const theta = THREE.MathUtils.degToRad(this.lon)
      this.camera.position.x = 100 * Math.sin(phi) * Math.cos(theta)
      this.camera.position.y = 100 * Math.cos(phi)
      this.camera.position.z = 100 * Math.sin(phi) * Math.sin(theta)
      this.camera.lookAt(this.scene.position)
      this.renderer.render(this.scene, this.camera)
    },
    onDocumentMouseDown (event) {
      event.preventDefault()

      this.onPointerDownPointerX = event.clientX
      this.onPointerDownPointerY = event.clientY

      this.onPointerDownLon = this.lon
      this.onPointerDownLat = this.lat

      document.addEventListener('mousemove', this.onDocumentMouseMove, false)
      document.addEventListener('mouseup', this.onDocumentMouseUp, false)
    },
    onDocumentMouseMove (event) {
      this.lon = (event.clientX - this.onPointerDownPointerX) * 0.1 + this.onPointerDownLon
      this.lat = (event.clientY - this.onPointerDownPointerY) * 0.1 + this.onPointerDownLat
    },
    onDocumentMouseUp () {
      document.removeEventListener('mousemove', this.onDocumentMouseMove, false)
      document.removeEventListener('mouseup', this.onDocumentMouseUp, false)
    },
    onDocumentMouseWheel (event) {
      const fov = this.camera.fov + event.deltaY * 0.05
      this.camera.fov = THREE.MathUtils.clamp(fov, 10, 75)
      this.camera.updateProjectionMatrix()
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
