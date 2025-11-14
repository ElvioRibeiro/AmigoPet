<template>
  <div>
    <q-menu v-model="menuOpen" anchor="top right" self="top right">
      <template v-slot:reference>
        <div style="display:flex; align-items:center">
          <q-btn round flat class="notification-btn" @click="openMenu">
            <q-icon name="notifications" size="24px" />
            <q-badge v-if="notifications.length > 0" :label="notifications.length" color="red" floating rounded />
          </q-btn>
          <!-- If permission is undecided, show a small action button to request it (user gesture) -->
          <q-btn v-if="notificationPermission === 'default'" dense flat small class="q-ml-sm" @click.stop="askNotificationPermission" aria-label="Ativar notificações">
            <q-icon name="notifications_active" size="20px" />
          </q-btn>
        </div>
      </template>

      <q-list style="min-width: 320px; max-width: 420px">
        <q-item-label header class="text-weight-bold">Notificações</q-item-label>
        <q-separator />

        <div v-if="loading" class="q-pa-md flex flex-center">
          <q-spinner color="primary" size="24px" />
        </div>

        <div v-else>
          <q-item v-if="!notifications.length" clickable>
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

        <!-- Show an action to enable notifications when permission is default -->
        <q-item clickable v-if="notificationPermission === 'default'">
          <q-item-section side>
            <q-btn flat dense label="Ativar notificações" @click.stop="askNotificationPermission" />
          </q-item-section>
        </q-item>

        <q-item clickable v-if="notifications.length">
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
import { LocalStorage } from 'quasar'
import { usePetStore } from 'src/stores/pet'

export default {
  name: 'NotificationMenu',
  data() {
    return {
      menuOpen: false,
      notifications: [],
      loading: false,
      pollInterval: null,
      notificationPermission: 'default', // 'granted' | 'denied' | 'default' | 'unsupported'
    }
  },
  computed: {
    unreadCount() {
      return this.notifications.length
    },
  },
  mounted() {
    // initialize permission state without requesting permission
    if ('Notification' in window) {
      this.notificationPermission = Notification.permission
    } else {
      this.notificationPermission = 'unsupported'
    }

    this.fetchNotifications()
    // poll every 2 minutes
    this.pollInterval = setInterval(() => this.fetchNotifications(), 2 * 60 * 1000)
  },
  beforeUnmount() {
    if (this.pollInterval) clearInterval(this.pollInterval)
  },
  methods: {
    openMenu() {
      this.menuOpen = !this.menuOpen
    },
    async askNotificationPermission() {
      // must be called from a user gesture (click)
      try {
        if (!('Notification' in window)) {
          this.notificationPermission = 'unsupported'
          this.$q.notify({ message: 'Notificações não são suportadas pelo navegador.', color: 'negative' })
          return
        }

        const perm = await Notification.requestPermission()
        this.notificationPermission = perm

        if (perm === 'granted') {
          // show browser notifications for current items (that haven't been notified)
          const notifiedIds = LocalStorage.getItem('@amigopet-notified-care-ids') || []
          for (const note of this.notifications) {
            if (!notifiedIds.includes(note.careId)) {
              try {
                new Notification(`${note.petName} - lembrete`, {
                  body: `Tem um cuidado para ${note.whenLabel} (${note.displayDate})`,
                  tag: `amigopet-care-${note.careId}`,
                })
                notifiedIds.push(note.careId)
              } catch (err) {
                console.error('Erro ao exibir notificação após permissão', err)
              }
            }
          }
          LocalStorage.set('@amigopet-notified-care-ids', notifiedIds)
        } else if (perm === 'denied') {
          this.$q.notify({ message: 'Permissão de notificações negada. Você pode habilitar nas configurações do navegador.', color: 'warning' })
        }
      } catch (err) {
        console.error('Erro ao solicitar permissão de notificações', err)
      }
    },
    async fetchNotifications() {
      const petStore = usePetStore()
      const pets = petStore.pets || []
      if (!pets.length) return (this.notifications = [])

      this.loading = true
      const notifiedIds = LocalStorage.getItem('@amigopet-notified-care-ids') || []
      const notifications = []

      try {
        // Fetch care history for each pet and aggregate
        for (const pet of pets) {
          try {
            const res = await api.get('/pet-care-history', { params: { pet_id: pet.id } })
            const data = Array.isArray(res.data) ? res.data : res.data?.rows || res.data?.data || []
            const cares = data.filter((c) => c.status === 'active')

            for (const care of cares) {
              const whenLabel = this.getWhenLabel(care.care_date)
              // Build a friendly display date
              const displayDate = new Date(care.care_date).toLocaleDateString('pt-BR')

              // Create a simple unique id per care
              const id = `care-${care.id}`

              // Only show cares that are today or within the next 14 days (configurable rule)
              const daysDiff = this.daysFromNow(care.care_date)
              if (daysDiff < 0 || daysDiff > 14) continue

              // Include even if already notified — user may want to see them; but we keep track to avoid re-notifying browser notifications
              notifications.push({ id, petName: pet.name, whenLabel, displayDate, careId: care.id })
            }
          } catch (err) {
            // ignore per-pet errors but continue
            console.error('Erro ao buscar cuidados do pet', pet.id, err)
          }
        }

        // Sort by soonest
        notifications.sort((a, b) => new Date(a.displayDate) - new Date(b.displayDate))

        this.notifications = notifications

        // Only trigger browser notifications if permission already granted
        if (this.notificationPermission === 'granted') {
          for (const note of notifications) {
            if (!notifiedIds.includes(note.careId)) {
              try {
                new Notification(`${note.petName} - lembrete`, {
                  body: `Tem um cuidado para ${note.whenLabel} (${note.displayDate})`,
                  tag: `amigopet-care-${note.careId}`,
                })
                notifiedIds.push(note.careId)
              } catch (err) {
                console.error('Erro ao mostrar notificação', err)
              }
            }
          }

          LocalStorage.set('@amigopet-notified-care-ids', notifiedIds)
        }
      } catch (err) {
        console.error('Erro ao agregar notificações', err)
      } finally {
        this.loading = false
      }
    },
    daysFromNow(dateStr) {
      const today = new Date()
      today.setHours(0, 0, 0, 0)
      const d = new Date(dateStr)
      d.setHours(0, 0, 0, 0)
      const diff = Math.round((d - today) / (1000 * 60 * 60 * 24))
      return diff
    },
    getWhenLabel(dateStr) {
      const diff = this.daysFromNow(dateStr)
      if (diff === 0) return 'hoje'
      if (diff > 0 && diff <= 7) return 'esta semana'
      if (diff > 7 && diff <= 14) return 'semana que vem'
      return new Date(dateStr).toLocaleDateString('pt-BR')
    },
    acknowledge(note) {
      // mark single notification as read for UI
      this.notifications = this.notifications.filter((n) => n.careId !== note.careId)
      // also persist acknowledged so browser notifications won't be re-sent (we already push to LocalStorage when showing browser notification)
    },
    acknowledgeAll() {
      this.notifications = []
    },
  },
}
</script>

<style scoped>
.notification-btn {
  position: relative;
  color: white; /* ensure icon/button is visible on purple header */
  display: inline-flex;
  align-items: center;
  z-index: 1100; /* bring to front */
  margin-right: 8px;
}
.notification-btn :deep(.q-icon),
.notification-btn :deep(.q-badge) {
  color: white !important;
}
.notification-btn :deep(.q-badge) {
  transform: translate(6px, -6px);
}
/* increase click area and keep it aligned */
.notification-btn { padding: 6px; }
</style>
