<template>
  <div class="map">
    <h3>Карта офиса</h3>

    <div v-if="!isLoading" class="map-root">
      <MapSVG ref="svg" />
      <Table v-show="false" ref="table" />
    </div>
    <div v-else>Loading...</div>
  </div>
</template>

<script>
import MapSVG from "@/assets/images/map.svg";
import Table from "@/assets/images/workPlace.svg";
import tables from "@/assets/data/tables.json";
import legend from "@/assets/data/legend.json";
import people from "@/assets/data/people.json";
import * as d3 from "d3";

export default {
  components: {
    MapSVG,
    Table,
  },
  created() {
    this.tables = tables;
  },
  data() {
    return {
      isLoading: false,
      svg: null,
      g: null,
      tableSVG: null, // добавляем шаблон стола
      tables: [],
    };
  },
  mounted() {
    this.isLoading = true;

    this.svg = d3.select(this.$refs.svg);
    this.g = this.svg.select("g");
    this.tableSVG = d3.select(this.$refs.table); // получаем шаблон стола

    if (this.g) {
      this.drawTables();
    } else {
      alert("SVG is incorrect");
    }

    this.isLoading = false;
  },
  methods: {
    emitPerson(tableId) {
      this.$emit(
        "set-person",
        people.find((p) => p.tableId === tableId)
      );
    },
    drawTables() {
      // создаем группу для рабочик мест
      const svgTablesGroupPlace = this.g
        .append("g")
        .classed("groupPlaces", true);

      this.tables.map((table) => {
        // создает группу для рабочего стола
        const targetSeat = svgTablesGroupPlace
          .append("g")
          .attr("transform", `translate(${table.x}, ${table.y}) scale(0.5)`)
          .attr("id", table._id)
          .classed("employer-place", true)
          .on("click", (event) => {
            event.stopPropagation();
            this.emitPerson(table._id);
          });

        // устанавливает стол в группу
        targetSeat
          .append("g")
          .attr("transform", `rotate(${table.rotate || 0})`)
          .attr("group_id", table.group_id)
          .classed("table", true)
          .html(this.tableSVG.html());

        targetSeat
          .append("g")
          .attr("transform", `rotate(${table.rotate || 0})`)
          .attr("group_id", table.group_id)
          .classed("table", true)
          .html(this.tableSVG.html())
          .attr(
            // устанавливаем цвет подразделения
            "fill",
            legend.find((it) => it.group_id === table.group_id)?.color ??
              "transparent"
          );
      });
    },
  },
};
</script>

<style scoped>
.map {
  height: 100%;
  width: 100%;
  padding: 24px;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.map-root {
  height: 100%;
  width: 100%;
  overflow: hidden;
  box-sizing: border-box;
}

h3 {
  margin-top: 0;
}

::v-deep svg {
  height: 100%;
  width: 100%;
}

::v-deep .table {
  cursor: pointer;
}
</style>
