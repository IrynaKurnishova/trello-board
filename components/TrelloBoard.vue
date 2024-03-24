<template>
  <div class="flex items-start overflow-x-auto gap-4">
    <draggableComponent
        v-model="columns"
        group="columns"
        :animation="150"
        handle=".drag-handle"
        item-key="id"
        class="flex gap-4 items-start"
    >
      <template #item="{element: column}: { element: Column}">
        <div
            class="column bg-gray-200 p-5 rounded min-w-[250px]"
        >
          <header class="font-bold mb-4 flex items-center">
            <DragHandle/>
            <input
                class="title-input bg-transparent focus:bg-white rounded px-1 w-4/5"
                @keyup.enter="($event.target as HTMLInputElement).blur()"
                @keydown.backspace="
                column.title === ''
                ? columns = columns.filter(c => c.id !== column.id)
                : null
"
                type="text"
                v-model="column.title"
            />
          </header>
          <draggableComponent
              v-model="column.tasks"
              :group="{name: 'tasks', pull: alt ? 'clone' : true}"
              handle=".drag-handle"
              :animation="150"
              item-key="id"
          >
            <template #item="{element: task} : {element: Task}">
              <div>
                <TrelloBoardTask :task="task" @delete="column.tasks = column.tasks.filter((t) => t.id !== $event)"/>
              </div>
            </template>
          </draggableComponent>
          <footer>
            <NewTask @add="column.tasks.push($event)"/>
          </footer>
        </div>
      </template>
    </draggableComponent>
    <button
        @click="createColumn"
        class="bg-gray-200 whitespace-nowrap p-2 rounded opacity-50"
    >
      + Add Another Column
    </button>
  </div>
</template>

<script setup lang="ts">
import type { Column, Task } from "~/types";
import {nanoid} from "nanoid";
import draggableComponent from "vuedraggable";
import DragHandle from "~/components/DragHandle.vue";
const columns = useLocalStorage<Column[]>("trelloBoard", [
  {
    id: nanoid(),
    title: "Backlog",
    tasks: [
      {
        id: nanoid(),
        title: "Create marketing landing page",
        createdAt: new Date()
      },
      {
        id: nanoid(),
        title: "Develop cool new feature",
        createdAt: new Date()
      },
      {
        id: nanoid(),
        title: "Fix page nav bug",
        createdAt: new Date()
      }
    ],
  },
  {title: "Selected for dev", id: nanoid(), tasks: []},
  {title: "In progress", id: nanoid(), tasks: []},
  {title: "QA", id: nanoid(), tasks: []},
  {title: "Complete", id: nanoid(), tasks: []},
],
    {
      serializer: {
        read: (value) => {
          return JSON.parse(value).map((column: Column) => {
            column.tasks = column.tasks.map((task: Task) => {
              task.createdAt = new Date(task.createdAt);
              return task;
            });
            return column;
          });
        },
        write: (value) => JSON.stringify(value),
      }
    }
)
const alt = useKeyModifier("Alt")

// if need to refresh backend (not localstorage)
// watch(
//     columns,
//     () => {
//       // ajax request
//     },
//     {
//       deep: true,
//     }
// )
function createColumn() {
  const column: Column = {
    id: nanoid(),
    title: "",
    tasks: []
  }
  columns.value.push(column);
  nextTick(() => {
    (document.querySelector(
        ".column:last-of-type .title-input"
    ) as HTMLInputElement
    ).focus()
  })
}
</script>
