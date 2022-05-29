<template>
  <v-data-table
    :headers="headers"
    :items="listPhoneBooks"
    sort-by="calories"
    class="elevation-0"
    hide-default-footer
  >
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>Phone Book</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              New Item
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-form ref="form1" v-model="valid">
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.name"
                        label="Name"
                        :rules="rules"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.organization"
                        label="Organization"
                        :rules="rules"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.mobilenumber"
                        label="Mobile number"
                        :rules="rules"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.homephone"
                        label="Home phone"
                        :rules="rules"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.officenumber"
                        label="Office number"
                        :rules="rules"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.email"
                        label="Email"
                        :rules="rules"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-file-input
                        v-model="editedItem.image"
                        accept="image/*"
                        label="Image"
                        prepend-icon="mdi-image"
                        :rules="rules"
                      ></v-file-input>
                    </v-col>
                  </v-row>
                </v-form>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close"> Cancel </v-btn>
              <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5"
              >Are you sure you want to delete this item?</v-card-title
            >
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete"
                >Cancel</v-btn
              >
              <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                >OK</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:[`item.image`]="{ item }">
      <v-col v-if="item.image">
        <v-img
          :src="pathImage + item.image"
          :lazy-src="pathImage + item.image"
          aspect-ratio="1"
          class="grey lighten-2"
          width="50"
          height="50"
        >
        </v-img>
      </v-col>
      <v-col v-else>
        <v-img aspect-ratio="1" class="grey lighten-2" width="50" height="50">
        </v-img>
      </v-col>
    </template>
    <template v-slot:[`item.actions`]="{ item }">
      <v-icon small class="mr-2" @click="showItem(item)"> mdi-eye </v-icon>
      <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
      <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="getListPhoneBook"> Reset </v-btn>
    </template>
  </v-data-table>
</template>

<script>
import config from "@/config";
export default {
  data: () => ({
    rules: [(value) => !!value || "จำเป็นต้องระบุ."],
    valid: true,
    dialog: false,
    dialogDelete: false,
    headers: [
      { text: "", value: "image" },
      {
        text: "Name",
        align: "start",
        sortable: false,
        value: "name",
      },
      { text: "Organization", value: "organization" },
      { text: "Mobile number", value: "mobilenumber" },
      { text: "Home phone", value: "homephone" },
      { text: "Office number", value: "officenumber" },
      { text: "Email", value: "email" },

      { text: "Actions", value: "actions", sortable: false },
    ],
    listPhoneBooks: [],
    editedId: null,
    editedItem: {
      name: "",
      organization: "",
      mobilenumber: "",
      homephone: "",
      officenumber: "",
      email: "",
      image: [],
    },
    defaultItem: {
      name: "",
      organization: "",
      mobilenumber: "",
      homephone: "",
      officenumber: "",
      email: "",
      image: [],
    },
    pathImage: "",
  }),

  computed: {
    formTitle() {
      return this.editedId === null ? "New Item" : "Edit Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.getListPhoneBook();
    this.pathImage = config.apiImage;
  },

  methods: {
    getListPhoneBook() {
      this.$axios
        .$get(`${config.apiUrl}/${config.user_id}/index`)
        .then((response) => {
          this.listPhoneBooks = response.data;
        });
    },
    createPhoneBook() {
      this.$refs.form1.validate();
      if (this.valid) {
        let formData = new FormData();
        formData.append("name", this.editedItem.name);
        formData.append("organization", this.editedItem.organization);
        formData.append("mobilenumber", this.editedItem.mobilenumber);
        formData.append("homephone", this.editedItem.homephone);
        formData.append("officenumber", this.editedItem.officenumber);
        formData.append("email", this.editedItem.email);
        formData.append("image", this.editedItem.image);

        let url = `${config.apiUrl}/${config.user_id}/store`;

        this.$axios
          .post(url, formData, {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          })
          .then((response) => {
            this.getListPhoneBook();
          });
      }
    },
    updatePhoneBook() {
      this.$refs.form1.validate();
      if (this.valid) {
        let data = {
          name: this.editedItem.name,
          organization: this.editedItem.organization,
          mobilenumber: this.editedItem.mobilenumber,
          homephone: this.editedItem.homephone,
          officenumber: this.editedItem.officenumber,
          email: this.editedItem.email,
        };

        let url = `${config.apiUrl}/${config.user_id}/update/${this.editedId}`;

        this.$axios.put(url, data).then((response) => {
          this.getListPhoneBook();
        });
      }
    },
    showItem(item) {
      this.$router.push(`/show/?id=${item.id}`);
    },

    editItem(item) {
      this.editedItem = Object.assign({}, item);
      this.editedId = item.id;
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedId = item.id;
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.$axios
        .$delete(`${config.apiUrl}/${config.user_id}/destroy/${this.editedId}`)
        .then((response) => {
          this.getListPhoneBook();
        });
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedId = null;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedId = null;
      });
    },

    save() {
      if (this.editedId) {
        this.updatePhoneBook();
      } else {
        this.createPhoneBook();
      }
      this.close();
    },
  },
};
</script>

<style>
</style>