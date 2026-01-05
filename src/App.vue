<template>
  <div class="app">
    <header class="app-header">
      <h1 class="app-title">DISCO BINGO</h1>
      <p class="app-subtitle">The Party Edition</p>
    </header>

    <div class="controls">
      <h2>Custom Disco Nummers</h2>
      <p>
        Optioneel: voeg je eigen liedjes toe, anders worden automatisch
        willekeurige disco hits gebruikt.
      </p>

      <div class="input-group">
        <input
          v-model="newNumber"
          type="text"
          placeholder="Voeg een liedje toe (bijv. 'Dancing Queen - ABBA', 'Stayin' Alive - Bee Gees')"
          @keyup.enter="addNumber"
        />
        <button class="btn btn-primary" @click="addNumber">Toevoegen</button>
      </div>

      <div class="numbers-list" v-if="availableNumbers.length > 0">
        <div
          v-for="(number, index) in availableNumbers"
          :key="index"
          class="number-tag"
        >
          {{ number }}
          <button class="remove-btn" @click="removeNumber(index)">×</button>
        </div>
      </div>

      <div class="input-group" style="margin-top: 20px">
        <button class="btn btn-secondary" @click="generateCard">
          Genereer Nieuwe Kaart
        </button>
        <button
          class="btn btn-success"
          @click="exportToPDF"
          :disabled="bingoCards.length === 0"
        >
          Download PDF
        </button>
        <button class="btn btn-primary" @click="generateMultipleCards">
          Genereer 4 Kaarten
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
          <div v-for="(song, index) in allSongs" :key="index" class="song-item">
            <span class="song-name">{{ song }}</span>
            <button
              v-if="availableNumbers.includes(song)"
              class="remove-btn"
              @click="removeFromMasterList(song)"
              title="Verwijder uit lijst"
            >
              ×
            </button>
          </div>
        </div>

        <div class="add-to-master">
          <div class="input-group">
            <input
              v-model="newMasterSong"
              type="text"
              placeholder="Voeg liedje toe aan master lijst..."
              @keyup.enter="addToMasterList"
            />
            <button class="btn btn-primary" @click="addToMasterList">
              Toevoegen
            </button>
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
        "Rock With You - Michael Jackson",
        "Off the Wall - Michael Jackson",
        "Another Brick in the Wall - Pink Floyd",
      ],
      bingoCards: [],
      showMasterList: false,
    };
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
        } else {
          alert("Dit liedje bestaat al!");
        }
      } else {
        alert("Voer een geldig liedje in");
      }
    },
    removeFromMasterList(song) {
      const index = this.availableNumbers.indexOf(song);
      if (index > -1) {
        this.availableNumbers.splice(index, 1);
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
      this.bingoCards = [];

      for (let i = 0; i < 4; i++) {
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
        const cardsPerPage = 2;

        for (let i = 0; i < this.bingoCards.length; i++) {
          const cardElement = document.getElementById(`card-${i}`);

          if (cardElement) {
            const canvas = await html2canvas(cardElement, {
              scale: 2,
              useCORS: true,
              backgroundColor: "#ffffff",
            });

            const imgData = canvas.toDataURL("image/png");
            const imgWidth = 180;
            const imgHeight = (canvas.height * imgWidth) / canvas.width;

            // Calculate position
            const pageIndex = Math.floor(i / cardsPerPage);
            const cardIndex = i % cardsPerPage;

            // Add new page if needed
            if (i > 0 && cardIndex === 0) {
              pdf.addPage();
            }

            const x = 15;
            const y = cardIndex * (imgHeight + 20) + 15;

            pdf.addImage(imgData, "PNG", x, y, imgWidth, imgHeight);
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
</style>
