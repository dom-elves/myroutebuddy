<template>
  <div
    :class="[
      'p-4 bg-white rounded-lg shadow dark:bg-gray-800',
      isFinalView ? 'w-full' : '',
    ]"
  >
    <RouteStats :route="route" />
    <PinnedTasks
      :pinned-tasks="pinnedTasks"
      :route="route"
      @unpin="unpinTask"
      @clear-all-pins="clearAllPins"
    />

    <!-- Banner indicating Final View -->
    <div
      v-if="isFinalView"
      class="bg-blue-500 text-white text-center p-2 rounded mb-4"
    >
      You are in Final View. Click tasks to mark them as complete.
    </div>

    <h2 class="text-xl font-bold m-4 text-gray-800 dark:text-gray-200">
      Your Route
    </h2>

    <!-- Hide search bar in final view -->
    <input
      v-if="!isFinalView"
      type="text"
      v-model="searchQuery"
      placeholder="Search your route..."
      class="w-full border rounded-lg p-2 mb-4 focus:ring-2 focus:ring-blue-500 focus:outline-none dark:bg-gray-700 dark:border-gray-600 dark:text-gray-200"
    />

    <Container
      v-if="!isFinalView"
      :get-child-payload="getTaskPayload"
      group-name="tasks"
      @drop="onDrop"
    >
      <Draggable
        v-for="(task, index) in filteredRoute"
        :key="task.id"
        :id="`task-${task.id}`"
        :class="[
          'p-2 py-3 rounded-lg shadow border transition mb-2 relative cursor-pointer',
          task.completed
            ? 'bg-gray-100 hover:bg-gray-200 dark:bg-gray-700 dark:hover:bg-gray-600'
            : getColorClass(task.color),
          !task.completed &&
            (getHoverClass(task.color) ||
              'hover:bg-gray-100 dark:hover:bg-gray-600'),
          task.completed
            ? 'border-gray-200 dark:border-gray-700'
            : getBorderClass(task.color) ||
              (task.custom
                ? 'border-blue-600 dark:border-blue-500'
                : 'border-gray-200 dark:border-gray-700'),
        ]"
        drag-handler="handle"
      >
        <TaskContent
          :task="task"
          :index="index"
          :is-final-view="isFinalView"
          :is-pinned="isPinned(task)"
          :max-position="route.length"
          @toggle-completion="toggleCompletion"
          @edit-task="editTask"
          @save-edit="saveEdit"
          @cancel-edit="cancelEdit"
          @update-task-color="updateTaskColor"
          @pin-task="pinTask"
          @unpin-task="unpinTask"
          @insert-after="insertAfter"
          @remove-task="removeTaskById"
          @move-task="moveTaskToPosition"
        />
      </Draggable>
    </Container>

    <!-- Final view list -->
    <div v-else>
      <div
        v-for="(task, index) in filteredRoute"
        :key="task.id"
        :id="`task-${task.id}`"
        :class="[
          'p-2 py-3 rounded-lg shadow border transition mb-2 relative',
          task.completed
            ? 'bg-gray-100 hover:bg-gray-200 dark:bg-gray-700 dark:hover:bg-gray-600'
            : getColorClass(task.color),
          !task.completed &&
            (getHoverClass(task.color) ||
              'hover:bg-gray-100 dark:hover:bg-gray-600'),
          task.completed
            ? 'border-gray-200 dark:border-gray-700'
            : getBorderClass(task.color) ||
              (task.custom
                ? 'border-blue-600 dark:border-blue-500'
                : 'border-gray-200 dark:border-gray-700'),
          'cursor-pointer',
        ]"
        @click="toggleCompletion(task)"
      >
        <TaskContent
          :task="task"
          :index="index"
          :is-final-view="isFinalView"
          @toggle-completion="toggleCompletion"
        />
      </div>
    </div>

    <RouteNav :route="route" />
  </div>
</template>

<script>
import { Container, Draggable } from "vue3-smooth-dnd";
import ColorSelect from "./ColorSelect.vue";
import { colorMap } from "./ColorSelect.vue";
import RegionIcon from "./RegionIcon.vue";
import RouteStats from "./RouteStats.vue";
import PinnedTasks from "./PinnedTasks.vue";
import RouteNav from "./RouteNav.vue";
import TaskContent from "./TaskContent.vue";

