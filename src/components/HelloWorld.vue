<template>
  <div class="main">
    <router-link
      class="buttonTop"
      :to="{
        name: 'Form',
        params: {
          edit: 'no'
        }
      }"
    >
      + Tambah
    </router-link>
    <div class="grid">
      <div class="grid-item_left record">
        <select v-model="perPage">
          <option value="2">2</option>
          <option value="5">5</option>
          <option value="10">10</option>
          <option value="25">25</option>
        </select>
        <span>records per page</span>
      </div>
      <div class="route grid-item_right">
        <span>Search:</span>
        <input type="text" v-model="searchQuery" />
      </div>
    </div>
    <table class="tableMain">
      <thead>
        <tr>
          <th>ID</th>
          <th>NIK</th>
          <th>Nama</th>
          <th>Tempat Lahir</th>
          <th>Tanggal Lahir</th>
          <th>Gender</th>
          <th>Alamat</th>
          <th>Aksi</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="psn in person" :key="psn.id">
          <td>{{ psn.id }}</td>
          <td>{{ psn.nik }}</td>
          <td>{{ psn.nama }}</td>
          <td>{{ psn.tmptl }}</td>
          <td>{{ psn.tgll }}</td>
          <td>{{ psn.gender }}</td>
          <td>{{ psn.alamat }}</td>
          <td class="buttonClass">
            <router-link
              :to="{ name: 'Form', params: { userId: psn.id, edit: 'yes' } }"
              >Edit</router-link
            >
            <button v-on:click="deletePerson(psn.id)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>
    <div class="grid">
      <div>
        <span
          >Showing {{ response.from }} to {{ response.to }} of
          {{ response.total }} entries</span
        >
      </div>
      <div class="route grid-item_right">
        <span v-on:click="prevPage()">pref</span>
        <button v-if="last === true" v-on:click="page = 1">1</button>
        <button v-if="last === false" v-on:click="page = currentState">
          {{ currentState }}
        </button>
        <span v-if="last === true">...</span>
        <button v-on:click="page = nextState">{{ nextState }}</button>
        <button v-on:click="page = doubleState">{{ doubleState }}</button>
        <span v-if="last === false">...</span>
        <button v-on:click="page = lastState">{{ lastState }}</button>
        <span v-on:click="nextPage()">next</span>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "HelloWorld",
  data() {
    return {
      person: [],
      response: [],
      page: 1,
      searchQuery: "",
      perPage: 5,
      currentState: 0,
      nextState: 0,
      doubleState: 0,
      lastState: 0,
      last: false
    };
  },
  mounted() {
    let self = this;
    self.updateData();
    // console.log(self.$route.params)
    self.openTab(self.$route.params.tab);
  },
  methods: {
    deletePerson(ids) {
      let self = this;
      let link = "http://127.0.0.1:8000/api/person/{id}";
      let linknew = link.replace("{id}", ids);
      axios.delete(linknew).then(response => {
        console.log(response);
        self.updateData();
        self.updateState();
      });
    },
    updateData() {
      let self = this;
      axios
        .get("http://127.0.0.1:8000/api/person", {
          params: {
            page: self.page,
            find: self.searchQuery,
            perPage: self.perPage
          }
        })
        .then(response => {
          // console.log(response.data);
          self.person = response.data.data;
          self.response = response.data;
          self.updateState();
        })
        .catch(err => {
          console.log(err);
        });
    },
    nextPage() {
      let self = this;
      let page = self.page;
      if (page + 1 <= self.response.last_page) {
        self.page += 1;
      }
    },
    prevPage() {
      let self = this;
      let page = self.page;
      if (page - 1 >= 1) {
        self.page -= 1;
      }
    },
    updateState() {
      let self = this;
      self.currentState = self.response.current_page;
      self.lastState = self.response.last_page;
      let nextState = self.currentState + 1;
      if (nextState <= self.lastState - 2) {
        if (nextState + 1 >= self.lastState) {
          self.nextState = self.lastState - 2;
        } else {
          self.nextState = nextState;
        }
      } else {
        self.nextState = self.lastState - 2;
      }
      let doubleState = self.currentState + 2;
      if (doubleState <= self.lastState - 1) {
        if (doubleState >= self.lastState) {
          self.doubleState = self.lastState - 1;
        } else {
          self.doubleState = doubleState;
        }
      } else {
        self.doubleState = self.lastState - 1;
      }
      if (self.currentState + 2 >= self.lastState) {
        self.last = true;
      } else {
        self.last = false;
      }
    },
    openTab(id) {
      let self = this;
      self.$store.dispatch("openTabAct", id);
    }
  },
  watch: {
    page: function() {
      let self = this;
      self.updateData();
      // self.updateState();
    },
    searchQuery: function() {
      let self = this;
      self.updateData();
      self.updateState();
      self.page = 1;
    },
    perPage: function() {
      let self = this;
      self.updateData();
      self.updateState();
      self.page = 1;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
td,
th {
  border: 1px black solid;
  padding: 1em;
}
table {
  border-collapse: collapse;
  width: 100%;
  table-layout: fixed;
  .buttonClass {
    display: grid;
    a {
      display: inline-block;
      padding: 5px 20px;
      margin: 5px;
      border: 1px solid black;
      border-radius: 15px;
    }
  }
}

.buttonTop {
  display: inline-block;
  padding: 15px 30px;
  margin: 0px 0 10px 0;
  border: 1px solid black;
  border-radius: 15px;
}

a {
  text-decoration: none;
}
.route {
  width: 300px;
  margin: 20px 50px;
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
  span {
    margin: 0 auto;
    cursor: pointer;
  }
}
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
}
.grid-item {
  &_left {
    justify-self: start;
  }
  &_right {
    justify-self: end;
  }
}
.record {
  margin: 20px;
}
span {
  margin: 5px;
  line-height: 1.4em;
}
</style>
