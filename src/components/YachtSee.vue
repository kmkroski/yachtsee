<template>
  <div>
    <nav class="navbar fixed-top navbar-dark bg-secondary">
      <a class="navbar-brand" href="/">YachtSee</a>
      <span class="ml-auto mr-0 text-light">
        Score&nbsp;
        <strong>{{ scores.total }}</strong>
      </span>
    </nav>

    <div class="d-flex flex-row justify-content-center text-center mb-2">
      <div
        v-for="index in 5" :key="index"
        :class="{'die': true, 'active': selected.indexOf(index - 1) != -1 }"
        @click="selectDie(index - 1)"
      >
        <span v-if="!dice_rolling[index - 1]">{{ dice[index - 1] || '' }}</span>
        <span v-if="dice_rolling[index - 1]" class="spinner">&nbsp;</span>
      </div>
    </div>

    <hr>

    <div class="d-flex flex-row justify-content-center text-center mb-3">
      <button
        type="button"
        class="btn btn-success col-8"
        @click="rollDice"
        :disabled="roll >= 3 || rolling"
      >
        <span v-if="roll === 0">First Roll</span>
        <span v-if="roll === 1">Second Roll</span>
        <span v-if="roll === 2">Last Roll</span>
        <span v-if="roll === 3">Choose a Score</span>
      </button>
    </div>

    <hr>

    <div class="row">
      <div class="col">
        <button
          type="button"
          v-for="(title, type) in score_types.left" :key="type"
          :class="{'btn btn-secondary btn-block btn-score': 1, 'btn-info': scores[type] !== null}"
          @click="addScore(type)"
          :disabled="scores[type] !== null || roll === 0 || rolling"
        >
          {{ title }}
          <span class="badge badge-light">
            {{ scores[type] === null ? calcScore(type) : scores[type] }}
          </span>
        </button>
      </div>

      <div class="col">
        <button
          type="button"
          :class="{'btn btn-secondary btn-block btn-score': 1, 'btn-info': scores[type] !== null}"
          v-for="(title, type) in score_types.right" :key="type"
          @click="addScore(type)"
          :disabled="scores[type] !== null || roll === 0 || rolling"
        >
          {{ title }}
          <span class="badge badge-light">
            {{ scores[type] === null ? calcScore(type) : scores[type] }}
          </span>
        </button>
      </div>
    </div>

    <b-modal ref="fiveOfAKindModal1" title="YachtSee" centered>
      <p class="my-1 text-center">
        You rolled a five of a kind!
      </p>

      <p class="my-1 text-center">
        <strong>+50 points!</strong>
      </p>

      <div slot="modal-footer" class="w-100 text-center">
        <button type="button" class="btn btn-primary" @click="closeFiveModals()">
          OK
        </button>
      </div>
    </b-modal>

    <b-modal ref="fiveOfAKindModal2" title="YachtSee" centered>
      <p class="my-1 text-center">
        You rolled <em>another</em> five of a kind!
      </p>

      <p class="my-1 text-center">
        <strong>+100 points!</strong>
      </p>

      <div slot="modal-footer" class="w-100 text-center">
        <button type="button" class="btn btn-primary" @click="closeFiveModals()">
          OK
        </button>
      </div>
    </b-modal>

    <b-modal ref="completeModal" title="YachtSee" centered>
      <p class="my-1 text-center">
        <strong>Game Over!</strong>
      </p>

      <p class="my-1 text-center">
        You scored {{ scores.total }} points.
      </p>

      <div slot="modal-footer" class="w-100 text-center">
        <button class="btn btn-primary" @click="resetGame()">
          Play Again
        </button>

        <button class="btn btn-secondary" @click="showShareModal()">
          Share Score
        </button>
      </div>
    </b-modal>

    <b-modal ref="shareModal" title="YachtSee" centered hide-footer>
      <social-sharing url="https://kro.ski/yachtsee"
        :title="`I scored ${scores.total} points on #YachtSee! Can you do better?`"
        :quote="`I scored ${scores.total} points on #YachtSee! Can you do better?`"
        description="YachtSee is an online dice game at https://kro.ski/yachtsee"
        network-tag="button"
        inline-template
      >
        <div class="social-networks">
          <network network="facebook" class="btn btn-block btn-primary">
            <i class="fa fa-facebook"></i> Facebook
          </network>

          <network network="twitter" class="btn btn-block btn-primary">
            <i class="fa fa-twitter"></i> Twitter
          </network>

          <network network="googleplus" class="btn btn-block btn-primary">
            <i class="fa fa-google-plus"></i> Google +
          </network>

          <network network="sms" class="btn btn-block btn-primary">
            <i class="fa fa-commenting-o"></i> SMS
          </network>

          <network network="email" class="btn btn-block btn-primary">
            <i class="fa fa-envelope"></i> Email
          </network>
        </div>
      </social-sharing>

      <hr>

      <button class="btn btn-block btn-secondary" @click="resetGame()">
        Play Again
      </button>

      <div slot="modal-footer"></div>
    </b-modal>
  </div>
