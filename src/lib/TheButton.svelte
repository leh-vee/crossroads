<script>
  import { goto } from '$app/navigation';
  import { tweened } from 'svelte/motion';
  import { untrack } from 'svelte';
  import { circOut, elasticOut, elasticIn, elasticInOut, quartIn, cubicOut } from 'svelte/easing';
  import { poemMetadata, percentRead, isPoemUnread, isPoemRevealed,
    isMooning, isTyping, isGerminating, isBreaking } from '$lib/store.js';

  let { x, y, r, animateBtnIn } = $props();

  const maxDiagonal = untrack(() => Math.hypot(x, y));
  const id = $poemMetadata.sefirahId;

  let circumference = $derived(Math.round(2 * Math.PI * r));
  let readOffset = $derived(Math.round(circumference * (1 - $percentRead)));
  let unreadOffset = $derived($percentRead === 0 ? circumference : circumference - readOffset);  

  const btnRadius = tweened(0, {
    duration: Math.PI * 1000,
    easing: circOut
  });

  $effect(async () => { if (animateBtnIn) btnRadius.set(r) });

  const glowRadius = tweened(0, {
    duration: Math.PI * 1000,
    easing: elasticOut
  });

  $effect(async () => { 
    while ($isTyping || $isBreaking) {
      await glowRadius.set(r * 1.5, { duration: 1570, easing: elasticOut });
      await glowRadius.set(r * 1.3, { duration: 1570, easing: elasticIn });
    }
    if ($isGerminating) glowRadius.set(0, { duration: 3141, easing: cubicOut });
  });

  let isGreyedOut = $state(false);
  let isNova = $state(false);

  // $effect(async () => {
  //   if ($isPoemRevealed) {
  //     if (isGreyedOut) {
  //       animateOut();
  //     } else {
  //       $isTyping = false;
  //     }
    // } else if ($isTyping && isLabelVisible) {
    //   elasticGlow();
    // }
  // });

  async function animateOut() {
    const duration = 314;
    isNova = true;
    glowRadius.set(maxDiagonal * 5, { duration, easing: quartIn });
    await btnRadius.set(maxDiagonal, { duration, easing: quartIn });
    goto('/');
  }

  function onBtnAnimationEnd(event) {
    const animeName = event.animationName.split('-').pop();
    if (animeName === 'greyOut') isGreyedOut = true;
  }

</script>

<defs>
  <radialGradient id="glow-grad-{id}">
    <stop offset="0%" stop-color="gold" stop-opacity="0.8"/>
    <stop offset="50%" stop-color="gold" stop-opacity="0.4"/>
    <stop offset="100%" stop-color="gold" stop-opacity="0"/>
  </radialGradient>
</defs>

<g class:period={ $isPoemRevealed }>
  <circle class="btn whole" class:cool={ animateBtnIn }
    class:show={ $isPoemUnread } cx={x} cy={y} r={ $btnRadius } 
    transform-origin="{x}px {y}px" />
  <g transform={`translate(${x} ${y}) scale(-1 1) translate(${-x} ${-y})`}>
    <circle
      class="btn partial"
      class:show={ !$isPoemUnread && ($isGerminating || $isMooning) }
      fill="none"
      cx={x}
      cy={y}
      r={r}
      transform={`rotate(-90 ${x} ${y})`}
      stroke-dasharray={circumference}
      style:--circumference="{circumference}px"
      style:--unread-offset="{unreadOffset}px"
    />
  </g>
  <circle class="progress" cx={x} cy={y} r={ r }
    transform={`rotate(-90 ${x} ${y})`} fill="none"
    stroke-dasharray={ circumference } 
    style:--circumference="{circumference}px"
    style:--read-offset="{readOffset}px"
    class:show={ !$isPoemUnread && ($isGerminating || $isMooning) } />
  <circle class="sun" cx={x} cy={y} r={ $glowRadius } fill="url(#glow-grad-{id})" />
  {#if $isMooning}
    <text x={x} y={y + 2} >{id}</text>
  {/if}
</g>

<style>
  circle.btn {
    stroke-width: 0;
    stroke: none;
  }

  circle.whole.btn.show {
    stroke-width: 2;
    stroke: gold;
  }

  circle.btn.whole.show.cool {
    animation: coolDown 3.14s ease-in forwards;
    stroke: var(--moon-glow-stroke);
  }

  circle.btn.whole.show.cool {
    animation: coolDown 3.14s ease-in forwards;
    stroke: var(--moon-glow-stroke);
  }

  @keyframes coolDown {
    0%   { 
      stroke: gold; 
    }
    100% { 
      stroke: var(--moon-glow-stroke);
    }
  }

  circle.partial.btn {
    stroke-dashoffset: var(--circumference);
  }

  circle.partial.btn.show {
    stroke-width: 2;
    stroke: var(--moon-glow-stroke);
    animation: drawUnread 3.14s ease-out forwards;
  }

  @keyframes drawUnread {
    from { stroke-dashoffset: var(--circumference); }
    to   { stroke-dashoffset: var(--unread-offset); }
  }

  circle.progress {
    stroke-dashoffset: var(--circumference);
    stroke-width: 0;
    stroke: none;
  }

  circle.progress.show {
    stroke-width: 2;
    stroke: gold;
    animation: drawRead 3.14s ease-out forwards;
  }

  @keyframes drawRead {
    from { stroke-dashoffset: var(--circumference); }
    to   { stroke-dashoffset: var(--read-offset); }
  }

  text {
    text-anchor: middle;
    alignment-baseline: middle;
    fill: gold;
    font-family: "Rubik", sans-serif;
    font-size: 6dvh;
    z-index: 2;
    pointer-events: none;
  }

  .period text {
    transition: fill 3.14s ease-out;
    fill: black;
  }

  circle.sun {
    stroke: none;
    pointer-events: none;
  }

  .period circle.btn {
    animation: greyOut 3.14s ease-out forwards;
    stroke: grey;
  }

  @keyframes greyOut {
    0% { 
      stroke-width: 2;
      stroke: var(--moon-glow-stroke);
    }
    100% { 
      stroke-width: 1;
      stroke: grey;
    }
  }
</style>