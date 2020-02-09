<script>
	export let name;
  let canvas, ctx, w_width;
  import { Random, MersenneTwister19937 } from 'random-js';
  import sha256 from 'tiny-sha256';

  import Noise from './perlin-noise.js';

  let seed = window && window.location.hash ? window.location.hash.replace('#', '') : sha256(new Date());

  let random;

  function parseHexString(str) {
      var result = [];
      while (str.length >= 8) {
          result.push(parseInt(str.substring(0, 8), 16));

          str = str.substring(8, str.length);
      }
    return result;
  }

  function setupCanvas(canvas) {
      let dpr = window.devicePixelRatio || 1;
      console.log(dpr);
      let rect = canvas.getBoundingClientRect();

      canvas.width = 1500 *dpr;//rect.width * dpr;
      canvas.height = 1500 * dpr;//rect.height * dpr;
      canvas.getContext('2d').scale(dpr*2, dpr*2);
  }

  function pickColorFromPallete(index) {
      const pallete = [
          [50.9, '#a7a9c8'],
          [23.7, '#4b5f8a'],
          [7.9, '#c12934'],
          [5.2, '#942e38'],
          [4.7, '#f5f5f5'],
          [2.2, '#8b4559'],
          [2.0, '#322535'],
          [1.5, '#bb89a0'],
          [1.3, '#774268'],
          [0.4, '#d9b6b8'],
      ];
      let sumPercentage = 0;
      for (let i = pallete.length - 1 ; i > 0; i--) {
          sumPercentage += pallete[i][0];
          if (sumPercentage/100 > index) return pallete[i][1];
      }
      return pallete[0][1];
  }



  function map_range(value, low1, high1, low2, high2) {
    return low2 + (high2 - low2) * (value - low1) / (high1 - low1);
  }

  // NOT USED
  function generateCircleGrid({num_rows, num_columns}) {
      let grid = [];
      let default_angle = Math.PI * 0.25
      for (let column =0; column < num_columns; column++) {
          grid[column] = [];
          for (let row = 0; row < num_rows; row++) {
              let angle = (row / num_rows) * Math.PI
              grid[column][row] = angle
          }
      }
      return grid;
  }

  function generateNoiseGrid({random, num_rows, num_columns}) {
      let grid = [];
      const scale = random.real(0.001, 0.02);
      for (let column =0; column < num_columns; column++) {
          grid[column] = [];
          for (let row = 0; row < num_rows; row++) {
              // Processing's noise() works best when the step between
              // points is approximately 0.005, so scale down to that
              let scaled_x = column * scale;
              let scaled_y = row * scale;
              // get our noise value, between 0.0 and 1.0
              let noise_val = Noise.perlin2(scaled_x, scaled_y)
              // translate the noise value to an angle (betwen 0 and 2 * PI)
              let angle = map_range(noise_val, 0.0, 1.0, 0.0, Math.PI * 2.0)
              grid[column][row] = angle
          }
      }
      return grid;
  }


  function drawCurve({ctx, grid, startX, startY, colorScale, num_steps, resolution, left_x, top_y, step_length}) {
      let x = startX;
      let y = startY;

      ctx.beginPath()
      ctx.lineTo(x, y)
      for (let n = 0; n < num_steps; n++) {
          ctx.lineTo(x, y)
          let x_offset = x - left_x
          let y_offset = y - top_y
          let column_index = Math.floor(x_offset / resolution)
          let row_index = Math.floor(y_offset / resolution)
          // NOTE: normally you want to check the bounds here
          let grid_angle = grid[column_index][row_index]
          let x_step = step_length * Math.cos(grid_angle)
          let y_step = step_length * Math.sin(grid_angle)
          x = x + x_step
          y = y + y_step
//          console.log(x,y);
      }
      ctx.strokeStyle = pickColorFromPallete(Noise.perlin2(x/ colorScale, y / colorScale));

      ctx.stroke();
  }

  function render(ctx, seed) {
      let width = 1000;
      let height = 1000;
      let num_steps;// = random.integer(5, 20);
      let step_length;// = random.integer(5, 30);

      let left_x = Math.floor(width * -0.5)
      let right_x = Math.floor(width * 1.5)
      let top_y = Math.floor(height * -0.5)
      let bottom_y = Math.floor(height * 1.5)
      let resolution = Math.floor(width * 0.01)
      let num_columns = (right_x - left_x) / resolution
      let num_rows = (bottom_y - top_y) / resolution

      random = new Random(MersenneTwister19937.seedWithArray(parseHexString(seed)));
      let colorScale = random.integer(1, 1000);

      Noise.seed(random.integer(1, 1000000000));

      num_steps = random.integer(5, 20);
      step_length = random.integer(5, 30);

      let grid = generateNoiseGrid({ random, num_rows, num_columns });
      let num_lines = random.integer(5000, 20000)
      for (let i = 0; i < num_lines; i++) {
          const x = random.integer(0, width);
          const y = random.integer(0, height);
          drawCurve({ctx, grid, startX: x, startY: y, colorScale, num_steps, resolution, left_x, top_y, step_length});
      }

  }

  function handleClick(e) {
      let ctx = canvas.getContext('2d');
      seed = sha256(new Date());
      ctx.fillStyle = 'black';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      render(ctx, seed)
  }

  $: if(canvas) {
      setupCanvas(canvas);
      render(canvas.getContext('2d'), seed)
  };
  $: if(canvas && w_width) {
      console.log(w_width);
      setupCanvas(canvas);
      render(canvas.getContext('2d'), seed)
  }
</script>

<svelte:window bind:innerWidth={w_width} />
<main>
	<h1><a href="/">GRASS</a></h1>
  <canvas
    on:click={handleClick}
    on:touchend={handleClick}
    bind:this={canvas}
    ></canvas>
  <p>
    <a href={`#${seed}`}>{seed}</a>
  </p>
  <p>
    <a href="https://gitlab.com/sergey-larionov/grass">gitlab.com/sergey-larionov/grass</a>
  </p>
</main>

<style>
	main {
		  text-align: center;
		  padding: 1em;
		  max-width: 100vw;
      min-width: 320px;
	}
  canvas {
      width: calc(100vw - 3em);
      height: calc(100vw - 3em);
      min-width: min(1000px, calc(100vw - 3em));
      min-height: min(1000px, calc(100vw - 3em));
      max-height: 1000px;
		  margin: 0 auto;
  }
  a {
		  font-size: 0.7em;
      color:  darkgray;/*#f5f5f5;*/
		  font-weight: 100;
  }
	h1  a {
		  color:#a7a9c8;
		  text-transform: uppercase;
		  font-size: 2em;
		  font-weight: 100;
      text-decoration: none;
	}

  p {
      margin: 0;
  }



	@media (min-width: 1000px) {
      a {
		      font-size: 1em;
      }
		  canvas {
			    width: 1000px;
		  }
	}
</style>
