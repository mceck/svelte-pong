<script lang="ts">
	import { onMount } from "svelte";

	const X = 20;
	const mapSize = X ** 2;
	const center = Math.round(mapSize / 2 + X / 2);

	const dirMap = {
		u: -X,
		ul: -X - 1,
		ur: -X + 1,
		d: X,
		dl: X - 1,
		dr: X + 1,
	};

	let boxes: string[];
	let ball = center;
	let ballDirection = "u";
	let player1 = {
		length: 3,
		x: Math.round(X / 2),
	};
	let p1Score = 0;

	let player2 = {
		length: 3,
		x: Math.round(X / 2),
	};
	let p2Score = 0;

	$: {
		boxes = new Array(mapSize).fill("empty");
		boxes[ball] = "ball";
		//player1
		for (let i = 0; i < player1.length; i++)
			boxes[player1.x + i] = "player";
		//player2
		for (let i = 0; i < player2.length; i++)
			boxes[X * (X - 1) + player2.x + i] = "player";
	}

	function moveBall() {
		// check collisions
		const x = ball % X,
			y = Math.floor(ball / X);
		let dist1 = x - player1.x;
		let dist2 = x - player2.x;
		if (
			//player1
			(y === 1 && dist1 >= -1 && dist1 <= player1.length) ||
			//player2
			(y === X - 2 && dist2 >= -1 && dist2 <= player2.length)
		) {
			if (ballDirection[0] === "u") {
				// player1
				if (dist1 <= 0) ballDirection = "dl";
				else if (dist1 >= player1.length - 1) ballDirection = "dr";
				else ballDirection = ballDirection.replace("u", "d");
			} else {
				// player2
				if (dist2 <= 0) ballDirection = "ul";
				else if (dist2 >= player2.length - 1) ballDirection = "ur";
				else ballDirection = ballDirection.replace("d", "u");
			}
		} else if (
			// walls
			(x === 0 || x === X - 1) &&
			ballDirection.length > 1
		) {
			if (ballDirection[1] === "l")
				ballDirection = ballDirection.replace("l", "r");
			else ballDirection = ballDirection.replace("r", "l");
		}
		// goal
		else if (y <= -1 || y >= X - 1) {
			ball = center;
			if (y > 0) {
				// player1 win
				p1Score++;
				ballDirection = "d";
			} else {
				//player2 win
				p2Score++;
				ballDirection = "u";
			}
			console.log(`Score ${p1Score} - ${p2Score}`);
		}
		ball += dirMap[ballDirection];
	}

	const onKeyDown: svelte.JSX.EventHandler<KeyboardEvent, Window> = (e) => {
		switch (e.key) {
			// player1
			case "ArrowLeft":
				if (player1.x > 0) player1.x--;
				break;
			case "ArrowRight":
				if (player1.x + player1.length < X) player1.x++;
				break;
			// player2
			case "a":
			case "q":
				if (player2.x > 0) player2.x--;
				break;
			case "d":
			case "e":
				if (player2.x + player2.length < X) player2.x++;
				break;
		}
	};

	onMount(() => {
		setInterval(moveBall, 150);
	});
</script>

<svelte:window on:keydown={onKeyDown} />
<main>
	<header>Score {p1Score} - {p2Score}</header>
	<div class="wnd" style={`max-width: ${X * 20}px`}>
		{#each boxes as box}
			<div class={`box ${box}`} />
		{/each}
	</div>
</main>
