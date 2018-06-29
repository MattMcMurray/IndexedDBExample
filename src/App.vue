<template>
  <div id="app">
    <h1>{{ msg }}</h1>
    <h2>DB is <span v-if="this.dbOpen">open</span><span v-else>closed</span></h2>
    <h3 v-show="objectStoresReady">Object stores are ready</h3>
    <input type="text" name="goal" id="goalInput" v-model="goal.goalName">
    <button v-on:click="addGoal()">Add Customer</button>
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
      }
    }
  },

  mounted: function (params) {
    let request = window.indexedDB.open("MyTestDatabase", 2);
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

      request.onsuccess = function (event) {
        console.log('Added successfully');
      }


      request.error = function (event) {
        console.group();
        console.error('Error adding to IndexedDB');
        console.error(event);
        console.groupEnd();
      }

    },

    addCustomer: function () {
      let customerObjectStore = this.db.transaction("customers", "readwrite").objectStore("customers");
      let request = customerObjectStore.add({ssn: 12345, name: 'Matt', email: 'test@example.com'});

      request.onsuccess = function (event) {
        console.log('Added to db successfully');
      }

      request.onerror = function (event) {
        console.group();
        console.log('Error adding to object store');
        console.log(event);
        console.groupEnd();
      }

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

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
