<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <h2>{{ form.id ? "Edit Article" : "New Article" }}</h2>
      <label for="title">Title</label>
      <input type="text" v-model="form.title" id="title" placeholder="Enter title" required /><br />
      
      <label for="content">Content</label>
      <textarea v-model="form.content" id="content" placeholder="Enter content" required></textarea><br />
      
      <div class="form-buttons">
        <button type="submit" class="btn save-btn">{{ form.id ? "Update" : "Save" }}</button>
        <button type="button" @click="resetForm" class="btn cancel-btn" v-if="form.id">Cancel</button>
      </div>
    </form>
    
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <div class="article-buttons">
          <button @click="edit(article)" class="btn edit-btn">Edit</button>
          <button @click="remove(article.id)" class="btn delete-btn">Delete</button>
        </div>
      </li>
    </ul>
    
    <button @click="load" class="btn load-btn">Load Articles</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id 
          ? `http://localhost:3000/articles/${form.id}` 
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        // Update UI after save
        if (form.id) {
          const index = articles.value.findIndex(article => article.id === form.id);
          articles.value.splice(index, 1, response.data);
        } else {
          articles.value.push(response.data);
        }
        
        resetForm();
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    function resetForm() {
      form.id = null;
      form.title = '';
      form.content = '';
    }

    onMounted(load);

    return { form, articles, save, edit, remove, resetForm, load };
  }
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
  background-color: #f4f4f9;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.form {
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

.form h2 {
  margin-bottom: 20px;
  color: #333;
}

.form label {
  display: block;
  margin: 10px 0 5px;
  font-weight: bold;
  color: #333;
}

.form input,
.form textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  transition: border-color 0.3s;
}

.form input:focus,
.form textarea:focus {
  border-color: #4caf50;
}

.form-buttons {
  display: flex;
  justify-content: flex-start;
  gap: 10px;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.save-btn {
  background-color: #4caf50;
  color: white;
}

.save-btn:hover {
  background-color: #45a049;
}

.cancel-btn {
  background-color: #f44336;
  color: white;
}

.cancel-btn:hover {
  background-color: #e53935;
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  background-color: #fff;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 10px;
  margin-bottom: 10px;
  transition: transform 0.2s;
}

.article-item:hover {
  transform: translateY(-5px);
}

.article-item h3 {
  margin: 0 0 10px;
  color: #333;
}

.article-item p {
  margin: 0 0 20px;
  color: #666;
}

.article-buttons {
  display: flex;
  justify-content: flex-start;
  gap: 10px;
}

.edit-btn {
  background-color: #2196f3;
  color: white;
}

.edit-btn:hover {
  background-color: #1e88e5;
}

.delete-btn {
  background-color: #f44336;
  color: white;
}

.delete-btn:hover {
  background-color: #e53935;
}

.load-btn {
  background-color: #2196f3;
  color: white;
  padding: 10px 20px;
  margin-top: 20px;
  display: block;
  width: 100%;
  text-align: center;
}

.load-btn:hover {
  background-color: #1e88e5;
}
</style>