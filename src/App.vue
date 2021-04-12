<template>
  <div id="app">
    <!--    <img alt="Vue logo" src="./assets/logo.png" />-->
    <!--    <HelloWorld msg="Welcome to Your Vue.js + TypeScript App" />-->
    <kanban-board
      :columns.sync="columns"
      :items="items"
      column-key="id"
      column-title-key="title"
      items-column-key="status_id"
    ></kanban-board>
    <hr>
    <div>columns</div>
    <pre>{{ columns }}</pre>
    <hr>
    <div>items</div>
    <pre>{{ items }}</pre>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
// import HelloWorld from "./components/HelloWorld.vue";
import KanbanBoard from "@/components/KanbanBoard.vue";

export default Vue.extend({
  name: "App",
  components: {
    // HelloWorld,
    KanbanBoard,
  },
  data() {
    return {
      columns: [
        {
          id: 1,
          title: 'To Do',
        },
        {
          id: 2,
          title: 'In Progress',
        },
        {
          id: 3,
          title: 'Done',
        },
      ],
      items: [
        {
          id: 1,
          status_id: 1,
          text: '1'
        },
        {
          id: 2,
          status_id: 1,
          text: '2'
        },
        {
          id: 3,
          status_id: 1,
          text: '3'
        },
        {
          id: 4,
          status_id: 2,
          text: '4'
        },
        {
          id: 5,
          status_id: 3,
          text: '5'
        },
      ],
    }
  },
});
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  //text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
