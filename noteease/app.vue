<template>
  <div class="app-container">
    <!-- Top Bar -->
    <header class="top-bar">
      <input
        type="text"
        class="search-input"
        v-model="searchQuery"
        placeholder="Search notes..."
        @input="filterNotes"
        aria-label="Search notes"
      />
      <button class="create-btn" @click="createNote" aria-label="Create Note">
        + New Note
      </button>
    </header>

    <div class="main-content">
      <!-- Sidebar: Notes List -->
      <aside class="sidebar">
        <div class="notes-list">
          <div
            v-for="note in filteredNotes"
            :key="note.id"
            class="note-list-item"
            :class="{ active: note.id === selectedNoteId }"
            @click="selectNote(note.id)"
          >
            <div class="note-title">{{ note.title || '(Untitled Note)' }}</div>
            <div class="note-snippet">{{ note.content.slice(0, 40) || 'No content yet.' }}</div>
            <div class="note-category" v-if="note.category">
              #{{ note.category }}
            </div>
          </div>
          <div v-if="filteredNotes.length === 0" class="empty-list-msg">
            No notes found.
          </div>
        </div>
      </aside>

      <!-- Note Display & Actions -->
      <section class="note-detail" v-if="selectedNote">
        <input
          class="note-title-input"
          v-model="selectedNote.title"
          placeholder="Title"
          @change="saveNote"
        />
        <textarea
          class="note-content-input"
          v-model="selectedNote.content"
          placeholder="Start typing your note here..."
          @change="saveNote"
          rows="12"
        />
        <div class="note-actions">
          <input
            class="note-category-input"
            v-model="selectedNote.category"
            placeholder="Add category/tag..."
            @change="saveNote"
          />
          <button
            class="delete-btn"
            @click="deleteNote(selectedNote.id)"
            aria-label="Delete Note"
          >
            Delete
          </button>
        </div>
      </section>
      <section class="note-detail-empty" v-else>
        <div class="empty-note-text">
          Select a note or create a new one to get started.
        </div>
      </section>
    </div>
  </div>
</template>

<script setup>
// PUBLIC_INTERFACE
import { ref, computed, watch } from 'vue'

const COLOR_PRIMARY = '#4A90E2'
const COLOR_SECONDARY = '#F5F7FA'
const COLOR_ACCENT = '#F8E71C'

// Local state: notes array, selected note, search, etc.
const notes = ref([
  // Example initial note
  // { id: 1, title: 'First Note', content: 'This is a sample note.', category: 'Personal' },
])
const selectedNoteId = ref(null)
const searchQuery = ref('')

const filteredNotes = computed(() => {
  if (!searchQuery.value) return notes.value
  const q = searchQuery.value.trim().toLowerCase()
  return notes.value.filter(
    (note) =>
      (note.title && note.title.toLowerCase().includes(q)) ||
      (note.content && note.content.toLowerCase().includes(q)) ||
      (note.category && note.category.toLowerCase().includes(q))
  )
})

const selectedNote = computed({
  get() {
    return notes.value.find((n) => n.id === selectedNoteId.value) || null
  },
  set(updatedNote) {
    if (!updatedNote) return
    const idx = notes.value.findIndex((n) => n.id === updatedNote.id)
    if (idx !== -1) {
      notes.value[idx] = { ...updatedNote }
    }
  },
})

function filterNotes() {
  // Filtering is reactive via computed
}

function selectNote(noteId) {
  selectedNoteId.value = noteId
}

function createNote() {
  const newNote = {
    id: Date.now(),
    title: '',
    content: '',
    category: '',
  }
  notes.value.unshift(newNote)
  selectedNoteId.value = newNote.id
}

function saveNote() {
  // In a later step, sync with DB; here, just ref update.
  // The v-model and computed setter take care of updating the local note.
}

function deleteNote(noteId) {
  const idx = notes.value.findIndex((n) => n.id === noteId)
  if (idx !== -1) {
    notes.value.splice(idx, 1)
    if (selectedNoteId.value === noteId) {
      selectedNoteId.value = notes.value.length ? notes.value[0].id : null
    }
  }
}

// If no notes, encourage starting from a blank
watch(notes, (val) => {
  if (!val.length) selectedNoteId.value = null
})
</script>

<style scoped>
.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: #F5F7FA;
  font-family: "Segoe UI", Arial, sans-serif;
}

