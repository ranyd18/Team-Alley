<template>
  <div>
    <div v-if="user" class="page-header clear-filter" color="orange">
      <div
        class="page-header-image"
        style="background-image: url('img/inputdata-bg.jpg'); opacity: 0.4"
        v-bind:style="(height = '100px')"
      ></div>

      <form>
        <h3 class="greeting" style="color: black">
          Set up your goals and start tracking!
        </h3>
        <h2 style="color: black">Exercise</h2>

        <div class="form-group">
          <label for="inputExerciseDuration" style="color: black"
            >Duration of Exercise (min/day)*</label
          >
          <input
            type="number"
            class="form-control"
            id="inputExerciseDuration"
            placeholder="45"
            v-model.number="item.exercise"
            required
          />
        </div>

        <h2 style="color: black">Wellness</h2>
        <div class="form-group">
          <label for="inputWorkDuration" style="color: black"
            >Duration of Working (hr/day)*</label
          >
          <input
            type="number"
            class="form-control"
            id="inputWorkDuration"
            placeholder="7"
            v-model.number="item.work"
            required
          />
        </div>

        <div class="form-group">
          <label for="inputLeisureDuration" style="color: black"
            >Duration of Leisure (hr/day)*</label
          >
          <input
            type="number"
            class="form-control"
            id="inputLeisureDuration"
            placeholder="1.5"
            v-model.number="item.leisure"
            required
          />
        </div>
        <h2 style="color: black">Hygiene</h2>
        <div class="form-row">
          <div class="form-group col-md-4">
            <label for="inputTemperature" style="color: black"
              >Temperature Taking (times/day)*</label
            >
            <input
              type="number"
              class="form-control"
              id="inputTemperature"
              placeholder="2"
              v-model.number="item.temperature"
              required
            />
          </div>
          <div class="form-group col-md-4">
            <label for="inputMask" style="color: black"
              >Washing masks (times/day)*</label
            >
            <input
              type="number"
              class="form-control"
              id="inputMask"
              placeholder="1"
              v-model.number="item.mask"
              required
            />
          </div>
          <div class="form-group col-md-4">
            <label for="inputHands" style="color: black"
              >Wasing hands (times/day)*</label
            >
            <input
              type="number"
              class="form-control"
              id="inputHands"
              placeholder="5"
              v-model.number="item.hand"
              required
            />
          </div>
        </div>

        <h2 style="color: black">Food</h2>
        <h5 style="color: black">
          Choose a level between our recommended range:{{ getdata2 }}
          {{ data.calorieMin }} - {{ data.calorieMax }}
        </h5>

        <div class="form-group">
          <label for="inputCalory" style="color: black"
            >Daily calories intake (kCal/day) *</label
          >
          <input
            type="number"
            class="form-control"
            id="inputHands"
            placeholder="1850"
            v-model.number="item.calorie"
            required
          />
        </div>

        <button class="btn btn-primary btn-round" v-on:click.prevent="addItem">
          SUBMIT
        </button>
      </form>
    </div>

    <div v-else class="section">
      <div  class="container">
        <div class="alert alert-danger">Please log in first</div>
      </div>
    </div>
  </div>
</template>
<script>
//import { Tabs, TabPane } from '@/components';
import { database, storage } from "../firebase.js";
import firebase from "firebase";
import auth from "../firebase.js";
import VueSimpleAlert from "vue-simple-alert";

export default {
  name: "editgoal",
  bodyClass: "form-page",
  components: {
    //Tabs,
    //TabPane
  },
  data() {
    return {
      item: {
        exercise: "",
        work: "",
        leisure: "",
        mask: "",
        temperature: "",
        calorie: "",
        hand: "",
      },
      data: {},
    };
  },
  computed: {
    getdata2() {
      this.getdata();
      return null;
    },
    user() {
      return auth.currentUser;
    }
  },
  methods: {
    checkCalorie() {
      if (this.item.calorie < this.data.calorieMin) {
        return true;
      } else if (this.item.calorie > this.data.calorieMax) {
        return true;
      } else {
        return false;
      }
    },
    getdata() {
      database
        .collection("Users")
        .doc(auth.currentUser.email)
        .get()
        .then((doc) => {
          this.data = doc.data();
        })
        .catch((err) => {
          console.log("Error getting document:", err);
        });
    },
    checkdata() {
      if ("weight" in this.data) {
        return false;
      } else {
        return true;
      }
    },
    addItem() {
      if (
        this.item.exercise === "" ||
        this.item.work === "" ||
        this.item.leisure === "" ||
        this.item.mask === "" ||
        this.item.temperature === "" ||
        this.item.calorie === "" ||
        this.item.hand === ""
      ) {
        VueSimpleAlert.alert("Please fill in empty fields!",'','error');
      } else if (
        this.item.exercise < 0 ||
        this.item.work < 0 ||
        this.item.leisure < 0 ||
        this.item.mask < 0 ||
        this.item.temperature < 0 ||
        this.item.calorie < 0 ||
        this.item.hand < 0
      ) {
        VueSimpleAlert.alert(
          "Input value cannot be negative. Please check again before submission!",'','error'
        );
      } else if (this.checkdata()) {
        VueSimpleAlert.alert(
          "Please fill in your health data first for valid calorie range recommendation!",'','error'
        );
        this.$router.replace({ name: "editdata" });
      } else if (this.checkCalorie()) {
        VueSimpleAlert.alert(
          "Your daily calorie goal is not within the recommended range, please enter again!",'','error'
        );
      } else {
        console.log("User daily goals input");
        database
          .collection("Users")
          .doc(auth.currentUser.email)
          .update({
            dailyTarget: {
              exercise: this.item.exercise,
              work: this.item.work,
              leisure: this.item.leisure,
              mask: this.item.mask,
              temperature: this.item.temperature,
              calorie: this.item.calorie,
              hand: this.item.hand,
            },
          })
          .catch((err) => {
            this.item.error = err.message;
          });
        VueSimpleAlert.alert("You have successfully submitted daily goals!",'','success');
        this.$router.replace({ name: "profile" });
      }
    },
  },
};
</script>
<style scoped>
form {
  padding-top: 80px;
  padding-bottom: 0px;
  padding-left: 50px;
  padding-right: 50px;
  width: 90%;
  height: 1500px;
  min-height: calc(100vh - 80px);
}

.form-control {
  height: 80%;
}
</style>
