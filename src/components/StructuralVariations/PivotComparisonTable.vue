<template>
  <div class="structural-variations-table">
    <!--    <div class="table-tabs-container">-->
    <!--      <v-tabs v-model="selectedTab" class="table-tabs d-flex">-->
    <!--        <v-tab class="elevation-2 ml-5">-->
    <!--          Millipides-->
    <!--        </v-tab>-->
    <!--        <v-tab class="elevation-2 ml-5">-->
    <!--          Pan Alignment-->
    <!--        </v-tab>-->
    <!--      </v-tabs>-->
    <!--    </div>-->
    <div class="table-container">
      <v-card tile>
        <div v-if="pivot && pivot.path && pivot.path.steps" class="data-area d-flex flex-row">

          <!-- "Header" column -->
          <div class="data-labels col-2 d-flex flex-column pr-0 mr-n1">

            <div v-for="assembly in beforePivotRows" :key="`assembly-header-${assembly.name}`"
                 class="data-label elevation-1 px-3">
              {{ assembly.name }}
            </div>

            <div class="data-label elevation-1 pivot-label-row px-3 d-flex">
              <div class="arrows d-flex flex-column justify-space-around">
                <a class="up-arrow" @click="movePivotUp"></a>
                <a class="down-arrow" @click="movePivotDown"></a>
              </div>
              <div class="pivot-label">
                {{ pivot.name }}
              </div>
            </div>

            <div v-for="assembly in afterPivotRows" :key="`assembly-header-${assembly.name}`"
                 class="data-label elevation-1 px-3">
              {{ assembly.name }}
            </div>

          </div>

          <!-- "Data" column rows -->
          <div class="data-block-rows col-10 d-flex flex-column pl-0 mr-1">

            <div v-for="(assembly, aIndex) in beforePivotRows" :key="`assembly-row-${assembly.name}`"
                 class="data-block-row d-flex flex-row">
              <!--              , {'selected-block': isBlockSelected(block, assembly)}-->
              <!--              @click="selectBlock(block, assembly)"-->
              <div v-for="(pivotStep, psIndex) in pivot.path.steps"
                   :key="`assembly-row-${assembly.name}-step-${pivotStep.panBlock}`"
                   :class="['data-block-column', `block-${psIndex % 2}`, `elevation-1`, 'above-pivot',
                   {
                     'pivot-neighbor': aIndex === beforePivotRows.length - 1,
                     'selected': selectedBlock && selectedBlock.assemblyName === assembly.name && selectedBlock.blockName === pivotStep.panBlock,
                     'outside-range':
                     pivotStep.startPosition < positionFilter[0] ||
                     pivotStep.startPosition > positionFilter[1] ||
                     (

                       (
                         getVariationLength(pivotStep.panBlock, assembly.name) < lengthFilter[0] ||
                         getVariationLength(pivotStep.panBlock, assembly.name) > lengthFilter[1]
                         )
                     ) ||
                     isNotInSVSelection(pivotStep.panBlock, assembly.name),
                   }]"
                   @click="selectBlock(assembly.name, pivotStep.panBlock)"
              >
                <div class="block-count-label">{{ getVariationLength(pivotStep.panBlock, assembly.name) }}</div>
                <div v-for="blockClass in blockClasses(pivotStep.panBlock, assembly.name)"
                     :key="`assembly-row-${assembly.name}-step-${pivotStep.panBlock}-block-${blockClass}`"
                     :class="[ blockClass, 'data-block-cell']"></div>
                <!--                , `block-type-${getUpperBlock(block, assembly)}`-->
                <!--                <div :class="['data-block-cell-top']"></div>-->
                <!--                , `block-type-${getLowerBlock(block, assembly)}`-->
                <!--                <div :class="['data-block-cell-bottom']"></div>-->
              </div>
            </div>

            <div class="pivot-data-block-row d-flex flex-row">
              <div v-for="(pivotStep, psIndex) in pivot.path.steps"
                   :key="`pivot-row-${pivot.name}-step-${pivotStep.panBlock}`"
                   :class="[
                   'data-block-column', `pivot-block-${psIndex % 2}`, `elevation-1`,
                   {
                      'outside-range':
                     pivotStep.startPosition < positionFilter[0] ||
                     pivotStep.startPosition > positionFilter[1]
                   }]">
                <div :class="['pivot-data-block-cell']"
                     :style="{ background: pivotColor(pivotStep.panBlock, assemblies) }"></div>
              </div>
            </div>


            <div v-for="(assembly, aIndex) in afterPivotRows" :key="`assembly-row-${assembly.name}`"
                 class="data-block-row d-flex flex-row">
              <!--              , {'selected-block': isBlockSelected(block, assembly)}-->
              <!--              @click="selectBlock(block, assembly)"-->
              <div v-for="(pivotStep, psIndex) in pivot.path.steps"
                   :key="`assembly-row-${assembly.name}-step-${pivotStep.panBlock}`"
                   :class="['data-block-column', `block-${psIndex % 2}`, `elevation-1`, 'bloe-pivot',
                   {
                     'pivot-neighbor': aIndex === 0,
                     'selected': selectedBlock && selectedBlock.assemblyName === assembly.name && selectedBlock.blockName === pivotStep.panBlock,
                     'outside-range':
                     pivotStep.startPosition < positionFilter[0] ||
                     pivotStep.startPosition > positionFilter[1] ||
                     (

                       (
                         getVariationLength(pivotStep.panBlock, assembly.name) < lengthFilter[0] ||
                         getVariationLength(pivotStep.panBlock, assembly.name) > lengthFilter[1]
                         )
                     ) ||
                     isNotInSVSelection(pivotStep.panBlock, assembly.name),
                   }]"
                   @click="selectBlock(assembly.name, pivotStep.panBlock)"
              >
                <div class="block-count-label">{{ getVariationLength(pivotStep.panBlock, assembly.name) }}</div>
                <div v-for="blockClass in blockClasses(pivotStep.panBlock, assembly.name)"
                     :key="`assembly-row-${assembly.name}-step-${pivotStep.panBlock}-block-${blockClass}`"
                     :class="[ blockClass, 'data-block-cell']"></div>
                <!--                , `block-type-${getUpperBlock(block, assembly)}`-->
                <!--                <div :class="['data-block-cell-top']"></div>-->
                <!--                , `block-type-${getLowerBlock(block, assembly)}`-->
                <!--                <div :class="['data-block-cell-bottom']"></div>-->
              </div>
            </div>

          </div>
        </div>
      </v-card>
    </div>
  </div>
