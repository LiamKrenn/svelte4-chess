<script lang="ts">
  import Chess, { Engine } from "$lib/Chess.svelte";

  let chess: Chess;
</script>

<div style="max-width:512px;margin:0 auto;">
  <Chess
    bind:this={chess}
    orientation="b"
    engine={new Engine({ depth: 20, moveTime: 1500, color: "w" })}
    config={{
      events: {
        move: () => {
          chess.playPremove();
        },
      },
      movable: {
        free: false,
      },
      premovable: {
        enabled: true,
      },
    }}
  />
  <button on:click={() => chess?.reset()}>Reset board</button>
  <button on:click={() => chess?.undo()}>Undo</button>
  <button on:click={() => chess?.playEngineMove()}>Play engine move</button>
</div>

<style>
  button {
    margin-top: 16px;
  }
</style>
