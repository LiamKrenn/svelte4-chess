<script lang="ts" context="module">
  export type GameOverEvent = CustomEvent<GameOver>;
  export type MoveEvent = CustomEvent<Move>;
  export type UciEvent = CustomEvent<string>;
  export type { Square, Color, PieceSymbol, Move, GameOver };
  export { Engine } from "$lib/engine.js";
</script>

<script lang="ts">
  import { Chessground } from "svelte4-chessground";
  import type { Config } from "chessground/config";
  import PromotionDialog from "$lib/PromotionDialog.svelte";
  import {
    Api,
    type Square,
    type Color,
    type PieceSymbol,
    type Move,
    type GameOver,
  } from "$lib/api.js";
  import type { Engine } from "$lib/engine.js";

  import { onMount, createEventDispatcher } from "svelte";
  import { type Key } from "chessground/types";

  const dispatch = createEventDispatcher<{
    move: Move;
    gameOver: GameOver;
    ready: {};
    uci: string;
    update: {};
  }>();

  let chessground: Chessground;
  let container: HTMLElement;

  /*
   * Props
   */

  // bindable read-only props
  export let moveNumber = 0;
  export let turn: Color = "w";
  export let inCheck = false;
  export let history: string[] = [];
  export let isGameOver = false;

  // Initial values used, also bindable
  export let fen = "rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1";
  export let orientation: Color = "w";

  export let config: Config = {};

  $: chessground?.set(config);

  // non-bindable
  export let engine: Engine | undefined = undefined;
  let className: string | undefined = undefined;
  export { className as class };

  // API: only accessible through props and methods
  let api: Api | undefined = undefined;

  /*
   * Methods -- passed to API
   */

  export function load(newFen: string) {
    if (!api) throw new Error("component not mounted yet");
    api.load(newFen);
  }

  export function moveFromTo(from: Key, to: Key) {
    if (!api) throw new Error("component not mounted yet");
    api.move({
      from: from,
      to: to,
    });
  }

  export function setOwnColor(color: "white" | "black") {
    if (!api) throw new Error("component not mounted yet");
    ownColor = color;
    api.setOwnColor(color);
  }

  export let ownColor: "white" | "black" = "white";

  export function setConfig(config: Config) {
    if (!api) throw new Error("component not mounted yet");
    api.set(config);
  }

  export function moveLan(moveLan: string) {
    if (!api) throw new Error("component not mounted yet");
    api.moveLan(moveLan);
  }

  export function move(moveSan: string) {
    if (!api) throw new Error("component not mounted yet");
    console.log("ooppoonennt");

    api.move(moveSan);
  }
  export function getHistory(): string[];
  export function getHistory({ verbose }: { verbose: true }): Move[];
  export function getHistory({ verbose }: { verbose: false }): string[];
  export function getHistory({
    verbose,
  }: {
    verbose: boolean;
  }): string[] | Move[];
  export function getHistory({ verbose = false }: { verbose?: boolean } = {}) {
    if (!api) throw new Error("component not mounted yet");
    return api.history({ verbose });
  }
  export function getBoard() {
    if (!api) throw new Error("component not mounted yet");
    return api.board();
  }
  export function undo(): Move | null {
    if (!api) throw new Error("component not mounted yet");
    return api.undo();
  }
  export function reset(): void {
    if (!api) throw new Error("component not mounted yet");
    api.reset();
  }
  export function toggleOrientation(): void {
    if (!api) throw new Error("component not mounted yet");
    api.toggleOrientation();
  }
  export async function playEngineMove(): Promise<void> {
    if (!api) throw new Error("component not mounted yet");
    return api.playEngineMove();
  }

  export function playPremove() {
    chessground.playPremove();
  }
  /*
   * API Construction
   */

  function stateChangeCallback(api: Api) {
    fen = api.fen();
    orientation = api.orientation();
    moveNumber = api.moveNumber();
    turn = api.turn();
    inCheck = api.inCheck();
    history = api.history();
    isGameOver = api.isGameOver();
    dispatch("update");
  }

  function promotionCallback(square: Square): Promise<PieceSymbol> {
    return new Promise((resolve) => {
      const element = new PromotionDialog({
        target: container,
        props: {
          square,
          orientation,
          callback: (piece: PieceSymbol) => {
            element.$destroy();
            resolve(piece);
          },
        },
      });
    });
  }

  function moveCallback(move: Move) {
    dispatch("move", move);
  }
  function gameOverCallback(gameOver: GameOver) {
    dispatch("gameOver", gameOver);
  }

  onMount(async () => {
    if (engine) {
      engine.setUciCallback((message) => dispatch("uci", message));
    }
    api = new Api(
      chessground,
      fen,
      stateChangeCallback,
      promotionCallback,
      moveCallback,
      gameOverCallback,
      orientation,
      ownColor,
      engine,
    );
    api.init().then(() => {
      // Dispatch ready-event: Simply letting the parent observe when the component is mounted is not enough due to async onMount.
      dispatch("ready");
    });
  });
</script>

<div style="position:relative;" bind:this={container}>
  <Chessground bind:config bind:this={chessground} class={className} />
</div>
