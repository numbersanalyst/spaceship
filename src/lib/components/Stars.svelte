<script>
	import { T, useFrame } from '@threlte/core';
	import { useTexture, InstancedMesh, Instance } from '@threlte/extras';
	import { Color, DoubleSide, Vector3 } from 'three';

	const map = useTexture('textures/star.png');

	const STARS_COUNT = 350;
	const COLORS = ['#fcaa67', '#c76d59', '#ffffc7', '#8cc5c6', '#a5898c'];

	let stars = [];

	function r(min, max) {
		let diff = Math.random() * (max - min);
		return min + diff;
	}

	function resetStar(star) {
        if (r(0,1) > 0.8) {
            star.pos = new Vector3(r(-30, -10), r(-5, 5), r(-6, 6));
            star.len = r(1.5, 15);
        } else {
            star.pos = new Vector3(r(-45, -15), r(-10.5, 1.5), r(30, -45));
		    star.len = r(2.5, 20);
        }
		star.speed = r(19.5, 42);
		star.color = new Color(COLORS[Math.floor(Math.random() * COLORS.length)])
			.convertSRGBToLinear()
			.multiplyScalar(1.3);

		return star;
	}

	for (let i = 0; i < STARS_COUNT; i++) {
		let star = {
			pos: null,
			len: null,
			speed: null,
			color: null
		};

		stars.push(resetStar(star));
	}

	useFrame((_, delta) => {
		stars.forEach((star) => {
			star.pos.x += star.speed * delta;
			if (star.pos.x > 40) {
				resetStar(star);
			}
		});
        stars = stars;
	});
</script>

{#await map then value}
	<InstancedMesh limit={STARS_COUNT} range={STARS_COUNT}>
		<T.PlaneGeometry args={[1, 0.05]} />
		<T.MeshBasicMaterial alphaMap={value} transparent side={DoubleSide} />
		{#each stars as star}
			<Instance
				position={[star.pos.x, star.pos.y, star.pos.z]}
				scale={[star.len, 1, 1]}
				color={star.color}
			/>
		{/each}
	</InstancedMesh>
{/await}
