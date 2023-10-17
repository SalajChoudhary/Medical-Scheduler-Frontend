<template>
  <v-container>
    <navbar />
    <v-divider></v-divider>
    <h1>Weekly Calendar</h1>
    <v-spacer></v-spacer>
    <div>
      <v-sheet tile height="54" class="d-flex">
        <v-btn icon class="ma-2" @click="$refs.calendar.prev()">
          <v-icon>mdi-chevron-left</v-icon>
        </v-btn>
        <v-select
          v-model="type"
          :items="types"
          dense
          outlined
          hide-details
          class="ma-2"
          label="type"
        ></v-select>
        <v-select
          :items="this.$store.state.doctors"
          v-model="doctorObj"
          item-text="firstName"
          item-value="doctorId"
          return-object
          dense
          outlined
          hide-details
          class="ma-2"
          label="doctors"
          @change="chosenDoctor"
        ></v-select>
        <v-select
          v-model="weekday"
          :items="weekdays"
          dense
          outlined
          hide-details
          label="weekdays"
          class="ma-2"
        ></v-select>
        <v-spacer></v-spacer>
        <v-btn icon class="ma-2" @click="$refs.calendar.next()">
          <v-icon>mdi-chevron-right</v-icon>
        </v-btn>
      </v-sheet>
      <v-sheet height="600">
        <v-calendar
          ref="calendar"
          v-model="value"
          :weekdays="weekday"
          :type="type"
          :events="filteredAppointments"
          @click:event="showEventDetails"
          :now="now"
        ></v-calendar>
      </v-sheet>
      <!-- Dialog for displaying appointment details -->
      <v-dialog v-model="dialog" max-width="400px">
        <v-card>
          <v-card-title>Appointment Details</v-card-title>
          <v-card-text>
            <p><strong>Doctor:</strong> {{ selectedEvent.doctorName }}</p>
            <p><strong>Patient:</strong> {{ selectedEvent.patientName }}</p>
            <p><strong>Time:</strong> {{ selectedEvent.start }} - {{ selectedEvent.end }}</p>
          </v-card-text>
          <v-card-actions>
            <v-btn color="blue darken-1" text @click="dialog = false">Close</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>
  </v-container>
</template>
<script>
import Navbar from "../components/Navbar.vue";
import AppointmentService from "../services/AppointmentService";
import PatientService from "@/services/PatientService";

export default {
  name: "calendar",
  components: {
    Navbar,
  },
  data() {
    return {
      type: "month",
      types: ["month", "week", "day"],
      mode: "stack",
      weekday: [0, 1, 2, 3, 4, 5, 6],
      weekdays: [
        { text: "Sun - Sat", value: [0, 1, 2, 3, 4, 5, 6] },
        { text: "Mon - Sun", value: [1, 2, 3, 4, 5, 6, 0] },
        { text: "Mon - Fri", value: [1, 2, 3, 4, 5] },
      ],
      value: "",
      events: [],
      colors: ["blue"],
      names: ["Appointment", "Holiday", "PTO"],
      appointments: [],
      now: "2023-04-20",
      selectedDoctorId: null, // id of doctor,
      doctors: [],
      patients: [],
      doctorObj: {},
      dialog: false,
      selectedEvent: {}
    };
  },
  methods: {
    chosenDoctor() {
      this.selectedDoctorId = this.doctorObj.doctorId;
    },
    getAppointments() {
      AppointmentService.getAppointments().then((response) => {
        this.$store.commit("SET_APPOINTMENTS", response.data);
        this.appointments = this.$store.state.appointments;
        this.getEvents();
      });
    },
    showEventDetails(event) {
      this.selectedEvent = event; // This is assuming the event contains doctorName and patientName. You might need to update this based on your actual data structure.
      this.dialog = true;
    },
    getEvents() {
      for (let i = 0; i < this.appointments.length; i++) {
        let temp = this.appointments[i];
        let time = this.appointments[i].appointmentTime;
        let endTime;
        if (time.slice(3, 5) === "30") {
          endTime = time.slice(0, 2);
          parseInt(endTime);
          endTime++;
          endTime = endTime.toString() + ":00:00";
        } else {
          endTime = time.slice(0, 2) + ":30:00";
        }
        let event = {
          docId: this.appointments[i].doctorId,
          name: "Appointment",
          doctorName: this.getDoctorName(temp.doctorId),
          patientName: this.getPatientName(temp.patientId),
          start: temp.appointmentDate + "T" + temp.appointmentTime,
          end: (temp.appointmentDate += "T" + endTime),
          color: "blue",
          timed: false,
        };
        this.events.push(event);
      }
    },
    getDoctorName(id) {
      let doctor = this.doctors.find(doc => doc.doctorId === id);
      return doctor ? doctor.firstName : 'Unknown';
    },

    getPatientName(id) {
      let patient = this.patients.find(pat => pat.patientId === id);
      return patient ? patient.firstName : 'Unknown';
    },
    getPatients() {
      PatientService.getAllPatients().then((response) => {
        this.$store.commit("SET_PATIENTS", response.data);
        this.patients = this.$store.state.patients;
      });
    }
  },
  computed: {
    // This should filter doctor apponintments by id - attached to :events for calendar
    filteredAppointments() {
      return this.events.filter((appt) => appt.docId === this.selectedDoctorId);
    },
  },
  created() {
    this.getAppointments();
    this.doctors = this.$store.state.doctors;
    this.getPatients();
  },
};
</script>
