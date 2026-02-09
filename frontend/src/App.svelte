<script lang="ts">
  import { onMount } from "svelte";

  let videoElement: HTMLVideoElement;
  let canvasElement: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;

  let prevFrame: ImageData | null = null;

  let box = { x: 0, y: 0, width: 150, height: 150 };
  let prediction = "";

  const predictEveryNFrames = 10;
  let frameCount = 0;

  onMount(async () => {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ video: true });
      videoElement.srcObject = stream;
      await videoElement.play();

      ctx = canvasElement.getContext("2d")!;
      trackLoop();
    } catch (err) {
      console.error(err);
    }
  });

  async function sendBoxToBackend() {
    // Create a temporary canvas for the box region
    const tempCanvas = document.createElement("canvas");
    tempCanvas.width = 64; // match model input
    tempCanvas.height = 64;
    const tempCtx = tempCanvas.getContext("2d")!;

    // Draw the box region rotated 180 degrees
    tempCtx.translate(32, 32); // center
    tempCtx.rotate(Math.PI); // 180°
    tempCtx.drawImage(
      videoElement,
      box.x,
      box.y,
      box.width,
      box.height,
      -32,
      -32,
      64,
      64,
    );

    const blob: Blob | null = await new Promise((resolve) =>
      tempCanvas.toBlob(resolve, "image/jpeg"),
    );

    if (!blob) return;

    const formData = new FormData();
    formData.append("file", blob, "frame.jpg");

    try {
      const res = await fetch("http://127.0.0.1:8000/predict", {
        method: "POST",
        body: formData,
      });
      const data = await res.json();
      prediction = data.prediction;
    } catch (err) {
      console.error("Prediction error:", err);
    }
  }

  function trackLoop() {
    const width = canvasElement.width;
    const height = canvasElement.height;

    // Draw current video frame
    ctx.drawImage(videoElement, 0, 0, width, height);

    const frame = ctx.getImageData(0, 0, width, height);

    if (prevFrame) {
      let sumX = 0,
        sumY = 0,
        count = 0;

      // Simple motion detection
      for (let i = 0; i < frame.data.length; i += 4) {
        const diff =
          Math.abs(frame.data[i] - prevFrame.data[i]) +
          Math.abs(frame.data[i + 1] - prevFrame.data[i + 1]) +
          Math.abs(frame.data[i + 2] - prevFrame.data[i + 2]);

        if (diff > 50) {
          // motion threshold
          const pixelIndex = i / 4;
          const x = pixelIndex % width;
          const y = Math.floor(pixelIndex / width);
          sumX += x;
          sumY += y;
          count++;
        }
      }

      if (count > 0) {
        const avgX = sumX / count;
        const avgY = sumY / count;
        box.x = avgX - box.width / 2;
        box.y = avgY - box.height / 2;
      }
    }

    prevFrame = frame;

    // Draw bounding box
    ctx.strokeStyle = "red";
    ctx.lineWidth = 3;
    ctx.strokeRect(box.x, box.y, box.width, box.height);

    // Draw prediction text
    if (prediction) {
      ctx.fillStyle = "yellow";
      ctx.font = "20px sans-serif";
      ctx.fillText(prediction, box.x, box.y - 10);
    }

    // Send to backend every N frames
    frameCount++;
    if (frameCount % predictEveryNFrames === 0) {
      sendBoxToBackend();
    }

    requestAnimationFrame(trackLoop);
  }
</script>

<main>
  <div style="position: relative; width: 640px; height: 480px;">
    <video
      bind:this={videoElement}
      autoplay
      playsinline
      width="640"
      height="480"
      style="position: absolute; top: 0; left: 0;"
    ></video>

    <canvas
      bind:this={canvasElement}
      width="640"
      height="480"
      style="position: absolute; top: 0; left: 0; pointer-events: none;"
    ></canvas>
  </div>
</main>
