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
            @drag-start="onCardDragStart(column, $event)"
            @drag-end="onCardDragEnd(column, $event)"
            @drop-ready="onCardDropReady(column, $event)"
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
              <slot name="card" :card="card" :column="column" :columnKey="columnKey" :itemsOrderKey="itemsOrderKey">
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

function groupBy<T = any>(
  arr: T[],
  groupByKey: string,
  withKeys = false
): Array<T[]> | Record<string, T[]> {
  const groups: Record<string, T[]> = {};
  arr.forEach((item) => {
    groups[item[groupByKey]] = groups[item[groupByKey]] || [];
    groups[item[groupByKey]].push(item);
  });
  if (withKeys) {
    return groups;
  }
  return Object.values(groups);
}

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
      draggingItemCurrentColumnId: null,
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
  created() {
    // this.prepareItemsOrders();
  },
  methods: {
    // TODO
    onColumnDrop(dropResult: any): void {
      const columns = applyDrag(this.columns, dropResult);
      this.$emit("update:columns", columns);
    },
    onCardDropReady(column: any, dropResult: any) {
      const { removedIndex, addedIndex, payload } = dropResult;
      // console.log('onCardDropReady', column[this.columnKey], removedIndex, addedIndex, payload, payload[this.itemsColumnKey]);
    },
    onCardDragStart(column: any, dragResult: any) {
      const { isSource, payload, willAcceptDrop } = dragResult;
      // console.log('onCardDragStart', isSource, payload, willAcceptDrop, payload[this.itemsColumnKey]);
      if (isSource) {
        this.draggingItemCurrentColumnId = payload[this.itemsColumnKey]
      }
    },
    onCardDragEnd(column: any, dragResult: any) {
      const { isSource, payload, willAcceptDrop } = dragResult
      // console.log('onCardDragEnd', isSource, payload, willAcceptDrop, payload[this.itemsColumnKey]);
      // if (isSource) {
      //   this.draggingItemCurrentColumnId = null
      // }
    },
    onCardDrop(column: any, dropResult: any): void {
      const { removedIndex, addedIndex, payload: item, element } = dropResult;
      const previousColumnId = this.draggingItemCurrentColumnId;
      const currentColumnId = item[this.itemsColumnKey];
      // this.draggingItemCurrentColumnId = null;
      // there's problem with indexes - item removed earlier than removeIndex calculates and removeIndex is grater than real by 1
      const isDroppedToLeftSide = previousColumnId as any > currentColumnId;
      console.log('isDroppedToLeftSide', isDroppedToLeftSide, previousColumnId, currentColumnId);
      // this.draggingItemCurrentColumnId = null;

      if (
        (removedIndex !== null || addedIndex !== null) &&
        removedIndex !== addedIndex
      ) {
        const itemsInColumn = this.getItemsByColumn(column);

        const varyItemsOrderKey = (
          items: any[],
          mode: "increment" | "decrement" = "increment"
        ) => {
          const modeSign = mode === "increment" ? 1 : -1;
          items.forEach((item) => {
            if (!isNaN(item[this.itemsOrderKey])) {
              item[this.itemsOrderKey] += 1 * modeSign;
            } else if (/.+\d+$/.test(item[this.itemsOrderKey])) {
              const oldNumberString = item[this.itemsOrderKey].match(
                /.+(\d+)$/
              )[1];
              let number = +oldNumberString;
              number += 1 * modeSign;
              item[this.itemsOrderKey].replace(oldNumberString, "" + number);
            } else {
              console.error(
                "item's order key has no any number. Allowed number and string with numbers at the end (e.g. ord3)."
              );
            }
          });
        };

        const decrementItemsOrderKey = (items: any[]) => {
          varyItemsOrderKey(items, "decrement");
        };

        const incrementItemsOrderKey = (items: any[]) => {
          varyItemsOrderKey(items, "increment");
        };

        if (removedIndex !== null && addedIndex === null) {
          const needleStartIndex = isDroppedToLeftSide
            ? removedIndex
            : removedIndex + 1;
          const itemsAfterRemovedIndex = itemsInColumn.slice(needleStartIndex);

          decrementItemsOrderKey(itemsAfterRemovedIndex);

          // console.log(
          //   "removed",
          //   item[this.itemsColumnKey],
          //   column[this.columnKey],
          //   itemsAfterRemovedIndex,
          //   removedIndex,
          //   itemsInColumn
          // );
        }

        if (addedIndex !== null && removedIndex === null) {
          const itemsAfterAddedIndex = itemsInColumn.slice(addedIndex);

          incrementItemsOrderKey(itemsAfterAddedIndex);

          // console.log(
          //   "added",
          //   item[this.itemsColumnKey],
          //   column[this.columnKey],
          //   itemsAfterAddedIndex,
          //   addedIndex,
          //   itemsInColumn
          // );
        }

        // const result = [...arr];
        // let itemToAdd = payload;
        //
        // if (removedIndex !== null) {
        //   itemToAdd = result.splice(removedIndex, 1)[0];
        // }
        //
        // if (addedIndex !== null) {
        //   result.splice(addedIndex, 0, itemToAdd);
        // }
        //
        // return result;
      }

      /// Change column of item if needed
      if (addedIndex !== null) {
        item[this.itemsColumnKey] = column[this.columnKey];
      }
    },
    getCardPayload(
      column: Record<string, unknown>
    ): (index: number) => Record<string, unknown> {
      return (index: number) => {
        return this.getItemsByColumn(column)[index];
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
          if (
            (a[this.itemsOrderKey] as number | string) <
            (b[this.itemsOrderKey] as number | string)
          ) {
            return -1;
          }
          if (
            (a[this.itemsOrderKey] as number | string) >
            (b[this.itemsOrderKey] as number | string)
          ) {
            return 1;
          }
          return 0;
        });
    },

    // prepareItemsOrders(): void {
    //   this.items.forEach((item) => {
    //     if (!(this.itemsOrderKey in item)) {
    //       item[this.itemsOrderKey] = 1;
    //     }
    //   });
    //
    //   const itemsGroupedByColumn = groupBy(
    //     this.items,
    //     this.itemsColumnKey
    //   ) as any[][];
    //   itemsGroupedByColumn.forEach((itemsByColumn) => {
    //     const itemsGroupedBySameOrder = groupBy(
    //       itemsByColumn,
    //       this.itemsOrderKey
    //     );
    //
    //     // count of previous same values - 1 -- how much need to increase values of next (current) group
    //     let additionalAddendum = 0;
    //
    //     itemsGroupedBySameOrder.forEach((itemsWithSameOrder) => {
    //       additionalAddendum = itemsWithSameOrder.length - 1;
    //
    //       itemsWithSameOrder.forEach((item, itemIndex) => {
    //         // var++ is not safe for string values, but var += 1 is safe
    //         item[this.itemsOrderKey] += additionalAddendum + itemIndex;
    //       });
    //     });
    //
    //     additionalAddendum = 0; // todo
    //   });
    // },
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
