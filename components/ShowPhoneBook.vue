<template>
  <div>
    <v-card v-if="phoneBook" class="mx-auto my-12" max-width="374">
      <template slot="progress">
        <v-progress-linear
          color="deep-purple"
          height="10"
          indeterminate
        ></v-progress-linear>
      </template>

      <v-img height="250" :src="pathImage + phoneBook.image"></v-img>

      <v-card-title>{{ phoneBook.name }}</v-card-title>

      <v-card-text>
        <v-row align="center" class="mx-0">
          <div class="grey--text ms-4">{{ phoneBook.organization }}</div>
        </v-row>
        <div class="mt-4 text-subtitle-1">Email : {{ phoneBook.email }}</div>
        <div class="text-subtitle-1">
          Home phone : {{ phoneBook.homephone }}
        </div>

        <div class="text-subtitle-1">
          Office number : {{ phoneBook.officenumber }}
        </div>
        <div class="text-subtitle-1">
          Mobile number : {{ phoneBook.mobilenumber }}
        </div>
      </v-card-text>

      <v-divider class="mx-4"></v-divider>

      <v-card-actions>
        <v-btn @click="$router.back()" color="deep-purple lighten-2" text>
          Back
        </v-btn>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script>
import config from "@/config";

export default {
  data() {
    return {
      phoneBook: null,
      pathImage: "",
    };
  },
  computed: {
    phoneBookId() {
      return this.$route.query.id;
    },
  },
  mounted() {
    this.getPhonBook();
    this.pathImage = config.apiImage;
  },
  methods: {
    getPhonBook() {
      this.$axios
        .$get(`${config.apiUrl}/${config.user_id}/show/${this.phoneBookId}`)
        .then((response) => {
          this.phoneBook = response.data;
        });
    },
  },
};
</script>

<style>
</style>