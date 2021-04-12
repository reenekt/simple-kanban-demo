<template>
  <div class="kanban-board">
    <Container
      orientation="horizontal"
      @drop="onColumnDrop($event)"
      drag-handle-selector=".kanban-board-column-handle"
      :drop-placeholder="upperDropPlaceholderOptions"
    >
      <Draggable
        v-for="(column, columnIndex) in columns"
        :key="column[columnKey] || `unknown-column-${columnIndex}`"
      >
        <div class="kanban-board-column">
          <div class="kanban-board-column__header">
            <span class="kanban-board-column-handle">&#x2630;</span>
            {{ column[columnTitleKey] || `Unknown column ${columnIndex + 1}` }}
          </div>
          <Container
            group-name="col"
            @drop="onCardDrop(column, $event)"
            :get-child-payload="getCardPayload(column)"
            drag-class="kanban-board-card--ghost"
            drop-class="kanban-board-card--ghost-drop"
            :drop-placeholder="dropPlaceholderOptions"
          >
            <Draggable
              v-for="(card, cardIndex) in getItemsByColumn(column)"
              :key="card.id || `column-${columnIndex}-card-${cardIndex}`"
            >
              <div class="kanban-board-card">
                <p>
                  {{ card.text }} | col: {{ column[columnKey] }} | ID:
                  {{ card[itemsColumnKey] }}
                </p>
              </div>
            </Draggable>
          </Container>
        </div>
      </Draggable>
    </Container>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from "vue";
import { Container, Draggable } from "vue-smooth-dnd";

const applyDrag = (arr, dragResult) => {
  const { removedIndex, addedIndex, payload } = dragResult;
  if (removedIndex === null && addedIndex === null) return arr;

  const result = [...arr];
  let itemToAdd = payload;

  if (removedIndex !== null) {
    itemToAdd = result.splice(removedIndex, 1)[0];
  }

  if (addedIndex !== null) {
    result.splice(addedIndex, 0, itemToAdd);
  }

  return result;
};

export default Vue.extend({
  name: "KanbanBoard",
  components: {
    Container,
    Draggable,
  },
  props: {
    columns: {
      required: true,
      type: Array as PropType<Record<string, unknown>[]>,
    },
    items: {
      required: false,
      type: Array as PropType<Record<string, unknown>[]>,
      default: () => [],
    },
    columnKey: {
      required: false,
      type: String as PropType<string>,
      default: "id",
    },
    columnTitleKey: {
      required: false,
      type: String as PropType<string>,
      default: "title",
    },
    itemsColumnKey: {
      required: false,
      type: String as PropType<string>,
      default: "group_id",
    },
  },
  data() {
    return {
      upperDropPlaceholderOptions: {
        className: "kanban-board-container-preview",
        animationDuration: "150",
        showOnTop: true,
      },
      dropPlaceholderOptions: {
        className: "kanban-board-cards-container-preview",
        animationDuration: "150",
        showOnTop: true,
      },
    };
  },
  methods: {
    // TODO
    onColumnDrop(dropResult) {
      const columns = applyDrag(this.columns, dropResult);
      this.$emit("update:columns", columns);
    },
    onCardDrop(column, dropResult) {
      const { removedIndex, addedIndex, payload: item, element } = dropResult;
      if (addedIndex !== null) {
        item[this.itemsColumnKey] = column[this.columnKey];
      }
    },
    getCardPayload(column: Record<string, unknown>) {
      return (index: number) => {
        return this.getItemsByColumn(column)[index];
      };
    },

    getItemsByColumn(
      column: Record<string, unknown>
    ): Record<string, unknown>[] {
      return this.items.filter((item) => {
        return item[this.itemsColumnKey] === column[this.columnKey];
      });
    },
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.kanban-board-container-preview {
  background: red;
}
.kanban-board-cards-container-preview {
  background: blue;
}

.kanban-board {
  //
}
.kanban-board-column {
  border: 1px solid #474747;
  padding: 5px;

  &__header {
    font-weight: bold;
  }
}
.kanban-board-column-handle {
  //
}
.kanban-board-card {
  margin: 2px;
  padding: 7px;
  border: 1px solid #474747;

  &--ghost {
    border: 1px dashed #b10000;
  }
  &--ghost-drop {
    border: 1px dashed #0041b1;
    min-width: 50px;
    min-height: 50px;
  }
}

.kanban-board-container-preview {
  background: #b10000;
  border: 1px solid #b10000;
}
.kanban-board-cards-container-preview {
  background: #0041b1;
  border: 1px solid #0041b1;
}
</style>
