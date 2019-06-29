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


  function generatePlayer(name, id, scores) {
    const newPlayer = {
      name: name,
      id: id,
      scores: scores,
      total: function() {
        if (this.scores.length > 0) {
          return this.scores.reduce(reducer);
        }
        return 0;
      },
      average: function() {
        if (this.scores.length > 0) {
          const av  =  this.scores.reduce(reducer) / this.scores.length;
          return Math.round(av);
        } else {
          return 0;
        }
      },
      addScore: function(newScore) {
        this.scores = [...this.scores, newScore];
      }
    };
    return newPlayer;
  }

  function addPlayer() {
    if (playerName.value) {

      const newPlayer = generatePlayer( playerName.value, players.length, [] );
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

        saveScores();


      }
    }
  }

  function endGame() {
    players = [];
    gameStarted = false;
        localStorage.removeItem('scrabble_scores' );
  }


  function saveScores() {
    localStorage.setItem('scrabble_scores', JSON.stringify(players)  );
  }



  function loadScores() {
    const saved_players_string = localStorage.getItem('scrabble_scores');

    if (saved_players_string) {

      const saved_players = JSON.parse(saved_players_string);

      if (saved_players.length > 0) {
        let loaded_players = [];
        saved_players.forEach( (np) => {
          console.log(np.scores);
          const new_player = generatePlayer( np.name, np.id, np.scores );
          loaded_players.push(new_player);
        });

        players =  loaded_players;
        gameStarted = true;
      }

    }
  }

  //  schedules a callback to run as soon as the component has been mounted to the DOM.
  onMount(() => {

    loadScores();
  });

  //  when component is destroy, unsubscribe from any subscriptions to prevent memory leaks
  onDestroy(() => {
  });
</script>

<style>

</style>




<header>

  <h1>{appName}</h1>
  <nav>
    {#if gameStarted}
  <a on:click|preventDefault={endGame} href="#end_game">End Game</a>
  {/if}
  </nav>




</header>



<div class="container">



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

                  <div class="score hidden">
                  Average: {player.average()}
                  </div>

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
