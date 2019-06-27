<script>
  import { onDestroy, onMount } from "svelte";
  import Scorechart from "./Scorechart.svelte";
  const appName = "Scrabble App";
  const reducer = (accumulator, currentValue) => accumulator + currentValue;

  let gameStarted = false;

  let players = []; // {name, id, scores, total(), addScore() }
  let playerName;
  let currentIndex = 0;

  $: playerSummary = players.map(p => p.name).join(" & ");

  function startGame() {
    if (players.length > 0) {
      gameStarted = true;
      updateCurrentPlayer();
    }
  }

  function addPlayer() {
    if (playerName.value) {
      let newPlayer = {
        name: playerName.value,
        id: players.length,
        scores: [],
        total: function() {
          if (this.scores.length > 0) {
            return this.scores.reduce(reducer);
          }
          return 0;
        },
        addScore: function(newScore) {
          this.scores = [...this.scores, newScore];
        }
      };
      players = [...players, newPlayer];
      playerName.value = null;
    }
  }

  function updateCurrentPlayer() {
    let cur_index = 0;
    let cur_score_length = 9999;
    players.forEach((p, index) => {
      if (p.scores.length < cur_score_length) {
        cur_score_length = p.scores.length;
        cur_index = index;
      }
    });
    currentIndex = cur_index;
  }

  function recordScore(e, player) {
    if (e.target.value && e.key === "Enter") {
      const newScore = parseInt(e.target.value, 10);
      if (newScore >= 0 || newScore <= 0) {
        player.addScore(newScore);
        players = players;
        e.target.value = null;
        updateCurrentPlayer();
      }
    }
  }

  //  schedules a callback to run as soon as the component has been mounted to the DOM.
  onMount(() => {});

  //  when component is destroy, unsubscribe from any subscriptions to prevent memory leaks
  onDestroy(() => {});
</script>

<style>

</style>

<div class="container">

  <h1>{appName}</h1>

  {#if gameStarted}
    <div class="columns">

      <div class="column">

        <table>

          <thead>
            <tr>
              {#each players as player}
                <th class:current={currentIndex === player.id}>
                   {player.name} ({player.total()})
                </th>
              {/each}
            </tr>
          </thead>

          <tbody>

            <tr>
              {#each players as player}
                <td>

                  <div class="score">
                    <input on:keyup={e => recordScore(e, player)} type="text" />
                  </div>

                  {#each player.scores as score}
                    <div class="score"> {score} </div>
                  {/each}

                </td>
              {/each}
            </tr>

          </tbody>
        </table>

      </div>

      <div class="column">
        <Scorechart {players} />
      </div>
    </div>
  {:else}
    <div class="center">
      <form on:submit|preventDefault={addPlayer}>
        <div class="form_field">
          <input placeholder="Player name" type="text" bind:this={playerName} />
          <button>Add Player</button>
        </div>
      </form>
      {#each players as player}
        <p>{player.name}</p>
      {/each}

      <p>
        <a class="button" href="#start" on:click|preventDefault={startGame}>
          Start Game
        </a>
      </p>

    </div>
  {/if}

</div>
