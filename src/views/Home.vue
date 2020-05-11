<template>
  <div class="home">
    <v-container fill-height>
      <v-row align="center">
        <v-col>
          <Form  v-on:created="getExpenses"></Form>
        </v-col>
        <v-col>
          <line-chart v-if="loaded" :chartdata="chartData" :options="chartOptions" />
        </v-col>
      </v-row>
      <v-row>
        <v-data-table :headers="headers" :items="expenses" :items-per-page="15" class="elevation-1">
          <template v-slot:item.actions="{ item }">
            <v-icon small @click="deleteItem(item)">mdi-delete</v-icon>
          </template>
        </v-data-table>
      </v-row>
    </v-container>
  </div>
</template>

<script>
// @ is an alias to /src
import db from "@/firebase/init";
import color from "nice-color-palettes";
import lineChart from "@/components/LineChart";
import Form from "@/components/Form";
export default {
  name: "Home",
  components: { lineChart, Form },
  data() {
    return {
      expenses: [],
      headers: [
        { text: "Name", value: "name" },
        { text: "Cost", value: "cost" },
        {text: "category", value: "category"},
        { text: "Actions", value: "actions", sortable: false }
      ],
      loaded: false,
      chartData: null,
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false
      }
    };
  },

  methods: {
    getExpenses: function() {
      this.loaded = false;
      this.expenses = [];
      db.collection("expenses")
        .get()
        .then(snapshot => {
          var i = 0;
          var costs = [];
          var labels = [];
          var bgcolors = [];
          var j = 1;
          var k = 26;
          bgcolors.push(color[k][j]);
          snapshot.forEach(doc => {
            costs.push(doc.data().cost);
            labels.push(doc.data().name);
            var expense = doc.data();
            expense.id = doc.id;
            this.expenses.push(expense);
            if (i === snapshot.size - 1) {
              if (j === 4) {
                k = Math.floor(Math.random() * 100);
                j = 0;
                bgcolors.push(color[k][j]);
              } else {
                bgcolors.push(color[k][j]);
                j++;
              }
              this.loaded = true;
              this.chartData = {
                labels: labels,
                datasets: [
                  {
                    label: "Data One",
                    backgroundColor: bgcolors,
                    data: costs
                  }
                ]
              };
            } else {
              i++;
              if (j === 4) {
                k = Math.floor(Math.random() * 100);
                j = 0;
                bgcolors.push(color[k][j]);
              } else {
                j++;
                bgcolors.push(color[k][j]);
              }
            }
          });
        });
    },
    deleteItem: function(item) {
      console.log(item.id);
      db.collection("expenses")
        .doc(item.id)
        .delete()
        .then(() => {
          this.getExpenses();
        });
    }
  },
  created() {
    this.getExpenses();
  }
};
</script>
