<template>
  <v-container>
    <v-snackbar top v-model="snackbar" :color="color" :timeout="timeout">{{
      text
    }}</v-snackbar>
    <v-row class="px-4">
      <h2 v-if="$route.name == 'eventosAdm'">Eventos</h2>
      <h2 v-else>Meus Eventos</h2>
      <v-spacer></v-spacer>
      <v-btn
        v-if="$route.name == 'eventosAdm'"
        @click="btnEvento = true"
        title="Marcar Evento"
        small
        color="secondary"
        fab
      >
        <v-icon>mdi-plus</v-icon>
      </v-btn>
    </v-row>
    <v-row class="px-4 pt-3">
      <v-flex>
        <v-card class="w-100">
          <v-card-title>
            <v-text-field
              style="max-width: 300px;"
              v-model="search"
              append-icon="mdi-magnify"
              label="Pesquisar"
              single-line
              hide-details
            ></v-text-field>
          </v-card-title>

          <v-data-table
            no-data-text="Nenhum evento cadastrado"
            no-results-text="Nenhum evento encontrado"
            :headers="getHeaders()"
            :items="eventos"
            :search="search"
            :sort-by="['date', 'time']"
            :sort-desc="[true, true]"
            @click:row="eventShow"
            class="dataTable"
          >
            <template v-slot:item.type="{ item }">
              <span> {{ formatTypeEvent(item.type) }} </span>
            </template>
            <template v-slot:item.member="{ item }">
              <span> {{ item.member.first_name }} </span>
            </template>
            <template v-slot:item.date="{ item }">
              <span>{{ formatDate(item.date) }}</span>
            </template>
            <template v-slot:item.time="{ item }">
              <span>{{ formatTime(item.time) }}</span>
            </template>
            <template v-slot:item.engagement="{ item }">
              <span>{{ formatPercentage(item.engagement) }}</span>
            </template>
            <template v-slot:item.attendance="{ item }">
              <input type="checkbox" disabled v-model="item.attendance" />
            </template>
          </v-data-table>
        </v-card>
      </v-flex>
    </v-row>
    <NovoEvento
      :show="btnEvento"
      @close="btnEvento = false"
      v-on:showSnackbar="showSnackbar"
      @getAllEvents="getEvents()"
    >
    </NovoEvento>
    <modalDetail
      v-if="showDetail"
      :show="showDetail"
      @close="showDetail = false"
      :event="eventDetail"
      v-on:showSnackbar="showSnackbar"
      @getAllEvents="getEvents()"
    >
    </modalDetail>
  </v-container>
</template>

<script>
import NovoEvento from "./CadastrarEvento.vue";
import participationController from "../../controllers/ParticipationController";
import eventController from "../../controllers/EventController";
import modalDetail from "./ModalDetail.vue";
import moment from "moment";

export default {
  components: {
    NovoEvento,
    modalDetail,
  },

  async created() {
    let res = this.getEvents();
    this.eventos = res.data;
  },

  data() {
    return {
      btnEvento: false,
      search: "",
      eventController,
      participationController,
      eventDetail: null,
      showDetail: false,
      snackbar: false,
      text: "",
      timeout: 3000,
      color: "",
      eventos: [],
      userID: null,
    };
  },
  methods: {
    formatDate(date) {
      return moment(date).format("DD/MM/YYYY");
    },
    formatTime(time) {
      let hora = time.split(":");
      return `${hora[0]}:${hora[1]}`;
    },
    formatPercentage(item) {
      if (item > 100) {
        return 100 + " %";
      } else return item + " %";
    },
    eventShow(event) {
      this.eventDetail = event;
      this.showDetail = true;
    },
    showSnackbar(snackbarDetails) {
      this.snackbar = true;
      this.text = snackbarDetails.text;
      this.color = snackbarDetails.color;
    },
    getHeaders() {
      if (this.$route.name == "eventosAdm") {
        return [
          {
            text: "Nome",
            align: "center",
            value: "type",
          },
          { text: "Responsável", value: "member", align: "center" },
          { text: "Data", value: "date", align: "center" },
          { text: "Hora", value: "time", align: "center" },
          { text: "% de Presença", value: "engagement", align: "center" },
        ];
      } else {
        return [
          {
            text: "Nome",
            align: "center",
            value: "type",
          },
          { text: "Responsável", value: "member", align: "center" },
          { text: "Data", value: "date", align: "center" },
          { text: "Hora", value: "time", align: "center" },
          { text: "Presenca", value: "attendance", align: "center" },
        ];
      }
    },
    async getEvents() {
      this.eventos = [];
      if (this.$route.name == "eventosAdm") {
        const res = await this.eventController.getAllEvents(this.$api);
        this.eventos = res.data;
      } else if (this.$route.name == "meus-eventos") {
        const memberID = localStorage.getItem("user_id");
        const minhasParticipacoes = await this.participationController.getMemberParticipationEvent(
          this.$api,
          memberID
        );
        const meusEventos = [];
        minhasParticipacoes.forEach((participacao) => {
          participacao.event.participacaoID = participacao.id;
          participacao.event.attendance = participacao.attendance;
          meusEventos.push(participacao.event);
        });
        this.eventos = meusEventos;
      }
    },
    async deleteEvent() {
      await this.eventController.deleteEvent(this.$api, this.event);
    },
    formatTypeEvent(sigla) {
      if (sigla == "RG") return "Reunião Geral";
      else if (sigla == "AS") return "Assembléia";
      else if (sigla == "CF") return "Conferência";
      else if (sigla == "OU") return "Outros";
    },
  },
};
</script>
<style scoped>
.dataTable:hover {
  cursor: pointer;
}
</style>
