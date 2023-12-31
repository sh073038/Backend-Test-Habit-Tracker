<template>
  <transition name="fade">
    <yayy v-show="yayy"></yayy>
  </transition>
  <div class="container">
    <!-- CREATE HABIT -->
    <div class="row">
      <div class="col-sm-12">
        <div class="d-flex justify-content-end align-items-center mb-4">
          <toggle
            color="dark"
            @toggle="toggleHabitsForm"
            :visibility="habitFormVisible"
          >
            Create habit
          </toggle>
        </div>
      </div>
    </div>

    <transition name="slide-in">
      <div
        v-if="habitFormVisible"
        class="row justify-content-center habit-form"
      >
        <div class="col-lg-6 mb-5">
          <habit-form
            @onSubmit="onHabitFormSubmit"
            :loading="habitFormLoading"
            :error="habitFormError"
          ></habit-form>
        </div>
      </div>
    </transition>
    <!--/ CREATE HABIT -->

    <!-- INDEX HABITS -->
    <div class="row">
      <div class="col-sm-12">
        <div
          class="
            d-flex
            flex-column
            align-items-center
            justify-content-center
            py-2
          "
        >
          <div class="d-flex justify-content-center align-items-center">
            <h1 class="m-0">Habits</h1>
            <info-tooltip
              title="Your existing habits."
              placement="right"
            ></info-tooltip>
          </div>
          <div class="py-3" v-if="habits.length < 1">
            <p class="alert alert-info p-2">No habits found.</p>
          </div>
        </div>
        <div class="row py-4" v-if="habits.length > 0">
          <!-- INDEX GOOD HABITS -->
          <div class="col-lg-6 mb-4 mb-lg-0">
            <div class="text-center mb-3">
              <h4 class="">Good Habits</h4>
            </div>
            <div
              v-for="habit in goodHabits"
              :key="habit.id"
              class="habit-wrapper mb-2"
            >
              <habit
                :habit="habit"
                @createRep="createHabitRep"
                @deleteRep="deleteHabitRep"
                :repetitionLoading="repetitionLoading"
              ></habit>
            </div>
          </div>
          <!--/ INDEX GOOD HABITS -->

          <!-- INDEX BAD HABITS -->
          <div class="col-lg-6">
            <div class="text-center mb-3">
              <h4 class="">Bad Habits</h4>
            </div>
            <div
              v-for="habit in badHabits"
              :key="habit.id"
              class="habit-wrapper mb-2"
            >
              <habit
                :habit="habit"
                @createRep="createHabitRep"
                @deleteRep="deleteHabitRep"
                :repetitionLoading="repetitionLoading"
              ></habit>
            </div>
          </div>
          <!--/ INDEX BAD HABITS -->
        </div>
      </div>
    </div>
    <!-- INDEX HABITS -->
  </div>
</template>
<style lang="scss" scoped></style>
<script>
import Yayy from '@/components/Yayy.vue';
import Habit from '@/components/Habit.vue';
import HabitForm from '@/components/HabitForm.vue';
import InfoTooltip from '@/components/InfoTooltip.vue';
import Toggle from '@/components/Toggle.vue';
import { ref, computed } from 'vue';
import { useStore } from 'vuex';

export default {
  name: 'Habits',
  components: { Yayy, Habit, HabitForm, Toggle, InfoTooltip },
  setup() {
    const store = useStore();
    const habitFormVisible = ref(false);
    const habitFormLoading = ref(false);
    const habitFormError = ref(null);
    const yayy = ref(false);
    const onHabitFormSubmit = async (formData) => {
      habitFormLoading.value = true;
      try {
        await store.dispatch('habits/create', {
          formData: formData,
        });
      } catch (error) {
        habitFormError.value = error;
      } finally {
        habitFormLoading.value = false;
        if (!habitFormError.value) {
          habitFormVisible.value = false;
        }
      }
    };
    const toggleHabitsForm = () => {
      habitFormVisible.value = !habitFormVisible.value;
      habitFormError.value = null;
    };

    const repetitionLoading = ref(null);
    const createHabitRep = async (habit) => {
      repetitionLoading.value = habit.id;
      habitFormError.value = null;
      let error;
      try {
        await store.dispatch('habits/createRepetition', {
          habit: habit.id,
          repetition: {
            date: new Date(),
            habit: habit.id,
          },
        });
      } catch (e) {
        error = e;
      } finally {
        repetitionLoading.value = null;
        if (!error && habit.type == 'G') {
          // Show success message only when a good habit is completed
          yayy.value = true;
          setTimeout(() => {
            yayy.value = false;
          }, 1000);
        }
      }
    };
    const deleteHabitRep = async (habit) => {
      repetitionLoading.value = habit.id;
      try {
        await store.dispatch('habits/deleteRepetition', {
          habit: habit.id,
          pk: habit.todays_repetition.id,
          repetition: habit.todays_repetition,
        });
      } finally {
        repetitionLoading.value = null;
      }
    };

    return {
      habitFormVisible,
      habitFormLoading,
      onHabitFormSubmit,
      toggleHabitsForm,
      repetitionLoading,
      createHabitRep,
      deleteHabitRep,
      habitFormError,
      yayy,
      goodHabits: computed(() => store.getters['habits/good']),
      badHabits: computed(() => store.getters['habits/bad']),
      habits: computed(() => {
        return store.state.habits.models;
      }),
    };
  },
};
</script>
