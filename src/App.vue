<script setup lang="ts">
import { ref, computed } from "vue";
import ArrowIcon from "./components/ArrowIcon.vue";

interface Rule {
  left: string;
  right: string;
}
const rules = ref<Rule[]>([
  { left: "S", right: "AB" },
  { left: "S", right: "BC" },
  { left: "A", right: "BA" },
  { left: "A", right: "a" },
  { left: "B", right: "CC" },
  { left: "B", right: "b" },
  { left: "C", right: "AB" },
  { left: "C", right: "a" },
]);
const wordToCheck = ref("baababaababaababaa");
const rerenderCounter = ref(0);

const Vn = computed(() => {
  //all uppercase letters from rules, left side and right side

  const Vn = new Set<string>();
  rules.value.forEach((rule) => {
    Vn.add(rule.left);
    rule.right.split("").forEach((char) => {
      if (char.toUpperCase() === char) {
        Vn.add(char);
      }
    });
  });
  return Array.from(Vn);
});

const Vt = computed(() => {
  // all characters from rules, except uppercase letters
  const Vt = new Set<string>();
  rules.value.forEach((rule) => {
    rule.right.split("").forEach((char) => {
      if (char.toLowerCase() === char) {
        Vt.add(char);
      }
    });
  });
  return Array.from(Vt);
});

function addRule() {
  rules.value.push({ left: "", right: "" });
}

let matrix = ref<any>(new Array(wordToCheck.value.length + 1));

function solve() {
  //build matrix.value borders
  // let matrix = ref<any>(new Array(wordToCheck.value.length + 1));
  for (let i = 0; i < matrix.value.length; i++) {
    matrix.value[i] = new Array<string>(wordToCheck.value.length + 1);
  }

  for (let i = 0; i < wordToCheck.value.length; i++) {
    matrix.value[0][i + 1] = wordToCheck.value[i];
  }
  for (let i = 0; i < wordToCheck.value.length; i++) {
    matrix.value[i + 1][0] = i + 1;
  }
  for (let i = 1; i < wordToCheck.value.length + 1; i++) {
    for (let j = 1; j < wordToCheck.value.length + 1; j++) {
      matrix.value[i][j] = "";
    }
  }

  // fill the first line of matrix.value using Cocke - Younger - Kassami algorithm
  for (let i = 0; i < wordToCheck.value.length; i++) {
    for (let j = 0; j < rules.value.length; j++) {
      if (rules.value[j].right === wordToCheck.value[i]) {
        matrix.value[1][i + 1] += rules.value[j].left;
      }
    }
  }

  for (let i = 2; i < wordToCheck.value.length + 1; i++) {
    for (let j = 1; j < wordToCheck.value.length - i + 2; j++) {
      let nonTerminalCombinations: String[] = [];
      for (let k = 1; k < i; k++) {
        const first = matrix.value[k][j].split("");
        const second = matrix.value[i - k][j + k].split("");

        for (let l = 0; l < first.length; l++) {
          for (let m = 0; m < second.length; m++) {
            nonTerminalCombinations.push(first[l] + second[m]);
          }
        }
      }

      //check nonTerminalCombinations
      for (let k = 0; k < rules.value.length; k++) {
        for (let l = 0; l < nonTerminalCombinations.length; l++) {
          if (
            rules.value[k].right === nonTerminalCombinations[l] &&
            !matrix.value[i][j].includes(rules.value[k].left)
          ) {
            matrix.value[i][j] += rules.value[k].left;
          }
        }
      }
    }
  }

  //show the matrix.value
  console.log(matrix.value);
  rerenderCounter.value++;
}
</script>

<template>
  <v-card class="ma-2">
    <v-card-title>Algoritmul Cocke - Younger - Kassami</v-card-title>
  </v-card>
  <v-card class="d-flex justify-space-between">
    <v-responsive max-width="400">
      <v-card>
        <v-form @submit.prevent>
          <div v-for="(rule, index) in rules" :key="index" class="d-flex text">
            <div>{{ index + 1 }}.</div>
            <v-responsive max-width="50">
              <v-text-field v-model="rule.left" />
            </v-responsive>
            <ArrowIcon />
            <v-responsive max-width="150">
              <v-text-field v-model="rule.right" />
            </v-responsive>
          </div>
        </v-form>
      </v-card>
    </v-responsive>

    <v-responsive max-width="350">
      <v-card>
        <v-card-title>Word to check:</v-card-title>
        <v-text-field v-model="wordToCheck" @keyup.enter="solve"/>
        <div class="d-flex">
          <div class="ma-2">Word length:</div>
          <div class="ma-2">{{ wordToCheck.length }}</div>
        </div>
        <v-btn @click="solve" class="ma-2">Solve</v-btn>
      </v-card>
    </v-responsive>

    <v-responsive max-width="350">
      <v-card>
        <v-card-title>Grammar general info:</v-card-title>
        <div class="d-flex">
          <div class="ma-2">Axiom:</div>
          <div class="ma-2">{{ rules[0].left }}</div>
        </div>
        <div class="d-flex">
          <div class="ma-2">Vn:</div>
          <div class="ma-2">{{ Vn }}</div>
        </div>
        <div class="d-flex">
          <div class="ma-2">Vt:</div>
          <div class="ma-2">{{ Vt }}</div>
        </div>
      </v-card>
    </v-responsive>
  </v-card>
  <v-card class="d-flex">
    <v-btn @click="addRule" class="ma-2">Add rule</v-btn>
    <v-btn @click="rules.pop()" class="ma-2">Remove rule</v-btn>
  </v-card>

  <v-card>
    <v-card-title> Results: </v-card-title>
    <v-table :key="rerenderCounter">
      <tbody>
        <tr
          v-for="(row, indexRow) in matrix"
          :key="indexRow"
          :class="{ highlight: indexRow % 2 === 0 }"
        >
          <td v-for="(element, index) in row" :key="index">{{ element }}</td>
        </tr>
      </tbody>
    </v-table>
  </v-card>
</template>

<style scoped>
.text {
  font-size: 50px;
}

.highlight {
  background-color: #d9d9d9;
}
</style>
