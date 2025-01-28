<script>
  import { onMount } from 'svelte';
  import { animate } from 'motion';

  export let selfId;
  export let duration;

  onMount(() => {
    const el = document.getElementById(selfId);
    animate(el, {
      opacity: [0, 1],
      y: [-10, 0],
      duration: duration,
    });
  });
</script>

<slot></slot>