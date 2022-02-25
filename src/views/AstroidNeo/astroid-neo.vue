<template>
  <div class="astroid-container">
    <div class="astroid-main-title">ASTROID NEO-STS</div>

    <form class="form-inline" role="search" method="post">
      <div class="form-group">
        <div>
          <label class="date-text">Start Date :</label>&nbsp; &nbsp;
          <input
            type="date"
            class="form-control"
            placeholder="Order Date From"
            name="searchDateFrom"
            @change="datePicker()"
            v-model="startdate"
            id="searchDateFrom"
          />
        </div>
        <div>
          <label class="date-text">End Date :</label>&nbsp; &nbsp;
          <input
            type="date"
            class="form-control"
            placeholder="Order Date To"
            name="searchDateTo"
            id="searchDateTo"
            v-model="enddate"
            @change="datePicker()"
          />
        </div>
        <div class="btn-section">
          <el-button
            type="primary"
            round
            v-on:click="Getdata()"
            v-loading="processlodading"
            v-bind:class="{ 'button-disabled': disableCutinue() }"
            >Submit</el-button
          >
        </div>
      </div>
    </form>
    <div v-if="astroidArray.length" style="margin:2rem 0;">
      <el-button type="success" plain @click="display = 'table'"
        >Table</el-button
      >
      <el-button type="info" plain @click="display = 'graph'">Chart</el-button>
    </div>

    <div
      v-if="astroidArray.length && display == 'table'"
      style="overflow: auto; height: 300px"
    >
      <table style="width: 100%; margin-top: 2rem">
        <tr>
          <th class="heading">Sr NO.</th>
          <th class="heading">ID</th>
          <th class="heading">Date</th>
          <th class="heading">Time</th>
          <th class="heading">Miss Distance(Km/hour)</th>
        </tr>
        <tr v-for="(asteriods, index) in astroidArray" :key="index">
          <td>{{ index + 1 }}</td>
          <td>{{ asteriods.id }}</td>
          <td>{{ dateformat(asteriods.close_approch_date) }}</td>
          <td>{{ timeformat(asteriods.close_approch_date) }}</td>
          <td>{{ asteriods.miss_distance.kilometers }}</td>
        </tr>
      </table>
    </div>
    <div>
      <GChart
        v-if="astroidArray.length && display == 'graph'"
        :astroGraphdata="astroGraphdata"
      />
    </div>
  </div>
</template>


<script>
import axios from "axios";
import moment from "moment";
import GChart from "../../components/vue-google-charts.vue";

export default {
  name: "AstroidNeo",
  components: {
    GChart,
  },
  data() {
    return {
      startdate: "",
      enddate: "",
      asteriodData: [],
      processlodading: false,
      astroidArray: [],
      astroGraphdata: [],
      display: "table",
    };
  },

  methods: {
    dateformat(date) {
      return moment(date).format("ll");
    },
    timeformat(time) {
      return moment(time).format("LT");
    },
    disableCutinue() {
      if (this.enddate == "" || this.startdate == "") {
        return true;
      } else {
        return false;
      }
    },
    datePicker() {
      let dtFrom = document.querySelector("#searchDateFrom").value;
      let dtTo = new Date(dtFrom);
      dtTo.setDate(dtTo.getDate() + 7);
      let sMonthTo =
        dtTo.getMonth() + 1 < 10
          ? "0" + (dtTo.getMonth() + 1)
          : dtTo.getMonth() + 1;
      let sDayTo = dtTo.getDate() < 10 ? "0" + dtTo.getDate() : dtTo.getDate();
      document
        .querySelector("#searchDateTo")
        .setAttribute(
          "max",
          dtTo.getFullYear() + "-" + sMonthTo + "-" + sDayTo
        );
    },
    Getdata() {
      if (this.disableCutinue()) {
        this.$message({
          message: "Please Select The Start & End Date First",
        });
        return;
      }
      this.processlodading = true;

      axios
        .get(
          `https://api.nasa.gov/neo/rest/v1/feed?start_date=${this.startdate}&end_date=${this.enddate}&api_key=DEMO_KEY`,
          {
            "response.headers": ["x-ratelimit-limit"],
            method: "POST",
            mode: "no-cors",
            cache: "no-cache",
            credentials: "same-origin",

            headers: {
              "content-type": "application/json",
            },
          }
        )
        .then((response) => {
          if (response.status == 200) {
            this.processlodading = false;
            var astroid = response.data.near_earth_objects;
            var count = {};
            for (const property in astroid) {      
              for (var i = 0; i < astroid[property].length; i++) {
                var obj = {
                  id: astroid[property][i].id,
                  close_approch_date:
                    astroid[property][i].close_approach_data[0]
                      .close_approach_date_full,
                  miss_distance:
                    astroid[property][i].close_approach_data[0].miss_distance,
                };
                count[
                  this.dateformat(
                    astroid[property][i].close_approach_data[0]
                      .close_approach_date_full
                  )
                ] =
                  (count[
                    this.dateformat(
                      astroid[property][i].close_approach_data[0]
                        .close_approach_date_full
                    )
                  ] || 0) + 1;
                this.astroidArray.push(obj);
              }
            }
            this.astroGraphdata.push(["Date", "No. Of Astroids"]);

            Object.keys(count).forEach((key) =>
              this.astroGraphdata.push([moment(key).calendar(), count[key]])
            );
          }
        })
        .catch((err) => {
          console.log(err);
        });
    },
  },
};
</script>


<style >
@import url("./astroid-neo.css");
</style>
