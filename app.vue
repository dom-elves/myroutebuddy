<template>
  <div class="min-h-screen flex bg-gray-50 dark:bg-gray-900">
    <!-- Sidebar -->
    <div :class="{'lg:fixed lg:inset-y-0 lg:z-50 flex flex-col transition-all duration-300 ': true, 'lg:w-0 md:w-0': isCollapsed, 'lg:w-2/6 md:w-full': !isCollapsed, 'absolute  top-0 left-0 z-40': true,
      }">
      <!-- Moved button toggle to parent div for UX, started untoggled  -->
      <div class="flex items-center justify-center p-2 bg-blue-600 rounded-lg cursor-pointer " @click="toggleSidebar" type="button" aria-label="Toggle progress sidebar"> 
          <span class="text-lg text-white">
            <span v-if="isCollapsed">&gt;</span> <!-- Right arrow when collapsed -->
            <span v-else>&lt;</span> <!-- Left arrow when expanded -->
          </span>
      </div>
      <div v-show="!isCollapsed" class="flex grow flex-col gap-y-5 overflow-y-auto scrollbar-hide lg:border-r lg:border-r-gray-700 border-b-2 border-b-gray-700 lg:border-b-0  bg-white px-6 pb-4 dark:bg-gray-800 " >
        <!-- Progress Section -->
        <div class="mt-4">
          <h3 class="text-lg font-semibold text-gray-700 mb-4 dark:text-gray-200">Progress</h3>
          <div class="w-full bg-gray-200 rounded-full h-6">
            <div
              class="bg-blue-500 h-6 rounded-full transition-all"
              :style="{ width: progressPercentage + '%' }"
            ></div>
          </div>
          <p class="text-md text-gray-500 mt-2 dark:text-gray-400">
            {{ completedPoints }} of {{ totalPoints }} points completed
          </p>
        </div>

        <!-- Time Until Launch -->
        <div class="mt-4">
          <h3 class="text-lg font-semibold text-gray-700 dark:text-gray-200">Time Until Launch</h3>
          <p class="text-md text-gray-500 dark:text-gray-400">{{ timeUntilLaunch }}</p>
        </div>

        <!-- Region Selection -->
        <div class="mt-8">
          <RegionFilter
            :regions="regions"
            :selectedRegions="selectedRegions"
            @update-regions="updateRegions"
          />
        </div>

        <!-- Manage Routes Section -->
        <nav class="flex flex-1 flex-col mt-8">
          <h3 class="text-lg font-semibold text-gray-700 mb-4 dark:text-gray-100">Manage Your Routes</h3>
            <p class="text-sm text-gray-700 dark:text-gray-200 mb-4">
              Have an awesome route and want to share it with everyone? See how you can add it to the pre-loaded default routes.
              <a
                href="https://github.com/KennethLuczko/myroutebuddy?tab=readme-ov-file#how-to-add-your-own-route-to-the-default-routes"
                class="text-blue-500 underline hover:text-blue-700"
                target="_blank"
                rel="noopener noreferrer"
              >
                Learn more.
              </a>
            </p>
          <div class="space-y-3">
            <!-- Save Route -->
            <button
              @click="saveRoute"
              class="w-full bg-blue-500 text-white px-4 py-2 rounded-lg shadow-sm hover:bg-blue-600 transition dark:bg-blue-700 dark:hover:bg-blue-800"
            >
              Save Route
            </button>

            <!-- Load Route -->
            <div class="space-y-2">
              <select
                v-model="selectedRoute"
                class="w-full bg-gray-100 border border-gray-300 rounded-lg p-2 focus:outline-none focus:ring-2 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-200"
              >
                <option value="" disabled>Select a Route</option>
                <option
                  v-for="(routeName, index) in Object.keys(savedRoutes)"
                  :key="index"
                  :value="routeName"
                >
                  {{ routeName }}
                </option>
              </select>
              <button
                @click="loadRoute"
                :disabled="!selectedRoute"
                class="w-full bg-gray-300 text-gray-700 py-2 rounded-lg shadow-sm hover:bg-gray-400 transition disabled:opacity-50 dark:bg-gray-700 dark:text-gray-300"
              >
                Load Route
              </button>
            </div>

            <!-- Share Route -->
            <button
              @click="shareRoute"
              class="w-full bg-green-500 text-white px-4 py-2 rounded-lg shadow-sm hover:bg-green-600 transition dark:bg-green-700 dark:hover:bg-green-800"
            >
              Share Route
            </button>

            <!-- Import Route -->
            <div>
              <textarea
                v-model="importedRoute"
                placeholder="Paste shared route JSON here..."
                class="w-full bg-gray-100 border border-gray-300 rounded-lg p-2 focus:outline-none focus:ring-2 focus:ring-blue-500 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-200"
              ></textarea>
              <button
                @click="importRoute"
                :disabled="!importedRoute"
                class="w-full bg-purple-500 text-white py-2 rounded-lg shadow-sm hover:bg-purple-600 transition disabled:opacity-50 dark:bg-purple-700 dark:hover:bg-purple-800"
              >
                Import Route
              </button>
            </div>

            <!-- Reset App -->
            <button
              @click="resetApp"
              class="w-full bg-red-500 text-white px-4 py-2 rounded-lg shadow-sm hover:bg-red-600 transition dark:bg-red-700 dark:hover:bg-red-800"
            >
              Reset
            </button>
          </div>

          <!-- Dark Mode Toggle Section -->
          <div class="mt-6 pt-4 border-t border-gray-200 dark:border-gray-700">
            <h3 class="text-lg font-semibold text-gray-700 dark:text-gray-100 mb-4">Theme</h3>
            <button
              @click="toggleDarkMode"
              class="w-full bg-gray-800 text-white px-4 py-2 rounded-lg shadow-sm hover:bg-gray-700 transition dark:bg-gray-100 dark:text-gray-800 dark:hover:bg-gray-200"
            >
              Toggle Dark Mode
            </button>
          </div>

          <!-- Change Log Section -->
          <div class="mt-6 pt-4 border-t border-gray-200 dark:border-gray-700">
            <h3 class="text-lg font-semibold text-gray-700 dark:text-gray-100">Change Log (latest 3 changes)</h3>
            <a
              href="https://github.com/KennethLuczko/myroutebuddy"
              class="text-blue-500 underline hover:text-blue-700 text-sm"
              target="_blank"
              rel="noopener noreferrer"
            >
              Contribute to this project.
            </a>
            <ul>
              <li
                v-for="(log, index) in changeLog"
                :key="index"
                class="p-4 bg-gray-100 dark:bg-gray-900 rounded-lg mb-2 shadow border dark:border-gray-700"
              >
                <p class="text-sm text-gray-800 dark:text-gray-200">
                  <span class="font-semibold">Date:</span> {{ log.date }}
                </p>
                <p class="text-sm text-gray-800 dark:text-gray-200">
                  <span class="font-semibold">Time:</span> {{ log.time }}
                </p>
                <p class="text-sm text-gray-800 dark:text-gray-200">
                  <span class="font-semibold">Change: </span>
                  <a
                    :href="log.url"
                    class="text-blue-500 underline hover:text-blue-700"
                    target="_blank"
                    rel="noopener noreferrer"
                  >
                    {{ log.change }}
                  </a>
                </p>
                <p class="text-sm text-gray-800 dark:text-gray-200">
                  <span class="font-semibold">Contributor:</span> {{ log.contributor }}
                </p>
              </li>
            </ul>
          </div>
        </nav>
      </div>
    </div>
    <!-- Sticky Scroll to Top Button for Mobile UX -->
    <div
      ref="scrollTopButton"
      class="fixed bottom-0 right-0 pb-3 pr-5 transition z-20">
      <button
        class="flex justify-center items-center w-12 h-12 rounded-full bg-white text-black cursor-pointer text-3xl transition transform hover:scale-110 hover:bg-blue-500 hover:text-white hover:shadow-lg"
        @click="scrollToTop"
        role="button"
        aria-label="Scroll to top of the page">
        ↑
      </button>
    </div>

    <!-- Main Content Area -->
    <div :class="{'lg:pl-1/3vw': !isCollapsed, 'flex-1': true}">
      <div class="p-6">
        <h1 class="text-4xl font-bold mb-8 text-gray-800 dark:text-white">My Route Buddy - Leagues 5: Raging Echoes Task Route Planner</h1>
        <h2 class="text-md mb-2 text-gray-800 dark:text-gray-200">
          If you have any issues, want to report a bug, have a suggestion, or want to help develop this web app yourself, please visit
          <a href="https://github.com/KennethLuczko/myroutebuddy" 
            class="text-blue-500 underline hover:text-blue-700" 
            target="_blank" 
            rel="noopener noreferrer">
            this GitHub and contribute.
          </a>
        </h2>
        <h3 class="text-md mb-8 text-gray-800 dark:text-gray-200">
          Not sure where to begin? Discover your regions, combat masteries, and more
          <a href="https://docs.google.com/spreadsheets/d/1LJVEIIR9D56FF9PyUzb4nhofaCjpzSWTRGsUNsH7lNw/copy" 
            class="text-blue-500 underline hover:text-blue-700" 
            target="_blank" 
            rel="noopener noreferrer">
            here.
          </a>
        </h3>

        <!-- Final View Toggle Button -->
        <button
          @click="toggleFinalView"
          class="mb-4 bg-gray-800 text-white px-4 py-2 rounded-lg shadow-sm hover:bg-gray-700 transition dark:bg-gray-100 dark:text-gray-800 dark:hover:bg-gray-200"
        >
          {{ isFinalView ? 'Exit Final View (removes available tasks)' : 'Enter Final View (removes available tasks)' }}
        </button>

        <!-- Adjusted Content Based on Final View State -->
        <div>
          <div v-if="!isFinalView" class="grid grid-cols-1 md:grid-cols-2 gap-8">
            <!-- Available Tasks -->
            <TaskList :tasks="filteredTasks" @add-task="addTask" />

            <!-- Your Route -->
            <RouteBuilder :route="route" @update-route="updateRoute" />
          </div>

          <div v-else>
            <!-- Your Route Spanning Full Width -->
            <RouteBuilder :route="route" @update-route="updateRoute" :isFinalView="true" />
          </div>
        </div>
      </div>
    </div>
    
  </div>
