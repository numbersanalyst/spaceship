<script>
	import { T, useFrame, useThrelte } from '@threlte/core';
	import { Grid, OrbitControls } from '@threlte/extras';
	import { onMount } from 'svelte';
	import { Raycaster, Vector2, Vector3 } from 'three';

	import Spaceship from './models/spaceship.svelte';

	let planeRef;
	let sphereRef;

	let intersectionPoint;

	const { scene, camera, renderer } = useThrelte();

	let translY = 0;
	let translAccelleration = 0;

	let angleZ = 0;
	let angleAccelleration = 0;

	useFrame(({ scene }) => {
		if (intersectionPoint) {
			const targetY = intersectionPoint?.y || 0;
			translAccelleration += (targetY - translY) * 0.003; // stiffness
			translAccelleration *= 0.95; // damping
			translY += translAccelleration;

			const dir = intersectionPoint.clone().sub(new Vector3(0, translY, 0)).normalize();
			const dirCos = dir.dot(new Vector3(0, 1, 0));
			const angle = Math.acos(dirCos) - Math.PI * 0.5;
			angleAccelleration += (angle - angleZ) * 0.01; // stiffness
			angleAccelleration *= 0.85; // damping
			angleZ += angleAccelleration;
		}
	});

	onMount(() => {
		const raycaster = new Raycaster();
		const pointer = new Vector2();

		function onPointerMove(event) {
			pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
			pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;

			raycaster.setFromCamera(pointer, $camera);

			const intersects = raycaster.intersectObject(planeRef);
			intersectionPoint = intersects[0]?.point;

			if (intersectionPoint) {
				intersectionPoint.x = -3;
				sphereRef.position.copy(intersectionPoint);
			}
		}

		window.addEventListener('pointermove', onPointerMove);
		return () => {
			window.removeEventListener('pointermove', onPointerMove);
		};
	});
</script>

<T.PerspectiveCamera makeDefault position={[-5, 5, 15]} fov={25}>
	<OrbitControls enableDamping target={[0, 0, 0]} />
</T.PerspectiveCamera>

<T.DirectionalLight
	intensity={0.8}
	position.x={5}
	position.y={10}
	castShadows
	shadow-bias={-0.0001}
/>
<T.AmbientLight intensity={0.2} />

<Grid
	position.y={-0.001}
	cellColor="#ffffff"
	sectionColor="#ffffff"
	sectionThickness={0}
	fadeDistance={25}
	cellSize={2}
/>

<Spaceship position={[0,translY,0]} rotation = {[angleZ,0,angleZ, 'ZYX']} />

<T.Mesh renderOrder={2} bind:ref={planeRef} visible={false} >
	<T.PlaneGeometry args={[20, 20]} />
	<T.MeshBasicMaterial color={[1, 0, 1]} transparent opacity={0.25}/>
</T.Mesh>

<T.Mesh position={[1, 2, 0]} bind:ref={sphereRef} visible={false} >
	<T.SphereGeometry args={[0.1, 20, 20]} />
	<T.MeshBasicMaterial color={[1, 0, 0]} />
</T.Mesh>
