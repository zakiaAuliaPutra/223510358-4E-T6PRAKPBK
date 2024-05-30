<script>
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios'

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
        console.error('Error loading articles: ', error);
      }
    }

    async function save() {
      try {
        let url, method;
        if (form.id === null) {
          const lastArticle = articles.value[articles.value.length - 1];
          const newId = lastArticle ? parseInt(lastArticle.id) + 1 : 1;
          form.id = newId.toString(); 
          url = 'http://localhost:3000/articles/';
          method = 'post';
        } else {
          url = 'http://localhost:3000/articles/' + form.id;
          method = 'put';
        }
        const response = await axios[method](url, form);
        articles.value = articles.value.map((article) =>
          article.id === response.data.id ? response.data : article
        );
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.log('Error saving article: ', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article: ', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, remove, edit, load }; 
  },
};
</script>

<template>
  <q-layout view="lHh Lpr lFf">
    <q-header flat>
      <q-toolbar>
        <q-toolbar-title>Tugas 6 Praktikum PBK</q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page padding>
        <q-form @submit.prevent="save" class="q-gutter-md">
          <q-input filled v-model="form.title" label="Title" />
          <q-input filled v-model="form.content" label="Content" type="textarea" />
          <q-btn type="submit" label="Save" color="primary" />
        </q-form>
        
        <q-list bordered class="q-mt-md">
          <q-item v-for="article in articles" :key="article.id">
            <q-item-section>
              <q-item-label>{{ article.title }}</q-item-label>
              <q-item-label caption>{{ article.content }}</q-item-label>
            </q-item-section>
            <q-item-section side>
              <q-btn icon="edit" text-color="white" color="green" @click="edit(article)" />
            </q-item-section>
            <q-item-section side>
              <q-btn icon="delete" text-color="white" color="red" @click="remove(article.id)" />
            </q-item-section>
          </q-item>
        </q-list>
        
        <q-btn label="Load" @click="load" color="secondary" class="q-mt-md" />
      </q-page>
    </q-page-container>
  </q-layout>
</template>