export default {
  props: {
    route: Array,
    isFinalView: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      searchQuery: "",
    };
  },
  computed: {
    pinnedTasks() {
      return this.route.filter((task) => task.pinned);
    },
    filteredRoute() {
      if (!this.searchQuery) return this.route;
      const query = this.searchQuery.toLowerCase();
      return this.route.filter((task) =>
        task.task.toLowerCase().includes(query)
      );
    },
  },
  methods: {
    getColorClass(color) {
      return color && colorMap[color]
        ? colorMap[color].bg
        : "bg-gray-50 dark:bg-gray-700";
    },
    getHoverClass(color) {
      return color && colorMap[color] ? colorMap[color].hover : null;
    },
    getBorderClass(color) {
      return color && colorMap[color] ? colorMap[color].border : null;
    },
    getTaskPayload(index) {
      return this.filteredRoute[index];
    },
    onDrop(dropResult) {
      if (dropResult.removedIndex !== null || dropResult.addedIndex !== null) {
        let newRoute = [...this.route];

        if (dropResult.removedIndex !== null) {
          newRoute.splice(dropResult.removedIndex, 1);
        }

        if (dropResult.addedIndex !== null) {
          const task = dropResult.payload;

          if (task.completed === undefined) {
            task.completed = false;
          }

          if (!newRoute.some((t) => t.id === task.id)) {
            newRoute.splice(dropResult.addedIndex, 0, task);
          } else {
            if (dropResult.removedIndex !== null) {
              newRoute.splice(dropResult.addedIndex, 0, task);
            } else {
              alert("Task is already in your route.");
            }
          }
        }

        this.$emit("update-route", newRoute);
      }
    },
    removeTaskById(taskId) {
      const updatedRoute = this.route.filter((task) => task.id !== taskId);
      this.$emit("update-route", updatedRoute);
    },
    toggleCompletion(task) {
      const updatedTask = { ...task, completed: !task.completed };
      if (updatedTask.completed && updatedTask.pinned) {
        updatedTask.pinned = false;
      }
      this.updateTask(updatedTask);
    },
    editTask(task) {
      const updatedTask = { ...task, isEditing: true, editableTask: task.task };
      this.updateTask(updatedTask);
    },
    saveEdit(task) {
      const updatedTask = {
        ...task,
        task: task.editableTask,
        isEditing: false,
      };
      delete updatedTask.editableTask;
      this.updateTask(updatedTask);
    },
    cancelEdit(task) {
      const updatedTask = { ...task, isEditing: false };
      delete updatedTask.editableTask;
      this.updateTask(updatedTask);
    },
    updateTask(updatedTask) {
      const index = this.route.findIndex((t) => t.id === updatedTask.id);
      if (index !== -1) {
        const newRoute = [...this.route];
        newRoute.splice(index, 1, updatedTask);
        this.$emit("update-route", newRoute);
      }
    },
    insertAfter(task) {
      const index = this.route.findIndex((t) => t.id === task.id);
      const newTask = {
        id: Date.now(),
        task: "",
        points: 0,
        custom: true,
        region: "Global",
        completed: false,
        isEditing: true,
        editableTask: "",
      };

      const newRoute = [...this.route];
      newRoute.splice(index + 1, 0, newTask);
      this.$emit("update-route", newRoute);
    },
    updateTaskColor(updatedTask) {
      this.updateTask(updatedTask);
    },
    pinTask(task) {
      const updatedTask = { ...task, pinned: true };
      this.updateTask(updatedTask);
    },
    unpinTask(task) {
      const updatedTask = { ...task, pinned: false };
      this.updateTask(updatedTask);
    },
    clearAllPins(tasksToUnpin) {
      const updatedRoute = this.route.map((task) => {
        if (tasksToUnpin.some((pinnedTask) => pinnedTask.id === task.id)) {
          return { ...task, pinned: false };
        }
        return task;
      });
      this.$emit("update-route", updatedRoute);
    },
    isPinned(task) {
      return task.pinned === true;
    },
    moveTaskToPosition(task, newIndex) {
      const oldIndex = this.route.findIndex((t) => t.id === task.id);
      if (oldIndex !== -1) {
        const newRoute = [...this.route];
        newRoute.splice(oldIndex, 1);
        newRoute.splice(newIndex, 0, task);
        this.$emit("update-route", newRoute);
      }
    },
  },
  components: {
    Container,
    Draggable,
    ColorSelect,
    RegionIcon,
    RouteStats,
    PinnedTasks,
    RouteNav,
    TaskContent,
  },
};
</script>
