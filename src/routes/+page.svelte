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

	// create a renderer
	var render = Render.create({
		element: document.body,
		options: {
			background: 'white',
			width: 200,
			height: 200,
			wireframes: false
		},
		engine: engine
	});

	// create two boxes and a ground
	var boxA = Bodies.rectangle(50, 0, 80, 80, {
		render: {
			fillStyle: 'red'
		}
	});
	var boxB = Bodies.rectangle(150, 0, 80, 80, {
		render: {
			fillStyle: 'blue'
		}
	});
	var ground = Bodies.rectangle(100, 199.5, 200, 1, {
		isStatic: true,
	});
    var wallLeft = Bodies.rectangle(-0.5, 100, 1, 200, {
		isStatic: true,
	});
    var wallRight = Bodies.rectangle(199.5, 100, 1, 200, {
		isStatic: true,
	});
    var ceiling = Bodies.rectangle(100, 0.5, 200, 1, {
		isStatic: true,
	});

	// add all of the bodies to the world
	Composite.add(engine.world, [boxA, boxB, ground, wallLeft, wallRight, ceiling]);

	// run the renderer
	Render.run(render);

	// create runner
	var runner = Runner.create();

	// run the engine
	Runner.run(runner, engine);

	const canvas = document.querySelector('canvas')!;
	const mirrorRightCanvas = canvas.parentElement!.appendChild(document.createElement('canvas'));
	mirrorRightCanvas.style.transform = 'rotateY(180deg)';
	const mirrorLeftCanvas = canvas.parentElement!.insertBefore(
		document.createElement('canvas'),
		canvas
	);
	mirrorLeftCanvas.style.transform = 'rotateY(180deg)';
	Matter.Events.on(runner, 'afterUpdate', () => {
		mirror(canvas, mirrorRightCanvas);
		cloneCanvas(mirrorRightCanvas, mirrorLeftCanvas);
	});

	function cloneCanvas(oldCanvas: HTMLCanvasElement, dest: HTMLCanvasElement): HTMLCanvasElement {
		//create a new canvas
		// var newCanvas = document.createElement('canvas');
		var context = dest.getContext('2d')!;

		//set dimensions
		dest.width = oldCanvas.width;
		dest.height = oldCanvas.height;

		//apply the old canvas to the new one
		context.drawImage(oldCanvas, 0, 0);

		//return the new canvas
		return dest;
	}

	function mirror(canvas: HTMLCanvasElement, dest: HTMLCanvasElement): HTMLCanvasElement {
		return cloneCanvas(canvas, dest);
	}

	function shake() {
		let force = { x: Math.random() * 0.5, y: -Math.random() * 0.5 };
		[boxA, boxB].forEach((b) =>
			Matter.Body.applyForce(b, { x: b.position.x, y: b.position.y }, force)
		);
	}
</script>

<button type="button" on:click={shake}>shake</button>
