<template>
  <v-card class="mx-auto overflow-hidden" height="100%">
    <v-app-bar color="deep-purple" dark>
      <v-app-bar-nav-icon @click="drawer = true"></v-app-bar-nav-icon>

      <v-toolbar-title>Información de atenciones medicas</v-toolbar-title>
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
        <!-- <v-row>
          <v-col cols="12" md="4">
            <v-text-field
              v-model="nameCompleted"
              label="Nombre"
              :rules="[(v) => !!v || 'Este campo es requerido']"
              required
            ></v-text-field>
          </v-col>
        </v-row> -->
        <v-col cols="3">
          <v-row>
            <v-menu
              ref="menu"
              v-model="menu"
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              offset-y
              max-width="290px"
              min-width="290px"
            >
              <template v-slot:activator="{ on }">
                <v-text-field
                  v-model="startDate"
                  label="Fecha de Inicio"
                  hint="YYYY-MM-DD formato"
                  persistent-hint
                  lenguaje="es"
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker
                v-model="startDate"
                no-title
                locale="es-es"
                @input="menu = false"
              ></v-date-picker>
            </v-menu>
            <v-spacer></v-spacer>
            <v-menu
              ref="menu1"
              v-model="menu1"
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              offset-y
              max-width="290px"
              min-width="290px"
            >
              <template v-slot:activator="{ on }">
                <v-text-field
                  v-model="endDate"
                  label="Fecha de Fin"
                  hint="YYYY-MM-DD formato"
                  persistent-hint
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker
                v-model="endDate"
                no-title
                locale="es-es"
                @input="menu1 = false"
              ></v-date-picker>
            </v-menu>
          </v-row>
        </v-col>
        <!-- <v-col cols="4">
          <v-row aling="start">
            <v-select
              class="pr-2"
              v-model="select"
              :items="companyInfo"
              :rules="[(v) => !!v || 'seleccionar una empresa']"
              label="Selecciona Empresa"
              required
            ></v-select>
          </v-row>
        </v-col> -->
      </v-container>
      <v-container>
        <v-col cols="6">
          <v-row>
            <v-btn
              :disabled="!valid"
              color="success"
              class="mr-4"
              @click="validateWithGetInformation"
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
      :items="dataInfo"
      :items-per-page="15"
      class="elevation-1"
      item-key="name"
      :search="search"
      :custom-filter="filterOnlyCapsText"
    >
      <template v-slot:items="props">
        <td>{{ props.item.codigo }}</td>
        <td>{{ props.item.fecha }}</td>
        <td>{{ props.item.empresa }}</td>
        <td>{{ props.item.subcontrata }}</td>
        <td>{{ props.item.proyecto }}</td>
        <td>{{ props.item.tExam }}</td>
        <td>{{ props.item.paciente }}</td>
        <td>{{ props.item.certificado }}</td>
        <td>{{ props.item.certificadoDownloaded }}</td>
        <td>{{ props.item.imp }}</td>
        <td>{{ props.item.impDownloaded }}</td>
        <td>{{ props.item.fechaDownloaded }}</td>
      </template>
      <template #item.certificado="{ value }">
        <a target="_blank" @click="getDownload(value)">
          {{ value }}
        </a>
      </template>
      <template #item.imp="{ value }">
        <a target="_blank" @click="getDownload(value)">
          {{ value }}
        </a>
      </template>
      <template v-slot:top>
        <v-text-field
          v-model="search"
          label="Buscar (Solo Mayusculas)"
          class="mx-4"
        ></v-text-field>
      </template>
      <!-- <template v-slot:no-data>
        <v-btn color="primary" @click="initialize">Reset</v-btn>
      </template> -->
    </v-data-table>
  </v-card>
