<template>
  <div>
    <div v-if="user" class="page-header clear-filter" color="orange">
      <div
        class="page-header-image"
        style="background-image: url('img/inputdata-bg.jpg'); opacity: 0.4"
        v-bind:style="(height = '100px')"
      ></div>

      <form>
        <h2 class="greeting" style="color: black">We need your health data!</h2>
        <div class="form-row">
          <div class="form-group col-md-6">
            <label for="inputGender" style="color: black">Gender*</label>
            <select
              id="inputGender"
              class="form-control"
              v-model="item.gender"
              required
            >
              <option selected>FEMALE</option>
              <option>MALE</option>
            </select>
          </div>
          <div class="form-group col-md-6">
            <label for="inputAge" style="color: black">Age*</label>
            <input
              type="number"
              class="form-control"
              id="inputAge"
              v-model.number="item.age"
              required
              placeholder="30"
            />
          </div>
        </div>

        <div class="form-group">
          <label for="inputHeight" style="color: black">Height (cm)*</label>
          <input
            type="number"
            class="form-control"
            id="inputTemperature"
            placeholder="170"
            v-model.number="item.height"
            required
          />
        </div>
        <div class="form-group">
          <label for="inputWeight" style="color: black">Weight (kg)*</label>
          <input
            type="float"
            class="form-control"
            id="inputWeight"
            placeholder="60.0"
            v-model.number="item.weight"
            required
          />
        </div>

        <div class="form-group">
          <label for="inputPhysicalActivity" style="color: black"
            >Physical Activity Level*</label
          >
          <select
            id="inputPhysicalActivity"
            class="form-control"
            v-model="item.pal"
            required
          >
            <option selected>Sedentary (little or no exercise)</option>
            <option>Moderately active</option>
            <option>Vigorously active</option>
            <option>Extremely active</option>
          </select>
        </div>
        <div class="form-group">
          <label for="inputWeightGoal" style="color: black"
            >Weight Goal (kg)*</label
          >
          <input
            type="float"
            class="form-control"
            id="inputExerciseDuration"
            placeholder="60.0"
            v-model.number="item.weightGoal"
            required
          />
        </div>
        <div class="form-group">
          <label for="inputSpecialCondition" style="color: black"
            >Special Physical Condition*</label
          >
          <input
            type="text"
            class="form-control"
            id="inputExerciseDuration"
            placeholder="None"
            v-model="item.specialPhysicalCondition"
            required
          />
        </div>
        <button class="btn btn-primary btn-round" v-on:click.prevent="addItem">
          SUBMIT{{ getdata2 }}
        </button>
      </form>
    </div>

    <div v-else class="section">
      <div class="container">
        <div class="alert alert-danger">Please log in first</div>
      </div>
    </div>
  </div>
</template>
<script>
import { database, storage } from "../firebase.js";
import firebase from "firebase";
import auth from "../firebase.js";
import VueSimpleAlert from "vue-simple-alert";

export default {
  name: "editdata",
  bodyClass: "form-page",
  components: {},
  data() {
    return {
      item: {
        name: "",
        gender: "",
        height: 0,
        weight: 0,
        pal: "",
        weightGoal: 0,
        specialPhysicalCondition: "",
        age: 0,
        calorieMax: 0,
        calorieMin: 0,
        l: 0,
        s: 0,
        BMR: 0,
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
    },
  },
  methods: {
    writeData() {
      console.log("User health data input");
      database
        .collection("Users")
        .doc(auth.currentUser.email)
        .update({
          gender: this.item.gender,
          height: this.item.height,
          weight: this.item.weight,
          pal: this.item.pal,
          weightGoal: this.item.weightGoal,
          specialPhysicalCondition: this.item.specialPhysicalCondition,
          age: this.item.age,
          calorieMax: this.item.calorieMax,
          calorieMin: this.item.calorieMin,
          name: auth.currentUser.displayName,
        })
        .catch((err) => {
          this.item.error = err.message;
        });
      VueSimpleAlert.alert(
        "You have successfully submitted health data!",
        "",
        "success"
      );
    },
    checkCalorieGoal() {
      if ("dailyTarget" in this.data) {
        if (this.data.dailyTarget.calorie < this.item.calorieMin) {
          return true;
        } else if (this.data.dailyTarget.calorie > this.item.calorieMax) {
          return true;
        } else {
          return false;
        }
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
    computeCal() {
      if (this.item.pal == "Sedentary (little or no exercise)") {
        this.item.l = 1.2;
      } else if (this.item.pal == "Moderately active") {
        this.item.l = 1.375;
      } else if (this.item.pal == "Vigorously active") {
        this.item.l = 1.55;
      } else {
        this.item.l = 1.8;
      }
      if (this.item.gender == "FEMALE") {
        this.item.s = -161;
      } else {
        this.item.s = 5;
      }
      this.item.BMR =
        (10 * this.item.weight +
          6.25 * this.item.height -
          5 * this.item.age +
          this.item.s) *
        this.item.l;
      this.item.calorieMax = Math.round(this.item.BMR + 500);
      this.item.calorieMin = Math.round(this.item.BMR - 500);
    },
    addItem() {
      this.computeCal();
      if (
        this.item.gender === "" ||
        this.item.height === "" ||
        this.item.weight === "" ||
        this.item.pal === "" ||
        this.item.weightGoal === "" ||
        this.item.specialPhysicalCondition === "" ||
        this.item.age === ""
      ) {
        VueSimpleAlert.alert("Please fill in empty fields!", "", "error");
      } else if (
        this.item.height <= 0 ||
        this.item.weight <= 0 ||
        this.item.weightGoal <= 0 ||
        this.item.age <= 0
      ) {
        VueSimpleAlert.alert(
          "Input value is not valid. Please check again before submission!",
          "",
          "error"
        );
      } else if (this.checkCalorieGoal()) {
        alert(
          "Your current daily calorie goal is no longer within our recommended range, you will be directed to goal page to modify your daily goal!",
          "",
          "error"
        );
        this.writeData();
        this.$router.replace({ name: "profile" });
        this.$router.replace({ name: "editgoal" });
      } else {
        this.writeData();
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