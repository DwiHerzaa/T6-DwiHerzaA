<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <strong>{{ article.title }}</strong><br />
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="button edit-button">Edit</button>
        <button @click="deleteArticle(article.id)" class="button delete-button">Delete</button>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load</button>
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
      content: ''
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

        if (form.id) {
          // Update existing article
          articles.value = articles.value.map(article =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          // Add new article
          articles.value.push(response.data);
        }

        // Reset form
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function deleteArticle(id) {
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

    onMounted(load);

    return { form, articles, save, edit, deleteArticle, load };
  }
};
</script>

<style scoped>
.container {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  font-family: Arial, sans-serif;
}

.form {
  margin-bottom: 20px;
}

.input, .textarea {
  width: 100%;
  padding: 8px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  box-sizing: border-box;
}

.textarea {
  min-height: 80px;
  resize: vertical;
}

.button {
  padding: 10px 15px;
  color: #fff;
  background-color: #007bff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  margin-right: 5px;
}

.button:hover {
  background-color: #0056b3;
}

.edit-button {
  background-color: #28a745;
}

.edit-button:hover {
  background-color: #218838;
}

.delete-button {
  background-color: #dc3545;
}

.delete-button:hover {
  background-color: #c82333;
}

.article-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.article-item {
  padding: 10px;
  border-bottom: 1px solid #ddd;
  background-color: #fafafa;
  border-radius: 4px;
  margin-bottom: 10px;
}

.article-item strong {
  display: block;
  font-size: 1.1em;
  margin-bottom: 5px;
}

.load-button {
  width: 100%;
  margin-top: 10px;
  background-color: #17a2b8;
}

.load-button:hover {
  background-color: #138496;
}
</style>
