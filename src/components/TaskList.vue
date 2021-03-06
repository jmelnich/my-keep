<template>
    <div class="content-layout">
        <side-navigation
                v-on:filter-tasks="updateFilterQuery"
                v-bind:filterQuery="filterQuery"
        ></side-navigation>
        <div  v-if="error" class="error">{{error}}</div>
        <ul class="task-list">
          <task-item class="item"
                     v-for="task in filteredTasks()"
                     v-bind:task="task"
                     v-bind:key="task._id"
                     v-on:remove-task="removeTask"
                     v-on:complete-task="completeTask"
                     v-on:archive-task="archiveTask"
                     v-bind:class="[task.completed ? 'task-item-completed' : '',
                     task.archived ? 'task-item-archived' : '']"
          >
          </task-item>
        </ul>
    </div>
</template>

<script>

import axios from 'axios';
import TaskItem from './TaskItem.vue';
import SideNavigation from './SideNavigation.vue';
import api from '../api';

export default {
  components: { TaskItem, SideNavigation },
  name: 'task-list',
  props: {
    newTask: Object,
  },
  data() {
    return {
      tasks: [],
      filterQuery: 'todo',
      error: null,
    };
  },
  mounted() {
    this.$root.$on('addTaskEvent', taskObj => this.tasks.push(taskObj));
    axios
      .get(`${api.URL}`, {
        headers: {
        'x-apikey': api.secretKey
        }
      })
      .then((res) => {
        this.tasks = res.data;
      })
      .catch((error) => {
        this.error = error;
    });
  },
  methods: {
    updateFilterQuery(filterQuery) {
      this.filterQuery = filterQuery;
    },
    filteredTasks() {
      if (this.filterQuery === 'todo') {
        return this.tasks.filter(task => !task.archived);
      } else if (this.filterQuery === 'archived') {
        return this.tasks.filter(task => task.archived);
      } else if (this.filterQuery === 'all') {
        return this.tasks;
      }
      return this.tasks;
    },
    removeTask(id) {
      axios
        .delete(`${api.URL}/${id}`, { headers: { 'x-apikey': api.secretKey }})
        .then(() => this.tasks = this.tasks.filter(task => task._id !== id))
        .catch((error) => this.error = error);
    },
    completeTask(id) {
      const taskToUpdate = this.tasks.find(task => task._id === id);
      if (!taskToUpdate) {
        return;
      }
      taskToUpdate.completed = !taskToUpdate.completed;
      axios
        .put(`${api.URL}/${id}`, taskToUpdate, { headers: { 'x-apikey': api.secretKey }})
        .then((res) => {
          this.tasks = this.tasks.map(task => (task._id === id ? res.data : task));
        })
        .catch((error) => this.error = error);
    },
    archiveTask(id) {
      const taskToUpdate = this.tasks.find(task => task._id === id);
      if (!taskToUpdate) {
        return;
      }
      taskToUpdate.archived = !taskToUpdate.archived;
      axios
        .put(`${api.URL}/${id}`, taskToUpdate, { headers: { 'x-apikey': api.secretKey }})
        .then((res) => {
          this.tasks = this.tasks.map(task => (task._id === id ? res.data : task));
        })
        .catch((error) => this.error = error);
    },
  },
};
</script>

<style scoped lang="scss">

    .content-layout {
        display: flex;
    }

    .task-list {
      display: flex;
      flex-wrap: wrap;
    }
</style>
