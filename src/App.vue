<template>
  <v-app>
    <!-- Navigation Drawer -->
    <v-navigation-drawer v-model="drawer" app>
      <v-list>
        <v-list-item>
          <v-list-item-avatar>
            <v-img :src="config.ui_settings.logo_url" alt="Logo" />
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title class="text-h6">{{ config.ui_settings.company_name }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>

        <v-divider></v-divider>

        <v-list-item
          v-for="(item, i) in menuItems"
          :key="i"
          :to="item.to"
          link
        >
          <v-list-item-icon>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <!-- App Bar -->
    <v-app-bar app color="primary" dark elevation="4">
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>{{ config.ui_settings.company_name }}</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn icon @click="theme.global.name.value = theme.global.current.value.dark ? 'light' : 'dark'">
        <v-icon>{{ theme.global.current.value.dark ? 'mdi-weather-sunny' : 'mdi-weather-night' }}</v-icon>
      </v-btn>
    </v-app-bar>

    <!-- Main Content -->
    <v-main class="grey lighten-3">
      <v-container fluid>
        <v-row>
          <!-- Single Image Processing -->
          <v-col cols="12" v-if="config.ui_settings.show_single_image">
            <v-card class="mb-4" elevation="2">
              <v-card-title class="text-h6">
                <v-icon left color="primary">mdi-image</v-icon>
                Process Single Image
              </v-card-title>
              <v-card-text>
                <v-file-input
                  v-model="singleFile"
                  accept="image/*"
                  label="Select an image"
                  prepend-icon="mdi-camera"
                  show-size
                  truncate-length="25"
                  @change="processSingleImage"
                ></v-file-input>
              </v-card-text>
            </v-card>
          </v-col>

          <!-- Multiple Images Processing -->
          <v-col cols="12" v-if="config.ui_settings.show_multiple_images">
            <v-card class="mb-4" elevation="2">
              <v-card-title class="text-h6">
                <v-icon left color="primary">mdi-image-multiple</v-icon>
                Process Multiple Images
              </v-card-title>
              <v-card-text>
                <v-file-input
                  v-model="multipleFiles"
                  accept="image/*"
                  label="Select multiple images"
                  prepend-icon="mdi-folder-multiple-image"
                  show-size
                  truncate-length="25"
                  multiple
                  @change="processMultipleImages"
                ></v-file-input>
              </v-card-text>
            </v-card>
          </v-col>

          <!-- Folder Processing -->
          <v-col cols="12" v-if="config.ui_settings.show_folder_processing">
            <v-card class="mb-4" elevation="2">
              <v-card-title class="text-h6">
                <v-icon left color="primary">mdi-folder</v-icon>
                Process Folder
              </v-card-title>
              <v-card-text>
                <v-text-field
                  v-model="folderPath"
                  label="Folder Path"
                  placeholder="/path/to/folder"
                  prepend-icon="mdi-folder-open"
                  @keyup.enter="processFolder"
                ></v-text-field>
                <v-btn
                  color="primary"
                  block
                  :loading="loading"
                  @click="processFolder"
                  class="mt-2"
                >
                  Process Folder
                </v-btn>
              </v-card-text>
            </v-card>
          </v-col>

          <!-- S3 Processing -->
          <v-col cols="12" v-if="config.ui_settings.show_s3_processing">
            <v-card class="mb-4" elevation="2">
              <v-card-title class="text-h6">
                <v-icon left color="primary">mdi-cloud</v-icon>
                Process S3 Images
              </v-card-title>
              <v-card-text>
                <v-text-field
                  v-model="s3Url"
                  label="S3 URL"
                  placeholder="s3://bucket-name/path/to/images"
                  prepend-icon="mdi-cloud"
                  @keyup.enter="processS3Images"
                ></v-text-field>
                <v-btn
                  color="primary"
                  block
                  :loading="loading"
                  @click="processS3Images"
                  class="mt-2"
                >
                  Process S3 Images
                </v-btn>
              </v-card-text>
            </v-card>
          </v-col>

          <!-- Results Table -->
          <v-col cols="12" v-if="results.length > 0">
            <v-card elevation="2">
              <v-card-title class="text-h6">
                <v-icon left color="success">mdi-check-circle</v-icon>
                Results
                <v-spacer></v-spacer>
                <v-btn
                  color="success"
                  @click="downloadCSV"
                  :disabled="!csvData"
                  class="ml-2"
                >
                  <v-icon left>mdi-download</v-icon>
                  Download CSV
                </v-btn>
              </v-card-title>
              <v-card-text>
                <v-data-table
                  :headers="headers"
                  :items="results"
                  :items-per-page="10"
                  class="elevation-1"
                  :search="search"
                >
                  <template v-slot:top>
                    <v-text-field
                      v-model="search"
                      label="Search"
                      prepend-icon="mdi-magnify"
                      class="mx-4"
                    ></v-text-field>
                  </template>
                </v-data-table>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <!-- Snackbar for notifications -->
    <v-snackbar
      v-model="snackbar.show"
      :color="snackbar.color"
      :timeout="3000"
    >
      {{ snackbar.text }}
      <template v-slot:actions>
        <v-btn
          variant="text"
          @click="snackbar.show = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-app>
</template>

<script>
import axios from 'axios';
import config from './config.json';
import { useTheme } from 'vuetify';

export default {
  name: 'App',
  setup() {
    const theme = useTheme();
    return { theme };
  },
  data: () => ({
    config: config,
    drawer: true,
    singleFile: null,
    multipleFiles: [],
    folderPath: '',
    s3Url: '',
    loading: false,
    results: [],
    csvData: null,
    search: '',
    snackbar: {
      show: false,
      text: '',
      color: 'success'
    },
    menuItems: [
      { title: 'Dashboard', icon: 'mdi-view-dashboard', to: '/' },
      { title: 'History', icon: 'mdi-history', to: '/history' },
      { title: 'Settings', icon: 'mdi-cog', to: '/settings' }
    ],
    headers: [
      { title: 'Filename', key: 'filename', align: 'start' },
      { title: 'Date', key: 'date' },
      { title: 'Payee', key: 'payee' },
      { title: 'Amount (Words)', key: 'amount_words' },
      { title: 'Amount (Digits)', key: 'amount_digits' },
    ],
  }),
  methods: {
    showNotification(text, color = 'success') {
      this.snackbar.text = text;
      this.snackbar.color = color;
      this.snackbar.show = true;
    },
    
    async processSingleImage() {
      if (!this.singleFile) return;
      
      this.loading = true;
      try {
        const formData = new FormData();
        formData.append('file', this.singleFile);
        
        const response = await axios.post('http://35.193.72.57:8000/process-single', formData, {
          headers: { 'Content-Type': 'multipart/form-data' },
        });
        
        this.results = [{
          filename: response.data.filename,
          date: response.data.date,
          payee: response.data.payee,
          amount_words: response.data.amount_words,
          amount_digits: response.data.amount_digits
        }];
        
        const csvContent = [
          ['filename', 'date', 'payee', 'amount_words', 'amount_digits'],
          [
            response.data.filename,
            response.data.date,
            response.data.payee,
            response.data.amount_words,
            response.data.amount_digits
          ]
        ].map(row => row.join(',')).join('\n');
        
        this.csvData = new Blob([csvContent], { type: 'text/csv' });
        this.showNotification('Image processed successfully');
      } catch (error) {
        console.error('Error processing single image:', error);
        this.showNotification('Error processing image. Please try again.', 'error');
      } finally {
        this.loading = false;
      }
    },
    
    async processMultipleImages() {
      if (!this.multipleFiles.length) return;
      
      this.loading = true;
      try {
        const formData = new FormData();
        this.multipleFiles.forEach(file => {
          formData.append('files', file);
        });
        
        const response = await axios.post('http://35.193.72.57:8000/process-folder', formData, {
          headers: { 'Content-Type': 'multipart/form-data' },
          responseType: 'blob',
        });
        
        this.csvData = response.data;
        const reader = new FileReader();
        reader.onload = (e) => {
          const text = e.target.result;
          const rows = text.split('\n').map(row => row.split(','));
          this.results = rows.slice(1).map(row => ({
            filename: row[0],
            date: row[1],
            payee: row[2],
            amount_words: row[3],
            amount_digits: row[4]
          }));
        };
        reader.readAsText(response.data);
        this.showNotification('Multiple images processed successfully');
      } catch (error) {
        console.error('Error processing multiple images:', error);
        this.showNotification('Error processing images. Please try again.', 'error');
      } finally {
        this.loading = false;
      }
    },
    
    async processFolder() {
      if (!this.folderPath) return;
      
      this.loading = true;
      try {
        const response = await axios.post('http://35.193.72.57:8000/process-folder-path', {  
          folder_path: this.folderPath
        }, {
          responseType: 'blob'
        });
        
        this.csvData = response.data;
        const reader = new FileReader();
        reader.onload = (e) => {
          const text = e.target.result;
          const rows = text.split('\n').map(row => row.split(','));
          this.results = rows.slice(1).map(row => ({
            filename: row[0],
            date: row[1],
            payee: row[2],
            amount_words: row[3],
            amount_digits: row[4]
          }));
        };
        reader.readAsText(response.data);
        this.showNotification('Folder processed successfully');
      } catch (error) {
        console.error('Error processing folder:', error);
        this.showNotification('Error processing folder. Please try again.', 'error');
      } finally {
        this.loading = false;
      }
    },
    
    async processS3Images() {
      if (!this.s3Url) return;
      
      this.loading = true;
      try {
        const response = await axios.post('http://35.193.72.57:8000/process-s3', {
          s3_url: this.s3Url,
        }, {
          responseType: 'blob',
        });
        
        this.csvData = response.data;
        const reader = new FileReader();
        reader.onload = (e) => {
          const text = e.target.result;
          const rows = text.split('\n').map(row => row.split(','));
          this.results = rows.slice(1).map(row => ({
            filename: row[0],
            date: row[1],
            payee: row[2],
            amount_words: row[3],
            amount_digits: row[4]
          }));
        };
        reader.readAsText(response.data);
        this.showNotification('S3 images processed successfully');
      } catch (error) {
        console.error('Error processing S3 images:', error);
        this.showNotification('Error processing S3 images. Please try again.', 'error');
      } finally {
        this.loading = false;
      }
    },
    
    downloadCSV() {
      if (!this.csvData) return;
      
      const url = window.URL.createObjectURL(this.csvData);
      const link = document.createElement('a');
      link.href = url;
      link.setAttribute('download', 'results.csv');
      document.body.appendChild(link);
      link.click();
      link.remove();
      this.showNotification('CSV downloaded successfully');
    },
  },
};
</script>

<style>
.v-application {
  font-family: 'Roboto', sans-serif;
}

.v-card {
  transition: all 0.3s ease-in-out;
}

.v-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1) !important;
}

.theme--dark.v-application {
  background: #121212 !important;
}

.v-data-table {
  border-radius: 8px;
}
</style>