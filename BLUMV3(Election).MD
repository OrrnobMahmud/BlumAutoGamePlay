```
(() => {
  if (window.BlumAC) return;
  window.BlumAC = true;

  const autoPlay = true;
  const gc = [208, 216, 0];
  const t = 5;

  // Election color triggers
  const ELECTIONS_COLOR_TRIGGERS = [
    {
      red: { min: 250, max: 255 },
      green: { min: 130, max: 140 },
      blue: { min: 90, max: 105 },
    },
    {
      red: { min: 220, max: 230 },
      green: { min: 160, max: 175 },
      blue: { min: 125, max: 135 },
    },
  ];

  // Bomb color triggers (adjust these values based on the actual bomb color)
  const BOMB_COLOR_TRIGGERS = [
    {
      red: { min: 200, max: 210 },
      green: { min: 0, max: 20 },
      blue: { min: 0, max: 20 },
    },
  ];

  if (autoPlay) {
    setInterval(() => {
      const playButton = document.querySelector("button.is-primary, .play-btn");
      if (playButton && playButton.textContent.toLowerCase().includes("play")) {
        playButton.click();
      }
    }, 5000);
  }

  function startObjectDetection() {
    const canvas = document.querySelector("canvas");
    if (canvas) {
      findAndClickObjects(canvas);
    }
    requestAnimationFrame(startObjectDetection);
  }

  startObjectDetection();

  function findAndClickObjects(screenCanvas) {
    const context = screenCanvas.getContext("2d");
    const { width, height } = screenCanvas;
    const imageData = context.getImageData(0, 0, width, height);
    const pixels = imageData.data;

    const bombPositions = new Set();

    // Detect bomb positions
    for (let x = 0; x < width; x += 5) {
      for (let y = 70; y < height; y += 5) {
        const index = (y * width + x) * 4;
        const r = pixels[index];
        const g = pixels[index + 1];
        const b = pixels[index + 2];

        const isBomb = BOMB_COLOR_TRIGGERS.some(color =>
          r >= color.red.min && r <= color.red.max &&
          g >= color.green.min && g <= color.green.max &&
          b >= color.blue.min && b <= color.blue.max
        );

        if (isBomb) {
          bombPositions.add(`${x},${y}`);
        }
      }
    }

    // Click on objects while avoiding bombs
    for (let x = 0; x < width; x += 5) {
      for (let y = 70; y < height; y += 5) {
        const index = (y * width + x) * 4;
        const r = pixels[index];
        const g = pixels[index + 1];
        const b = pixels[index + 2];

        const greenRange =
          r > gc[0] - t && r < gc[0] + t &&
          g > gc[1] - t && g < gc[1] + t &&
          b > gc[2] - t && b < gc[2] + t;

        const isElectionColor = ELECTIONS_COLOR_TRIGGERS.some(color =>
          r >= color.red.min && r <= color.red.max &&
          g >= color.green.min && g <= color.green.max &&
          b >= color.blue.min && b <= color.blue.max
        );

        // Skip clicking if near a bomb
        if (greenRange || isElectionColor) {
          // Check surrounding pixels to see if a bomb is nearby
          let nearBomb = false;
          for (let dx = -5; dx <= 5; dx += 5) {
            for (let dy = -5; dy <= 5; dy += 5) {
              if (bombPositions.has(`${x + dx},${y + dy}`)) {
                nearBomb = true;
                break;
              }
            }
            if (nearBomb) break;
          }

          if (!nearBomb) {
            simulateClick(screenCanvas, x, y);
          }
        }
      }
    }
  }

  function simulateClick(canvas, x, y) {
    const prop = {
      clientX: x,
      clientY: y,
      bubbles: true,
    };
    canvas.dispatchEvent(new MouseEvent("click", prop));
    canvas.dispatchEvent(new MouseEvent("mousedown", prop));
    canvas.dispatchEvent(new MouseEvent("mouseup", prop));
  }
})();
