<template>
  <v-form ref="form" v-model="valid" lazy-validation>
    <v-text-field v-model="name" :rules="nameRules" label="Name" required outlined></v-text-field>

    <v-text-field v-model="cost" :rules="costRules" label="Cost" required outlined suffix="€"></v-text-field>

    <v-autocomplete
      v-model="select"
      :items="items"
      :rules="[v => !!v || 'Item is required']"
      label="Category"
      required
      outlined
      item-text="name"
    ></v-autocomplete>

    <v-btn :disabled="!valid" color="success" class="mr-4" @click="validate">Validate</v-btn>
  </v-form>
</template>

<script>
import db from "@/firebase/init";
export default {
  data: () => ({
    valid: true,
    name: "",
    nameRules: [v => !!v || "Name is required"],
    cost: "",
    costRules: [v => !!v || "Cost is required"],
    select: null,
    items: [],
  }),

  methods: {
    validate() {
      if (this.$refs.form.validate()) {
        this.sendData();
        this.$emit("created");

        this.name = "";
        this.cost = "";
        this.select = "";
        this.$refs.form.reset();
      }
    },
    sendData: function() {
      db.collection("expenses")
        .add({
          cost: this.cost,
          name: this.name,
          date: Date.now(),
          category: this.select
        })
        .then(res => {
          console.log(res);
        })
        .catch(err => {
          console.log("Error", err);
        });
    }
  },

  created() {
    db.collection("categories")
      .get()
      .then(snapshot => {
        snapshot.forEach(doc => {
          let item = doc.data();
          item.id = doc.id;
          this.items.push(item);
          console.log(this.items);
        });
      })
      .catch(err => {
        console.log(err);
      });
  }
};
</script>