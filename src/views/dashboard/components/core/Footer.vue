<template>
  <v-footer id="dashboard-core-footer">
    <v-container>
      <v-row
        align="center"
        no-gutters
      >
        <v-col
          cols="12"
        >
          <div class="text-body-1 font-weight-light pt-6 pt-md-0 text-center">
            &copy; 2021 Chaingreen - All rights are reserved
          </div>
        </v-col>
      </v-row>
    </v-container>
  </v-footer>
</template>

<script>
  export default {
    name: 'DashboardCoreFooter',

    data: () => ({
      links: [],
    }),
  }
</script>

<style lang="sass">
#dashboard-core-footer
  a
    font-size: .825rem
    font-weight: 500
    text-decoration: none
    text-transform: uppercase
</style>