</template>

<script>

import { computed, defineComponent, ref } from "vue";
import { useStore } from "vuex";
import { getData } from "@/data/data-source";
import { reactiveVuex } from "@/store/helper";
// import {selectedAssemblyNameKeys, selectedChromosome, selectedPivotName} from '@/data/some-data-source';

export default defineComponent({
  name: "StructuralVariationsPivotComparisonTable",
  components: {},
  setup() {
    const store = useStore();
    const selectedPivotName = reactiveVuex(store, "selectedPivotName", "setselectedPivotName");
    const selectedAssemblyNameKeys = reactiveVuex(store, "selectedAssemblyNameKeys", "setselectedAssemblyNameKeys");
    const selectedSVTypeNames = reactiveVuex(store, "selectedSVTypeNames", "setselectedSVTypeNames");
    const selectedBlock = reactiveVuex(store, "selectedBlock", "setSelectedBlock");

    const lengthFilter = reactiveVuex(store, "lengthFilter", "setLengthFilter"); // ref((Math.round(limitLength * 0.01) / lengthStep) * lengthStep);
    const positionFilter = reactiveVuex(store, "positionFilter", "setPositionFilter"); // ref((Math.round(limitPosition * 0.01) / positionStep) * positionStep);

    const paths = ref({});
    const pangenome = ref();
    const pivots = ref();

    getData().then((data) => {
      if (data) {
        pangenome.value = data.pangenome;
        pivots.value = data.pivots;

        const pathNames = Object.keys(pangenome.value.paths);

        paths.value = pathNames.reduce((result, pathName) => ({
          ...result,
          [pathName]: data.pangenome.paths[pathName]
        }), {});
      }
    });

    const pivotRowIndex = ref(1);

    const pivot = computed(() => ({ name: selectedPivotName.value, path: paths.value[selectedPivotName.value] }));
    const assemblies = computed(() => Object.keys(paths.value).filter(pathName => selectedAssemblyNameKeys.value[pathName]).map(pathName => ({
      name: pathName,
      path: paths.value[pathName]
    })));

    const getBlock = (nodeName, pathName) => {
      if (pivots.value) {
        const nodes = pivots.value[selectedPivotName.value];
        if (nodes) {
          const node = nodes[nodeName];
          if (node) {
            const pathBlock = node[pathName];
            return pathBlock;
          }
        }
      }
    };

    const getVariationLength = (nodeName, pathName) => {
      const block = getBlock(nodeName, pathName);
      return block.variationLength;
    };


    const isNotInSVSelection = (nodeName, pathName) => {
      if (!selectedSVTypeNames.value || !selectedSVTypeNames.value.length) {
        return false;
      }
      const block = getBlock(nodeName, pathName);
      const found = Object.entries(block).filter(([key, value]) => !!value).find(([key, value]) => selectedSVTypeNames.value.includes(key));
      return !found;
    };

    // const ispresent = (pivotName, nodeName, pathName) => {
    //   const pathBlock = getBlock(pivotName, nodeName, pathName);
    //   console.log("ispresent", pivotName, nodeName, pathName, pathBlock && pathBlock.present);
    //   return pathBlock && pathBlock.present;
    // };

    const pivotColor = (nodeName, paths) => {
      const blocks = paths.map(path => getBlock(nodeName, path.name));
      const presentCount = blocks.reduce((result, block) => {
        if (block && block.present) {
          result++;
        }
        return result;
      }, 0);
      const totalCount = blocks.length;
      const percentpresent = (presentCount / totalCount);
      const presentColor = {
        red: 236,
        green: 135,
        blue: 1
      };
      const absentColor = {
        red: 0,
        green: 159,
        blue: 236
      };
      const colors = {
        red: ((presentColor.red - absentColor.red) * percentpresent) + (absentColor.red),
        green: ((presentColor.green - absentColor.green) * percentpresent) + (absentColor.green),
        blue: ((presentColor.blue - absentColor.blue) * percentpresent) + (absentColor.blue)
      };
      return `rgb(${colors.red}, ${colors.green}, ${colors.blue}`;
    };

    const blockClasses = (nodeName, pathName) => {
      const pathBlock = getBlock(nodeName, pathName);
      const cssClasses = [`block-${nodeName}`];
      if (pathBlock) {
        const props = Object.keys(pathBlock);

        cssClasses.push(...props.map((prop) => {
          const value = pathBlock[prop];
          if (value === true) {
            return `block-${prop.toLowerCase()}`;
          } else if (typeof value === "string") {
            return `block-${prop.toLowerCase()}-${value.toLowerCase()}`;
          }
        }).filter((value) => !!value));
      }
      return cssClasses;
    };

    const beforePivotRows = computed(() => assemblies.value.slice(0, pivotRowIndex.value));
    const afterPivotRows = computed(() => assemblies.value.slice(pivotRowIndex.value));

    const movePivotUp = () => (pivotRowIndex.value > 0) ? pivotRowIndex.value-- : undefined;
    const movePivotDown = () => (pivotRowIndex.value < assemblies.value.length) ? pivotRowIndex.value++ : undefined;

    const selectBlock = (assembly, block) => {
      selectedBlock.value = { pivot: selectedPivotName.value, assembly, block };
      console.log("selectedBlock", selectedBlock.value);
    };

    return {
      pivot,
      assemblies,
      blockClasses,
      pivotColor,
      pivotRowIndex,
      beforePivotRows,
      afterPivotRows,
      movePivotUp,
      movePivotDown,
      selectedBlock,
      selectBlock,

      lengthFilter,

      positionFilter,

      getBlock,

      getVariationLength,

      isNotInSVSelection
    };
  }
});
</script>

