<template>
  <div class="bingo-card" :id="cardId">
    <div class="card-header">
      <h3 class="card-title">DISCO</h3>
      <h4 class="card-title">BINGO</h4>
      <p class="card-subtitle">The Party Edition</p>
    </div>

    <div class="bingo-grid-container">
      <div class="bingo-headers">
        <div class="bingo-header">M</div>
        <div class="bingo-header">U</div>
        <div class="bingo-header">S</div>
        <div class="bingo-header">I</div>
        <div class="bingo-header">C</div>
      </div>

      <div class="bingo-grid">
        <div
          v-for="(number, index) in numbers"
          :key="index"
          :class="['bingo-cell', { free: number === 'FREE' }]"
          @click="toggleCell(index)"
          :data-marked="markedCells.includes(index)"
        >
          {{ number }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "BingoCard",
  props: {
    numbers: {
      type: Array,
      required: true,
    },
    cardId: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      markedCells: [],
    };
  },
  methods: {
    toggleCell(index) {
      const cellIndex = this.markedCells.indexOf(index);
      if (cellIndex > -1) {
        this.markedCells.splice(cellIndex, 1);
      } else {
        this.markedCells.push(index);
      }
    },
  },
};
</script>

<style scoped>
.bingo-card {
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
  border-radius: 15px;
  padding: 20px;
  margin: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  max-width: 500px;
  width: 100%;
  font-family: "Arial", sans-serif;
}

.card-header {
  text-align: center;
  margin-bottom: 20px;
  color: white;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.card-title {
  margin: 0;
  font-weight: bold;
  font-size: 2.5rem;
  letter-spacing: 3px;
}

.card-subtitle {
  margin: 5px 0 0 0;
  font-size: 1rem;
  font-weight: normal;
  opacity: 0.9;
}

.bingo-grid-container {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  padding: 10px;
}

.bingo-headers {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 2px;
  margin-bottom: 5px;
}

.bingo-header {
  background: rgba(255, 255, 255, 0.9);
  color: #333;
  font-weight: bold;
  font-size: 1.2rem;
  text-align: center;
  padding: 8px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.bingo-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 2px;
}

.bingo-cell {
  min-height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: white;
  color: #333;
  font-weight: bold;
  font-size: 0.7rem;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  padding: 5px;
  text-align: center;
  line-height: 1.2;
  overflow: hidden;
  word-wrap: break-word;
  hyphens: auto;
}

.bingo-cell:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.bingo-cell.free {
  background: linear-gradient(135deg, #ffeaa7 0%, #fdcb6e 100%);
  color: #2d3436;
  font-size: 0.8rem;
  font-weight: bold;
}

.bingo-cell[data-marked="true"] {
  background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%) !important;
  color: white !important;
  transform: scale(0.95);
}

.bingo-cell[data-marked="true"]::after {
  content: "✓";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 1.5rem;
  font-weight: bold;
  color: white;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}
</style>
