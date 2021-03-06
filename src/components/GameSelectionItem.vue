<template>
  <div class="game-card" v-if="bothCountriesAvailable()">
    <div class="header" :class="{ selected: selected }">
      <h5 class="countries">{{ game.country_1 }} vs {{ game.country_2 }}</h5>
    </div>
    <div class="game-card-body">
      <div class="game-info">
        <div>
          <p>Game {{ game.game }}</p>
          <p class="datetime">{{ game.datetime | moment('timezone', 'Australia/Perth', "dddd, Do MMM, h:mma") }}</p>
          <p class="stadium">{{ game.stadium }}</p>
        </div>
        <div v-if="game.group">
          <p class="group">Group {{ game.group }}</p>
        </div>
      </div>
      <div class="game-picker">
        <p v-if="selected" class="selected-item" :class="winnerPicked()">Selected {{ selected }}</p>
        <p v-else class="selected-none">None</p>
        <div class="btn-group" v-if="winnerPicked() === '' && canSelect && !isLocked()">
          <button type="button" class="btn btn-primary" @click="selectCountry(game.country_1)">{{ game.country_1 }}</button>
          <button type="button" class="btn btn-primary draw" @click="selectCountry('Draw')" v-if="!finals">Draw</button>
          <button type="button" class="btn btn-primary" @click="selectCountry(game.country_2)">{{ game.country_2 }}</button>
        </div>
        <div class="game-played" v-else>
          <span>Game Played</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import firebase from 'firebase';
  import { db } from '../firebase';
  import moment from 'moment';

  export default {
    name: 'GameSelectionItem',
    props: ['locked', 'game', 'email', 'winners', 'finals'],
    data () {
      return {
        hidden: true,
        selected: null,
        selection: null,
        canSelect: this.timeLeft(),
      }
    },
    methods: {
      addSelection: function() {
        if (moment() < moment(this.game.datetime)) {
          if (this.winnerPicked() === '') {
            let obj = {};
            obj[this.game.game] = this.selected;
            const email = this.email.replace(/\./g, '*');
            db.ref(`selections/${email}`).update(obj);
          }
        }
      },
      selectCountry: function(choice) {
        if (moment() < moment(this.game.datetime)) {
          this.selected = choice;
          this.addSelection();
        }
      },
      winnerPicked: function() {
        let winLose = '';
        if (this.winners.length) {
          this.winners.forEach(item => {
            if (item['.key'] === this.game.game) {
              return winLose = (this.selected === item['.value']) ? 'win' : 'lose';
            }
          });
        }
        return winLose;
      },
      isLocked: function() {
        let lock = false;
        this.locked.forEach(item => {
          if (item['.key'] === this.game.game) {
            return lock = (item['.value'] === true);
          }
        });
        return lock;
      },
      timeLeft: function() {
        let time = false;
        let obj = {};
        if (moment() < moment(this.game.datetime)) {
          return time = true;
        } else {
          obj[`${this.game.game}`] = true;
          db.ref('locked').update(obj);
          return time = false;
        }
        return time;
      },
      bothCountriesAvailable: function() {
        return (this.game.country_1.length <= 3 || this.game.country_2.length <= 3) ? false : true;
      },
    },
    beforeMount: function() {
      const email = this.email.replace(/\./g, '*');
      this.$bindAsArray('selection', db.ref(`selections/${email}/`));
    },
    watch: {
      selection: function(val) {
        if (this.selection.length) {
          val.forEach(element => {
            if (element['.key'] == (this.game.game)) {
              return this.selected = element['.value'];
            }
          });
          this.hidden = false;
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  .game-card {
    background-color: whitesmoke;
    margin-top: 15px;
    width: 100%;

    .header {
      color: white;
      background: rgb(66, 65, 65);
      padding: 10px;

      .countries {
        font-size: 1em;
        margin: 0;
        text-align: center;
        font-weight: bold;
      }
    }

  }

  .game-picker {
    width: 100%;
    
    .btn-group {
      width: 100%;

      .btn {
        border-radius: 0;
        flex-grow: 1;
        padding: 10px;
      }
    }

    .selected-item {
      background: darken(#2c7caa, 5%);
      color: white;
      text-align: center;
      padding: 10px;
      width: 100%;
      margin: 0;

      &.win {
        background: rgb(71, 185, 71);
      }
      &.lose {
        background: rgb(196, 76, 76);
      }
    }

    .selected-none {
      background: #8b8d8f;
      color: white;
      text-align: center;
      padding: 10px;
      width: 100%;
      margin: 0;
    }
  }

  .game-played {
    background: darken(#3287b8, 5%);
    color: white;
    text-align: center;
    padding: 10px;
    width: 100%;
    margin: 0;
  }

  .game-info {
    display: flex;
    justify-content: space-between;
    padding: 10px;

    p {      
      margin-top: 0;
      margin-bottom: 5px;
    }
    .datetime {
      font-size: 0.75em;
      color: rgb(112, 112, 112);
      font-style: italic;
    }
    .group {
      font-size: 0.95em;
      text-transform: uppercase;
      font-weight: 700;
    }
    .stadium,
    .location {
      color: rgb(80, 79, 79);
      font-size: 0.82em;
      text-transform: capitalize;
    }
  }

  @media screen and (min-width: 768px) {
    .game-card {
    background-color: whitesmoke;
    margin-top: 15px;
    width: 48%;
    }
  }

  @media screen and (min-width: 1200px) {
    .game-card {
    background-color: whitesmoke;
    margin-top: 15px;
    width: 30%;
    }
  }
</style>

