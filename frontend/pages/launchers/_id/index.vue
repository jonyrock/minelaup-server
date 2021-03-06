<template>
  <div v-if="launcher" class="flex flex-col">
    <div class="h-32 bg-gray-200 dark:bg-gray-600 px-10 py-5">
      <div class="flex flex-row">
        <h1
          class="font-bold uppercase text-3xl text-gray-900 dark:text-white flex-1"
        >
          {{ launcher.name }}
        </h1>
        <div
          v-if="launcher.userPerms.owner || launcher.userPerms.manage_modpack"
        >
          <i
            class="fas text-gray-700 dark:text-white hover:text-green-400 cursor-pointer mr-2 text-xl"
            :class="{
              'fa-toggle-on': !launcher.disabled,
              'fa-toggle-off': launcher.disabled,
            }"
            @click="updateLauncherState(launcher.id, launcher.disabled)"
          />
          <nuxt-link :to="'/launchers/' + $route.params.id + '/edit'">
            <i class="fas fa-pen hover:text-green-400"></i>
          </nuxt-link>
          <i
            class="fas fa-trash text-gray-900 dark:text-white hover:text-red-500 cursor-pointer ml-1"
            @click="openDeleteLauncherModal"
          ></i>
        </div>
      </div>
      <div class="mx-8">
        <p class="text-gray-800">
          {{ $t('pages.launchers.index.team_name', [launcher.team.name]) }}
        </p>
        <p class="text-gray-800 italic">
          {{ launcher.summary }}
        </p>
      </div>
    </div>
    <div class="flex-1">
      <div class="p-10">
        <t-input
          id="api_key"
          class="w-2/3 mb-4 select-text"
          :label="$t('pages.launchers.view.api_key')"
          type="password"
          autocomplete="off"
          :disabled="true"
          :value="launcher.api_key"
          icon="key"
        />
        <t-button @click="regenerateKey">
          {{ $t('pages.launchers.view.regenerate_key') }}
        </t-button>
      </div>

      <div class="p-10">
        <div
          class="flex flex-col border border-gray-800 shadow-lg rounded-md overflow-hidden"
        >
          <div class="flex flex-row justify-between bg-gray-800 text-white p-4">
            <h1 class="uppercase font-bold">
              {{ $t('pages.launchers.view.assigned_modpack') }}
            </h1>
            <div>
              <i
                class="fas fa-plus-circle hover:text-green-400 cursor-pointer"
                @click="openSearchModpackModal"
              ></i>
            </div>
          </div>
          <div
            v-for="modpack in launcher.modpacks"
            :key="modpack.id"
            class="border-b-gray-600 last:border-b-0 p-4 hover:bg-gray-100 dark:bg-gray-600 dark-hover:bg-gray-500 transition-colors duration-75 flex flex-row justify-between"
          >
            <span>
              {{ modpack.name }}
            </span>
            <span>
              <i
                class="fas text-gray-700 dark:text-white hover:text-green-400 cursor-pointer mr-2 text-xl"
                :class="{
                  'fa-toggle-on': !modpack.disabled,
                  'fa-toggle-off': modpack.disabled,
                }"
                @click="updateModpackState(modpack.id, modpack.disabled)"
              ></i>
              <i
                class="fas fa-trash hover:text-red-500 dark:text-white text-gray-900 cursor-pointer"
                @click="deleteModpack(modpack.id)"
              ></i>
            </span>
          </div>
          <div
            v-if="Object.keys(launcher.modpacks).length === 0"
            class="p-4 hover:bg-gray-100 dark:bg-gray-600 dark-hover:bg-gray-500 transition-colors duration-75"
          >
            {{ $t('pages.launchers.view.no_modpacks') }}
          </div>
        </div>
      </div>
    </div>

    <t-modal
      v-if="Object.keys(deleteModal).length > 0"
      @close-modal="deleteModal = {}"
    >
      <h1 slot="title">{{ $t(deleteModal.title) }}</h1>
      <div slot="actions" class="flex flex-col">
        <t-button
          class="mb-2"
          bg-hover-color="red-500"
          dark-bg-hover-color="red-500"
          @click="deleteLauncher"
        >
          {{ $t('components.modal.yes') }}
        </t-button>
        <t-button
          class="mb-2"
          bg-hover-color="gray-900"
          dark-color="white"
          dark-bg-hover-color="white"
          dark-hover-color="gray-900"
          @click="deleteModal = {}"
        >
          {{ $t('components.modal.no') }}
        </t-button>
      </div>
    </t-modal>

    <t-modal
      v-if="Object.keys(searchModal).length > 0"
      @close-modal="searchModal = {}"
    >
      <h1 slot="title">{{ $t(searchModal.title) }}</h1>
      <div slot="actions" class="flex flex-col">
        <t-input
          id="searchModpack"
          v-model="searchField"
          class="mb-4"
          icon="search"
          :label="$t('pages.launchers.view.search_modpack')"
          @input="searchField"
        />

        <div class="border border-gray-400 m-5">
          <div
            v-for="modpack in modpacks"
            :key="modpack.id"
            class="p-4 flex flex-row justify-between hover:bg-gray-200 dark-hover:bg-gray-700 border-b border-gray-300 last:border-b-0"
          >
            <span>
              {{ modpack.name }}
            </span>
            <span>
              <i
                class="fas fa-plus hover:text-green-400 transition-colors duration-75 cursor-pointer"
                @click="addModpack(modpack.id, modpack.disabled)"
              ></i>
            </span>
          </div>
          <div v-if="modpacks.length === 0" class="p-4">
            {{ $t('pages.launchers.view.no_modpack_found') }}
          </div>
        </div>

        <t-button
          class="mb-2"
          bg-hover-color="gray-900"
          dark-color="white"
          dark-bg-hover-color="white"
          dark-hover-color="gray-900"
          @click="searchModal = {}"
        >
          {{ $t('components.modal.close') }}
        </t-button>
      </div>
    </t-modal>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'nuxt-property-decorator'
