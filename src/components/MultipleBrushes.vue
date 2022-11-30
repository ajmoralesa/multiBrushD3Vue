<template>
  <div>
    <svg :height="height" :width="width">
      <g>
        <rect :width="width" :height="height" class="rect"></rect>
        <g ref="gBrushes"></g>
      </g>
    </svg>
  </div>
</template>

<script setup>
import { computed } from "@vue/reactivity";
import { onMounted, reactive, ref } from "vue";
import * as d3 from "d3";

const margin = reactive({
  top: 10,
  right: 10,
  bottom: 10,
  left: 10,
});
const width = 960 - margin.left - margin.right;
const height = 500 - margin.top - margin.bottom;

const gBrushes = ref();

// Keep the actual d3-brush functions and their IDs in a list:
const brushes = ref([]);

onMounted(() => {
  d3.select(gBrushes.value).call(newBrush.value);
});

const newBrush = computed(() => {
  var brush = d3.brush().on("end", brushend);

  function brushend() {
    brushes.value.push({ id: brushes.value.length, brush: brush });
    // // Figure out if our latest brush has a selection
    // var lastBrushID = brushes.value[brushes.value.length - 1].id;
    // var lastBrush = document.getElementById("brush-" + lastBrushID);
    // console.log("lastBrush:", lastBrush);

    // var selection = d3.brushSelection(lastBrush);

    // // If it does, that means we need another one
    // if (selection && selection[0] !== selection[1]) {
    //   newBrush();
    // }

    // Always draw brushes
    drawBrushes();
  }

  return brush;
});

function drawBrushes() {
  console.log("hey there!");

  var brushSelection = d3
    .select(gBrushes.value)
    .selectAll(".brush")
    .data(brushes.value, function (d) {
      return d.id;
    });

  // Set up new brushes
  brushSelection
    .enter()
    .insert("g", ".brush")
    .attr("class", "brush")
    .attr("id", function (brush) {
      return "brush-" + brush.id;
    })
    .each(function (brushObject) {
      //call the brush
      d3.select(this).call(d3.brush(brushObject));
    });

  /* REMOVE POINTER EVENTS ON BRUSH OVERLAYS
   *
   * This part is abbit tricky and requires knowledge of how brushes are implemented.
   * They register pointer events on a .overlay rectangle within them.
   * For existing brushes, make sure we disable their pointer events on their overlay.
   * This frees the overlay for the most current (as of yet with an empty selection) brush to listen for click and drag events
   * The moving and resizing is done with other parts of the brush, so that will still work.
   */
  brushSelection.each(function (brushObject) {
    d3.select(this)
      .attr("class", "brush")
      .selectAll(".overlay")
      .style("pointer-events", function () {
        var brush = brushObject.brush;
        if (
          brushObject.id === brushes.value.length - 1 &&
          brush !== undefined
        ) {
          return "all";
        } else {
          return "none";
        }
      });
  });

  brushSelection.exit().remove();
}
</script>

<style scoped>
.rect {
  fill: #eee;
}
</style>