<style lang="scss" scoped>

.arrows {
  position: absolute;
  height: 100%;
  left: 0.25rem;
  padding: 0.125rem 0;
}

.up-arrow, .down-arrow {
  display: block;
}

.up-arrow {
  width: 0;
  height: 0;
  border-left: 0.3rem solid transparent;
  border-right: 0.3rem solid transparent;

  border-bottom: 0.5rem solid #aaa;

}

.down-arrow {
  width: 0;
  height: 0;
  border-left: 0.3rem solid transparent;
  border-right: 0.3rem solid transparent;

  border-top: 0.5rem solid #aaa;

}

.structural-variations-table {
}

.table-tabs-container {
  .table-tabs {
    ::v-deep .v-item-group.v-slide-group.v-tabs-bar {

    }

    ::v-deep .v-slide-group__wrapper {
      padding-top: 1rem;
      margin-top: -1rem;
    }

    ::v-deep .v-slide-group__content.v-tabs-bar__content {
      padding-right: 1rem;
    }

    ::v-deep .v-item-group {
    }
  }
}

.data-area {
  overflow: auto;
}

.data-label {
  line-height: 3rem;
}

.data-block-rows {
  overflow: auto;
}


.data-block-column {
  position: relative;

  width: 1.5rem;
  height: 3rem;
  background: #eee;
  border: 1px solid transparent;


  &.block-1 {
    background: white;
    //border: 1px solid white;

    .data-block-cell-top, .data-block-cell-bottom {
    }
  }

  .pivot-data-block-cell {
    width: calc(1.5rem - 2px);
    height: calc(3rem - 2px);
  }

  &:hover {
    border-color: rgba(0, 255, 255, 1);
    background-color: rgba(0, 255, 255, 0.75);
  }

  &.selected {
    border-color: rgba(0, 255, 0, 1);
    background-color: rgba(0, 255, 0, 0.75);
  }
}


