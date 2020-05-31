<template>
  <div v-if="isPairsSet" class="game">
    <div class="game-board">
      <div v-if="isLoading" class="loader">
        <div>
          <img src="@/assets/loading.gif" alt="Loading images" />
        </div>
        <span>Loading game assets...</span>
      </div>

      <div v-else>
        <div class="game-information">
          <span>Turns: {{ turns }}</span>
          <span>Pairs: {{ pairsMatched }} of {{ cardPairs }}</span>
        </div>

        <div class="game-layout">
          <div class="card-layout">
            <Card v-for="card in deck" :key="card.number" :card="card" />
          </div>
        </div>
      </div>
      <div v-if="pairsMatched === cardPairs" class="overlay">
        <div class="victory">
          <span>Congratulations</span>
          <button v-if="pairsMatched === cardPairs" @click="resetGame">Start new game</button>
        </div>
      </div>
    </div>
  </div>

  <div v-else class="overlay">
    <div class="settings">
      <span>Please select difficulty</span>
      <button @click="setGameSettings(6)">Easy</button>
      <button @click="setGameSettings(12)">Medium</button>
      <button @click="setGameSettings(16)">Hard</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Card from "./Card.vue";

export default {
  components: {
    Card
  },
  data: () => {
    return {
      isLoading: true,
      isPairsSet: false,
      deck: [],
      openedCards: [],
      pairsMatched: 0,
      pairVisibleInMilliseconds: 1000,
      turns: 0,
      cardImages: [],
      cardPairs: 0,
      imageApi: "https://picsum.photos/200"
    };
  },
  mounted() {
    this.$on("onCardOpen", function(card) {
      this.openCard(card);
    });
  },
  watch: {
    cardImages: function(val) {
      if (val.length === this.cardPairs) {
        this.generateCards();
        this.isLoading = false;
      }
    }
  },
  methods: {
    setGameSettings: function(value) {
      this.cardPairs = value;
      this.isPairsSet = true;
      this.getCardImages();
    },
    getCardImages: async function() {
      while (this.cardImages.length < this.cardPairs) {
        const response = await axios.get(this.imageApi);

        if (response.request.responseURL) {
          this.cardImages.push(response.request.responseURL);
        }
      }
    },
    generateCards: function() {
      var cards = [];
      var cardNumber = 0;

      this.cardImages.forEach((image, key) => {
        for (var i = 0; i < 2; i++) {
          cardNumber += 1;

          cards.push({
            number: cardNumber,
            pair: key,
            image: image,
            open: false,
            matched: false
          });
        }
      });

      this.deck = this.shuffleDeck(cards);
    },
    shuffleDeck: function(cards) {
      return cards
        .map(a => [Math.random(), a])
        .sort((a, b) => a[0] - b[0])
        .map(a => a[1]);
    },
    openCard: function(card) {
      if (this.openedCards.length === 2) {
        return;
      }

      this.deck.forEach((element, index) => {
        if (element.number === card.number) {
          this.deck[index].open = true;
          return;
        }
      });

      this.openedCards.push(card);

      if (this.openedCards.length === 2) {
        setTimeout(() => {
          this.handlePossibleMatch(this.openedCards);
        }, this.pairVisibleInMilliseconds);
      }
    },
    handlePossibleMatch: function(openedCards) {
      if (this.cardsMatch(openedCards)) {
        const openedCardNumber = [openedCards[0].number, openedCards[1].number];

        this.deck.forEach((element, index) => {
          if (openedCardNumber.includes(element.number)) {
            this.deck[index].open = false;
            this.deck[index].matched = true;
          }
        });

        this.pairsMatched += 1;
      } else {
        this.closeCards();
      }

      this.turns += 1;
      this.openedCards = [];
    },
    resetGame: function() {
      this.generateCards();
      this.turns = 0;
      this.pairsMatched = 0;
    },
    cardsMatch: function(cards) {
      return cards[0].pair === cards[1].pair;
    },
    closeCards: function() {
      this.deck.forEach(card => {
        card.open = false;
      });
    }
  }
};
</script>

<style>
.overlay {
  z-index: 99;
  height: 100vh;
  width: 100%;
  background-color: black;
  opacity: 0.7;
  display: grid;
  position: absolute;
}
.victory,
.settings {
  color: #f7fff7;
  justify-self: center;
  align-self: center;
  display: grid;
}
.victory > button,
.settings > button {
  background-color: #0ac2c2;
  border: none;
  padding: 5px;
  color: #f7fff7;
  margin: 5px;
}
.game-board {
  display: grid;
  height: 100vh;
}
.loader {
  align-self: center;
  color: #0ac2c2;
  display: grid;
  font-size: 17px;
  font-weight: 500;
  justify-self: center;
}
.loader > div {
  justify-self: center;
}
.loader > div > img {
  max-height: 120px;
  margin: 0 auto;
}
.game-layout {
  display: grid;
  user-select: none;
  justify-content: center;
}
.card-layout {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 1rem;
  padding: 1rem;
}
.game-information {
  background: #293132;
  color: #f7fff7;
  display: grid;
  padding: 5px;
  text-align: center;
}
@media (min-width: 375px) {
  .card-layout {
    grid-template-columns: repeat(5, 1fr);
  }
}

@media (min-width: 768px) {
  .card-layout {
    grid-template-columns: repeat(6, 1fr);
  }
}

@media (min-width: 1024px) {
  .card-layout {
    grid-template-columns: repeat(8, 1fr);
  }
}
</style>