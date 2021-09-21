<template>
  <section class="app-container">
    <loading :active.sync="isLoading"></loading>
    <h1 class="title">
      Membership Builder
    </h1>
    <div class="description">
      <div class="description-text">
        What type of membership would you like to purchase?
      </div>
    </div>

    <div class="family-wrapper">
      <div :key="index"  class="label-row" v-for="(group, index) in age_groups">
        <div class="label-row">
          <label class="label" :for="group.uuid">{{ group.title }}</label>
          <div class="value">
            <input type="checkbox" @change="setTotal" :value="group.uuid" :id="group.uuid"  v-model="$store.state.family[group.uuid].count" />
          </div>
        </div>
      </div>
    </div>
    <div class="navigation" v-if="totalCount">
      <div class="container">
        <button class="btn btn-next" @click="$emit('go-next')">Next</button>
      </div>
    </div>
  </section>
</template>

<script>
import Loading from "vue-loading-overlay";
import "vue-loading-overlay/dist/vue-loading.css";
import Cart from '../helpers/Cart';
export default {
  mounted() {

    this.isLoading = true;
    Cart.getAgeGroups().then(json => {
      let leave_items = {};
      this.age_groups = Object.keys(json).map((key) => {
        leave_items[json[key].uuid] = true;
        this.$store.commit('setFamilyTerm', {
          key: json[key].uuid,
          value: key
        })
        return {key: key, title: json[key].title, uuid: json[key].uuid};
      })
      Object.keys(this.$store.state.family).forEach((key) => {
        if (!leave_items[key]) {
          this.$store.commit('deleteFamilyKey', key);
        }
      })
      this.isLoading = false;
    }).catch(()=>{
      this.isLoading = false;
    });
  },
  computed: {
    family() {
      return this.$store.state.family;
    }
  },
  components: {
    Loading
  },
  data () {
    return {
      isLoading: false,
      age_groups: [],
      totalCount: this.getTotal(),
    }
  },
  methods: {
    updateFamily(key, value) {
      this.$store.commit('setFamily', {
        key,
        value
      })
    },
    setTotal() {
      let total = this.getTotal();
      this.totalCount = total;
    },
    getTotal() {
      let count = 0;
      Object.keys(this.$store.state.family).forEach(element => {
        let eCount = this.$store.state.family[element].count;
        if(eCount) {
          count = count + eCount;
        }
      });
      return count;
    },
  }
}
</script>
