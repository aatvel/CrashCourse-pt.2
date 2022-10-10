<template>
  <div>
    <canvas ref="canvas"></canvas>
    <div
      id="container"
      class="absolute text-white text-center w-full max-w-2xl px-6"
      style="top: 50%; transform: translate(-50%, -50%); left: 50%;"
    >
      <h1
        id="aatvel"
        class="font-space-mono text-sm  tracking-wide opacity-0"
        style="transform: translateY(30px)"
      >
      aatvel
      </h1>
      <p
        id="oneWithAn"
        class="font-exo text-4xl opacity-0"
        style="transform: translateY(30px)"
      >
      CRASH COURSE
      </p>
      <a
        id="viewWorkBtn"
        href="https://chriscourses.com/"
        class="border px-4 py-2 rounded-lg text-sm font-space-mono uppercase mt-8 hover:bg-white hover:text-gray-800 inline-block opacity-0"
        style="transform: translateY(30px)"
      >
        View Work
      </a>
    </div>
  </div>
</template>

<script>
import gsap from '../node_modules/gsap'
import OrbitControls from '../node_modules/three/examples/jsm/controls/OrbitControls'
import {
  PlaneGeometry,
  BufferAttribute,
  Raycaster,
  Scene,
  PerspectiveCamera,
  WebGLRenderer,
  MeshPhongMaterial,
  DoubleSide,
  FlatShading,
  Mesh,
  DirectionalLight,
  PointLight,
  BufferGeometry,
  PointsMaterial,
  Float32BufferAttribute,
  Points
} from '../node_modules/three'