import debounce from 'lodash/debounce'
import TButton from '~/components/forms/TButton.vue'
import TModal from '~/components/bases/TModal.vue'
import TInput from '~/components/forms/TInput.vue'

@Component({
  components: {
    TButton,
    TModal,
    TInput,
  },
})
export default class LauncherViewIndex extends Vue {
  launcher: Partial<any> | null = null
  modpacks: Array<Partial<any>> = []

  deleteModal: Partial<any> = {}
  searchModal: Partial<any> = {}

  searchField: string = ''

  async fetch() {
    // Fetch launcher informations
    const launcher = await this.$axios
      .$get(`/api/launcher/${this.$route.params.id}`)
      .catch(() => {
        this.$router.replace('/launchers')
      })

    this.launcher = launcher
  }

  // open the delete launcher modal confirmation
  openDeleteLauncherModal() {
    this.deleteModal = {
      title: 'pages.launchers.index.confirm_delete',
    }
  }

  // open the search modpack modal
  async openSearchModpackModal() {
    this.searchModal = {
      title: 'pages.launchers.view.add_modpacks',
    }

    const modpacks = await this.$axios.$get('/api/launchers/list-modpacks', {
      params: {
        // eslint-disable-next-line camelcase
        teamId: this.launcher?.team_id,
      },
    })

    this.modpacks = modpacks
  }

  searchModpacks = debounce(async () => {
    this.modpacks = await this.$axios
      .$get('/api/launchers/list-modpacks', {
        params: {
          // eslint-disable-next-line camelcase
          teamId: this.launcher?.team_id,
          search: this.searchField,
        },
      })
      .catch((error) => {
        // eslint-disable-next-line no-console
        console.error(error)
      })
  }, 250)

  // Add modpack to the launcher list
  addModpack(id: number) {
    this.$axios
      .$post(`/api/launcher/${this.$route.params.id}/modpacks`, {
        id,
      })
      .then(async () => {
        await this.$fetch()
        this.searchModal = {}
      })
  }

  // Add modpack to the launcher list
  deleteModpack(id: number) {
    this.$axios
      .$delete(`/api/launcher/${this.$route.params.id}/modpacks`, {
        params: {
          id,
        },
      })
      .then(async () => {
        await this.$fetch()
      })
  }

  // Update modpack state (disabled/enabled)
  updateModpackState(id: number, state: boolean) {
    this.$axios
      .$put(`/api/modpack/${id}/state`, {
        state,
      })
      .then(async () => {
        await this.$fetch()
      })
  }

  // Update launcher state (disabled/enabled)
  updateLauncherState(id: number, state: boolean) {
    this.$axios
      .$put(`/api/launcher/${id}/state`, {
        state,
      })
      .then(async () => {
        await this.$fetch()
      })
  }

  // Delete launcher action
  deleteLauncher() {
    // Request the APi to delete the modpack
    this.$axios
      .delete(`/api/launcher/${this.$route.params.id}`)
      .then(async () => {
        // On success, fetch teams list and update it
        let teams = await this.$axios.$get('/api/launchers')

        teams = teams.filter((team: Partial<any>) => {
          return team.launchers?.length
        })

        let launchers: Array<Partial<any>> = []

        teams.forEach((team: Partial<any>) => {
          launchers = launchers.concat(
            team.launchers.map((launcher: Partial<any>) => ({
              name: launcher.name,
              path: `/launchers/${launcher.id}`,
            }))
          )
        })

        if (teams) {
          this.$store.commit('menu/setList', launchers)
        }

        // Finally redirect the user to the main route
        this.$router.push('/launchers')
      })
      .catch((error) => {
        // If failed, log the error in the console
        // eslint-disable-next-line
        console.error(error)
      })
  }

  regenerateKey() {
    this.$axios
      .$post(`/api/launcher/${this.$route.params.id}/regenerate`)
      .then(() => {
        // On success, refresh datas
        this.$fetch()
      })
      .catch((error) => {
        // If failed, log the error in the console
        // eslint-disable-next-line
        console.error(error)
      })
  }

  @Watch('$route')
  onRouteChange(current: Partial<any>) {
    // On route change, if the page name still the same, then refetch team informations
    if (current.name === 'launchers-id') {
      this.$fetch()
    }
  }
}
</script>
