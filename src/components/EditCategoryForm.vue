<template>
  <div>
    <div class="modal fade" id="edit-category-modal" data-backdrop="static" data-keyboard="false" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel"
              aria-hidden="true">
      <div v-if="error">
        <ErrorMessage v-for="(message, index) in messages" :key="index" :message="message" ref="error"></ErrorMessage>
      </div>
      <div class="modal-dialog modal-dialog-centered" role="document">
          <div class="modal-content" style="background-color:transparent;border:none">
              <div class="add-edit-category-form-content">
                  <div class="add-edit-category-form-title">
                      <h1>Edit Category</h1>
                  </div>
                  <form class="add-edit-category-form">
                    <div class="form-group">
                      <label for="name">Name</label>
                      <input v-model="category.name" type="text" class="form-control add-edit-category-name" name="name">
                  </div>
                  <div class="form-group">
                      <select v-model="category.color" class="form-control color-select" name="color">
                        <option value="#ff3b30">Red</option>
                        <option value="#ff9500">Orange</option>
                        <option value="#ffcc00">Yellow</option>
                        <option value="#4cd964">Green</option>
                        <option value="#5ac8fa">Teal Blue</option>
                        <option value="#007aff">Blue</option>
                        <option value="#e066ff">Purple</option>
                        <option value="#ff4d88">Pink</option>
                      </select>
                  </div>
                  <div class="category-form-btn-container">
                      <button @click.prevent="editCategory" type="submit" class="btn mt-4 add-edit-category-btn">Edit</button>
                      <button @click="$emit('close-edit-category-form')" type="button" class="btn mt-2 category-cancel-btn">Cancel</button>
                  </div>
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
  name: "EditCategoryForm",
  props: ["CategoryId", "categoryEditData"],
  data() {
    return {
      error:false,
      messages: []
    }
  },
  methods: {
    editCategory() {
      axios({
          url: `https://kanban-server-rdport.onrender.com/categories/${this.CategoryId}`,
          method: "PUT",
          withCredentials: true,
          headers: {
              access_token: localStorage.getItem("access_token")
          },
          data: this.category
      })
      .then(({data}) => {
          this.$emit("fetchCategories");
          this.$emit("fetchTasks");
          this.$emit("close-edit-category-form");
          Swal.fire (
              "Edited",
              "The category has been edited.",
              "success"
          );
      })
      .catch((err) => {
          console.log(err);
          this.messages = err.response.data.messages;
          this.error = true;
          this.$nextTick(()=> {
             this.$refs.error[0].$el.scrollIntoView();
          });
      })
    }
  },
  computed: {
    category() {
      let obj = {
        name: this.categoryEditData.name,
        color: this.categoryEditData.color
      }
    return obj;
    } 
  },
  components: {
    ErrorMessage
  }
}
</script>
<style>
  
</style>