export default {
  mounted() {
    const dat = require('dat.gui')
    const OrbitControls = require('three-orbitcontrols')
   
    
    const gui = new dat.GUI()
    const world = {
      plane: {
        width: 260,
        height: 200,
        widthSegments: 50,
        heightSegments: 50
      }
    }
    gui.add(world.plane, 'width', 1, 500).onChange(generatePlane)
    gui.add(world.plane, 'height', 1, 500).onChange(generatePlane)
    gui.add(world.plane, 'widthSegments', 1, 100).onChange(generatePlane)
    gui.add(world.plane, 'heightSegments', 1, 100).onChange(generatePlane)

    function generatePlane() {
      planeMesh.geometry.dispose()
      planeMesh.geometry = new PlaneGeometry(
        world.plane.width,
        world.plane.height,
        world.plane.widthSegments,
        world.plane.heightSegments
      )
      // vertice position randomization
      const { array } = planeMesh.geometry.attributes.position
      const randomValues = []
      for (let i = 0; i < array.length; i++) {
        if (i % 3 === 0) {
          const x = array[i]
          const y = array[i + 1]
          const z = array[i + 2]
          array[i] = x + (Math.random() - 0.5) * 3
          array[i + 1] = y + (Math.random() - 0.5) * 3
          array[i + 2] = z + (Math.random() - 0.5) * 3
        }
        randomValues.push(Math.random() * Math.PI * 2)
      }

      planeMesh.geometry.attributes.position.randomValues = randomValues
      planeMesh.geometry.attributes.position.originalPosition =
      planeMesh.geometry.attributes.position.array

      const colors = []
      for (let i = 0; i < planeMesh.geometry.attributes.position.count; i++) {
        colors.push(0, 0.26, 0.6)
      }
      planeMesh.geometry.setAttribute(
        'color',
        new BufferAttribute(new Float32Array(colors), 3)
      )
    }

    //rscene
    const raycaster = new Raycaster()
    const scene = new Scene()
    const camera = new PerspectiveCamera(
      75,
      innerWidth / innerHeight,
      0.1,
      1000
    )
    const renderer = new WebGLRenderer({
      canvas: this.$refs.canvas
    })
    renderer.setSize(innerWidth, innerHeight)
    renderer.setPixelRatio(devicePixelRatio)

    new OrbitControls(camera, renderer.domElement)

    camera.position.z = 50

    //Plane
    const planeGeometry = new PlaneGeometry(
      world.plane.width,
      world.plane.height,
      world.plane.widthSegments,
      world.plane.heightSegments
    )
    const planeMaterial = new MeshPhongMaterial({
      side: DoubleSide,
      flatShading: FlatShading,
      vertexColors: true
    })

    const planeMesh = new Mesh(planeGeometry, planeMaterial)
    scene.add(planeMesh)


    generatePlane()


    const pointLight = new PointLight(0xffffff, 1)
    pointLight.position.x = 40
  pointLight.position.y = 20
  pointLight.position.z = 40
    scene.add(pointLight)

    const backLight = new PointLight(0xffffff, 1)
    backLight.position.x = - 40
    backLight.position.y = - 20
    backLight.position.z = - 30
    scene.add(backLight)

    //stars
    const starGeometry = new BufferGeometry()
    const starMaterial = new PointsMaterial({color: 0xffffff,
    size: 0.005,
    sizeAttenuation: true,
    opacity: 0.35, 
    transparent: true}
)
    
    const starVerticies = []
    for (let i = 0; i < 8000; i++) {
      const x = (Math.random() - 0.5) * 1500
      const y = (Math.random() - 0.5) * 1500
      const z = (Math.random() - 0.5) * 1500
      starVerticies.push(x, y, z)
    }
    starGeometry.setAttribute(
      'position',
      new Float32BufferAttribute(starVerticies, 3)
    )
    const stars = new Points(starGeometry, starMaterial)
    scene.add(stars)


    const mouse = {
      x: undefined,
      y: undefined
    }


    let frame = 0

    function animate() {
      requestAnimationFrame(animate)
      renderer.render(scene, camera)
      raycaster.setFromCamera(mouse, camera)


      frame += 0.01
      const {
        array,
        originalPosition,
        randomValues
      } = planeMesh.geometry.attributes.position
      for (let i = 0; i < array.length; i += 3) {
        // x
        array[i] =
          originalPosition[i] + Math.cos(frame + randomValues[i]) * 0.01
        // y
        array[i + 1] =
          originalPosition[i + 1] +
          Math.sin(frame + randomValues[i + 1]) * 0.001
      }
      planeMesh.geometry.attributes.position.needsUpdate = true


      const intersects = raycaster.intersectObject(planeMesh)
    if (intersects.length > 0) {
        // console.log(intersects[0].face)
    const {color} = intersects[0].object.geometry.attributes

        intersects[0].object.geometry.attributes.color.setX(intersects[0].face.a, 0.1)  // SetX = R ; Set Y = G; SetZ = B;  Setx(что изменяем, интенсивность)//
        intersects[0].object.geometry.attributes.color.setY(intersects[0].face.a, 0.5)
        intersects[0].object.geometry.attributes.color.setZ(intersects[0].face.a, 1)

        
        intersects[0].object.geometry.attributes.color.setX(intersects[0].face.b, 0.1)
        intersects[0].object.geometry.attributes.color.setY(intersects[0].face.b, 0.5)
        intersects[0].object.geometry.attributes.color.setZ(intersects[0].face.b, 1)
       

        
        intersects[0].object.geometry.attributes.color.setX(intersects[0].face.c, 0.1)
        intersects[0].object.geometry.attributes.color.setY(intersects[0].face.c, 0.5)
        intersects[0].object.geometry.attributes.color.setZ(intersects[0].face.c, 1)

        intersects[0].object.geometry.attributes.color.needsUpdate = true

        const initialColor = {
            r: 0,
            g: 0.25,
            b: 0.5
        }

        const hoverColor = {
            r: 0.8,
            g: 0.8,
            b: 1
        }

        gsap.to(hoverColor, {
            r: initialColor.r ,
            g: initialColor.g ,
            b: initialColor.b,
            duration: 2.5,
            onUpdate: () => {

                intersects[0].object.geometry.attributes.color.setX(intersects[0].face.a, hoverColor.r) 
                intersects[0].object.geometry.attributes.color.setY(intersects[0].face.a, hoverColor.g)
                intersects[0].object.geometry.attributes.color.setZ(intersects[0].face.a, hoverColor.b)
                
                intersects[0].object.geometry.attributes.color.setX(intersects[0].face.b, hoverColor.r)
                intersects[0].object.geometry.attributes.color.setY(intersects[0].face.b, hoverColor.g)
                intersects[0].object.geometry.attributes.color.setZ(intersects[0].face.b, hoverColor.b)
            
                intersects[0].object.geometry.attributes.color.setX(intersects[0].face.c, hoverColor.r)
                intersects[0].object.geometry.attributes.color.setY(intersects[0].face.c, hoverColor.g)
                intersects[0].object.geometry.attributes.color.setZ(intersects[0].face.c, hoverColor.b)
                color.needsUpdate = true    }
        })
      }
      stars.rotation.x += 0.0005
    }
    animate()
    addEventListener('mousemove', (event) => {
      mouse.x = (event.clientX / innerWidth) * 2 - 1
      mouse.y = -(event.clientY / innerHeight) * 2 + 1
    })
    gsap.to('#aatvel', {
      opacity: 1,
      duration: 1.5,
      y: 0,
      ease: 'expo'
    })
    gsap.to('#oneWithAn', {
      opacity: 1,
      duration: 1.5,
      delay: 0.3,
      y: 0,
      ease: 'expo'
    })
    gsap.to('#viewWorkBtn', {
      opacity: 1,
      duration: 1.5,
      delay: 0.6,
      y: 0,
      ease: 'expo'
    })
    document.querySelector('#viewWorkBtn').addEventListener('click', (e) => {
      e.preventDefault()
      gsap.to('#container',{
        opacity: 0,
        duration: 1.5
    })
    gsap.to(camera.position, {
        z:25,
        ease: 'expo.inOut',
        duration: 1.5, 
    })
    gsap.to(camera.rotation, {
        x: Math.PI / 2,
        delay: 1,
        duration: 1.5, 
    })
    gsap.to(camera.position, {
        y: 1000,
        delay: 1.25,
        ease: 'expo.in',
        duration: 1.7, 
        onComplete: () => {
          this.$router.push('/work')
        }
      })
    })


    addEventListener('resize', () => {
      camera.aspect = innerWidth / innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(innerWidth, innerHeight)
    })
  }
}
</script>

<style></style>