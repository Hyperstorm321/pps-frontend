<template>
  <div class="pop ml-12 mt-12">
    <v-layout row wrap>
      <v-flex>
        <h2>Examinations</h2>
      </v-flex>
    </v-layout>
    <v-layout row wrap class="ml-12 mt-5">
      <v-flex>
        <v-text-field
          prepend-inner-icon="mdi-magnify"
          rounded
          solo
          label="Search Exam"
        ></v-text-field>
      </v-flex>
      <v-flex class="ml-12">
        <v-btn
          class="primary rounded-lg mt-2"
          dark
          outlined
          v-on:click="createExamination"
        >
          <v-icon size="17" class="pr-1">mdi-plus-thick</v-icon> Create Exam
        </v-btn>
      </v-flex>
    </v-layout>
    <v-layout row wrap class="mt-5">
      <v-flex xs12 sm12 md10 xl7>
        <div class="text-center mr-12">
          <v-progress-circular
            v-if="loading == true"
            :rotate="90"
            :size="100"
            :width="15"
            :value="value"
            color="indigo"
          >
            {{ value }}
          </v-progress-circular>
        </div>

        <v-alert
          :value="typeof success !== 'undefined' && success != ''"
          transition="fade-transition"
          type="success"
        >
          {{ success }}
        </v-alert>
        <div class="pages" v-if="loading == false">
          <ul>
            <li>
              <v-btn small @click.prevent="loadExams(pagination.prev_page_url)"
                ><v-icon>mdi-menu-left</v-icon></v-btn
              >
            </li>
            <li class="ml-2 mr-2">
              <a class="page-link text-dark" href="#"
                >{{ pagination.current_page }} of {{ pagination.last_page }}</a
              >
            </li>
            <li>
              <v-btn small @click.prevent="loadExams(pagination.next_page_url)"
                ><v-icon>mdi-menu-right</v-icon></v-btn
              >
            </li>
          </ul>
        </div>
        <div v-if="loading == false">
          <v-expansion-panels
            accordion
            v-for="(exam, exam_no) in exams"
            :key="exam.exam_id"
          >
            <v-expansion-panel class="mt-3">
              <v-expansion-panel-header>
                <h3 class="indigo--text">{{ exam.exam_title }}</h3>
                <span> Room Code : {{ exam.exam_code }}</span>
                <span class="text-right"
                  ><b>Last update: </b>{{ exam.updated_on_f }}</span
                >
              </v-expansion-panel-header>
              <v-expansion-panel-content>
                <p>{{ exam.exam_desc }}</p>
                <p><b>Time Duration: </b>{{ exam.time_duration }} mins</p>
                <p>
                  <b>Passing Score: </b>{{ exam.passing_score }}/{{
                    exam.total_score
                  }}
                </p>
                <p>
                  <b>Number of questions: </b>{{ exam.total_num_questions }}
                </p>
                <p v-if="exam.is_randomized == true"><b>Randomized</b></p>

                <v-container class="text-center">
                  <v-btn
                    class="ma-2"
                    outlined
                    rounded
                    color="teal darken-4"
                    @click="viewExam(exam.exam_id)"
                  >
                    <v-icon left>mdi-eye</v-icon> View
                  </v-btn>
                  <v-btn
                    class="ma-2"
                    outlined
                    rounded
                    color="success"
                    @click="editExamConfirm(exam.exam_id)"
                  >
                    <v-icon left>mdi-pencil</v-icon> Edit
                  </v-btn>
                  <v-btn
                    class="ma-2"
                    outlined
                    rounded
                    color="red"
                    @click="deleteExamConfirm(exam.exam_id, exam_no)"
                  >
                    <v-icon left>mdi-delete</v-icon> Delete
                  </v-btn>
                  <v-divider></v-divider>
                  <v-btn
                    class="ma-2"
                    outlined
                    rounded
                    color="light-blue darken-3"
                    @click="manageExaminees(exam.exam_id, exam.exam_title)"
                  >
                    <v-icon left>mdi-account-group</v-icon> Manage Examinees
                  </v-btn>
                  <v-btn
                    class="ma-2"
                    outlined
                    rounded
                    color="deep-orange darken-1"
                    @click="viewResults(exam.exam_id, exam.exam_title)"
                  >
                    <v-icon left>mdi-folder-account</v-icon> View Results
                  </v-btn>
                  <v-btn
                    class="ma-2"
                    outlined
                    rounded
                    color="indigo"
                    @click="openChat(exam.exam_id)"
                  >
                    <v-icon left>mdi-chat</v-icon> View Chats
                  </v-btn>
                </v-container>
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>
        </div>
      </v-flex>

      <v-container>
        <v-dialog v-model="delExamDialog.show" persistent max-width="290">
          <v-card class="pop">
            <v-card-title class="pop"><h3>Delete Exam</h3></v-card-title>
            <v-card-text>{{ delExamDialog.message }}</v-card-text>
            <v-card-text
              >All the examinees' records of taking or taken the exam will also
              be deleted.</v-card-text
            >
            <v-card-text>Are you sure to delete the exam?</v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="error" text @click="delExamDialog.show = false"
                >No</v-btn
              >
              <v-btn color="primary" text @click="deleteExam()"
                >Yes, continue</v-btn
              >
            </v-card-actions>
          </v-card>
        </v-dialog>

        <v-dialog v-model="editExamDialog.show" persistent max-width="290">
          <v-card class="pop">
            <v-card-title class="pop"><h3>Edit Exam</h3></v-card-title>
            <v-card-text>{{ editExamDialog.message }}</v-card-text>
            <v-card-text
              >All the examinees already taken the exam will not be
              updated.</v-card-text
            >
            <v-card-text>Are you sure to edit the exam?</v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="error" text @click="editExamDialog.show = false"
                >No</v-btn
              >
              <v-btn color="primary" text @click="editExam()"
                >Yes, continue</v-btn
              >
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-container>
    </v-layout>

    <!-- chat dialog -->
    <v-dialog v-model="chatDialog" persistent max-width="550">
      <v-card class="font-body rounded-lg">
        <v-card-title class="pl-8 pr-8 pt-8 justify-center">
          Messages
        </v-card-title>
        <v-card-text>
          <ul class="list-unstyled" style="height:300px; overflow-y:scroll">
            <li class="p-2" v-for="message in messages" :key="message.id">
              <strong class="indigo--text mr-1 text-md-body-1"
                >{{ message.first_name }} {{ message.last_name }} :
              </strong>
              <span class="black--text text-md-body-1">
                {{ message.message }}</span
              >
            </li>
          </ul>
          <v-text-field
            v-model="newMessage"
            outlined
            dense
            rounded
            class="mt-5"
            placeholder="Enter your Message"
            v-on:keyup.enter="sendMessage"
          ></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <div class="mb-2">
            <v-btn
              color="indigo"
              outlined
              @click="closeChat"
              class="text-uppercase rounded-lg"
            >
              Close
            </v-btn>
          </div>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-main>
      <ExaminerDashboard />
    </v-main>
  </div>
