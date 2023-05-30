<script lang="ts">
	import * as Matter from 'matter-js';

	// module aliases
	var Engine = Matter.Engine,
		Render = Matter.Render,
		Runner = Matter.Runner,
		Bodies = Matter.Bodies,
		Composite = Matter.Composite;

	// create an engine
	var engine = Engine.create();
	const GRAVITIES = [
		{ x: -1, y: 0 },
		{ x: 0, y: -1 },
		{ x: 1, y: 0 },
		{ x: 0, y: 1 }
	];
	let gravity_i = -1;

	function toggle_gravity() {
		gravity_i++;
		if (gravity_i >= GRAVITIES.length) {
			gravity_i = 0;
		}
		return GRAVITIES[gravity_i];
	}

	const W = 900;
	const H = 900;
	// create a renderer
	var render = Render.create({
		element: document.body,
		options: {
			background: 'white',
			width: W,
			height: H,
			wireframes: false
		},
		engine: engine
	});

	const boundaries_opts = {
		isStatic: true,
		render: {
			fillStyle: 'transparent'
		}
	};
	const CONFINES = { x: [0.33 * W, 0.66 * W], y: [0.33 * H, 0.66 * H] };
	const CENTER = {
		x: (CONFINES.x[1] + CONFINES.x[0]) / 2,
		y: (CONFINES.y[1] + CONFINES.y[0]) / 2
	};
	var ground = Bodies.rectangle(
		CENTER.x,
		CONFINES.y[1],
		CONFINES.x[1] - CONFINES.x[0],
		1,
		boundaries_opts
	);
	var wallLeft = Bodies.rectangle(
		CONFINES.x[0],
		CENTER.y,
		1,
		CONFINES.y[1] - CONFINES.y[0],
		boundaries_opts
	);
	var wallRight = Bodies.rectangle(
		CONFINES.x[1],
		CENTER.y,
		1,
		CONFINES.y[1] - CONFINES.y[0],
		boundaries_opts
	);
	var ceiling = Bodies.rectangle(
		CENTER.x,
		CONFINES.y[0],
		CONFINES.x[1] - CONFINES.x[0],
		1,
		boundaries_opts
	);

	var boxA = Bodies.rectangle(CONFINES.x[0] + 40, CONFINES.y[0] + 40, 80, 80, {
		render: {
			fillStyle: 'rgba(255, 0, 0, 0.5)' //'red'
		}
	});
	var boxB = Bodies.rectangle(CONFINES.x[1] - 40, CONFINES.y[0] + 40, 80, 80, {
		render: {
			fillStyle: 'rgba(0, 0, 255, 0.5)' // 'blue'
		}
	});
	var rectA = Bodies.rectangle(CONFINES.x[1] - 40, CONFINES.y[0] + 40, 40, 160, {
		render: {
			fillStyle: 'rgba(255, 20, 147, 0.5)' // 'deeppink'
		}
	});
	var circA = Bodies.circle(CONFINES.x[1] - 40, CONFINES.y[0] + 40, 40, {
		render: {
			fillStyle: 'rgba(255, 255, 0, 0.5)' // 'yellow'
		}
	});
	var triA = Bodies.polygon(CONFINES.x[1] - 50, CONFINES.y[0] + 50, 3, 30, {
		render: {
			fillStyle: 'rgba(0, 255, 0, 0.5)' // 'green'
		}
	});
	const parts = [boxA, boxB, rectA, circA, triA];
	// add all of the bodies to the world
	Composite.add(engine.world, [...parts, ground, wallLeft, wallRight, ceiling]);

	// run the renderer
	Render.run(render);

	// create runner
	var runner = Runner.create();

	// run the engine
	Runner.run(runner, engine);

	const canvas = document.querySelector('canvas')!;
	canvas.style.placeSelf = "center";
	canvas.addEventListener('click', rotate);

	Matter.Events.on(runner, 'afterUpdate', () => {
		const ctx = canvas.getContext('2d')!;
		ctx.save();
		ctx.translate(CENTER.x, CENTER.y);

		ctx.rotate(Math.PI); // add 180 degrees rotation
		ctx.scale(-1, 1);
		ctx.translate(-CENTER.x, -CENTER.y + CONFINES.y[1] - CONFINES.y[0]);
		ctx.drawImage(canvas, 0, 0);
		ctx.drawImage(canvas, 0, -CONFINES.y[1]);
		ctx.restore();

		ctx.save();

		ctx.translate(CENTER.x, CENTER.y);
		ctx.scale(-1, 1);
		ctx.translate(-CENTER.x + CONFINES.x[0], -CENTER.y);
		ctx.drawImage(canvas, 0, 0);
		ctx.drawImage(canvas, -CONFINES.x[1], 0);
		ctx.restore();

		ctx.save();
		ctx.beginPath();
		ctx.arc(W / 2, H / 2, 1.41 * W, 0, 2 * Math.PI);
		ctx.arc(W / 2, H / 2, W / 3, 0, 2 * Math.PI, true);
		ctx.fillStyle = 'black';
		ctx.fill();
		ctx.restore();

		ligthen(ctx, W / 2, H / 2, W / 3, 'rgba(255,255,255,0.3)');
	});

	// https://stackoverflow.com/a/19795214
	function ligthen(ctx, x, y, radius, color) {
		ctx.save();
		var rnd = 0.03 * Math.sin((1.1 * Date.now()) / 1000);
		const radius_tag = radius * (1 + rnd);
		ctx.globalCompositeOperation = 'lighter';
		ctx.fillStyle = '#0B0B00';
		ctx.beginPath();
		ctx.arc(x, y, radius, 0, 2 * Math.PI);
		ctx.fill();
		ctx.fillStyle = color;
		ctx.beginPath();
		ctx.arc(x, y, radius_tag * 0.9 + rnd, 0, 2 * Math.PI);
		ctx.fill();
		ctx.beginPath();
		ctx.arc(x, y, radius_tag * 0.4 + rnd, 0, 2 * Math.PI);
		ctx.fill();
		ctx.restore();
	}

	function rotate() {
		engine.gravity = { ...engine.gravity, ...toggle_gravity() };
	}

	// function shake() {
	// 	let force = { x: Math.random() * 0.5, y: -Math.random() * 0.5 };
	// 	parts.forEach((b) => Matter.Body.applyForce(b, { x: b.position.x, y: b.position.y }, force));
	// }
</script>

<!-- <button type="button" on:click={shake}>shake</button> -->

<style>
	:global(body) {
		display: grid;
		background-color: black;
	}
</style>
