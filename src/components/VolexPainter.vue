<template>
  <div ref="container"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

export default {
  name: "VolexPainter",
  data() {
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(
      45,
      window.innerWidth / window.innerHeight,
      1,
      10000
    );

    new OrbitControls(camera, renderer.domElement);

    const cubeMaterial = new THREE.MeshLambertMaterial({
      color: 0xfeb74c,
    });

    const bufferGeo = new THREE.BufferGeometry();

    const lineMaterial = new THREE.LineBasicMaterial({
      color: 0x000000,
      opacity: 0.2,
      transparent: true,
    });

    const line = new THREE.LineSegments(bufferGeo, lineMaterial);

    const raycaster = new THREE.Raycaster();

    const planeBufferGeo = new THREE.PlaneBufferGeometry(1000, 1000);

    const plane = new THREE.Mesh(
      planeBufferGeo,
      new THREE.MeshBasicMaterial({ visible: false })
    );

    const ambientLight = new THREE.AmbientLight(0x606060);

    const directionalLight = new THREE.DirectionalLight(0xffffff);

    return {
      scene,
      camera,
      bufferGeo,
      cubeMaterial,
      line,
      raycaster,
      plane,
      ambientLight,
      directionalLight,
      renderer,
      objects: [],
      boxes: [
        {
          sizes: [30, 50, 50],
          vector: {
            x: -2.4915689024920766,
            y: 19.599681395701168,
            z: 0,
          },
        },
        {
          sizes: [50, 10, 20],
          vector: {
            x: 168.45821711558273,
            y: -153.19644522758358,
            z: 0,
          },
        },
        {
          sizes: [50, 50, 50],
          vector: {
            x: 173.67590004661594,
            y: 27.122338948066727,
            z: 0,
          },
        },
        {
          sizes: [50, 50, 50],
          vector: {
            x: 274.5690537943176,
            y: 253.04768248427763,
            z: 0,
          },
        },
        {
          sizes: [50, 70, 90],
          vector: {
            x: 270.32063962829824,
            y: -118.44841976291195,
            z: 0,
          },
        },
        {
          sizes: [150, 50, 50],
          vector: {
            x: 252.64073182599344,
            y: -268.3345584644135,
            z: 0,
          },
        },
      ],
    };
  },
  methods: {
    render() {
      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(this.render);
    },
    createPlane() {
      const points = [];
      let size = 500;
      let step = 50;
      for (var i = -size; i <= size; i += step) {
        points.push(new THREE.Vector3(-size, 0, i));
        points.push(new THREE.Vector3(size, 0, i));

        points.push(new THREE.Vector3(i, 0, -size));
        points.push(new THREE.Vector3(i, 0, size));
      }

      this.bufferGeo.setFromPoints(points);
    },
    renderBoxes() {
      let intersects = this.raycaster.intersectObjects(this.objects);

      if (intersects.length > 0) {
        let intersect = intersects[0];

        this.boxes.forEach(({ vector: vec, sizes }) => {
          const cubeGeo = new THREE.BoxGeometry(...sizes);
          const box = new THREE.Mesh(cubeGeo, this.cubeMaterial);
          box.position.copy(vec).add(intersect.face.normal);
          box.position
            .divideScalar(sizes[2])
            .floor()
            .multiplyScalar(sizes[2])
            .addScalar(sizes[2] / 2);

          this.scene.add(box);

          this.objects.push(box);
        });

        this.render();
      }
    },
    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();

      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },
  },
  created() {
    this.camera.position.set(
      -8.712355391925911,
      -1351.2523894806152,
      868.3553850736749
    );
    this.camera.lookAt(new THREE.Vector3());

    this.createPlane();

    this.scene.add(this.line);

    this.bufferGeo.rotateX(-Math.PI / 2);

    this.scene.add(this.plane);

    this.objects.push(this.plane);

    this.scene.add(this.ambientLight);
    this.directionalLight.position.set(1, 0.75, 0.5).normalize();
    this.scene.add(this.directionalLight);

    this.renderer.setClearColor(0xf0f0f0);
    this.renderer.setPixelRatio(window.devicePixelRatio);
    this.renderer.setSize(window.innerWidth, window.innerHeight);
  },
  mounted() {
    this.$refs.container.appendChild(this.renderer.domElement);
    this.renderBoxes();
  },
};
</script>

<style scoped></style>
