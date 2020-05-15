<template>
  <div id="three"></div>
</template>

<script>
// 转动像机
import * as THREE from 'three'
import { CSS3DSprite, CSS3DRenderer } from 'three-css3drenderer'
import TWEEN from 'three-tween'
import TrackballControls from 'three-trackballcontrols'

const PARTICLS_TOTAL = 512
export default {
  created () {
    this.objects = []
    this.positions = []
    this.current = 0
    this.init()
    this.render()
  },
  methods: {
    init () {
      this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 1, 5000)
      this.camera.position.set(600, 400, 1500)
      this.camera.lookAt(0, 0, 0)
      this.scene = new THREE.Scene()

      this.image = document.createElement('img')
      this.image.addEventListener('load', () => {
        for (let i = 0; i < PARTICLS_TOTAL; i++) {
          const object = new CSS3DSprite(this.image.cloneNode())
          object.position.x = Math.random() * 4000 - 2000
          object.position.y = Math.random() * 4000 - 2000
          object.position.z = Math.random() * 4000 - 2000
          this.scene.add(object)
          this.objects.push(object)
        }
        this.transition()
      }, false)
      this.image.src = '/static/sprite.png'

      this.plane()
      this.cube()
      for (let i = 0; i < PARTICLS_TOTAL; i++) {
        this.positions.push(
          Math.random() * 4000 - 2000,
          Math.random() * 4000 - 2000,
          Math.random() * 4000 - 2000
        )
      }
      this.sphere()
      this.renderer = new CSS3DRenderer()
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      document.body.appendChild(this.renderer.domElement)
      this.controls = new TrackballControls(this.camera, this.renderer.document)
      window.addEventListener('resize', this.onWindowResize, false)
    },
    render () {
      requestAnimationFrame(this.render)
      TWEEN.update()
      this.controls.update()
      // 变大小 太卡了
      //   const time = performance.now()
      //   for (let i = 0, l = this.objects.length; i < l; i++) {
      //     const object = this.objects[i]
      //     const scale = Math.sin((Math.floor(object.position.x) + time) * 0.002) * 0.3 + 1
      //     object.scale.set(scale, scale, scale)
      //   }
      this.renderer.render(this.scene, this.camera)
    },
    // 正方形
    cube () {
      const amount = 8
      const separation = 150
      const offset = ((amount - 1) * separation) / 2
      for (let i = 0; i < PARTICLS_TOTAL; i++) {
        const x = (i % amount) * separation
        const y = Math.floor((i / amount) % amount) * separation
        const z = Math.floor(i / (amount * amount)) * separation
        this.positions.push(x - offset, y - offset, z - offset)
      }
    },
    // 波浪
    plane () {
      const amountX = 16
      const amountZ = 32
      const separation = 150
      const offsetX = ((amountX - 1) * separation) / 2
      const offsetZ = ((amountZ - 1) * separation) / 2
      for (let i = 0; i < PARTICLS_TOTAL; i++) {
        const x = (i % amountX) * separation
        const z = Math.floor(i / amountX) * separation
        const y = (Math.sin(x * 0.5) + Math.sin(z * 0.5)) * 200
        this.positions.push(x - offsetX, y, z - offsetZ)
      }
    },
    // 球
    sphere () {
      const radius = 750
      for (let i = 0; i < PARTICLS_TOTAL; i++) {
        var phi = Math.acos(-1 + (2 * i) / PARTICLS_TOTAL)
        var theta = Math.sqrt(PARTICLS_TOTAL * Math.PI) * phi

        this.positions.push(
          radius * Math.cos(theta) * Math.sin(phi),
          radius * Math.sin(theta) * Math.sin(phi),
          radius * Math.cos(phi)
        )
      }
    },
    transition () {
      const self = this
      const offset = this.current * PARTICLS_TOTAL * 3
      const duration = 2000
      for (let i = 0, j = offset; i < PARTICLS_TOTAL; i++, j += 3) {
        const object = this.objects[i]
        new TWEEN.Tween(object.position)
          .to({
            x: this.positions[j],
            y: this.positions[j + 1],
            z: this.positions[j + 2]
          }, Math.random() * duration + duration)
          .easing(TWEEN.Easing.Exponential.InOut)
          .start()
      }
      new TWEEN.Tween(this)
        .to({}, duration * 3)
        .onComplete(self.transition)
        .start()
      this.current = (this.current + 1) % 4
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
