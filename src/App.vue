<template>
  <div id="app">
    <h1>{{ msg }}</h1>
    <h2>DB is <span v-if="this.dbOpen">open</span><span v-else>closed</span></h2>
    <h3 v-show="objectStoresReady">Object stores are ready</h3>
    <button v-on:click="addCustomer()">Add Customer</button>
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
      db: null
    }
  },

  mounted: function (params) {
    let request = window.indexedDB.open("MyTestDatabase", 1);
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
      let objectStore = db.createObjectStore("customers", { keyPath: "ssn" });
      objectStore.createIndex('name', 'name', { unique: false });
      objectStore.createIndex('email', 'email', { unique: true });
      this.objectStoresReady = true;
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
