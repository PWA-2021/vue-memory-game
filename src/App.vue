<template>
  <div id="app">
    <div class="gameOver" v-if="gameOver">
      <h2>Igra završena</h2>
      <p>Trebalo ti je {{ clicks }} klikova!</p>
      <button @click="resetGame">Kreni ponovo</button>
    </div>
    <p class="stats">Klikovi: {{ clicks }}</p>
    <ul class="cards" :class="{ 'is-clickable': clickable }">
      <Card
        v-for="(card, index) in shuffled"
        :key="index"
        :card="card"
        @click.native="cardClick(card, index)"
        :class="[
          {
            'is-revealed':
              firstClickIndex === index || secondClickIndex === index
          },
          { 'is-found': found.includes(card) }
        ]"
      />
    </ul>
  </div>
</template>

<script>
import Card from './components/Card.vue'

export default {
  name: 'App',
  components: {
    Card
  },
  watch: {
    found() {
      if (this.found.length === this.cards.length) {
        this.gameOver = true
      }
    }
  },
  data() {
    return {
      cards: ['🐶', '🐱', '🐵', '🐼', '🐻', '🦊'],
      clicks: 0,
      firstClickIndex: null,
      secondClickIndex: null,
      timeout: null,
      clickable: true,
      found: [],
      gameOver: true
    }
  },
  computed: {
    pairs() {
      return [...this.cards, ...this.cards]
    },
    shuffled() {
      return this.pairs
        .map((a) => [Math.random(), a])
        .sort((a, b) => a[0] - b[0])
        .map((a) => a[1])
    }
  },
  methods: {
    cardClick(card, index) {
      // šaljemo vrijednost kliknute karte (tipa 🐶) card i njen index odnosno redoslijed u arrayu, primjerice 0 ili 5 (gdje je 0 prvi element u nizu)

      // povećavamo klikove zbog statistika da znamo koliko smo dobri
      this.clicks++

      // dohvaćamo vrijednost prve karte, npr. 🐶
      const firstCard = this.shuffled[this.firstClickIndex]

      if (this.firstClickIndex === null) {
        // ako nema prvog klika onda ga postavljamo
        this.firstClickIndex = index
      } else {
        // ako prvi klik postoji moramo postaviti drugi
        this.secondClickIndex = index

        if (firstCard === card) {
          // ako se vrijednost prve kartice koju smo gore definirali podudara s vrijednosti kartice na koju smo upravo sad kliknuli onda poguraj vrijednosti u found i resetiraj prvi i drugi klik
          this.found.push(firstCard)
          this.firstClickIndex = null
          this.secondClickIndex = null
        } else {
          // ako se ne podudaraju kartice onda ne možemo klikati po ničemu dok ne resetiramo klikove
          this.clickable = false
          setTimeout(() => {
            // nakon što korisniku pokažemo njegove rezultate oslobodimo da može klikati ponovo i resetiramo vrijednost prvog i drugog klika, odnosno počinjemo ponovo klikati da nađemo prvi i potom drugi par
            this.clickable = true
            this.firstClickIndex = null
            this.secondClickIndex = null
          }, 1000)
        }
      }
    },
    resetGame() {
      this.cards = [...this.cards] // triggeramo da se shuffle computed prop ponovo okine
      this.clicks = 0
      this.found = []
      this.gameOver = false
    }
  }
}
</script>

<style>
#app {
  font-family: 'Arial';
}

.cards {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-gap: 20px;
  padding: 40px 0;
  max-width: 800px;
  margin: 0 auto;
  pointer-events: none;
}

.cards.is-clickable {
  pointer-events: all;
}

.stats {
  position: absolute;
  top: 10px;
  right: 20px;
}

.gameOver {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 200;
  box-shadow: 0 0 30px rgba(0, 0, 0, 0.3);
  padding: 50px;
  background: white;
  text-align: center;
}

.gameOver h2 {
  margin: 0 0 10px;
}

.gameOver button {
  all: unset;
  margin-top: 20px;
  padding: 10px 20px;
  background: black;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}

.gameOver button:hover {
  background: #333;
}
</style>
