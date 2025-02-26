<template>
  <div>
    <div class="header-container">
      <div class="header-title">{{ category.name }}</div>
      <div class="header-icon-container">
        <button @click="showAddTaskForm(category.id)" type="button" class="btn mt-2 btn-sm add-task-icon" ><img src="../assets/add-task.svg" width="30" height="30"></button>
        <button @click="showEditCatForm(category.id)" type="button" class="btn mt-2 btn-sm edit-category-icon"><img src="../assets/edit.svg" width="30" height="30"></button>
        <button @click="showDelCatConfirm(category.id)" type="button" class="btn mt-2 btn-sm delete-category-icon"><img src="../assets/delete.svg" width="30" height="30"></button>
      </div>
    </div>
    <div class="category-content" style="max-height:78vh">
      <div class="no-task-container" v-if="!filteredTasksLength"><p class="no-task">No Tasks ...</p></div>
      <div class="no-task-container" v-else-if="!filteredSearchLength"><p class="no-task">No results from your search ... </p></div>
      <draggable :id="category.id"
                v-bind="{group:'tasks'}" @start="drag=true" 
                @end="drag=false" @add="newTask">
        <div v-for="item in filteredSearch"
          :key="item.id">
          <draggable :id="item.id"
            v-model="filteredSearch"
            v-bind="{group: { name:'tasks', pull:'clone', put:'false'}}"
            @start="drag=true"
            @end="drag=false"
            :move="chooseTask">
            <Task
              :task="item"
              @setTaskId="setTaskId"
              @setTaskEditData="setTaskEditData"
              @show-task-details="showTaskDetails"
              class="task-card bg-light">
            </Task>
          </draggable>
        </div>
      </draggable>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import Task from "../components/Task";
import draggable from "vuedraggable";

export default {
  name: "Category",
  props: ["category", "tasks", "chosenTaskData", "searchKey"],
  data() {
    return {
      CategoryId: 0,
      TaskId: 0,
      categoryEditData: {},
      taskEditData: {}
    }
  },
  methods: {
    showAddTaskForm(CategoryId) {
      this.$emit("show-add-task-form");
      this.$emit("setCategoryId", CategoryId);
    },
    showEditCatForm(CategoryId) {
      this.fetchEditData(CategoryId);
      // this.$emit("show-edit-category-form");
      this.$emit("setCategoryId", CategoryId);
    },
    showDelCatConfirm(CategoryId) {
      this.$emit("show-delete-category-confirmation");
      this.$emit("setCategoryId", CategoryId);
    },
    showTaskDetails() {
      this.$emit("show-task-details");
    },
    setTaskId(TaskId) {
      this.$emit("setTaskId", TaskId);
    },
    setTaskEditData(taskEditData) {
      this.$emit("setTaskEditData", taskEditData);
    },
    fetchEditData(CategoryId) {
      axios({
          url: `https://kanban-server-rdport.onrender.com/categories/${CategoryId}`,
          method: "POST",
          withCredentials: true,
          headers: {
              access_token: localStorage.getItem("access_token")
          }
      })
      .then(({data}) => {
          this.categoryEditData.name = data.name;
          this.categoryEditData.color = data.color;
          this.$emit("setCategoryEditData", this.categoryEditData);
          this.$emit("show-edit-category-form");
      })
      .catch((err) => {
          console.log(err);
      })
    },
    fetchCategories(){
      this.$emit("fetchCategories");
    },
    fetchTasks(){
      this.$emit("fetchTasks");
    },
    changePage(pageName) {
      this.$emit("changePage", pageName);
    },
    chooseTask(event) {
      const filteredArray = this.filteredTasks.filter((task) => {
        return task.id === Number(event.from.id)
      })
      const chosenTask = filteredArray[0];
      this.setChosenTaskData(chosenTask);

    },
    setChosenTaskData(chosenTask) {
      this.$emit('setChosenTaskData', chosenTask);
    },
    newTask(event) {
      if (this.chosenTaskData.CategoryId !== Number(event.to.id)) {
        axios({
          url: `https://kanban-server-rdport.onrender.com/tasks/${this.chosenTaskData.id}`,
          method: "PATCH",
          withCredentials: true,
          headers: {
              access_token: localStorage.getItem("access_token")
          },
          data: {
            CategoryId: event.to.id
          }
        })
        .then(({data}) => {
          this.fetchCategories();
          this.fetchTasks();
        })
        .catch((err) => {
            console.log(err);
        })
      }
    }
  },
  computed: {
    filteredTasks() {
      return this.tasks.filter(task => {
          return (task.CategoryId === this.category.id);
      })
    },
    filteredTasksLength() {
      return this.filteredTasks.length;
    },
    filteredSearchLength() {
      return this.filteredSearch.length;
    },
    filteredSearch: function () {
      if (this.searchKey.words && this.searchKey.by && this.filteredTasks.length !== 0) {
        return this.filteredTasks.filter((task) => {
          if (this.searchKey.by === "name") {
            const fullName = `${task.User.first_name} ${task.User.last_name}`
            if (fullName.toLowerCase().includes(this.searchKey.words.toLowerCase())) {
                return task;
            }  
          } else if (this.searchKey.by === "due") {
              return task.due_date.includes(this.searchKey.words);
          } else {
              return task.title.toLowerCase().includes(this.searchKey.words.toLowerCase());
          }
        })
      } else {
        return this.filteredTasks;
      }
    }
  },
  components: {
    Task,
    draggable
  }
}
</script>
<style>

</style>