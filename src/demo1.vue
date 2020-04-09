<template>
  <div id="three"></div>
</template>

<script>
// 转动像机
import { Color, HemisphereLight, Scene, WebGLRenderer, PerspectiveCamera, BoxGeometry, MeshPhongMaterial, Mesh, MathUtils, Fog, AxisHelper } from 'three'
export default {
  created () {
    this.lon = 0
    this.lat = 0
    this.scene = new Scene()
    this.scene.background = new Color('rgba(20,40,70,0.8)')

    this.camera = new PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
    this.scene.fog = new Fog(0xf7d9aa, 100, 950)
    this.axes = new AxisHelper(10)
    this.renderer = new WebGLRenderer({ antialias: true })
    this.renderer.setSize(window.innerWidth, window.innerHeight)
    this.renderer.setPixelRatio(window.devicePixelRatio)

    // const textureLoader = new TextureLoader()
    // textureLoader.load(require('./assets/bg.jpg'), texture => {
    //   texture.mapping = UVMapping
    //   const options = {
    //     generateMipmaps: true,
    //     minFilter: LinearMipmapLinearFilter,
    //     magFilter: LinearFilter
    //   }
    //   this.scene.background = new WebGLCubeRenderTarget(1024, options).fromEquirectangularTexture(this.renderer, texture)
    // })

    document.body.appendChild(this.renderer.domElement)
    window.addEventListener('resize', () => this.onWindowResize())

    const boxGeometry = new BoxGeometry(20, 20, 20)
    const meshBasicMaterial = new MeshPhongMaterial({ color: '#fa5241' })

    const mesh = new Mesh(boxGeometry, meshBasicMaterial)
    mesh.name = 'box'
    const light = new HemisphereLight(0xbbbbff, 0x444422, 1.2)
    light.position.set(0, 1, 0)

    mesh.rotation.y = 30
    mesh.rotation.x = 50

    this.scene.add(light)
    this.scene.add(mesh)
    this.scene.add(this.axes)

    this.camera.position.set(0, 0, 15)

    document.addEventListener('mousedown', this.onDocumentMouseDown, false)
    document.addEventListener('wheel', this.onDocumentMouseWheel, false)

    this.render()
  },
  methods: {
    render () {
      window.requestAnimationFrame(() => this.render())
      // const skyboxMesh = this.scene.getObjectByName('box')
      // skyboxMesh.rotation.y += 0.01
      // skyboxMesh.rotation.x += 0.01
      this.lat = Math.max(-85, Math.min(85, this.lat))
      const phi = MathUtils.degToRad(90 - this.lat)
      const theta = MathUtils.degToRad(this.lon)
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
      this.camera.fov = MathUtils.clamp(fov, 10, 75)
      this.camera.updateProjectionMatrix()
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
