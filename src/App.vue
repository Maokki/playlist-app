<script setup>

//App.vue
import { ref, computed } from 'vue'
import AuthForm from './AuthForm.vue'

// Authentication state (in-memory only)
const isAuthenticated = ref(false)
const user = ref('')

// Data state (in-memory only)
const data = ref([])
const playlists = ref(['Games', 'Drama', 'Books']) // Default playlists
const newPlaylist = ref('')
const input_content = ref('')
const input_playlist = ref('Games') // Default to first playlist

// Computed properties
const data_asc = computed(() => [...data.value].sort((a, b) => a.createdAt - b.createdAt))

// Group data by playlist
const groupedData = computed(() => {
  const groups = {}
  data_asc.value.forEach(item => {
    if (!groups[item.playlist]) {
      groups[item.playlist] = []
    }
    groups[item.playlist].push(item)
  })
  return groups
})

// Auth methods
const onAuthenticated = (username) => {
  isAuthenticated.value = true
  user.value = username
}

const logout = () => {
  isAuthenticated.value = false
  user.value = ''
  data.value = [] // Clear data on logout
}

// Data methods
const addData = () => {
  if (input_content.value.trim() === '' || !input_playlist.value) return
  
  data.value.push({
    content: input_content.value,
    playlist: input_playlist.value,
    done: false,
    createdAt: new Date().getTime()
  })
  
  input_content.value = ''
}

const removeData = (item) => {
  data.value = data.value.filter(d => d !== item)
}

const addPlaylist = () => {
  const trimmed = newPlaylist.value.trim()
  if (!trimmed) return
  
  if (!playlists.value.includes(trimmed)) {
    playlists.value.push(trimmed)
    input_playlist.value = trimmed
    newPlaylist.value = ''
  }
}

const deletePlaylist = (playlist) => {
  playlists.value = playlists.value.filter(p => p !== playlist)
  if (input_playlist.value === playlist) {
    input_playlist.value = playlists.value[0] || null
  }
  data.value = data.value.filter(item => item.playlist !== playlist)
}
</script>

<template>
  <!-- Authentication Screen -->
  <AuthForm 
    v-if="!isAuthenticated" 
    @authenticated="onAuthenticated" 
  />
  
  <!-- Main App Interface -->
  <main v-else class="app">
    <button class="logout-btn" @click="logout">
      Logout ({{ user }})
    </button>

    <!-- Greeting Section -->
    <section class="greeting">
      <h2 class="title">
        What's up, <input 
          type="text" 
          placeholder="Your name" 
          v-model="user"
        >
      </h2>
    </section>

    <h4>Select playlist</h4>
    <div class="options">
      <label v-for="playlist in playlists" :key="playlist">
        <input 
          type="radio" 
          name="playlist" 
          :value="playlist"
          v-model="input_playlist" 
        />
        <span :class="`bubble ${playlist.toLowerCase()}`"></span>
        <div>{{ playlist }}</div>
        <button 
          class="delete-playlist" 
          @click.stop="deletePlaylist(playlist)"
          title="Delete playlist"
          v-if="playlists.length > 1"
        >
          ×
        </button>
      </label>
    </div>

    <!-- Create Data Section -->
    <section class="create-data">
      <h3>ADD NEW ITEM</h3>
      
      <form @submit.prevent="addData">
        <h4>What do you want to add?</h4>
        <input 
          type="text" 
          placeholder="e.g. Games (ML)"
          v-model="input_content"
          required
        >

        <!-- Add New Playlist -->
        <div class="add-playlist">
          <h4>Create new playlist</h4>
          <div class="playlist-input">
            <input 
              type="text" 
              v-model="newPlaylist"
              placeholder="e.g. Movies"
              @keyup.enter="addPlaylist"
            />
            <button 
              type="button" 
              @click="addPlaylist"
              :disabled="!newPlaylist.trim()"
            >
              Add
            </button>
          </div>
        </div>

        <input 
          type="submit" 
          value="Add Item"
          :disabled="!input_content.trim() || !input_playlist"
        >
      </form>
    </section>

    <!-- Data List Section - Grouped by Playlist -->
    <section class="data-list">
      <h3>YOUR ITEMS</h3>
      <div 
        class="playlist-containers"
        v-if="data.length > 0"
      >
        <div 
          class="playlist-container" 
          v-for="(items, playlist) in groupedData" 
          :key="playlist"
        >
          <h4 class="playlist-header">
            <span :class="`bubble ${playlist.toLowerCase()}`"></span>
            {{ playlist }}
            <span class="item-count">({{ items.length }})</span>
          </h4>
          
          <div class="list">
            <div 
              v-for="item in items" 
              :key="item.createdAt"
              :class="`data-item ${item.done && 'done'}`"
            >
              <label>
                <input type="checkbox" v-model="item.done" />
                <span :class="`bubble ${item.playlist.toLowerCase()}`"></span>
              </label>

              <div class="data-content">
                <input type="text" v-model="item.content" />
              </div>

              <div class="actions">
                <button class="delete" @click="removeData(item)">
                  Delete
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div v-else class="empty-state">
        No items yet. Add your first item above!
      </div>
    </section>
  </main>
</template>