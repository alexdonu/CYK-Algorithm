<script setup lang="ts">
import { ref, computed } from 'vue'

type Rule = {
  left: string
  right: string
}

const rules = ref<Rule[]>([
  { left: 'S', right: 'AB' },
  { left: 'S', right: 'BC' },
  { left: 'A', right: 'BA' },
  { left: 'A', right: 'a' },
  { left: 'B', right: 'CC' },
  { left: 'B', right: 'b' },
  { left: 'C', right: 'AB' },
  { left: 'C', right: 'a' },
])
const word = ref('baaba')

function combineSets(set1: Set<string>, set2: Set<string>) {
  return new Set(Array.from(set1).flatMap(value1 => Array.from(set2).map(value2 => value1 + value2)))
}

function mergeSets(...sets: Set<string>[]) {
  return new Set(Array.from(sets).flatMap(set => Array.from(set)))
}

function getRulesFor(value: string) {
  return rules.value.filter(rule => rule.right.includes(value))
}

function getRulesForSet(set: Set<string>) {
  return new Set(
    Array.from(set)
      .flatMap(getRulesFor)
      .map(rule => rule.left),
  )
}

const table = computed(() => {
  const firstRow = word.value.split('').map(char => new Set(getRulesFor(char).map(rule => rule.left)))

  const arr = [firstRow]

  for (let i = 1; i < word.value.length; i++) {
    const row = [] as Set<string>[]
    for (let j = 0; j < word.value.length - i; j++) {
      const vertical = arr.map(row => row[j])
      const diagonal = arr.map((row, index) => row[i + j - index]).reverse()

      const combined = vertical.map((set, index) => combineSets(set, diagonal[index]))
      row.push(getRulesForSet(mergeSets(...combined)))
    }
    arr.push(row)
  }

  return arr
})

const axiom = computed(() => rules.value[0]?.left ?? 'Missing axiom')
const nonterminals = computed(
  () =>
    new Set(
      rules.value
        .map(rule => rule.left)
        .join('')
        .split('')
        .filter(el => el >= 'A' && el <= 'Z'),
    ),
)
const terminals = computed(() => new Set(rules.value.map(rule => rule.right).filter(el => el >= 'a' && el <= 'z')))

function deleteRule(index: number) {
  rules.value.splice(index, 1)
}

const isValid = computed(() => {
  return (
    rules.value.length > 0 &&
    rules.value.every(rule => rule.left.length === 1 && rule.left >= 'A' && rule.left <= 'Z') &&
    rules.value.every(rule => rule.right.length > 0) &&
    rules.value.every(rule =>
      rule.right.split('').every(char => nonterminals.value.has(char) || terminals.value.has(rule.right)),
    ) &&
    rules.value.some(rule => rule.right.split('').some(char => terminals.value.has(char)))
  )
})
const wordSatisfiesRules = computed(() => {
  return table.value[table.value.length - 1][0].has(axiom.value)
})
</script>

<template>
  <main class="app">
    <aside>
      <header>
        <label>
          <span>Word:</span>
          <input v-model="word" type="text" placeholder="word" />
        </label>
        <div class="grammar">
          <h3>Grammar general info:</h3>
          <h4 v-if="!isValid" class="error">
            <span>Grammar is invalid</span>
          </h4>
          <h4 v-else-if="!wordSatisfiesRules" class="error">
            <span>Word does not satisfy grammar rules</span>
          </h4>
          <h4>
            <span>Axiom: </span>
            <span>{{ axiom }}</span>
          </h4>
          <h4>
            <span>Nonterminals: </span>
            <span>{{ [...nonterminals].join(', ') }}</span>
          </h4>
          <h4>
            <span>Terminals: </span>
            <span>{{ [...terminals].join(', ') }}</span>
          </h4>
        </div>
        <div class="controls">
          <button @click="rules.push({ left: '', right: '' })">Add</button>
        </div>
      </header>
      <ol class="rules">
        <li v-for="(rule, index) in rules" :key="index">
          <input v-model="rule.left" type="text" />
          <span>-></span>
          <input v-model="rule.right" type="text" />
          <button @click="deleteRule(index)">Delete</button>
        </li>
      </ol>
    </aside>
    <section>
      <table v-if="table.length > 0">
        <tr>
          <th v-for="(char, index) in word.split('')" :key="index">{{ char }}</th>
        </tr>
        <tr v-for="(row, rowIndex) in table" :key="rowIndex">
          <td v-for="(set, setIndex) in row" :key="setIndex">
            <ul v-if="set.size > 0">
              <li v-for="value in set" :key="value">{{ value }}</li>
            </ul>
            <span v-else>-</span>
          </td>
        </tr>
      </table>
    </section>
  </main>
</template>

<style scoped lang="less">
.app {
  display: flex;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
  flex-direction: row;

  aside {
    flex-grow: 0;
    flex-shrink: 0;
    width: 400px;
    background-color: #eee;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    padding: 16px;

    .error {
      color: hsl(0, 800%, 60%);
    }

    header {
      input {
        width: 100%;
        padding: 8px;
        border: 1px solid #000;
        border-radius: 4px;
        margin-bottom: 16px;
      }

      .controls {
        border-top: 1px solid #000;
        margin-top: 8px;
        padding-top: 8px;
        display: flex;
        flex-direction: row;
        justify-content: flex-end;

        button {
          padding: 8px 16px;
          border: 1px solid #000;
          border-radius: 4px;
          background-color: #fff;
          cursor: pointer;
        }
      }
    }

    ol.rules {
      flex-grow: 1;
      flex-shrink: 1;
      overflow: auto;
      padding: 0;
      margin: 16px 0 0 0;
      list-style: none;

      li {
        display: flex;
        flex-direction: row;
        align-items: center;
        margin-bottom: 8px;

        input {
          padding: 8px;
          border: 1px solid #000;
          border-radius: 4px;
          margin-right: 8px;
          width: 100px;
        }

        button {
          padding: 8px 16px;
          border: 1px solid #000;
          border-radius: 4px;
          background-color: #fff;
          cursor: pointer;
        }
      }
    }
  }

  section {
    flex-grow: 1;
    flex-shrink: 1;
    background-color: #ddd;
    overflow: auto;
    padding: 16px;

    table {
      border-collapse: collapse;
      width: 100%;

      tr:nth-child(even) {
        background-color: #fff;
      }

      th,
      td {
        border: 1px solid #000;
        padding: 8px;

        ul {
          margin: 0;
          padding: 0;
          list-style: none;
          display: flex;
          flex-direction: row;
        }
      }
    }
  }
}
</style>