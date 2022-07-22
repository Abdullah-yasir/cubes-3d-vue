<template>
  <div ref="container"></div>
</template>

<script>
import * as THREE from "three";

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

    const rollOverGeo = new THREE.BoxGeometry(50, 50, 50);
    const rollOverMaterial = new THREE.MeshBasicMaterial({
      color: 0xff0000,
      opacity: 0.5,
      transparent: true,
    });

    const rollOverMesh = new THREE.Mesh(rollOverGeo, rollOverMaterial);

    const cubeGeo = new THREE.BoxGeometry(50, 50, 50);
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
    const mouse = new THREE.Vector2();

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
      rollOverGeo,
      rollOverMaterial,
      rollOverMesh,
      cubeGeo,
      cubeMaterial,
      line,
      raycaster,
      mouse,
      plane,
      ambientLight,
      directionalLight,
      objects: [],
      renderer,
      isShiftDown: false,
      cursor: { x: 0, y: 0 },
    };
  },
  methods: {
    render() {
      this.renderer.render(this.scene, this.camera);
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
    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();

      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },
    onMouseMove(e) {
      e.preventDefault();

      this.cursor.x = e.clientX / window.innerWidth - 0.5;
      this.cursor.y = e.clientY / window.innerHeight - 0.5;

      // console.log(this.cursor.x, this.cursor.y);

      this.camera.position.x = this.cursor.x * 500;
      this.camera.lookAt(new THREE.Vector3());
      // this.camera.position.y = this.cursor.y * 100;
      // this.camera.position.z = this.cursor.x + this.cursor.y;

      this.mouse.set(
        (e.clientX / window.innerWidth) * 2 - 1,
        -(e.clientY / window.innerHeight) * 2 + 1
      );

      this.raycaster.setFromCamera(this.mouse, this.camera);

      var intersects = this.raycaster.intersectObjects(this.objects);

      if (intersects.length > 0) {
        var intersect = intersects[0];

        this.rollOverMesh.position
          .copy(intersect.point)
          .add(intersect.face.normal);
        this.rollOverMesh.position
          .divideScalar(50)
          .floor()
          .multiplyScalar(50)
          .addScalar(25);
      }

      this.render();
    },
    onMouseDown(e) {
      e.preventDefault();

      this.mouse.set(
        (e.clientX / window.innerWidth) * 2 - 1,
        -(e.clientY / window.innerHeight) * 2 + 1
      );

      this.raycaster.setFromCamera(this.mouse, this.camera);

      var intersects = this.raycaster.intersectObjects(this.objects);

      if (intersects.length > 0) {
        var intersect = intersects[0];

        // delete cube

        if (this.isShiftDown) {
          if (intersect.object != this.plane) {
            this.scene.remove(intersect.object);

            this.objects.splice(this.objects.indexOf(intersect.object), 1);
          }

          // create cube
        } else {
          const voxel = new THREE.Mesh(this.cubeGeo, this.cubeMaterial);
          voxel.position.copy(intersect.point).add(intersect.face.normal);
          console.log(intersect.point);
          voxel.position
            .divideScalar(50)
            .floor()
            .multiplyScalar(50)
            .addScalar(25);
          this.scene.add(voxel);

          this.objects.push(voxel);
        }

        this.render();
      }
    },
    onKeyDown(e) {
      switch (e.keyCode) {
        case 16:
          this.isShiftDown = true;
          break;
      }
    },
    onKeyUp(e) {
      switch (e.keyCode) {
        case 16:
          this.isShiftDown = false;
          break;
      }
    },
  },
  created() {
    this.camera.position.set(500, 800, 1300);
    this.camera.lookAt(new THREE.Vector3());
    this.scene.add(this.rollOverMesh);

    this.createPlane();

    this.scene.add(this.line);

    // this.bufferGeo.rotateX(-Math.PI / 2);

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

    window.addEventListener("mousemove", this.onMouseMove, false);
    window.addEventListener("mousedown", this.onMouseDown, false);
    window.addEventListener("keydown", this.onKeyDown, false);
    window.addEventListener("keyup", this.onKeyUp, false);
    window.addEventListener("resize", this.onWindowResize, false);
  },
  unmounted() {
    window.removeEventListener("mousemove", this.onMouseMove);
    window.removeEventListener("mousedown", this.onMouseDown);
    window.removeEventListener("keydown", this.onKeyDown);
    window.removeEventListener("keyup", this.onKeyUp);
    window.removeEventListener("resize", this.onWindowResize);
  },
};
</script>

<style scoped></style>
