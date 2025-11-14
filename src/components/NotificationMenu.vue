<template>
  <div>
    <q-btn round flat class="notification-btn" @click="toggleMenu">
      <q-icon name="notifications" size="24px" />
      <q-badge v-if="unreadCount > 0" :label="unreadCount" color="red" floating rounded />
    </q-btn>

    <q-menu v-model="menuOpen" anchor="top right" self="top right">
      <q-list style="min-width: 320px; max-width: 420px">
        <q-item-label header class="text-weight-bold">Notificações</q-item-label>
        <q-separator />

        <div v-if="loading" class="q-pa-md flex flex-center">
          <q-spinner color="primary" size="24px" />
        </div>

        <div v-else>
          <q-item v-if="!notifications.length">
            <q-item-section>
              <div class="text-grey">Sem notificações</div>
            </q-item-section>
          </q-item>

          <q-item v-for="(note, idx) in notifications" :key="note.id" clickable @click="acknowledge(note)">
            <q-item-section>
              <div class="text-subtitle2">Notificação {{ idx + 1 }} - {{ note.petName }} tem um cuidado para {{ note.whenLabel }}</div>
              <div class="text-caption text-grey-6">{{ note.displayDate }}</div>
            </q-item-section>
          </q-item>
        </div>

        <q-separator />
        <q-item v-if="notifications.length" clickable>
          <q-item-section side>
            <q-btn flat dense label="Marcar todas como lidas" @click.stop="acknowledgeAll" />
          </q-item-section>
        </q-item>
      </q-list>
    </q-menu>
  </div>
</template>

<script>
import { api } from 'boot/axios'
import { usePetStore } from 'src/stores/pet'
import { LocalStorage } from 'quasar'

export default {
  name: 'NotificationMenu',
  data() {
    return {
      menuOpen: false,
      notifications: [],
      loading: false,
      pollInterval: null,
    }
  },
  computed: {
    unreadCount() {
      return this.notifications.length
    }
  },
  mounted() {
    this.fetchNotifications()
    this.pollInterval = setInterval(() => this.fetchNotifications(), 2 * 60 * 1000)
  },
  beforeUnmount() {
    if (this.pollInterval) clearInterval(this.pollInterval)
  },
  methods: {
    toggleMenu() {
      this.menuOpen = !this.menuOpen
    },
    async fetchNotifications() {
      const petStore = usePetStore()
      const pets = petStore.pets || []
      if (!pets.length) return (this.notifications = [])

      this.loading = true
      const notifications = []
      try {
        for (const pet of pets) {
          const res = await api.get('/pet-care-history', { params: { pet_id: pet.id } })
          const data = Array.isArray(res.data) ? res.data : res.data?.rows || res.data?.data || []
          const cares = data.filter((c) => c.status === 'active')
          for (const care of cares) {
            const displayDate = new Date(care.care_date).toLocaleDateString('pt-BR')
            const whenLabel = this.getWhenLabel(care.care_date)
            notifications.push({ id: care.id, petName: pet.name, displayDate, whenLabel, careId: care.id })
          }
        }
        this.notifications = notifications.sort((a,b)=> new Date(a.displayDate) - new Date(b.displayDate))
      } catch (err) {
        console.error('Erro ao buscar notificações', err)
      } finally {
        this.loading = false
      }
    },
    acknowledge(note) {
      this.notifications = this.notifications.filter(n => n.careId !== note.careId)
    },
    acknowledgeAll() {
      this.notifications = []
    },
    daysFromNow(dateStr) {
      const today = new Date(); today.setHours(0,0,0,0)
      const d = new Date(dateStr); d.setHours(0,0,0,0)
      return Math.round((d - today) / (1000*60*60*24))
    },
    getWhenLabel(dateStr){
      const diff = this.daysFromNow(dateStr)
      if (diff===0) return 'hoje'
      if (diff>0 && diff<=7) return 'esta semana'
      if (diff>7 && diff<=14) return 'semana que vem'
      return new Date(dateStr).toLocaleDateString('pt-BR')
    }
  }
}
</script>

<style scoped>
.notification-btn {
  position: relative;
  color: white;
}
</style>
