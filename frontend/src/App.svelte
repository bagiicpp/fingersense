<script lang="ts">

  import { onMount } from "svelte";
  let videoElement: HTMLVideoElement;


  onMount(async () => {

    try {
      const stream = await navigator.mediaDevices.getUserMedia({video: true})
    
      videoElement.srcObject = stream
      videoElement.play()
    } catch (err) {
      console.error(err)
    }
    
  })

</script>

<main>
  <h1 class="font-bold text-6xl">Hiii</h1>

  <video bind:this={videoElement} autoplay playsinline height="800px" width="800px"></video>

</main>
