<script>
  import { onMount } from 'svelte';
  import { animate } from 'motion';

  export let selfId;

  onMount(() => {
    const el = document.getElementById(selfId);
    animate(el, {
      opacity: [0, 1],
      y: [-10, 0],
      duration: 0.5,
    });
  });
</script>

<slot></slot>