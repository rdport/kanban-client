<template>
  <div>
    <div class="modal fade" id="add-task-modal" data-backdrop="static" data-keyboard="false" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel"
    aria-hidden="true">
      <div v-if="error">
        <ErrorMessage v-for="(message, index) in messages" :key="index" :message="message" ref="error"></ErrorMessage>
      </div>
      <div class="modal-dialog modal-dialog-centered" role="document">
          <div class="modal-content" style="background-color:transparent;border:none">
              <div class="add-edit-task-form-content">
                  <div class="add-edit-task-form-title">
                      <h1>Add Task</h1>
                  </div>
                  <form class="add-edit-task-form">
                    <div class="form-group">
                        <label for="title">Title</label>
                        <input v-model="task.title" type="text" class="form-control" id="add-task-title" name="title">
                    </div>
                    <div class="form-group">
                        <label for="message-text" class="col-form-label">Description</label>
                        <textarea v-model="task.description" class="form-control text-area"></textarea>
                    </div>
                    <div class="form-group">
                        <label for="due_date">Due Date</label>
                        <input v-model="task.due_date" type="date" class="form-control" id="add-task-due_date" name="due_date">
                    </div>
                    <button type="submit" @click.prevent="addTask" class="btn mt-4 add-edit-task-btn">Add</button>
                    <button data-dismiss="modal" @click="$emit('close-add-task-form')" type="button" class="btn mt-2 add-edit-cancel-btn">Cancel</button>
                </form>
              </div>
          </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import ErrorMessage from "./ErrorMessage";
export default {
  name: "AddTaskForm",
  props: ["CategoryId"],
  data(){
    return {
      task: {
        title: "",
        description: "",
        due_date: "",
        CategoryId: 0
      },
      error: false,
      messages: []
    }
  },
  methods: {
    addTask() {
      this.task.CategoryId = this.CategoryId;
      axios({
          url: "https://kanban-server-rdport.onrender.com/tasks/add",
          method: "POST",
          withCredentials: true,
          headers: {
              access_token: localStorage.getItem("access_token")
          },
          data: this.task
      })
      .then(({data}) => {
          this.$emit("fetchCategories");
          this.$emit("fetchTasks");
          this.$emit("close-add-task-form");
          Swal.fire (
              "Added",
              "A new task has been added.",
              "success"
          );
      })
      .catch((err) => {
          console.log(err);
          this.messages = err.response.data.messages;
          this.error = true;
          this.$nextTick(()=> {
            // console.log(this.$refs.error)
             this.$refs.error[0].$el.scrollIntoView();
          });
      })
      .finally(() => {
          this.task.title = "";
          this.task.description = "";
          this.task.due_date = "";
      });
    }
  },
  components: {
    ErrorMessage
  }
}
</script>
<style>
  
</style>