</template>

<script>
export default {
  name: 'YachtSee',
  data() {
    return {
      dice: [],
      selected: [],
      roll: 0,
      rolling: false,
      dice_rolling: [],
      scores: {},
      score_types: {
        left: {
          ones: 'Ones',
          twos: 'Twos',
          threes: 'Threes',
          fours: 'Fours',
          fives: 'Fives',
          sixes: 'Sixes',
        },
        right: {
          three_of_a_kind: 'Three of a Kind',
          four_of_a_kind: 'Four of a Kind',
          full_house: 'Full House',
          small_straight: 'Small Straight',
          large_straight: 'Large Straight',
          five_of_a_kind: 'YachtSee',
          chance: 'Chance',
        },
      },
    };
  },
  created() {
    this.resetGame();
  },
  methods: {
    resetGame() {
      if (this.$refs.completeModal) {
        this.$refs.completeModal.hide();
      }

      if (this.$refs.shareModal) {
        this.$refs.shareModal.hide();
      }

      this.rolling = false;
      this.dice = ['?', '?', '?', '?', '?'];
      this.dice_rolling = [0, 0, 0, 0, 0];

      this.scores = { total: 0 };
      const scoresTypes = Object.keys(this.score_types.left)
        .concat(Object.keys(this.score_types.right));
      for (let i = 0; i < scoresTypes.length; i += 1) {
        this.scores[scoresTypes[i]] = null;
      }

      this.startRound();
    },
    startRound() {
      this.roll = 0;
      this.selected = [];

      const scoresTypes = Object.keys(this.scores);
      for (let i = 0; i < scoresTypes.length; i += 1) {
        if (this.scores[scoresTypes[i]] === null) {
          return;
        }
      }

      this.$refs.completeModal.show();
    },
    closeFiveModals() {
      if (this.$refs.fiveOfAKindModal1) {
        this.$refs.fiveOfAKindModal1.hide();
      }

      if (this.$refs.fiveOfAKindModal2) {
        this.$refs.fiveOfAKindModal2.hide();
      }
    },
    showShareModal() {
      if (this.$refs.completeModal) {
        this.$refs.completeModal.hide();
      }

      this.$refs.shareModal.show();
    },
    selectDie(index) {
      if (this.roll === 0) {
        return;
      }

      if (this.selected.indexOf(index) !== -1) {
        this.selected.splice(this.selected.indexOf(index), 1);
      } else {
        this.selected.push(index);
      }
    },
    rollDice() {
      this.roll += 1;

      for (let index = 0; index < 5; index += 1) {
        if (this.selected.indexOf(index) === -1) {
          this.rolling = true;
          this.dice_rolling[index] = 1;
          this.dice[index] = Math.floor(Math.random() * 6) + 1;
        }
      }

      if (this.rolling === true) {
        setTimeout(() => {
          this.dice_rolling = [0, 0, 0, 0, 0];
          this.rolling = false;
          this.checkFiveOfAKind();
        }, 300);
      } else {
        this.checkFiveOfAKind();
      }
    },
    checkFiveOfAKind() {
      if (
        this.dice[0] === this.dice[1] &&
        this.dice[0] === this.dice[2] &&
        this.dice[0] === this.dice[3] &&
        this.dice[0] === this.dice[4]
      ) {
        if (this.scores.five_of_a_kind === null) {
          this.$refs.fiveOfAKindModal1.show();
          this.scores.five_of_a_kind = 50;
          this.scores.total += 50;
          this.startRound();
        } else if (this.scores.five_of_a_kind === 50) {
          this.$refs.fiveOfAKindModal2.show();
          this.scores.five_of_a_kind = 150;
          this.scores.total += 100;
          this.startRound();
        }
      }
    },
    sumDiceByValue(value) {
      let score = 0;
      for (let index = 0; index < 5; index += 1) {
        if (this.dice[index] === value) {
          score += value;
        }
      }
      return score;
    },
    sumXOfAKind(value) {
      const count = [];
      for (let index = 0; index < 5; index += 1) {
        if (!count[this.dice[index]]) {
          count[this.dice[index]] = 0;
        }

        count[this.dice[index]] += 1;
      }

      if (count.indexOf(value) !== -1) {
        return value * count.indexOf(value);
      }

      if (count.indexOf(4) !== -1) {
        return value * count.indexOf(4);
      }

      if (count.indexOf(5) !== -1) {
        return value * count.indexOf(5);
      }

      return 0;
    },
    sumFullHouse() {
      const count = [];
      for (let index = 0; index < 5; index += 1) {
        if (!count[this.dice[index]]) {
          count[this.dice[index]] = 0;
        }

        count[this.dice[index]] += 1;
      }

      if (count.indexOf(2) !== -1 && count.indexOf(3) !== -1) {
        return 25;
      }

      return 0;
    },
    sumSmallStraight() {
      if (
        // 1-2-3-4
        (
          this.dice.indexOf(1) !== -1 && this.dice.indexOf(2) !== -1 &&
          this.dice.indexOf(3) !== -1 && this.dice.indexOf(4) !== -1
        ) ||
        // 2-3-4-5
        (
          this.dice.indexOf(2) !== -1 && this.dice.indexOf(3) !== -1 &&
          this.dice.indexOf(4) !== -1 && this.dice.indexOf(5) !== -1
        ) ||
        // 3-4-5-6
        (
          this.dice.indexOf(3) !== -1 && this.dice.indexOf(4) !== -1 &&
          this.dice.indexOf(5) !== -1 && this.dice.indexOf(6) !== -1
        )
      ) {
        return 30;
      }

      return 0;
    },
    sumLargeStraight() {
      if (
        // 1-2-3-4-5
        (
          this.dice.indexOf(1) !== -1 && this.dice.indexOf(2) !== -1 &&
          this.dice.indexOf(3) !== -1 && this.dice.indexOf(4) !== -1 &&
          this.dice.indexOf(5) !== -1
        ) ||
        // 2-3-4-5-6
        (
          this.dice.indexOf(2) !== -1 && this.dice.indexOf(3) !== -1 &&
          this.dice.indexOf(4) !== -1 && this.dice.indexOf(5) !== -1 &&
          this.dice.indexOf(6) !== -1
        )
      ) {
        return 40;
      }

      return 0;
    },
    sumChance() {
      let count = 0;
      for (let index = 0; index < 5; index += 1) {
        count += isNaN(this.dice[index]) ? 0 : this.dice[index];
      }
      return count;
    },
    calcScore(type) {
      switch (type) {
        case 'ones':
          return this.sumDiceByValue(1);
        case 'twos':
          return this.sumDiceByValue(2);
        case 'threes':
          return this.sumDiceByValue(3);
        case 'fours':
          return this.sumDiceByValue(4);
        case 'fives':
          return this.sumDiceByValue(5);
        case 'sixes':
          return this.sumDiceByValue(6);
        case 'three_of_a_kind':
          return this.sumXOfAKind(3);
        case 'four_of_a_kind':
          return this.sumXOfAKind(4);
        case 'full_house':
          return this.sumFullHouse();
        case 'small_straight':
          return this.sumSmallStraight();
        case 'large_straight':
          return this.sumLargeStraight();
        case 'chance':
          return this.sumChance();
        default:
          return 0;
      }
    },
    addScore(type) {
      const score = this.calcScore(type);
      this.scores.total += score;
      this.scores[type] = score;

      this.startRound();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.navbar-brand {
  font-weight: 600;
  font-style: italic;
}

.die {
  position: relative;
  display: inline-block;
  margin: 0 5px;
  width: 60px;
  height: 60px;
  border: 3px solid #6c757d;
  border-radius: 8px;
  font-size: 38px;
  line-height: 52px;
  font-weight: 200;
  cursor: pointer;
  user-select: none;
}
.die.active {
  border-color: #28a745;
  background-color: #28a745;
  color: #ffffff;
}

.spinner:before {
  content: '';
  box-sizing: border-box;
  position: absolute;
  top: 50%;
  left: 50%;
  width: 40px;
  height: 40px;
  margin-top: -20px;
  margin-left: -20px;
  border-radius: 50%;
  border: 3px solid #f8f9fa;
  border-top-color: #6c757d;
  animation: spinner .3s linear infinite;
}
@keyframes spinner {
  to {transform: rotate(360deg);}
}

.btn-score { text-align: left; }
.btn-score .badge {
  float: right;
  margin-top: 4px;
  line-height: 0.94;
}

.modal-dialog header > h5 {
  width: 100%;
  font-weight: 800;
  font-style: italic;
  text-align: center;
}
.modal-dialog header > button { display: none; }

.social-networks button.btn { text-align: left; }
.social-networks button.btn > i.fa {
  float: left;
  margin-top: 3px;
  margin-left: -0.7rem;
  margin-right: 1rem;
  width: 3rem;
  border-right: 1px solid #f8f9fa;
  text-align: center;
}
</style>
