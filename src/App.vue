<template>
  <div id="app"></div>
</template>

<script>
import { Scene, Camera, WebGLRenderer, PerspectiveCamera, BoxGeometry, MeshBasicMaterial, Mesh } from 'three'
export default {
  name: 'App',
  created () {
    this.scene = new Scene()
    this.Camera = new Camera()
    this.camera = new PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
    this.renderer = new WebGLRenderer({ antialias: true })
    this.renderer.setSize(window.innerWidth, window.innerHeight)
    document.body.appendChild(this.renderer.domElement)
    window.addEventListener('resize', () => this.onWindowResize())

    const boxGeometry = new BoxGeometry(5, 5, 5)
    const meshBasicMaterial = new MeshBasicMaterial({ color: '#FFFFFF' })
    const mesh = new Mesh(boxGeometry, meshBasicMaterial)
    mesh.name = 'box'
    this.scene.add(mesh)
    this.camera.position.set(0, 0, 15)
    this.render()
  },
  methods: {
    render () {
      window.requestAnimationFrame(() => this.render())
      const skyboxMesh = this.scene.getObjectByName('box')

      skyboxMesh.rotation.y += 0.01
      skyboxMesh.rotation.x += 0.01

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
