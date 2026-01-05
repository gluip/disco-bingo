<template>
  <div class="app">
    <header class="app-header">
      <h1 class="app-title">DISCO BINGO</h1>
      <p class="app-subtitle">The Party Edition</p>
    </header>

    <div class="controls">
      <h2>Disco Bingo Generator</h2>
      <p>
        Pas je master lijst aan in het tekstveld hieronder, of gebruik de standaard disco hits.
      </p>

      <div class="input-group" style="margin-top: 20px">
        <button class="btn btn-secondary" @click="generateCard">
          Genereer Enkele Kaart
        </button>
        <button
          v-if="bingoCards.length > 0"
          class="btn btn-success"
          @click="exportToPDF"
        >
          Download PDF ({{ bingoCards.length }} kaart{{ bingoCards.length !== 1 ? 'en' : '' }})
        </button>
      </div>
      
      <div class="input-group" style="margin-top: 15px">
        <input
          v-model.number="numberOfCards"
          type="number"
          min="1"
          max="50"
          placeholder="Aantal kaarten"
          class="number-input"
        />
        <button class="btn btn-primary" @click="generateMultipleCards">
          Genereer {{ numberOfCards || 'X' }} Kaarten
        </button>
      </div>
    </div>

    <!-- Master Songs List -->
    <div class="master-list-section">
      <button class="btn btn-outline" @click="showMasterList = !showMasterList">
        {{ showMasterList ? "Verberg" : "Toon" }} Master Lijst ({{
          allSongs.length
        }}
        liedjes)
      </button>

      <div v-if="showMasterList" class="master-list">
        <div class="master-list-header">
          <h3>Master Lijst - {{ allSongs.length }} liedjes</h3>
          <div class="list-controls">
            <button class="btn btn-sm btn-success" @click="printList">
              📄 Print Lijst
            </button>
          </div>
        </div>

        <div class="songs-list">
          <textarea
            v-model="masterListText"
            @blur="parseMasterList"
            placeholder="Voeg je liedjes toe, één per regel:
Dancing Queen - ABBA
Stayin' Alive - Bee Gees
I Will Survive - Gloria Gaynor"
            rows="15"
            class="master-list-textarea"
          ></textarea>
          <div class="list-info">
            <small>{{ allSongs.length }} liedjes totaal - Bewerk direct in het tekstveld</small>
          </div>
        </div>
      </div>
    </div>

    <div class="bingo-cards" v-if="bingoCards.length > 0">
      <BingoCard
        v-for="(card, index) in bingoCards"
        :key="index"
        :numbers="card"
        :card-id="'card-' + index"
      />
    </div>
  </div>
</template>

<script>
import BingoCard from "./components/BingoCard.vue";
import html2canvas from "html2canvas";
import jsPDF from "jspdf";

