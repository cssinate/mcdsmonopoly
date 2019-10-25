<template>
  <div id="app">
    <section class="search-wrapper">
      <input type="text" class="search" v-model="search" placeholder="Search" autofocus>
    </section>
    <section v-for="(section, index) in filteredSections" :key="index" class="section" :class="[section.className, {'filtered': search}]">
      <h1 class="sr-only">{{ section.className }}</h1>
      <div class="tiles">
        <div v-for="tile in section.tiles" :key="tile.number" class="tile" :class="{rare: tile.rare}">
          <h2>{{tile.name}} - {{tile.number}}</h2>
        </div>
      </div>
      <div class="prize">
        <p>{{section.prize.name}}</p>
        <p>{{section.prize.number}} prizes available</p>
        <p>1:{{section.prize.odds.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")}}</p>
      </div>
    </section>
  </div>
</template>

<script>
import { Prizes } from './prizes.js'
import { Tiles } from './tiles'

class Section {
  constructor (className, tiles, prize) {
    this.className = className
    this.tiles = tiles
    this.prize = prize
  }
}

export default {
  name: 'app',
  data () {
    return {
      year: 2019,
      sections: Tiles,
      search: ''
    }
  },
  mounted () {
    // this.sections.forEach((section, index) => {
    // this.sections[section]['prize'] = Prizes[this.year][index]
    // })
  },
  computed: {
    sectionList () {
      let list = []
      Object.keys(Tiles).forEach((section, index) => {
        /* eslint-disable-next-line */
        var newSection = new Section(
          Object.keys(Tiles)[index],
          this.sections[section].tiles,
          Prizes[this.year][index]
        )
        list.push(newSection)
      })
      return list
    },
    filteredSections () {
      return this.sectionList.filter(section => {
        var returned = false
        section.tiles.forEach(tile => {
          if (tile.name.toLowerCase().includes(this.search.toLowerCase())) {
            returned = true
          } else if (tile.number.toString().includes(this.search)) {
            returned = true
          }
        })
        if (returned) {
          return true
        }
      })
    }
  }
}
</script>

<style lang="scss">
  #app {
    display: contents;
  }

  *, *::before, *::after {
    box-sizing: border-box;
  }

  body {
    margin: 0;
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    font-family: sans-serif;
    background-color: white;
  }

  .sr-only:not(:focus):not(:active) {
    clip: rect(0 0 0 0);
    clip-path: inset(50%);
    height: 1px;
    overflow: hidden;
    position: absolute;
    white-space: nowrap;
    width: 1px;
  }

  .search-wrapper {
    width: 100%;
    position: sticky;
    top: 0;
    padding: .25em 1em;
    background-color: white;
    background-color: var(--app-bg, white);
    border-bottom: solid 2px currentColor;
  }

  .search {
    font-size: 2rem;
    padding: .5rem;
    width: 100%;
  }

  .section {
    border-style: solid;
    border-color: var(--color);
    border-left-width: 2ch;
    border-right-width: 2ch;
    flex-grow: 1;
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    padding-left: 2rem;
    padding-right: 2rem;
    justify-content: center;

    &.filtered {
      flex-grow: 0;
    }

    &:not(:last-child) {
      margin-bottom: .5rem;
    }
  }

  .tiles {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
  }

  .tile {
    background-color: white;
    background-color: var(--tilebg, white);
    border: solid 1px currentColor;
    padding: .5rem;
    flex: 0 0 16rem;
    margin-top: 1rem;
    order: 2;
    margin: .5rem;

    h2 {
      font-size: 1em;
      margin: 0;
      text-align: center;
      font-weight: normal;
    }

    &.rare {
      font-size: 1.1rem;
      border-width: 4px;
      order: 1;

      h2 {
        font-weight: bold;
      }
    }

    &::before {
      content: '';
      display: block;
      box-sizing: border-box;
      width: 100%;
      height: 1rem;
      margin-bottom: .5rem;
      background-color: var(--color);
      border: solid 2px currentColor;
    }
  }

  .prize {
    padding-bottom: 1rem;
    padding-top: 1rem;

    p {
      margin: 0;

      &:first-child {
        font-weight: bold;
      }
    }

  }

  .brown {
    --color: hsl(40, 100%, 25%);
  }

  .sky-blue {
    --color: hsl(190, 100%, 80%);
  }

  .pink {
    --color: hsl(310, 100%, 50%);
  }

  .orange {
    --color: hsl(30, 100%, 50%);
  }

  .red {
    --color: hsl(0, 90%, 50%);
  }

  .yellow {
    --color: hsl(50, 100%, 50%);
  }

  .green {
    --color: hsl(120, 70%, 50%);
  }

  .blue {
    --color: hsl(215, 100%, 60%);
  }

  .airports {
    --color: hsl(0, 0%, 80%);
  }

  .highways {
    --color: hsl(0, 0%, 25%);
  }

  @media (min-width: 79rem) {
    .section {
      justify-content: space-between;
    }

    .prize {
      text-align: right;
    }
  }

  @media (min-width: 45rem) {
    .tiles {
      flex-grow: 1;
      justify-content: flex-start;
    }
  }
</style>
