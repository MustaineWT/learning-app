<template>
  <v-card class="mx-auto overflow-hidden" height="100%">
    <v-app-bar color="deep-purple" dark>
      <v-app-bar-nav-icon @click="drawer = true"></v-app-bar-nav-icon>

      <v-toolbar-title>Title</v-toolbar-title>
    </v-app-bar>

    <v-navigation-drawer v-model="drawer" absolute temporary>
      <v-list nav dense>
        <v-list-item-group
          v-model="group"
          active-class="deep-purple--text text--accent-4"
        >
          <v-list-item>
            <v-list-item-icon>
              <v-icon>mdi-home</v-icon>
            </v-list-item-icon>
            <v-list-item-title>Home</v-list-item-title>
          </v-list-item>

          <v-list-item>
            <v-list-item-icon>
              <v-icon>mdi-account</v-icon>
            </v-list-item-icon>
            <v-list-item-title>Account</v-list-item-title>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>
    <v-form ref="form" v-model="valid" lazy-validation>
      <v-container>
        <v-row>
          <v-col cols="12" md="4">
            <v-text-field
              v-model="name"
              :counter="10"
              label="Nombre"
              required
            ></v-text-field>
          </v-col>
        </v-row>
        <v-col cols="6">
          <v-row>
            <v-text-field
              class="pr-2"
              v-model="startDate"
              label="Fecha de inicio"
              required
            ></v-text-field>
            <v-text-field
              class="pr-2"
              v-model="endDate"
              label="Fecha de fin"
              required
            ></v-text-field>
            <v-select
              class="pr-2"
              v-model="select"
              :items="items"
              :rules="[(v) => !!v || 'seleccionar una empresa']"
              label="Selecciona Empresa"
              required
            ></v-select>
          </v-row>
        </v-col>
      </v-container>
      <v-container>
        <v-col cols="6">
          <v-row>
            <v-btn
              :disabled="!valid"
              color="success"
              class="mr-4"
              @click="validate"
            >
              Buscar
            </v-btn>
            <v-btn color="error" class="mr-4" @click="reset"> Limpiar </v-btn>
          </v-row>
        </v-col>
      </v-container>
    </v-form>
    <v-data-table
      :headers="headers"
      :items="data"
      :items-per-page="15"
      class="elevation-1"
    ></v-data-table>
  </v-card>
</template>
<script>
import axios from "axios";
export default {
  data() {
    return {
      select: null,
      items: ["Company 1", "Company 2"],
      valid: false,
      name: "",
      startDate: "",
      endDate: "",
      nameRules: [
        (v) => !!v || "Name is required",
        (v) => v.length <= 10 || "Name must be less than 10 characters",
      ],
      email: "",
      headers: [
        {
          text: "Codigo",
          align: "start",
          sortable: false,
          value: "codigo",
        },
        { text: "Fecha", value: "fecha" },
        { text: "Empresa", value: "empresa" },
        { text: "SubContrata", value: "subcontrata" },
        { text: "Proyecto", value: "proyecto" },
        { text: "T_Exam", value: "t_exam" },
        { text: "Paciente", value: "paciente" },
        { text: "Certificado", value: "certificado" },
        { text: "Certificado_downloaded", value: "certificado_downloaded" },
        { text: "Imp", value: "imp" },
        { text: "Imp_downloaded", value: "imp_downloaded" },
      ],

      data: [],
      dataShow: 15,

      drawer: false,
      group: null,

      mounted() {
        axios
          .get(process.env.URL_ENDPOINT + "/doktuz")
          .then((response) => (this.data = response.data))
          .catch((error) => {
            console.log(error);
          });
      },
    };
  },
   created() {
    this.getData();
  },
  methods: {
    validate() {
      this.$refs.form.validate();
    },
    reset() {
      this.$refs.form.reset();
    },
     getData() {
      this.error = null;
      const axiosInstance = axios.create({        
        timeout: 10000,
        headers: {
          "Content-Type": "application/json",
          "Access-Control-Allow-Origin": "*",
        },
      });
      axiosInstance
        .get("http://localhost:3000/doktuz")
        //On success save response in articles variable
        .then(response => {
          this.data = response.data;
          console.log("Articles found: ", response.data);
        })
        //Catch and display any errors
        .catch(error => {
          this.error = error.toString();
          console.log("Error on retriving articles: " + error);
        });
    }
  },
};
</script>