<script setup>
import { ref, onMounted, reactive } from "vue";
import PieSocket from "piesocket-js";

const wins = ref(0);
const draws = ref(0);
const losses = ref(0);
const outcomes = {
  rock: {
    rock: "draw",
    paper: "loss",
    scissors: "win"
  },
  paper: {
    rock: "win",
    paper: "draw",
    scissors: "loss"
  },
  scissors: {
    rock: "loss",
    paper: "win",
    scissors: "draw"
  }
};
let verdict = ref(null);
let player1MadeChoice = ref(false);
let player2MadeChoice = ref(false);

let MakeMove;

var channel;
const pieSocket = new PieSocket({
  clusterId: "free.blr2",
  apiKey: "B9UKgvptNTWrZxfCUTquFp7nKVsYqu2LtmBao5Jg"
});

let isFirstPlayer = null;
let player1Choice = ref(null);
let player2Choice = ref(null);

pieSocket.subscribe("game-room").then(ch => {
  channel = ch;
  console.log("Channel is ready");

  MakeMove = move => {
    if (isFirstPlayer === null) {
      isFirstPlayer = true;
      player1Choice.value = move;
      player1MadeChoice.value = true;
      channel.publish(
        "move",
        JSON.stringify({ move: move, player: "player1" })
      );
    } else {
      player2Choice.value = move;
      player2MadeChoice.value = true;
      channel.publish(
        "move",
        JSON.stringify({ move: move, player: "player2" })
      );
    }
  };
  channel.listen("move", move => {
    if (move) {
      const data = JSON.parse(move);
      if (data.move) {
        if (isFirstPlayer === null) {
          isFirstPlayer = data.player === "player1";
        }
        if (data.player === "player1") {
          player1Choice.value = data.move;
          player1MadeChoice.value = true;
        } else {
          player2Choice.value = data.move;
          player2MadeChoice.value = true;
        }

        if (player1Choice.value && player2Choice.value) {
          const outcome = outcomes[player1Choice.value][player2Choice.value];
          console.log(
            `Player 1 chose ${player1Choice.value}, Player 2 chose ${player2Choice.value}`
          );
          if (outcome === "win") {
            console.log("Player 1 Wins");
            wins.value++;
            verdict.value =
              "Player 1 chose " +
              player1Choice.value +
              ", Player 2 chose " +
              player2Choice.value +
              ". Player 1 wins!";
          } else if (outcome === "loss") {
            console.log("Player 2 Wins");
            losses.value++;
            verdict.value =
              "Player 1 chose " +
              player1Choice.value +
              ", Player 2 chose " +
              player2Choice.value +
              ". Player 2 wins!";
          } else {
            console.log("Draw");
            draws.value++;
            verdict.value =
              "Player 1 chose " +
              player1Choice.value +
              ", Player 2 chose " +
              player2Choice.value +
              ". It's a draw!";
          }

          // Broadcast the verdict to both players
          channel.publish(
            "outcome",
            JSON.stringify({ verdict: verdict.value })
          );

          // Update the verdict for the current player
          verdict.value = verdict.value;

          // ...
          if (outcome === "win") {
            console.log("Player 1 Wins");
            wins.value++;
            verdict.value =
              "Player 1 chose " +
              player1Choice.value +
              ", Player 2 chose " +
              player2Choice.value +
              ". Player 1 wins!";
          } else if (outcome === "loss") {
            console.log("Player 2 Wins");
            losses.value++;
            verdict.value =
              "Player 1 chose " +
              player1Choice.value +
              ", Player 2 chose " +
              player2Choice.value +
              ". Player 2 wins!";
          } else {
            console.log("Draw");
            draws.value++;
            verdict.value =
              "Player 1 chose " +
              player1Choice.value +
              ", Player 2 chose " +
              player2Choice.value +
              ". It's a draw!";
          }
          channel.publish(
            "outcome",
            JSON.stringify({ verdict: verdict.value })
          );

          verdict.value = verdict.value;

          SaveGame();
          ResetRound();
        }
      }
    } else {
      console.error("Received undefined payload");
    }
  });

  channel.listen("outcome", outcome => {
    if (outcome) {
      const data = JSON.parse(outcome);
      if (data.verdict) {
        verdict.value = data.verdict; // Update the verdict ref
      }
      console.log("Verdict received: ", verdict.value);
    } else {
      console.error("Received undefined payload");
    }
  });
});

const SaveGame = () => {
  localStorage.setItem("wins", wins.value);
  localStorage.setItem("draws", draws.value);
  localStorage.setItem("losses", losses.value);
};

const LoadGame = () => {
  wins.value = localStorage.getItem("wins");
  draws.value = localStorage.getItem("draws");
  losses.value = localStorage.getItem("losses");
};

const ResetRound = () => {
  player1Choice.value = null;
  player2Choice.value = null;
};

onMounted(() => {
  window.addEventListener("keypress", e => {
    if (e.key === "r") {
      ResetRound();
    }
  });
});
</script>
<template>
  <div class="bg-gray-700 text-white text-center min-h-screen flex flex-col">
    <header class="container mx-auto p-6">
      <h1 class="text-4xl font-bold">Rock, Paper, Scissors!</h1>
    </header>
    <main class="container mx-auto p-6 flex-1">
      <div class="flex items-center justify-center -mx-6">
        <button
          @click="MakeMove('rock', 'player1')"
          class="bg-white rounded-full shadow-lg w-64 p-12 mx-6 transition-colors duration-300 hover:bg-pink-500"
        >
          <img src="./assets/RockIcon.svg" alt="Rock" class="w-full" />
        </button>

        <button
          @click="MakeMove('paper')"
          class="bg-white rounded-full shadow-lg w-64 p-12 mx-6 transition-colors duration-300 hover:bg-green-500"
        >
          <img src="./assets/PaperIcon.svg" alt="Paper" />
        </button>
        <button
          @click="MakeMove('scissors')"
          class="bg-white rounded-full shadow-lg w-64 p-12 mx-6 transition-colors duration-300 hover:bg-yellow-500"
        >
          <img src="./assets/ScissorsIcon.svg" alt="Scissors" />
        </button>
      </div>

      <div class="mt-4 text-xl">
        <p>{{ verdict }}</p>
      </div>
    </main>
  </div>
</template>
