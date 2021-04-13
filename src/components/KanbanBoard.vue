<template>
  <div class="kanban-board">
    <Container
      orientation="horizontal"
      @drop="onColumnDrop($event)"
      drag-handle-selector=".kanban-board-column-handle"
      :drop-placeholder="upperDropPlaceholderOptions"
    >
      <Draggable
        v-for="({ column, items }, columnIndex) in board"
        :key="column[columnKey] || `unknown-column-${columnIndex}`"
      >
        <div class="kanban-board-column">
          <slot
            name="columnHeader"
            :column="column"
            :columnTitleKey="columnTitleKey"
            :columnIndex="columnIndex"
          >
            <div class="kanban-board-column__header">
              <span class="kanban-board-column-handle">&#x2630;</span>
              {{
                column[columnTitleKey] || `Unknown column ${columnIndex + 1}`
              }}
            </div>
          </slot>
          <Container
            group-name="col"
            @drop="onCardDrop(column, $event)"
            :get-child-payload="getCardPayload(column)"
            drag-class="kanban-board-card--ghost"
            drop-class="kanban-board-card--ghost-drop"
            :drop-placeholder="dropPlaceholderOptions"
          >
            <Draggable
              v-for="(card, cardIndex) in items"
              :key="card.id || `column-${columnIndex}-card-${cardIndex}`"
            >
              <slot
                name="card"
                :card="card"
                :column="column"
                :columnKey="columnKey"
                :itemsOrderKey="itemsOrderKey"
              >
                <div class="kanban-board-card">
                  <p>
                    {{ card.text }} | col: {{ column[columnKey] }} | order:
                    {{ card[itemsOrderKey] }}
                  </p>
                </div>
              </slot>
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

const applyDrag = (arr: any, dragResult: any) => {
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

export type KanbanBoardColumnDataStructure = {
  column: Record<string, unknown>;
  items: Record<string, unknown>[];
};

export type KanbanBoardDataStructure = KanbanBoardColumnDataStructure[];

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
    itemsOrderKey: {
      required: false,
      type: String as PropType<string>,
      default: "order",
    },
  },
  data() {
    return {
      board: [] as KanbanBoardDataStructure,
      upperDropPlaceholderOptions: {
        className: "kanban-board-column-preview",
        animationDuration: "150",
        showOnTop: true,
      },
      dropPlaceholderOptions: {
        className: "kanban-board-card-preview",
        animationDuration: "150",
        showOnTop: true,
      },
      cardMoveEventData: {
        payload: null as Record<string, unknown> | null,
        from: null as KanbanBoardColumnDataStructure | null,
        to: null as KanbanBoardColumnDataStructure | null,
        removedIndex: null as number | null,
        addedIndex: null as number | null,
      },
    };
  },
  created() {
    // this.prepareItemsOrders();
    this.prepareBoardData();
  },
  watch: {
    cardMoveEventData: {
      handler(val, oldVal) {
        if (val.from !== null && val.to !== null) {
          this.dispatchCardMoveEvent();
        }
      },
      deep: true,
    },
  },
  methods: {
    // TODO
    onColumnDrop(dropResult: any): void {
      // const columns = applyDrag(this.columns, dropResult);
      // this.$emit("update:columns", columns);

      this.board = applyDrag(this.board, dropResult);
    },

    onCardDrop(column: any, dropResult: any): void {
      if (dropResult.removedIndex !== null || dropResult.addedIndex !== null) {
        const board = [...this.board];
        const columnDataIndex = board.findIndex((value) => {
          return value.column[this.columnKey] === column[this.columnKey];
        });
        if (columnDataIndex === -1) {
          return;
        }
        const columnData = board[columnDataIndex];

        const newColumnData = Object.assign({}, columnData);
        newColumnData.items = applyDrag(newColumnData.items, dropResult);
        this.reorderItemsInList(newColumnData.items);
        board.splice(columnDataIndex, 1, newColumnData);
        this.board = board;

        // For (single) event dispatching
        this.cardMoveEventData.payload = dropResult.payload;
        if (dropResult.removedIndex !== null) {
          this.cardMoveEventData.from = newColumnData;
          this.cardMoveEventData.removedIndex = dropResult.removedIndex;
        }
        if (dropResult.addedIndex !== null) {
          this.cardMoveEventData.to = newColumnData;
          this.cardMoveEventData.addedIndex = dropResult.addedIndex;
        }
      }
    },

    getCardPayload(
      column: Record<string, unknown>
    ): (index: number) => Record<string, unknown> | undefined {
      return (index: number) => {
        // return this.getItemsByColumn(column)[index];
        const needleBoardDataItem = this.board.find((value) => {
          // return value.column[this.columnKey] === column[this.columnKey];
          return value.column === column; // TODO test / use line above if test fails
        });
        if (needleBoardDataItem === undefined) {
          return undefined;
        }
        return needleBoardDataItem.items[index];
      };
    },

    getItemsByColumn(
      column: Record<string, unknown>
    ): Record<string, unknown>[] {
      return this.items
        .filter((item) => {
          return item[this.itemsColumnKey] === column[this.columnKey];
        })
        .sort((a, b) => {
          if (+a[this.itemsOrderKey] < +b[this.itemsOrderKey]) {
            return -1;
          }
          if (+a[this.itemsOrderKey] > +b[this.itemsOrderKey]) {
            return 1;
          }
          return 0;
        });
    },

    // TODO
    // reorderColumns() {
    //   this.board.forEach((columnData, columnDataIndex) => {
    //     columnData.column[this.columnsOrderKey] = columnDataIndex + 1;
    //   });
    // },

    reorderItemsInList(itemsList: Record<string, unknown>[]) {
      itemsList.forEach((item, itemIndex) => {
        item[this.itemsOrderKey] = itemIndex + 1;
      });
    },

    prepareBoardData(): void {
      const boardData: KanbanBoardDataStructure = [];
      this.columns.forEach((column) => {
        const itemsByColumn = this.getItemsByColumn(column);
        this.reorderItemsInList(itemsByColumn);
        const columnData: KanbanBoardColumnDataStructure = {
          column,
          items: itemsByColumn,
        };
        boardData.push(columnData);
      });
      this.board = boardData;
    },

    dispatchCardMoveEvent() {
      this.$emit("card:moved", this.cardMoveEventData);
      this.cardMoveEventData.from = null;
      this.cardMoveEventData.to = null;
      this.cardMoveEventData.payload = null;
      this.cardMoveEventData.removedIndex = null;
      this.cardMoveEventData.addedIndex = null;
    },
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
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

.kanban-board-column-preview {
  background: rgba(245, 184, 184, 0.15);
  border: 1px dashed rgba(245, 184, 184, 0.7);
}
.kanban-board-card-preview {
  background: rgba(184, 201, 245, 0.15);
  border: 1px dashed rgb(184, 201, 245);
}
</style>
