const SUITS = ["red", "yellow", "green", "blue"];
const VALUES = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "skip", "reverse", "draw2"];
const WILD_TYPES = ["wild", "wild4"];

const PLAYER_COUNT = 4;
const STARTING_HAND = 7;

const state = {
  players: [],
  currentPlayerIndex: 0,
  deck: [],
  discardPile: [],
  winner: null,
  wildColor: null,
  activeCard: null,
  round: 0,
  callUnoUsed: new Set(),
  gameOver: false,
};

const refs = {
  playersArea: document.getElementById("players-area"),
  currentPlayerName: document.getElementById("current-player-name"),
  drawCount: document.getElementById("draw-count"),
  discardTopLabel: document.getElementById("discard-top-label"),
  drawBtn: document.getElementById("draw-card-btn"),
  unoBtn: document.getElementById("uno-btn"),
  newGameBtn: document.getElementById("new-game-btn"),
  drawPile: document.getElementById("draw-pile"),
  discardPile: document.getElementById("discard-pile"),
};

function buildDeck() {
  const deck = [];

  for (const color of SUITS) {
    for (const value of VALUES) {
      deck.push({ color, value, type: "normal", id: `${color}-${value}-${Math.random()}` });
      if (value !== "0") {
        deck.push({ color, value, type: "normal", id: `${color}-${value}-${Math.random()}` });
      }
    }
  }

  for (let i = 0; i < 4; i += 1) {
    deck.push({ color: "wild", value: "wild", type: "wild", id: `wild-${i}-${Math.random()}` });
    deck.push({ color: "wild", value: "wild4", type: "wild4", id: `wild4-${i}-${Math.random()}` });
  }

  return shuffle(deck);
}

function shuffle(array) {
  const copy = [...array];
  for (let i = copy.length - 1; i > 0; i -= 1) {
    const j = Math.floor(Math.random() * (i + 1));
    [copy[i], copy[j]] = [copy[j], copy[i]];
  }
  return copy;
}

function createPlayers(count) {
  return Array.from({ length: count }, (_, index) => ({
    id: index + 1,
    name: `Player ${index + 1}`,
    hand: [],
    isHuman: true,
  }));
}

function drawCard(fromDeck) {
  if (!fromDeck.length) {
    const topCard = state.discardPile[state.discardPile.length - 1];
    const remaining = state.discardPile.slice(0, -1);
    const reshuffled = shuffle(remaining);
    state.deck = reshuffled;
    state.discardPile = [topCard];
  }

  return state.deck.pop();
}

function dealCards() {
  state.players = createPlayers(PLAYER_COUNT);
  state.deck = buildDeck();
  state.discardPile = [];
  state.winner = null;
  state.gameOver = false;
  state.callUnoUsed = new Set();

  for (const player of state.players) {
    player.hand = [];
    for (let i = 0; i < STARTING_HAND; i += 1) {
      player.hand.push(drawCard(state.deck));
    }
  }

  let starter = null;
  while (!starter || starter.color === "wild" || starter.type === "wild") {
    starter = drawCard(state.deck);
  }

  state.discardPile = [starter];
  state.activeCard = starter;
  state.wildColor = starter.color;
  state.currentPlayerIndex = 0;
}

function getPlayableCards(player) {
  const top = state.discardPile[state.discardPile.length - 1];
  return player.hand.filter((card) => canPlayCard(card, top));
}

function canPlayCard(card, top) {
  if (!top) return true;

  if (card.color === "wild") return true;
  if (top.color === "wild") {
    return card.color === state.wildColor || card.color === "wild";
  }

  if (card.color === top.color) return true;
  if (card.value === top.value) return true;
  if (card.type === "wild" || card.type === "wild4") return true;
  return false;
}

function cardLabel(card) {
  if (!card) return "";
  if (card.value === "skip") return "Skip";
  if (card.value === "reverse") return "Rev";
  if (card.value === "draw2") return "+2";
  if (card.value === "wild") return "Wild";
  if (card.value === "wild4") return "+4";
  return card.value;
}

function cardAssetPath(card) {
  if (!card) return "";
  const fileName = `${card.color}-${card.value}.svg`;
  return `assets/cards/${fileName}`;
}

function cardShouldUseCustomAsset(card) {
  if (!card) return false;
  if (card.color === "wild") return true;
  return false;
}

function renderCard(card, options = {}) {
  const wrapper = document.createElement("button");
  wrapper.type = "button";
  wrapper.className = "card";

  if (card) {
    if (card.color === "wild") wrapper.classList.add("wild");
    else wrapper.classList.add(card.color);
  }

  if (options.playable) wrapper.classList.add("playable");
  if (options.selected) wrapper.classList.add("selected");
  if (options.back) wrapper.classList.add("card-back");

  const customImagePath = cardAssetPath(card);
  const useCustomImage = !options.back && typeof card !== "undefined" && card && cardAssetExists(customImagePath);

  if (useCustomImage) {
    wrapper.dataset.art = "custom";
    wrapper.style.backgroundImage = `url('${customImagePath}')`;
  } else {
    wrapper.dataset.art = "fallback";
    const inner = document.createElement("div");
    inner.className = "card-surface";

    const top = document.createElement("div");
    top.className = "card-corner top-left";
    top.textContent = cardLabel(card);

    const bottom = document.createElement("div");
    bottom.className = "card-corner bottom-right";
    bottom.textContent = cardLabel(card);

    const center = document.createElement("div");
    center.className = "card-value";
    center.textContent = cardLabel(card);

    inner.append(top, center, bottom);
    wrapper.appendChild(inner);
  }

  if (options.onClick) wrapper.addEventListener("click", options.onClick);
  return wrapper;
}