</template>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Poppins&display=swap");
.pop {
  font-family: "Poppins", sans-serif;
}
.colored-title {
  color: #760d11;
}
.pages li {
  float: left;
}
.pages ul {
  list-style-type: none;
}
.pages a {
  text-decoration: none;
  color: black;
  display: block;
}
</style>

<script>
import ExaminerDashboard from "@/components/examiner/Dashboard";
import axios from "axios";

axios.defaults.withCredentials = true;
axios.defaults.baseURL = "http://localhost:8000";

export default {
  components: {
    ExaminerDashboard,
  },
  props: {
    success: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      exams: [],
      message: "",
      loading: true,
      user_id: "",
      interval: {},
      messages: [],
      newMessage: "",
      chatDialog: false,
      value: 0,
      pagination: {},
      delExamDialog: {
        show: false,
        message: "",
        exam_id: "",
        exam_no: 0,
      },
      editExamDialog: {
        show: false,
        message: "",
        exam_id: 0,
      },
    };
  },
  mounted() {
    this.loadExams();
    this.loadingButton();
    this.loadData();
  },
  // for loading button
  beforeDestroy() {
    clearInterval(this.interval);
  },
  methods: {
    loadData() {
      axios
        .get("/api/user")
        .then((response) => {
          this.user_id = response.data.user_id;
        })
        .catch((error) => {
          console.log("Call the Administrator.");
        });
    },
    createExamination() {
      this.$router.push({ name: "ExaminerCreateExamination" });
    },
    loadExams(page_url) {
      let main_url = "/api/exams";
      page_url = page_url || main_url;
      console.log(page_url);
      axios
        .get(page_url)
        .then((response) => {
          this.exams = response.data.data;
          this.makePagination(response.data);
          console.log(response.data);
          this.loading = false;
        })
        .catch((error) => {
          this.message = "error";
        });
    },
    makePagination(response) {
      let pagination = {
        current_page: response.current_page,
        last_page: response.last_page,
        prev_page_url: response.prev_page_url,
        next_page_url: response.next_page_url,
      };
      this.pagination = pagination;
    },
    manageExaminees(exam_id, exam_title) {
      this.$router.push({
        name: "ExaminerManageExaminees",
        params: {
          exam_id: exam_id,
          exam_title: exam_title,
        },
      });
    },
    viewResults(exam_id, exam_title) {
      this.$router.push({
        name: "ExaminerViewResults",
        params: {
          exam_id: exam_id,
          exam_title: exam_title,
        },
      });
    },
    deleteExamConfirm(exam_id, exam_no) {
      axios
        .post("/api/exam/examinees/count", {
          exam_id: exam_id,
        })
        .then((response) => {
          this.delExamDialog.exam_id = exam_id;
          this.delExamDialog.exam_no = exam_no;
          this.delExamDialog.message = response.data.message;
          this.delExamDialog.show = true;
        })
        .catch((error) => {
          console.log("Call the Administrator");
        });
    },
    deleteExam() {
      axios
        .post("/api/exam/delete", {
          exam_id: this.delExamDialog.exam_id,
        })
        .then((response) => {
          this.exams.splice(this.delExamDialog.exam_no, 1);
          this.delExamDialog.show = false;

          this.success = "The exam is successfully deleted.";
        })
        .catch((error) => {
          console.log("Call the Administrator");
        });
    },
    editExamConfirm(exam_id) {
      axios
        .post("/api/exam/examinees/count", {
          exam_id: exam_id,
        })
        .then((response) => {
          this.editExamDialog.exam_id = exam_id;
          this.editExamDialog.message = response.data.message;
          this.editExamDialog.show = true;
        })
        .catch((error) => {
          console.log("Call the Administrator");
        });
    },
    editExam() {
      this.$router.push({
        name: "ExaminerEditExam",
        params: {
          exam_id: this.editExamDialog.exam_id,
        },
      });
    },
    viewExam(exam_id) {
      this.$router.push({
        name: "ExaminerViewExam",
        params: {
          exam_id: exam_id,
        },
      });
    },
    loadingButton() {
      this.interval = setInterval(() => {
        if (this.value === 100) {
          return (this.value = 0);
        }
        this.value += 10;
      }, 1000);
    },
    openChat(exam_id) {
      this.exam_id = exam_id;
      this.chatDialog = true;
      this.realTime();
    },
    getMessages() {
      axios.get(`api/messages/${this.exam_id}`).then((response) => {
        this.messages = response.data;
      });
    },
    sendMessage() {
      axios
        .post("api/messages", {
          user_id: this.user_id,
          room_id: this.exam_id,
          message: this.newMessage,
        })
        .then((response) => {
          this.newMessage = "";
        })
        .catch((error) => {
          console.log(this.user_id);
        });
    },
    closeChat() {
      this.chatDialog = false;
    },
    realTime() {
      window.Echo.channel("chat" + this.exam_id).listen("Messages", (event) =>
        this.messages.push({
          message: event.message.message,
          last_name: event.user.last_name,
          first_name: event.user.first_name,
        })
      );
    },
  },
};
</script>