export default {
  name: "App",
  components: {
    BingoCard,
  },
  data() {
    return {
      newNumber: "",
      newMasterSong: "",
      masterListText: "",
      numberOfCards: 4,
      availableNumbers: [],
      // Lijst van populaire disco/party nummers
      songList: [
        "Dancing Queen - ABBA",
        "Stayin' Alive - Bee Gees",
        "I Will Survive - Gloria Gaynor",
        "Le Freak - Chic",
        "YMCA - Village People",
        "Super Freak - Rick James",
        "Billie Jean - Michael Jackson",
        "Don't Stop Me Now - Queen",
        "Uptown Funk - Bruno Mars",
        "Can't Stop the Feeling - Justin Timberlake",
        "September - Earth, Wind & Fire",
        "I Want Your Love - Chic",
        "Good Times - Chic",
        "Funky Town - Lipps Inc.",
        "Get Lucky - Daft Punk",
        "Upside Down - Diana Ross",
        "Hot Stuff - Donna Summer",
        "Last Dance - Donna Summer",
        "Bad Girls - Donna Summer",
        "Love Is in the Air - John Paul Young",
        "Celebration - Kool & The Gang",
        "Ladies Night - Kool & The Gang",
        "Get Down Tonight - KC and the Sunshine Band",
        "That's the Way (I Like It) - KC and the Sunshine Band",
        "Play That Funky Music - Wild Cherry",
        "Brick House - Commodores",
        "I Feel Good - James Brown",
        "Sex Machine - James Brown",
        "Car Wash - Rose Royce",
        "Best of My Love - The Emotions",
        "Boogie Wonderland - Earth, Wind & Fire",
        "I'm Every Woman - Chaka Khan",
        "We Are Family - Sister Sledge",
        "He's the Greatest Dancer - Sister Sledge",
        "Ring My Bell - Anita Ward",
        "Shake Your Body - The Jacksons",
        "Blame It on the Boogie - The Jacksons",
        "Rock With You - M ichael Jackson",
        "Off the Wall - Michael Jackson",
        "Another Brick in the Wall - Pink Floyd",
      ],
      bingoCards: [],
      showMasterList: false,
    };
  },
  mounted() {
    this.updateMasterListText();
  },
  computed: {
    allSongs() {
      // Combineer custom liedjes met default liedjes en verwijder duplicaten
      const combined = [...this.availableNumbers, ...this.songList];
      return [...new Set(combined)].sort();
    },
  },
  methods: {
    addToMasterList() {
      const song = this.newMasterSong.trim();
      if (song && song.length > 3) {
        if (!this.availableNumbers.includes(song)) {
          this.availableNumbers.push(song);
          this.newMasterSong = "";
          this.updateMasterListText();
        } else {
          alert("Dit liedje bestaat al!");
        }
      } else {
        alert("Voer een geldig liedje in");
      }
    },
    parseMasterList() {
      const songs = this.masterListText
        .split('\n')
        .map(song => song.trim())
        .filter(song => song.length > 3);
      
      this.availableNumbers = [...new Set(songs)]; // Remove duplicates
    },
    updateMasterListText() {
      this.masterListText = this.allSongs.join('\n');
    },
    removeFromMasterList(song) {
      const index = this.availableNumbers.indexOf(song);
      if (index > -1) {
        this.availableNumbers.splice(index, 1);
        this.updateMasterListText();
      }
    },
    printList() {
      const printWindow = window.open("", "_blank");
      printWindow.document.write(`
        <html>
          <head>
            <title>Disco Bingo - Master Lijst</title>
            <style>
              body { font-family: Arial, sans-serif; padding: 20px; }
              h1 { text-align: center; color: #333; }
              .song-list { column-count: 2; column-gap: 30px; }
              .song-item { 
                padding: 8px 0; 
                border-bottom: 1px solid #eee; 
                break-inside: avoid;
                display: flex;
                align-items: center;
              }
              .checkbox { 
                width: 20px; 
                height: 20px; 
                border: 2px solid #333; 
                margin-right: 10px; 
                flex-shrink: 0;
              }
            </style>
          </head>
          <body>
            <h1>DISCO BINGO - Master Lijst</h1>
            <p>Totaal: ${this.allSongs.length} liedjes</p>
            <div class="song-list">
              ${this.allSongs
                .map(
                  (song) => `
                <div class="song-item">
                  <div class="checkbox"></div>
                  <span>${song}</span>
                </div>
              `
                )
                .join("")}
            </div>
          </body>
        </html>
      `);
      printWindow.document.close();
      printWindow.print();
    },
    addNumber() {
      const song = this.newNumber.trim();
      if (song && song.length > 3) {
        if (!this.availableNumbers.includes(song)) {
          this.availableNumbers.push(song);
          this.newNumber = "";
        } else {
          alert("Dit liedje bestaat al!");
        }
      } else {
        alert("Voer een geldig liedje in (bijv. Dancing Queen - ABBA)");
      }
    },

    removeNumber(index) {
      this.availableNumbers.splice(index, 1);
    },
    shuffleArray(array) {
      const shuffled = [...array];
      for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
      }
      return shuffled;
    },
    generateCard() {
      const cardNumbers = this.generateBingoNumbers();
      this.bingoCards = [cardNumbers];
    },
    generateMultipleCards() {
      const numCards = this.numberOfCards || 4;
      this.bingoCards = [];

      for (let i = 0; i < numCards; i++) {
        const cardNumbers = this.generateBingoNumbers();
        this.bingoCards.push(cardNumbers);
      }
    },
    generateBingoNumbers() {
      const cardNumbers = [];

      // Gebruik custom songs als beschikbaar, anders default songs
      const songsToUse =
        this.availableNumbers.length > 0
          ? [...this.availableNumbers, ...this.songList]
          : [...this.songList];

      // Shuffle songs
      const shuffledSongs = this.shuffleArray(songsToUse);

      // Generate 5x5 grid
      for (let i = 0; i < 25; i++) {
        if (i === 12) {
          // Middle cell is always FREE
          cardNumbers.push("FREE");
        } else {
          cardNumbers.push(shuffledSongs[i % shuffledSongs.length]);
        }
      }

      return cardNumbers;
    },
    async exportToPDF() {
      try {
        const pdf = new jsPDF("p", "mm", "a4");
        const cardsPerPage = 4; // 4 kaarten per pagina in 2x2 grid
        const cardsPerRow = 2;
        const cardWidth = 85;
        const cardSpacing = 10;

        for (let i = 0; i < this.bingoCards.length; i++) {
          const cardElement = document.getElementById(`card-${i}`);

          if (cardElement) {
            // Nieuwe pagina toevoegen als nodig (na elke 4 kaarten)
            if (i > 0 && i % cardsPerPage === 0) {
              pdf.addPage();
            }

            const canvas = await html2canvas(cardElement, {
              scale: 1.5,
              useCORS: true,
              backgroundColor: "#ffffff",
            });

            const imgData = canvas.toDataURL("image/png");
            const imgHeight = (canvas.height * cardWidth) / canvas.width;

            // Bereken positie in 2x2 grid op huidige pagina
            const cardOnPage = i % cardsPerPage;
            const row = Math.floor(cardOnPage / cardsPerRow);
            const col = cardOnPage % cardsPerRow;

            const x = col * (cardWidth + cardSpacing) + 15;
            const y = row * (imgHeight + cardSpacing) + 15;

            pdf.addImage(imgData, "PNG", x, y, cardWidth, imgHeight);
          }
        }

        pdf.save("disco-bingo-kaarten.pdf");
      } catch (error) {
        console.error("Error generating PDF:", error);
        alert(
          "Er ging iets mis bij het genereren van de PDF. Probeer het opnieuw."
        );
      }
    },
  },
};
</script>