function cardAssetExists(path) {
  const img = new Image();
  img.src = path;
  return img.complete || path.includes("wild") || path.includes("draw2");
}

function renderPlayers() {
  refs.playersArea.innerHTML = "";

  state.players.forEach((player, index) => {
    const panel = document.createElement("div");
    panel.className = "player-panel";

    const header = document.createElement("div");
    header.className = "player-header";

    const name = document.createElement("span");
    name.className = "player-name";
    name.textContent = player.name;

    const tag = document.createElement("span");
    tag.className = "player-indicator";
    tag.textContent = index === state.currentPlayerIndex && !state.gameOver ? "Turn" : `${player.hand.length} cards`;

    header.append(name, tag);
    panel.appendChild(header);

    const handRow = document.createElement("div");
    handRow.className = "hand-row";

    player.hand.forEach((card, cardIndex) => {
      const cardNode = renderCard(card, {
        playable: index === state.currentPlayerIndex && canPlayCard(card, state.discardPile[state.discardPile.length - 1]),
        selected: false,
        onClick: () => handleCardClick(index, cardIndex),
      });
      handRow.appendChild(cardNode);
    });

    panel.appendChild(handRow);
    refs.playersArea.appendChild(panel);
  });

  refs.currentPlayerName.textContent = state.players[state.currentPlayerIndex]?.name ?? "-";
  refs.drawCount.textContent = `${state.deck.length} cards`;
  if (state.discardPile.length) {
    const top = state.discardPile[state.discardPile.length - 1];
    refs.discardTopLabel.textContent = `${top.color} / ${cardLabel(top)}`;
  } else {
    refs.discardTopLabel.textContent = "-";
  }

  refs.drawPile.innerHTML = "";
  refs.drawPile.appendChild(renderCard(null, { back: true }));

  refs.discardPile.innerHTML = "";
  const topCard = state.discardPile[state.discardPile.length - 1];
  if (topCard) {
    refs.discardPile.appendChild(renderCard(topCard));
  }
}

function handleCardClick(playerIndex, cardIndex) {
  if (state.gameOver) return;
  if (playerIndex !== state.currentPlayerIndex) return;

  const player = state.players[playerIndex];
  const card = player.hand[cardIndex];
  const top = state.discardPile[state.discardPile.length - 1];

  if (!canPlayCard(card, top)) {
    return;
  }

  player.hand.splice(cardIndex, 1);
  state.discardPile.push(card);
  state.activeCard = card;
  state.wildColor = card.color === "wild" ? "red" : card.color;

  if (card.value === "wild") {
    state.wildColor = chooseWildColor();
  }

  if (!player.hand.length) {
    state.winner = player.name;
    state.gameOver = true;
    alert(`${player.name} wins!`);
    renderPlayers();
    return;
  }

  resolveAfterPlay(card, player);
}

function chooseWildColor() {
  const options = ["red", "yellow", "green", "blue"];
  const chosen = window.prompt("Choose the next color: red, yellow, green, blue", "red");
  const normalized = chosen ? chosen.toLowerCase() : "red";
  return options.includes(normalized) ? normalized : "red";
}

function resolveAfterPlay(card, player) {
  if (card.value === "skip") {
    advanceTurn(2);
    return;
  }

  if (card.value === "reverse") {
    advanceTurn(2);
    return;
  }

  if (card.value === "draw2") {
    const targetIndex = nextPlayerIndex(state.currentPlayerIndex);
    const target = state.players[targetIndex];
    target.hand.push(drawCard(state.deck));
    target.hand.push(drawCard(state.deck));
    advanceTurn(2);
    return;
  }

  if (card.value === "wild4") {
    const targetIndex = nextPlayerIndex(state.currentPlayerIndex);
    const target = state.players[targetIndex];
    target.hand.push(drawCard(state.deck));
    target.hand.push(drawCard(state.deck));
    target.hand.push(drawCard(state.deck));
    target.hand.push(drawCard(state.deck));
    advanceTurn(2);
    return;
  }

  advanceTurn(1);
}

function nextPlayerIndex(index) {
  return (index + 1) % state.players.length;
}

function advanceTurn(step = 1) {
  if (state.gameOver) return;
  state.currentPlayerIndex = (state.currentPlayerIndex + step) % state.players.length;
  renderPlayers();
}

function handleDrawCard() {
  if (state.gameOver) return;
  const player = state.players[state.currentPlayerIndex];
  const top = state.discardPile[state.discardPile.length - 1];
  const drawn = drawCard(state.deck);
  player.hand.push(drawn);

  if (canPlayCard(drawn, top)) {
    alert(`${player.name} drew a playable card and may play it.`);
  }

  advanceTurn(1);
}

function handleUnoCall() {
  if (state.gameOver) return;
  const player = state.players[state.currentPlayerIndex];
  if (player.hand.length === 1) {
    state.callUnoUsed.add(player.name);
    alert(`${player.name} calls UNO!`);
  } else {
    alert(`${player.name} has ${player.hand.length} cards; UNO must be called only at 1 card.`);
  }
}

function initializeGame() {
  dealCards();
  renderPlayers();
}

refs.drawBtn.addEventListener("click", handleDrawCard);
refs.unoBtn.addEventListener("click", handleUnoCall);
refs.newGameBtn.addEventListener("click", initializeGame);

initializeGame();
