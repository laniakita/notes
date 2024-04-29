<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';

	import { createNoise3D, createNoise4D } from 'simplex-noise';
	import { createNoise2D } from 'simplex-noise';
	import * as THREE from 'three';
	import { OrbitControls } from '$threeAddOns/controls/OrbitControls'; //ignore warn

	let canvas: HTMLCanvasElement;

	// mandelbulb stuff
	let DIM = 96;
	class Spherical {
		r: number;
		theta: number;
		phi: number;
		constructor(r: number, theta: number, phi: number) {
			this.r = r;
			this.theta = theta;
			this.phi = phi;
		}
	}
	const spherical1 = (x: number, y: number, z: number) => {
		let r = Math.sqrt(x * x + y * y + z * z);
		let theta = Math.atan2(Math.sqrt(x * x + y * y), z);
		let phi = Math.atan2(y, x);

		return new Spherical(r, theta, phi);
	};
	let n: number = 8;

	let mandelbulb = [];


	let edge = false;
	for (let i = 0; i < DIM; i++) {
		for (let j = 0; j < DIM; j++) {
			for (let k = 0; k < DIM; k++) {
				// mandelbulb stuff
				let x = THREE.MathUtils.mapLinear(i, 0, DIM, -1, 1);
				let y = THREE.MathUtils.mapLinear(j, 0, DIM, -1, 1);
				let z = THREE.MathUtils.mapLinear(k, 0, DIM, -1, 1);

				let zeta = new THREE.Vector3(0, 0, 0);

				const maxIterations = 20;
				let iteration = 0;

				while (iteration < maxIterations) {
					let sphericalZ = spherical1(zeta.x, zeta.y, zeta.z);
					let newX =
						Math.pow(sphericalZ.r, n) *
						Math.sin(sphericalZ.theta * n) *
						Math.cos(sphericalZ.phi * n);
					let newY =
						Math.pow(sphericalZ.r, n) *
						Math.sin(sphericalZ.theta * n) *
						Math.sin(sphericalZ.phi * n);
					let newZ = Math.pow(sphericalZ.r, n) * Math.cos(sphericalZ.theta * n);
					iteration++;

					zeta.x = newX + x;
					zeta.y = newY + y;
					zeta.z = newZ + z;
					if (sphericalZ.r > 16) {
						if (edge) {
							edge = false;
						}
						break;
					}
					if (iteration == maxIterations) {
						if (!edge) {
							edge = true;
							mandelbulb.push(x * 100);
							mandelbulb.push(y * 100);
							mandelbulb.push(z * 100);

						}
						break;
					}
				}

				// three.js points stuff
				//makeInstancePoint(x, y, z);
			}
		}
	}

	onMount(() => {
		// boilerplate three.js
		const renderer = new THREE.WebGLRenderer({ antialias: true, canvas }); //ignore null
		const fov = 90;
		const aspect = window.innerWidth / window.innerHeight; // 2.0 is the canvas default
		const near = 1;
		const far = 410;
		const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
		//const controls = new OrbitControls(camera, renderer.domElement);
		camera.position.set(0, 0, 410);
		//controls.update();
		const scene = new THREE.Scene();

		const material = new THREE.PointsMaterial({
			color: 'white',
			sizeAttenuation: false,
			size: 1 // in pixels
		});
		const bulbGeometry = new THREE.BufferGeometry();
		bulbGeometry.setAttribute('position', new THREE.Float32BufferAttribute(mandelbulb, 3));

		let bulb = new THREE.Points(bulbGeometry, material);
		scene.add(bulb);

		const resizeRendererToDisplaySize = (renderer) => {
			const canvas = renderer.domElement;
			const pixelRatio = window.devicePixelRatio;
			const width = (canvas.clientWidth * pixelRatio) | 0;
			const height = (canvas.clientHeight * pixelRatio) | 0;
			const needResize = canvas.width !== width || canvas.height !== height;
			if (needResize) {
				renderer.setSize(width, height, false);
			}
			return needResize;
		};
		const render = (time) => {
			time *= 0.001; // convert time to seconds
			if (resizeRendererToDisplaySize(renderer)) {
				const canvas = renderer.domElement;
				camera.aspect = canvas.clientWidth / canvas.clientHeight;
				camera.updateProjectionMatrix();
			}

			bulb.rotation.y = time * 0.1;

			//controls.update();

			renderer.render(scene, camera);

			requestAnimationFrame(render);
		};
		requestAnimationFrame(render);
	});
</script>

<canvas id="c" bind:this={canvas} class="w-full h-full" />
