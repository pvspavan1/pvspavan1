- 👋 Hi, I’m @pvspavan1
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
pvspavan1/pvspavan1 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
var config = {
baseBet: { value: 100, type: "balance", label: "base bet" },
  payout: { value: 2, type: "multiplier" },
  stop: { value: 1e8, type: "balance", label: "stop if bet >" },
  loss: {
    value: "increase",
    type: "radio",
    label: "On Loss",
    options: {
      base: { type: "noop", label: "Return to base bet" },
      increase: { value: 2, type: "multiplier", label: "Increase bet by" },
    },
  },
  win: {
    value: "base",
    type: "radio",
    label: "On Win",
    options: {
      base: { type: "noop", label: "Return to base bet" },
      increase: { value: 2, type: "multiplier", label: "Increase bet by" },
    },
  },
};

log("Script is running..");

var currentBet = config.baseBet.value;

engine.on("GAME_STARTING", onGameStarted);
if (engine.gameState === "GAME_STARTING" || engine.gameState === "GAME_ENDED") {
  engine.bet(roundBit(currentBet), config.payout.value);
  engine.on("GAME_ENDED", onGameEnded);
} else {
  engine.once("GAME_STARTING", () => engine.on("GAME_ENDED", onGameEnded));
}

function onGameStarted() {
  engine.bet(roundBit(currentBet), config.payout.value);
}

function onGameEnded() {
  var lastGame = engine.history.first();

  // If we wagered, it means we played
  if (!lastGame.wager) {
    return;
  }

  // we won..
  if (lastGame.cashedAt) {
    if (config.win.value === "base") {
      currentBet = config.baseBet.value;
    } else {
      console.assert(config.win.value === "increase");
      currentBet *= config.win.options.increase.value;
    }
    log("We won, so next bet will be", currentBet / 100, "bits");
  } else {
    // damn, looks like we lost :(
    if (config.loss.value === "base") {
      currentBet = config.baseBet.value;
    } else {
      console.assert(config.loss.value === "increase");
      currentBet *= config.loss.options.increase.value;
    }
    log("We lost, so next bet will be", currentBet / 100, "bits");
  }

  if (currentBet > config.stop.value) {
    log("Was about to bet", currentBet / 100, "which triggers the stop");
    engine.removeListener("GAME_STARTING", onGameStarted);
    engine.removeListener("GAME_ENDED", onGameEnded);
  }
}

function roundBit(bet) {
  return Math.round(bet / 100) * 100;
}
