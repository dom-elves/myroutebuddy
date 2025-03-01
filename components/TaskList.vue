<template>
  <div class="p-4 bg-white rounded-lg shadow mb-4 dark:bg-gray-800">
    <!-- Add Custom Task Section -->
    <div class="mb-6">
      <h2 class="text-lg font-bold text-gray-800 dark:text-gray-200 mb-1">
        Add Custom Task / Note
      </h2>
      <p class="text-sm text-gray-600 dark:text-gray-400 mb-4">
        Add a custom task or note into your route. For example, "Have 10K GP".
      </p>
      <input
        type="text"
        v-model="customTaskName"
        placeholder="Custom Task or Note"
        class="w-full border rounded-lg p-2 focus:ring-2 focus:ring-blue-500 focus:outline-none mb-2"
      />
      <button
        @click="addCustomTask"
        class="w-full bg-blue-500 text-white px-4 py-2 rounded-lg shadow-sm hover:bg-blue-600 transition"
      >
        Add Custom Task
      </button>
    </div>

    <h2 class="text-xl font-bold mb-4 text-gray-800 dark:text-gray-200">
      Available Tasks
    </h2>

    <input
      type="text"
      v-model="searchQuery"
      placeholder="Search tasks..."
      class="w-full border rounded-lg p-2 mb-4 focus:ring-2 focus:ring-blue-500 focus:outline-none dark:bg-gray-700 dark:border-gray-600 dark:text-gray-200"
    />

    <!-- Draggable Task List -->
    <Container
      :get-child-payload="getTaskPayload"
      group-name="tasks"
      @drop="onDrop"
    >
      <Draggable
        v-for="task in filteredTasks"
        :key="task.id"
        :data-id="task.id"
        class="flex justify-between items-center bg-gray-50 hover:bg-gray-100 dark:bg-gray-700 dark:hover:bg-gray-600 p-4 rounded-lg shadow border dark:border-gray-700 cursor-pointer mb-2"
        @click="selectTask(task)"
      >
        <div class="font-medium text-gray-700 dark:text-gray-200">{{ task.task }}</div>
        <div class="text-sm text-gray-500 dark:text-gray-400 flex items-center gap-2">
          <RegionIcon :region="task.region" />
          <span>{{ task.points }} points</span>
        </div>
      </Draggable>
    </Container>
  </div>
</template>

<script>
import { Container, Draggable } from "vue3-smooth-dnd";
import RegionIcon from "./RegionIcon.vue";

export default {
  props: {
    tasks: Array,
  },
  data() {
    return {
      customTaskName: "",
      searchQuery: "",
    };
  },
  computed: {
    filteredTasks() {
      const query = this.searchQuery.toLowerCase();
      return this.tasks.filter((task) =>
        task.task.toLowerCase().includes(query)
      );
    },
  },
  methods: {
    addCustomTask() {
      if (this.customTaskName.trim()) {
        const customTask = {
          id: Date.now(),
          task: this.customTaskName.trim(),
          points: 0,
          custom: true,
          region: "Global",
          completed: false,
        };
        this.$emit("add-task", customTask);
        this.customTaskName = "";
      } else {
        alert("Please enter a task or note.");
      }
    },
    getTaskPayload(index) {
      return this.filteredTasks[index];
    },
    onDrop(dropResult) {
      if (dropResult.removedIndex !== null || dropResult.addedIndex !== null) {
        const updatedTasks = [...this.filteredTasks];

        if (dropResult.removedIndex !== null) {
          updatedTasks.splice(dropResult.removedIndex, 1);
        }

        if (dropResult.addedIndex !== null) {
          const task = dropResult.payload;
          updatedTasks.splice(dropResult.addedIndex, 0, task);
        }

        this.$emit("update-tasks", updatedTasks);
      }
    },
    selectTask(task) {
      this.$emit("add-task", task);
    },
  },
  components: {
    Container,
    Draggable,
    RegionIcon,
  },
};
</script>