</template>
<script>
import axios from "axios";
export default {
  data() {
    return {
      select: null,
      valid: false,
      search: "",
      nameCompleted: "",
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
        { text: "T_Exam", value: "tExam" },
        { text: "Paciente", value: "paciente" },
        { text: "Certificado", value: "certificado" },
        { text: "Certificado_downloaded", value: "certificadoDownloaded" },
        { text: "Imp", value: "imp" },
        { text: "Imp_downloaded", value: "impDownloaded" },
        { text: "fecha_downloaded", value: "fechaDownloaded" },
      ],

      dataShow: 15,

      drawer: false,
      group: null,
      dataInfo: [],
      companyInfo: [],
      menu: false,
      menu1: false,
    };
  },
  mounted() {
    this.getLearningData();
    this.getCompanysData();
  },
  methods: {
    forceFileDownload(response, item) {
      var headers = response.headers;
      var blob = new Blob([response.data], { type: headers["content-type"] });
      var link = document.createElement("a");
      link.href = window.URL.createObjectURL(blob);
      link.download = item;
      link.click();
      link.remove();
    },
    getDownload(item) {
      console.log("ingreso: " + item);
      console.log("Ingreso: getDownload");

      this.error = null;
      console.log();
      const axiosInstance = axios.create({
        timeout: 10000,
        responseType: "blob",
      });
      axiosInstance
        .get("http://138.197.103.244/Api/v1/doktuz/file/" + item)
        .then((response) => {
          this.forceFileDownload(response, item);
        })
        .catch((error) => {
          this.error = error.toString();
          console.log("Error on retriving file: " + error);
        });
    },
    validateWithGetInformation() {
      if (this.nameCompleted) {
        console.log("entro: " + this.nameCompleted + "getWithNamed");
        this.getWithNamed();
      } else if (this.startDate && this.endDate) {
        console.log(
          "entro: " + this.startDate + this.endDate + "getRangeWithDate"
        );
        this.getRangeWithDate();
      } else if (this.select) {
        console.log("entro: " + this.select + "getWithCompany");
        this.getWithCompany();
      } else {
        console.log("entro: " + "getLearningData");
        this.getLearningData();
      }
    },
    filterOnlyCapsText(value, search) {
      return (
        value != null &&
        search != null &&
        typeof value === "string" &&
        value.toString().toLocaleUpperCase().indexOf(search) !== -1
      );
    },
    getWithNamed() {
      this.error = null;
      console.log();
      const axiosInstance = axios.create({
        timeout: 10000,
        headers: {
          "Content-Type": "application/json",
        },
      });

      axiosInstance
        .get(
          "http://138.197.103.244/Api/v1/doktuz/allWithNameOrCompany/" +
            this.nameCompleted.toUpperCase() +
            "&" +
            "none"
        )
        .then((response) => {
          console.log(response.data);
          this.dataInfo = response.data.data;
        })
        .catch((error) => {
          this.error = error.toString();
          console.log("Error on retriving articles: " + error);
        });
    },
    getWithCompany() {
      this.error = null;
      console.log();
      const axiosInstance = axios.create({
        timeout: 10000,
        headers: {
          "Content-Type": "application/json",
        },
      });

      axiosInstance
        .get(
          "http://138.197.103.244/Api/v1/doktuz/allWithNameOrCompany/" +
            "none" +
            "&" +
            this.select
        )
        .then((response) => {
          console.log(response.data);
          this.dataInfo = response.data.data;
        })
        .catch((error) => {
          this.error = error.toString();
          console.log("Error on retriving articles: " + error);
        });
    },
    getRangeWithDate() {
      this.error = null;
      console.log();
      const axiosInstance = axios.create({
        timeout: 10000,
        headers: {
          "Content-Type": "application/json",
        },
      });

      axiosInstance
        .get(
          "http://138.197.103.244/Api/v1/doktuz/allWithRangeDate/" +
            this.startDate +
            "&" +
            this.endDate
        )
        .then((response) => {
          console.log(response.data);
          this.dataInfo = response.data.data;
        })
        .catch((error) => {
          this.error = error.toString();
          console.log("Error on retriving articles: " + error);
        });
    },

    validate() {
      this.$refs.form.validate();
    },
    reset() {
      this.$refs.form.reset();
    },
    getLearningData() {
      this.error = null;
      const axiosInstance = axios.create({
        timeout: 10000,
        headers: {
          "Content-Type": "application/json",
        },
      });
      axiosInstance
        .get("http://138.197.103.244/Api/v1/doktuz")
        .then((response) => {
          this.dataInfo = [];
          this.dataInfo = response.data.data;
        })
        .catch((error) => {
          this.error = error.toString();
          console.log("Error on retriving articles: " + error);
        });
    },

    getCompanysData() {
      this.error = null;
      const axiosInstance = axios.create({
        timeout: 10000,
        headers: {
          "Content-Type": "application/json",
        },
      });
      axiosInstance
        .get("http://138.197.103.244/Api/v1/doktuz/allCompanys")
        .then((response) => {
          this.companyInfo = response.data.companys;
        })
        .catch((error) => {
          this.error = error.toString();
          console.log("Error on retriving articles: " + error);
        });
    },
    formatDate(date) {
      if (!date) return null;

      const [year, month, day] = date.split("-");
      return `${year}-${month}-${day}`;
    },
    parseDate(date) {
      if (!date) return null;

      const [month, day, year] = date.split("/");
      return `${year}-${month.padStart(2, "0")}-${day.padStart(2, "0")}`;
    },
  },
};
</script>