/* Top Bar */
.top-bar {
  display: flex;
  align-items: center;
  background: #fff;
  border-bottom: 2px solid #F5F7FA;
  padding: 0.5rem 1.5rem;
  height: 60px;
  z-index: 2;
  gap: 1rem;
}

.search-input {
  flex: 1;
  padding: 0.5rem 1.1rem;
  font-size: 1rem;
  border: 1.5px solid #e2e4ea;
  border-radius: 6px;
  background: #F5F7FA;
  color: #222;
  outline: none;
  transition: border-color 0.2s;
}
.search-input:focus {
  border-color: #4A90E2;
}

.create-btn {
  background: #4A90E2;
  color: #fff;
  border: none;
  padding: 0.55rem 1.3rem;
  font-weight: 600;
  border-radius: 7px;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.2s, color 0.2s;
}
.create-btn:hover {
  background: #3174ba;
}

/* Main Layout */
.main-content {
  display: flex;
  flex: 1;
  min-height: 0;
}

.sidebar {
  width: 270px;
  background: #F5F7FA;
  border-right: 1.5px solid #e2e4ea;
  overflow-y: auto;
  padding: 0.85rem 0.5rem 0.85rem 0.85rem;
}

.notes-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.note-list-item {
  background: #fff;
  border-radius: 7px;
  box-shadow: 0 1px 3px 0 #e8ecf3;
  cursor: pointer;
  padding: 0.7rem 1rem;
  user-select: none;
  border-left: 5px solid transparent;
  transition: border 0.15s, background 0.2s;
  margin-right: 0.5rem;
}
.note-list-item.active {
  border-left: 5px solid #4A90E2;
  background: #e0efff;
}
.note-title {
  font-weight: 600;
  font-size: 1.08rem;
  color: #4A90E2;
  margin-bottom: 0.12rem;
  line-height: 1.2;
}
.note-snippet {
  font-size: 0.92rem;
  color: #888fa2;
  margin-bottom: 0.05rem;
  line-height: 1.3;
}
.note-category {
  display: inline-block;
  font-size: 0.87rem;
  color: #F8E71C;
  background: #fffbe3;
  border-radius: 5px;
  padding: 0px 6px;
  margin-top: 2px;
}

/* Empty msg */
.empty-list-msg {
  color: #bbb;
  padding: 1.6rem 0 0 0.7rem;
  font-size: 1.05rem;
}

/* Note Detail section */
.note-detail, .note-detail-empty {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 2.4rem 2.2rem;
  justify-content: flex-start;
  min-width: 0;
  background: #fff;
}

.note-title-input {
  width: 100%;
  font-size: 1.3rem;
  font-weight: 600;
  border: none;
  border-bottom: 2.5px solid #F8E71C;
  background: transparent;
  margin-bottom: 1rem;
  padding-bottom: 0.23rem;
  outline: none;
  margin-top: -0.4rem;
  color: #222;
  transition: border-color 0.18s;
}
.note-title-input:focus {
  border-bottom-color: #4A90E2;
}

.note-content-input {
  width: 100%;
  padding: 0.8rem 1rem;
  font-size: 1rem;
  min-height: 170px;
  border: 1.5px solid #e2e4ea;
  border-radius: 8px;
  background: #F5F7FA;
  color: #222;
  resize: vertical;
  margin-bottom: 1.2rem;
  outline: none;
  transition: border-color 0.18s;
}
.note-content-input:focus {
  border-color: #4A90E2;
}

.note-actions {
  display: flex;
  align-items: center;
  gap: 1.1rem;
}
.note-category-input {
  padding: 0.35rem 0.8rem;
  border: 1.3px solid #e2e4ea;
  border-radius: 6px;
  background: #F5F7FA;
  font-size: 1rem;
  color: #555;
  min-width: 120px;
  outline: none;
  transition: border-color 0.18s;
}
.note-category-input:focus {
  border-color: #F8E71C;
}

.delete-btn {
  background: #fff;
  color: #4A90E2;
  border: 1.7px solid #4A90E2;
  padding: 0.37rem 1.1rem;
  border-radius: 7px;
  font-weight: 500;
  font-size: 1rem;
  margin-left: auto;
  cursor: pointer;
  transition: background 0.18s, color 0.18s;
}
.delete-btn:hover {
  background: #FF6961;
  color: #fff;
  border-color: #FF6961;
}

.note-detail-empty {
  align-items: center;
  justify-content: center;
  background: #fff;
}
.empty-note-text {
  color: #b8bac8;
  font-size: 1.13rem;
  text-align: center;
  font-style: italic;
}
</style>
