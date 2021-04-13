<template>
  <div id="app">
    <div style="margin-bottom: 10px">
      <label>
        <input v-model="enableGifs" type="checkbox" />
        enable GIFs
      </label>
    </div>

    <kanban-board
      :columns.sync="columns"
      :items="items"
      column-key="id"
      column-title-key="title"
      items-column-key="status_id"
      items-order-key="order"
    >
      <template #columnHeader="{ column, columnTitleKey, columnIndex }">
        <div class="kanban-board-column__header d-flex">
          <strong class="kanban-board-column-handle">&#x2630;</strong>
          <b>{{
            column[columnTitleKey] || `Unknown column ${columnIndex + 1}`
          }}</b>
          <div class="spacer"></div>
          <small>{{ column.someNumber }}</small>
        </div>
      </template>
      <template #card="{ card, column, columnKey, itemsOrderKey }">
        <div class="kanban-board-card">
          <p>
            {{ card.text }} | col: {{ column[columnKey] }} | order:
            {{ card[itemsOrderKey] }}
          </p>

          <div
            v-if="enableGifs"
            style="
              width: 100%;
              height: 0;
              padding-bottom: 81%;
              position: relative;
            "
          >
            <iframe
              src="https://giphy.com/embed/jcxtvm2bsZDH2"
              width="100%"
              height="100%"
              style="position: absolute"
              frameBorder="0"
              class="giphy-embed"
              allowFullScreen
            ></iframe>
          </div>
          <p v-if="enableGifs">
            <a href="https://giphy.com/gifs/hair-template-gaps-jcxtvm2bsZDH2"
              >via GIPHY</a
            >
          </p>
        </div>
      </template>
    </kanban-board>
    <hr />
    <div>columns</div>
    <pre>{{ columns }}</pre>
    <hr />
    <div>items</div>
    <pre>{{ items }}</pre>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import KanbanBoard from "@/components/KanbanBoard.vue";

export default Vue.extend({
  name: "App",
  components: {
    // HelloWorld,
    KanbanBoard,
  },
  data() {
    return {
      enableGifs: false,
      columns: [
        {
          id: 1,
          title: "To Do",
          someNumber: 78,
        },
        {
          id: 2,
          title: "In Progress",
          someNumber: 115,
        },
        {
          id: 3,
          title: "Done",
          someNumber: 39,
        },
      ],
      items: [
        {
          id: 1,
          status_id: 1,
          text: "1",
          order: 1,
        },
        {
          id: 2,
          status_id: 1,
          text: "2",
          order: 3,
        },
        {
          id: 3,
          status_id: 1,
          text: "3",
          order: 2,
        },
        {
          id: 4,
          status_id: 2,
          text: "4",
          order: 1,
        },
        {
          id: 5,
          status_id: 3,
          text: "5",
          order: 1,
        },
      ],
    };
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
  //margin-top: 60px;
}

.d-flex {
  display: flex;
}
.spacer {
  flex: 1 1 auto;
}
</style>