</template>

<script>
import TaskList from './components/TaskList.vue';
import RegionFilter from './components/RegionFilter.vue';
import RouteBuilder from './components/RouteBuilder.vue';
import axios from 'axios';
import { onMounted, onUnmounted, ref } from "vue"; 
export default {
  components: { TaskList, RegionFilter, RouteBuilder },
  data() {
    return {
      regions: [
        { id: 2, name: 'Asgarnia' },
        { id: 3, name: 'Fremennik' },
        { id: 4, name: 'Kandarin' },
        { id: 5, name: 'Desert' },
        { id: 6, name: 'Morytania' },
        { id: 7, name: 'Tirannwn' },
        { id: 8, name: 'Wilderness' },
        { id: 9, name: 'Zeah' },
        { id: 10, name: 'Varlamore' },
      ],
      selectedRegions: [],
      tasks: [],
      route: [],
      savedRoutes: {},
      selectedRoute: '',
      importedRoute: '',
      isFinalView: false,
      changeLog: [],
      launchDate: new Date('2024-11-27T07:00:00-05:00'), // Launch date in EST
      intervalId: null,
      timeUntilLaunch: '', // Countdown string
      isCollapsed: false,
    };
  },
  computed: {
    filteredTasks() {
      const allowedRegions = ['Global', ...this.selectedRegions];
      return this.tasks.filter(
        (task) =>
          allowedRegions.includes(task.region) &&
          !this.route.some((r) => r.id === task.id)
      );
    },
    totalPoints() {
      return this.route.reduce((sum, task) => sum + task.points, 0);
    },
    completedPoints() {
      return this.route.reduce((sum, task) => (task.completed ? sum + task.points : sum), 0);
    },
    progressPercentage() {
      return this.totalPoints ? (this.completedPoints / this.totalPoints) * 100 : 0;
    },
    recentChangeLog() {
      return this.changeLog.slice(-3);
    },
  },
  methods: {
    updateTimeUntilLaunch() {
      const now = new Date();
      const timeDifference = this.launchDate - now;

      if (timeDifference <= 0) {
        clearInterval(this.intervalId);
        this.timeUntilLaunch = 'Launch time has arrived!';
        return;
      }

      const days = Math.floor(timeDifference / (1000 * 60 * 60 * 24));
      const hours = Math.floor((timeDifference % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
      const minutes = Math.floor((timeDifference % (1000 * 60 * 60)) / (1000 * 60));
      const seconds = Math.floor((timeDifference % (1000 * 60)) / 1000);

      this.timeUntilLaunch = `${days}d ${hours}h ${minutes}m ${seconds}s`;
    },
    updateTasks(updatedTasks) {
      this.tasks = updatedTasks;
      localStorage.setItem('tasks', JSON.stringify(this.tasks));
    },
    updateRegions(regions) {
      this.selectedRegions = regions;
      localStorage.setItem('selectedRegions', JSON.stringify(regions));
    },
    addTask(task) {
      if (!this.route.some((r) => r.id === task.id)) {
        this.route.push({ ...task, completed: false });
        localStorage.setItem('route', JSON.stringify(this.route));
      } else {
        alert('Task is already in your route.');
      }
    },
    updateRoute(newRoute) {
      this.route = newRoute;
      localStorage.setItem('route', JSON.stringify(this.route));
    },
    saveRoute() {
      const routeName = prompt('Enter a name for this route:');
      if (routeName) {
        this.savedRoutes[routeName] = JSON.stringify(this.route);
        localStorage.setItem('savedRoutes', JSON.stringify(this.savedRoutes));
        alert(`Route "${routeName}" has been saved.`);
      }
    },
    loadRoute() {
      if (this.selectedRoute && this.savedRoutes[this.selectedRoute]) {
        this.route = JSON.parse(this.savedRoutes[this.selectedRoute]);
        alert(`Route "${this.selectedRoute}" has been loaded.`);
      }
    },
    shareRoute() {
      const shareableRoute = JSON.stringify(this.route);
      navigator.clipboard.writeText(shareableRoute).then(() => {
        alert('Route copied to clipboard! Share it with others.');
      });
    },
    importRoute() {
      try {
        const imported = JSON.parse(this.importedRoute);
        const routeName = prompt('Enter a name for the imported route:');
        if (routeName) {
          this.savedRoutes[routeName] = JSON.stringify(imported);
          localStorage.setItem('savedRoutes', JSON.stringify(this.savedRoutes));
          alert(`Route "${routeName}" has been imported and saved.`);
          this.importedRoute = '';
        }
      } catch (error) {
        alert('Invalid route JSON. Please try again.');
      }
    },
    resetApp() {
      if (confirm('Are you sure you want to reset? This will clear your route and selected regions.')) {
        this.route = [];
        this.selectedRegions = [];
        localStorage.removeItem('route');
        localStorage.removeItem('selectedRegions');
        alert('App has been reset.');
      }
    },
    toggleDarkMode() {
      const html = document.documentElement;
      if (html.classList.contains('dark')) {
        html.classList.remove('dark');
        localStorage.setItem('theme', 'light');
      } else {
        html.classList.add('dark');
        localStorage.setItem('theme', 'dark');
      }
    },
    toggleFinalView() {
      this.isFinalView = !this.isFinalView;
    },
    loadTasks() {
      axios
        .get('./tasks.json')
        .then((response) => {
          this.tasks = response.data.map((task) => ({
            ...task,
            task: `${task.task}`,
          }));
          console.log('Tasks loaded:', this.tasks);
        })
        .catch((error) => {
          console.error('Error loading tasks:', error);
        });
    },
    fetchLatestCommits() {
      const cachedData = localStorage.getItem('changeLog');
      const cachedTime = localStorage.getItem('changeLogTime');
      const now = Date.now();

      if (cachedData && cachedTime && now - cachedTime < 15 * 60 * 1000) {
        this.changeLog = JSON.parse(cachedData);
        return;
      }

      const repoOwner = 'KennethLuczko';
      const repoName = 'myroutebuddy';

      axios
        .get(`https://api.github.com/repos/${repoOwner}/${repoName}/commits`, {
          params: { per_page: 3, sort: 'date', direction: 'desc' },
        })
        .then((response) => {
          this.changeLog = response.data.map((commit) => ({
            date: new Date(commit.commit.author.date).toLocaleDateString(),
            time: new Date(commit.commit.author.date).toLocaleTimeString(),
            change: commit.commit.message,
            contributor: commit.author ? commit.author.login : commit.commit.author.name,
            url: commit.html_url,
          }));

          localStorage.setItem('changeLog', JSON.stringify(this.changeLog));
          localStorage.setItem('changeLogTime', now);
        })
        .catch((error) => {
          console.error('Error fetching commits:', error);
        });
    },
    toggleSidebar() {
      this.isCollapsed = !this.isCollapsed;
    },
    // Scroll to Top Button
    scrollTopButton: ref(null),
    handleScroll() {
      if (window.scrollY > 0) {
        this.scrollTopButton.value.classList.remove("invisible");
      } else {
        this.scrollTopButton.value.classList.add("invisible");
      }
    },
    scrollToTop() {
      window.scrollTo({ top: 0, behavior: 'smooth' });
    }
  
  },
  mounted() {
    this.updateTimeUntilLaunch();
    this.intervalId = setInterval(this.updateTimeUntilLaunch, 1000);

    const savedTheme = localStorage.getItem('theme');
    if (savedTheme === 'dark' || (!savedTheme && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
      document.documentElement.classList.add('dark');
    }

    const savedRegions = localStorage.getItem('selectedRegions');
    if (savedRegions) {
      this.selectedRegions = JSON.parse(savedRegions);
    }

    const savedRoute = localStorage.getItem('route');
    if (savedRoute) {
      this.route = JSON.parse(savedRoute);
    }

    const savedRoutes = localStorage.getItem('savedRoutes');
    if (savedRoutes) {
      this.savedRoutes = JSON.parse(savedRoutes);
    } else {
      this.savedRoutes = {};
    }
    window.addEventListener("scroll", this.handleScroll);
    // Define the default routes
    const defaultRoutes = [
      { name: 'Wizzy (V/W/T) (6/4/0) (Harpoon) (Clue Relic) (Updated 11/23 12:36 AM)', file: './wizzy.json' },
      { name: 'Mazhar (Z/M/F) (Updated 11/25 9:09 AM)', file: './mazhar.json' },
      { name: 'Doubleshine 76+ Tasks, 1000+ Points, Relic Neutral, No Competition, Minimal Skilling/Combat Tasks (Updated 11/25 10:04 PM)', file: './doubleshine.json' },
      { name: 'Lilratbag (Updated 11/26 8:54 AM)', file: './lilratbag.json' },
    ];

    // Load each default route if not already present
    defaultRoutes.forEach((route) => {
      if (!this.savedRoutes[route.name]) {
        axios
          .get(route.file)
          .then((response) => {
            this.savedRoutes[route.name] = JSON.stringify(response.data);
            localStorage.setItem('savedRoutes', JSON.stringify(this.savedRoutes));
          })
          .catch((error) => {
            console.error(`Error loading default route (${route.name}):`, error);
          });
      }
    });

    const savedTasks = localStorage.getItem('tasks');
    if (savedTasks) {
      this.tasks = JSON.parse(savedTasks);
    } else {
      this.loadTasks();
    }

    this.fetchLatestCommits();
  },
  beforeDestroy() {
    clearInterval(this.intervalId);
    window.removeEventListener("scroll", this.handleScroll);
  },
};
</script>

<style scoped>
.transition {
  transition: width 0.3s ease;
}
</style>