<style scoped>
/* Master List Styling */
.master-list-section {
  margin: 30px 0;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 10px;
}

.btn-outline {
  background: transparent;
  border: 2px solid #007bff;
  color: #007bff;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

.btn-outline:hover {
  background: #007bff;
  color: white;
}

.master-list {
  margin-top: 20px;
  animation: slideDown 0.3s ease;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.master-list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  flex-wrap: wrap;
  gap: 10px;
}

.master-list-header h3 {
  margin: 0;
  color: #333;
}

.list-controls {
  display: flex;
  align-items: center;
  gap: 15px;
}

.counter {
  font-weight: bold;
  color: #007bff;
  background: white;
  padding: 5px 10px;
  border-radius: 15px;
  border: 1px solid #007bff;
}

.songs-list {
  max-height: 400px;
  overflow-y: auto;
  padding: 10px;
  border: 1px solid #dee2e6;
  border-radius: 5px;
  background: white;
  margin-bottom: 20px;
}

.song-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  border-bottom: 1px solid #f0f0f0;
  transition: background-color 0.2s ease;
}

.song-item:last-child {
  border-bottom: none;
}

.song-item:hover {
  background: #f8f9fa;
}

.song-item .remove-btn {
  background: #dc3545;
  color: white;
  border: none;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 16px;
  line-height: 1;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.song-item .remove-btn:hover {
  background: #c82333;
}

.add-to-master {
  padding: 15px;
  background: #f8f9fa;
  border-radius: 5px;
  border: 2px dashed #dee2e6;
}

.add-to-master .input-group {
  margin: 0;
}

.master-list-textarea {
  width: 100%;
  min-height: 300px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-family: 'Segoe UI', system-ui, sans-serif;
  font-size: 14px;
  line-height: 1.4;
  resize: vertical;
  background: white;
}

.master-list-textarea:focus {
  border-color: #007bff;
  outline: none;
  box-shadow: 0 0 8px rgba(0, 123, 255, 0.2);
}

.list-info {
  margin-top: 10px;
  text-align: center;
}

.list-info small {
  color: #666;
  font-style: italic;
}

.btn-success {
  background-color: #28a745;
  border-color: #28a745;
  color: white;
}

.btn-success:hover {
  background-color: #218838;
  border-color: #1e7e34;
}

.song-name {
  flex: 1;
  font-size: 14px;
  line-height: 1.3;
}

.btn-sm {
  padding: 5px 10px;
  font-size: 12px;
}

/* Responsive design */
@media (max-width: 768px) {
  .master-list-header {
    flex-direction: column;
    align-items: stretch;
  }

  .list-controls {
    justify-content: space-between;
  }

  .songs-list {
    max-height: 300px;
  }
}

.number-input {
  max-width: 120px !important;
  flex: none !important;
}
</style>
