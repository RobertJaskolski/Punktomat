<template>
  <v-app>
    <v-app-bar
      class="elevation-0"
      color="white"
      max-height="64px"
      height="64px"
    >
      <v-toolbar-title v-if="!$vuetify.breakpoint.xs"
        >Punktomat</v-toolbar-title
      >

      <v-spacer v-if="!$vuetify.breakpoint.xs"></v-spacer>

      <v-toolbar dense max-width="720px" class="rounded-lg elevation-2">
        <v-tooltip bottom :open-delay="200" transition="fade-transition">
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              icon
              v-bind="attrs"
              v-on="on"
              @click="searchBtnClicked()"
              class="mr-n3"
              style="z-index: 1"
            >
              <v-icon>mdi-magnify</v-icon>
            </v-btn>
          </template>
          <span>Wyszukaj</span>
        </v-tooltip>
        <v-text-field
          flat
          solo
          clearable
          single-line
          hide-details
          placeholder="Wyszukaj"
          v-model="searchText"
        ></v-text-field>
        <v-tooltip bottom :open-delay="200" transition="fade-transition">
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              icon
              v-bind="attrs"
              v-on="on"
              @click.stop="filtersDialog = true"
              class="mr-n3"
            >
              <v-icon>mdi-filter-outline</v-icon>
            </v-btn>
          </template>
          <span>Filtrowanie</span>
        </v-tooltip>
        <v-dialog
          v-model="filtersDialog"
          max-width="900"
          @click:outside="applyFilters"
        >
          <v-card>
            <v-card-title class="headline">
              Filtrowanie
            </v-card-title>

            <v-subheader>Wybierz przedział punktowy</v-subheader>

            <v-card-text>
              <v-range-slider
                v-model="range"
                min="20"
                max="200"
                step="10"
                thumb-label="always"
                append-icon="mdi-thumb-up-outline"
                prepend-icon="mdi-thumb-down-outline"
                class="mt-8 mb-n8"
                :value="this.range"
              ></v-range-slider>
            </v-card-text>

            <v-subheader>Wybierz Kategorie</v-subheader>

            <v-card-text>
              <v-chip-group v-model="filters" column multiple class="mb-n4">
                <v-chip
                  v-for="chip in allCategories"
                  v-bind:key="chip"
                  filter
                  outlined
                >
                  {{ chip }}
                </v-chip>
              </v-chip-group>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>

              <v-btn
                text
                @click="
                  filtersDialog = false;
                  applyFilters();
                "
                >Zatwierdź</v-btn
              >
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>

      <v-spacer v-if="!$vuetify.breakpoint.xs"></v-spacer>

      <v-tooltip bottom :open-delay="200" transition="fade-transition">
        <template v-slot:activator="{ on, attrs }">
          <v-btn icon v-bind="attrs" v-on="on" class="ml-2">
            <!-- <v-icon>mdi-pdf-box</v-icon> -->
            <v-icon>mdi-file-download-outline</v-icon>
          </v-btn>
        </template>
        <span>Zapisz zaznaczone do PDF</span>
      </v-tooltip>

      <v-tooltip bottom :open-delay="200" transition="fade-transition">
        <template v-slot:activator="{ on, attrs }">
          <v-btn icon v-bind="attrs" v-on="on">
            <v-icon>mdi-dots-vertical</v-icon>
          </v-btn>
        </template>
        <span>Więcej opcji</span>
      </v-tooltip>
    </v-app-bar>

    <v-main class="overflow-y-auto pa-0">
      <Table
        v-bind:loading="loading"
        v-bind:magazines="magazines"
        v-bind:totalMagazines="totalMagazines"
        @optionsChanged="tableOptionsChanged"
        @selectionChanged="tableSelectionChanged"
      />
    </v-main>
  </v-app>
</template>

<script>
import axios from "axios";
import Table from "./Table";

export default {
  name: "Main",

  components: { Table },

    mounted () {
      this.getDataFromApi()
      console.log(this.options.params)
    },

    methods: {
      getDataFromApi () {
        this.loading = true
        axios
          .post(`${process.env.VUE_APP_API_URL}/scienceMagazine`, this.options.data)
          .then(response => {
            this.magazines = response.data.results
            this.totalMagazines = response.data.total
            this.loading = false
          })
      },
      searchBtnClicked () {
        this.searchText = this.searchText.trim().replace(/\s+/g, ' ')
        // let q = this.searchText.trim().replace(/\s+/g, ' ')
        this.options.data.offset = 0
        this.options.data.search = this.searchText
        this.getDataFromApi()
      },
      applyFilters () {
        this.options.data.minPoints = this.range[0]
        this.options.data.maxPoints = this.range[1]
        this.getDataFromApi()
      },
      clearFilters () {
        this.options.data.categories = []
        this.options.data.minPoints = 20
        this.options.data.maxPoints = 200
        this.getDataFromApi()
      },
      tableOptionsChanged (op) {
        const options = JSON.parse(JSON.stringify(op))

        const lim = options.itemsPerPage
        this.options.data.limit = lim
        this.options.data.offset = (options.page - 1) * lim
        this.getDataFromApi()
      },
      tableSelectionChanged (selected) {
        this.selected = selected
        localStorage.setItem('starred', JSON.stringify(this.selected))
      },
    },
    tableOptionsChanged(op) {
      const options = JSON.parse(JSON.stringify(op));
      const lim = options.itemsPerPage;

      this.options.data.limit = lim;
      this.options.data.offset = (options.page - 1) * lim;
      this.getDataFromApi();
    },
    tableSelectionChanged(selected) {
      this.selected = selected;
      localStorage.setItem("starred", JSON.stringify(this.selected));
    },
  },
};
</script>

<style lang="scss">
@import "~/src/sass/variables.scss";

html {
  overflow-y: hidden;
}
</style>
