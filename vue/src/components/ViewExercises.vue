<template>
  <div class="main">
    <update-exercise v-if="$store.state.showEdit === true" />
    <add-workout v-if="$store.state.showAddWorkout === true" />
    <div
      v-show="$store.state.showEdit != true &&  $store.state.showAddWorkout != true"
      class="exerciseDiv"
      v-for="exercise in filteredExercises"
      v-bind:exercise="exercise"
      :key="exercise.id"
    >
      <h2>{{ exercise.name }}</h2>
      <div vr id="imgDiv">
        <div>
          <h4 class="group">{{ exercise.muscleGroup }}</h4>
          <h5 class="user_id" v-if="exercise.userId > 0"> {{exercise.userId}} </h5>
          <h5 class="type" v-show="exercise.type != 'Cardio'">
            {{ exercise.type }}
          </h5>
          <h6 v-show="exercise.type != 'Cardio'">Reps:</h6>
          <h4 class="reps" v-show="exercise.type != 'Cardio'">{{ exercise.repRange }}</h4>
          <h6>Time:</h6>
          <h4 class="time">{{ exercise.timeRange }} Mins </h4>
        </div>
        <img
          class="img"
          :src="`../WorkoutImages/${exercise.muscleGroup}.png`"
        />
      </div>
      <p id="desc">{{ exercise.description }}</p>
      
        <div class="btnDiv">
          <button class="add" 
          v-on:click="toggleAddWorkout();
           targetExercise(exercise)">Add <span id="AddTo"> To Workout </span> </button>
          <button
            class="edit"
            v-if="isAuthorized"
            v-on:click="
              toggleEditButton();  
              targetExercise(exercise);
            ">
            Edit <span>Exercise</span>
          </button>
          <button
            class="delete"
            v-if="isAuthorized"
            v-on:click="deleteExercise(exercise)"
          >
            Delete <span>Exercise</span>
          </button>
        </div>
      
    </div>
  </div>
</template>

<script>
import UpdateExercise from "../components/UpdateExercise.vue";
import exerciseService from "../services/ExerciseService";
import addWorkout from "../components/AddWorkout.vue";
import workoutService from "../services/WorkoutService";

export default {
  name: "view-exercises",

  data() {
    return {
      showEdit: false,

      targetedExercise: {},

      exercises: [],
    };
  },
  components: {
    UpdateExercise,
    addWorkout
  },

  created() {
    this.getExercises();
  },

  computed: {
    filteredExercises() {
      const exerciseFilter = this.$store.state.filter;
      const exercises = this.$store.state.exercises;
      return exercises.filter((exercise) => {
        return exercise.statusId === 2 && exerciseFilter == ""
          ? true
          : exerciseFilter == exercise.muscleGroup;
      });
    },

    isAuthorized() {
      if (this.$store.state.user.authorities[0].name === "ROLE_TRAINER") {
        return true;
      } else {
        return false;
      }
    },
  },

  methods: {
    getExercises() {
      exerciseService.getExercisesByStatus(2).then((response) => {
        let exercises = response.data;  
        this.$store.state.exercises = exercises;      
      });
    },

    toggleEditButton() {
      this.$store.state.showEdit = true
       this.$store.state.showAddWorkout = false;
    },

    toggleAddWorkout() {
      this.$store.state.showAddWorkout = true
      this.$store.state.showEdit = false;
    },

    targetExercise(exercise) {
      this.targetedExercise = exercise;
      this.$store.commit("SELECT_EXERCISE", this.targetedExercise);
    },

    deleteExercise(exercise) {
      workoutService.deleteExerciseWorkout(exercise.id).then((response) => {
          if (response.status == 200){
      exerciseService
        .deleteExercise(exercise)
        .then((response) => {
          console.log(response);
          if (response.status == 200) {
            this.$store.commit("DELETE_EXERCISE", exercise.id);
            this.getExercises();
          
          }
        })
        .catch((error) => {
          const response = error.response;

          if (response.status === 401) {
            this.createError = true;
          }
        });
          }
      })
        
    },

  },
};
</script>

<style scoped>
.main {
  display: flex;
  max-width: 100%;
  flex-wrap: wrap;
  min-width: 320px;
}

#imgDiv {
  display: flex;
  justify-content: space-between;
  align-content: center;
}

img {
  height: 100px;
  margin-bottom: 10px;
  border-radius: 5px;
  margin-right: 40px;
}
p{
  margin-top: 10px ;
}
.exerciseDiv {
  display: inline-block;
  margin: 20px;
  padding: 15px;
  border-radius: 8px/7px;
  background-color: #ebebeb;
  box-shadow: 1px 2px 5px black;
  border: solid 1px #cbc9c9;
  max-width: 275px;
  min-width: 275px;
  height: max-content;
}

.btnDiv {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}


.edit {
  background-color: rgb(161, 161, 38);
}

.delete {
  background-color: red;
}
button {
  font-size: 14px;
  font-weight: 600;
  color: white;
  text-decoration: none;
  width: 120px;
  height: 30px;
  border-radius: 5px;
  background-color: #00afef;
  box-shadow: 0 3px rgba(58, 87, 175, 0.75);
  padding: 5px;
  margin: 15px;
}

.add:hover {
  top: 3px;
  border: #4c4c4c;
}

@media only screen and (max-width: 600px) {
  .main {
    display: flex;
    justify-content: center;
  }
}
</style>