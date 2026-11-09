 REDLINE
// CONTROLLER POINTERS — FIXED
const raycaster = new THREE.Raycaster();
const controllers = [];

for (let i = 0; i < 2; i++) {
  const controller = renderer.xr.getController(i);
  scene.add(controller);

  const rayGeometry = new THREE.BufferGeometry()
    .setFromPoints([
      new THREE.Vector3(0, 0, 0),
      new THREE.Vector3(0, 0, -8)
    ]);

  const ray = new THREE.Line(
    rayGeometry,
    new THREE.LineBasicMaterial({
      color: 0xff263f,
      depthTest: false
    })
  );

  ray.renderOrder = 999;
  controller.add(ray);

  controller.addEventListener("select", () => {
    const origin = new THREE.Vector3();
    const direction = new THREE.Vector3();

    controller.getWorldPosition(origin);
    controller.getWorldDirection(direction);

    raycaster.set(origin, direction);
    raycaster.far = 8;

    const hits = raycaster.intersectObjects(buttons, false);

    if (hits.length > 0) {
      hits[0].object.userData.action();
    }
  });

  controllers.push(controller);
}