.pivot-label-row {
  position: relative;
  line-height: 2rem !important;
}

.pivot-data-block-row {
  .data-block-column {
    &:hover {
      border-color: transparent;
    }

    height: 2rem;

    .pivot-data-block-cell {
      height: calc(2rem - 2px);
    }
  }
}


.data-block-row {
  overflow: visible;

  .data-block-column {
    //border: 1px solid #eee;
    overflow: visible;

    &.above-pivot {
      transform: scaleY(-100%);
    }

    &.below-pivot {

    }

    &.pivot-neighbor {
      .block-dupe {
        transform: scaleY(-100%);
      }
    }

    .data-block-cell {
      display: block;
      width: calc(1.5rem - 2px);
      height: calc(1.5rem - 2px);

      position: absolute;
      overflow: visible;


      &.block-present {
        background: black;
        bottom: 0;
      }

      &.block-dupe {
        bottom: 50%;
        background: #0086CA;
        border-radius: 1rem 1rem 0 0;
      }

      &.block-insertion {
        bottom: 50%;
        left: -0.3rem;
        width: 0;
        height: 0;
        border-left: 0.25rem solid transparent;
        border-right: 0.25rem solid transparent;
        border-bottom: 1rem solid #81CD06;
        z-index: 1;
      }

      &.block-swap-start {
        //background: red;
        //background: green;
        left: 0;
        bottom: 0.2rem;
        width: 0;
        height: 0;
        border-top: 0.5rem solid transparent;
        border-bottom: 0.5rem solid transparent;

        border-left: 0.5rem solid #8148A4;
        //transform: translateY(10%);
      }

      &.block-swap-end {
        //background: red;
        //background: green;
        right: 0;
        bottom: 0.2rem;
        width: 0;
        height: 0;
        border-top: 0.5rem solid transparent;
        border-bottom: 0.5rem solid transparent;

        border-right: 0.5rem solid #8148A4;
        //transform: translateY(10%);
      }

      &.block-inversion {
        left: 0.2rem;
        bottom: 50%;
        width: 0;
        height: 0;
        border-left: 0.5rem solid transparent;
        border-right: 0.5rem solid transparent;

        border-bottom: 0.5rem solid #9D0D0D;
      }


      &.block-inversionChain, &.block-inversionChain-start, &.block-inversionChain-end {
        //left: 0.2rem;

        bottom: 50%;

        height: 50%;
        //border-left: 0.5rem solid transparent;
        //border-right: 0.5rem solid transparent;

        //border-bottom: 0.5rem solid #9D0D0D;
        //border-bottom: 0.125rem solid #9D0D0D;
        z-index: 1;

        &::before, &::after {
          display: block;
          content: "";
          position: absolute;
          bottom: 0;
        }

      }


      &.block-inversionChain::before {
        left: -1px;
        right: -1px;
        //width: calc(100% + 2px);
        border-bottom: 0.2rem solid #9D0D0D;
      }

      &.block-inversionChain-start::before {
        left: 50%;
        right: -1px;
        border-bottom: 0.2rem solid #9D0D0D;
      }

      &.block-inversionChain-end::before {
        left: -1px;
        right: 50%;
        border-bottom: 0.2rem solid #9D0D0D;
      }

      &.block-inversionChain::after, &.block-inversionChain-start::after, &.block-inversionChain-end::after {
        display: block;
        content: "";
        position: absolute;
        left: 0.2rem;
        bottom: 0;

        border-left: 0.5rem solid transparent;
        border-right: 0.5rem solid transparent;

        border-bottom: 0.5rem solid #9D0D0D;
      }

    }


  }
}

.outside-range {
  opacity: 0.1;
  //display: none;
}

.block-count-label {
  position: absolute;
  font-size: 0.5rem;
  top: 20%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 5;
}

.above-pivot {
  .block-count-label {
    transform: translate(-50%, -50%) scaleY(-100%);
  }
}

</style>