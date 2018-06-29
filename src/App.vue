<template>
  <div id="app">
    <h1>{{ msg }}</h1>
    <h2>DB is <span v-if="this.dbOpen">open</span><span v-else>closed</span></h2>
    <h3 v-show="objectStoresReady">Object stores are ready</h3>
    <form @submit.prevent="addGoal()">
      <input type="text" name="goal" id="goalInput" v-model="goal.goalName">
      <button type="submit">Add Goal</button>
    </form>
    <hr>
    <ul>
      <li v-for="goal in thisWeeksGoalsSorted">
        {{ goal.goalName }} - {{ goal.goalDate }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'app',

  data () {
    return {
      msg: 'IndexedDB Test',
      dbOpen: false,
      objectStoresReady: false,
      db: null,
      goal: {
        goalName: "",
        goalDate: new Date()
      },
      thisWeeksGoals: []
    }
  },

  computed: {
    thisWeeksGoalsSorted: function () {
      /**
       * Sort goals by date in descending order.
       *
       * Adapted from this answer: https://stackoverflow.com/a/10124053
       */
      return this.thisWeeksGoals.sort(function(a,b){
        // Turn your strings into dates, and then subtract them
        // to get a value that is either negative, positive, or zero.
        return new Date(b.goalDate) - new Date(a.goalDate);
      });
    }
  },

  mounted: function (params) {
    let request = window.indexedDB.open("GoalKeeperDatabase", 1);
    request.onerror = this.handleDBOpenError;
    request.onsuccess = this.handleDBOpenSuccess;
    request.onupgradeneeded = this.handleUpgradeNeededEvent;
  },

  methods: {

    handleDBOpenError: function (event) {
      console.group();
      console.error('There was an error opening IndexedDB');
      console.error(event);
      console.groupEnd();
      alert("IndexedDB Error. Check console for details.");
    },

    handleDBOpenSuccess: function (event) {
      this.db = event.target.result;
      this.dbOpen = true;
      this.fetchAllGoals();
    },

    handleUpgradeNeededEvent: function (event) {
      let db = event.target.result;
      let objectStore = db.createObjectStore("goals", { autoIncrement: true });
      objectStore.createIndex('goalName', 'goalName', { unique: false });
      objectStore.createIndex('goalDate', 'goalDate', { unique: false });
      this.objectStoresReady = true;
    },

    addGoal: function() {
      let objStore = this.db.transaction('goals', 'readwrite').objectStore('goals');
      let request = objStore.add(this.goal);

      request.onsuccess = event => {
        this.thisWeeksGoals.push({ goalName: this.goal.goalName, goalDate: this.goal.goalDate });
        this.goal.goalName = '';
        this.goal.goalDate = new Date();
      }


      request.onerror = function (event) {
        console.group();
        console.error('Error adding to IndexedDB');
        console.error(event);
        console.groupEnd();
      }

    },

    fetchAllGoals: function () {
      let transaction = this.db.transaction(["goals"]);
      let objectStore = transaction.objectStore("goals");
      let request = objectStore.getAll();

      request.onsuccess = event => {
        console.log(request.result);
        this.thisWeeksGoals = this.thisWeeksGoals.concat(request.result);
      };
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

a {
  color: #42b983;
}
</style>
