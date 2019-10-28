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
    <section class="about" v-if="showAbout">
      <h1>Here are the rare pieces for McDonald's Canada Monopoly 2019</h1>
      <p>Use this application to quickly check if your Monopoly stickers are winners or not. Type in the name or number of the property, and you can see quickly if it's worth hanging onto or not.</p>
      <p>This app was created by <a href="https://twitter.com/cssinate">Paul Grant</a>. Find and contribute to this project on <a href="https://github.com/cssinate/mcdsmonopoly">Github</a>.</p>
      <small>I collect a very small amount of anonymous usage data to see if people are actually using this tool. This is how I decide whether or not it's worth it for me to pay for the costs of running the site. On this site, I have information on how long you were here, how you got here, and whether or not you used the search field in this session (true or false).</small>
      <button @click="hideAbout()">Don't show this section again</button>
    </section>
    <a role="button" class="showAbout" @click="revealAbout()" v-if="!showAbout">About</a>
    <button class="darkModeToggle" :aria-pressed="theme" @click="toggleDark()">
      dark theme:
      <span aria-hidden="true">{{theme === 'dark' ? 'on' : 'off'}}</span>
    </button>
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
      search: '',
      trackedInteraction: false,
      theme: 'light',
      showAbout: true
    }
  },
  created () {
    const theme = localStorage.getItem('theme')
    const about = localStorage.getItem('about')
    const darkModeMediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
    if (!theme) {
      const darkModeOn = darkModeMediaQuery.matches
      darkModeOn && this.toggleDark('dark')
    }
    darkModeMediaQuery.addListener(() => {
      this.toggleDark()
    })
    if (theme === 'dark') {
      this.toggleDark('dark')
    }
    if (about) {
      this.showAbout = false
    }
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
  },
  methods: {
    toggleDark (setTo) {
      if (!setTo) {
        this.theme === 'light' ? this.theme = 'dark' : this.theme = 'light'
        this.theme === 'dark' ? document.body.classList.add('darkMode') : document.body.classList.remove('darkMode')
      } else if (setTo === 'dark') {
        this.theme = 'dark'
        document.body.classList.add('darkMode')
      }

      window.localStorage.setItem('theme', this.theme)
    },
    hideAbout () {
      this.showAbout = false
      window.localStorage.setItem('about', 'hidden')
    },
    revealAbout () {
      this.showAbout = true
      window.localStorage.removeItem('about')
      this.$nextTick(() => { window.scrollTo(0, document.body.scrollHeight) })
    }
  },
  watched: {
    search () {
      if (!this.trackedInteraction) {
        this.trackedInteraction = true
        this.$root._paq.push(['trackEvent', 'Interaction', 'Search'])
      }
    }
  }
}
</script>

<style lang="scss">
  :root {
    --bg: #ffffff;
    --fg: #000000;
    --link: #1460c2;
  }

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
    font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
    background-color: #ffffff;
    color: #000000;
    background-color: var(--bg);
    color: var(--fg);
    padding-bottom: 3rem;
  }

  a {
    color: var(--link);
  }

  button {
    background-color: var(--bg);
    color: var(--fg);
    outline: solid 2px currentColor;
    border: 0;

    &::-moz-focus-inner {
      border: 0;
    }

    &:focus {
      outline-width: 4px;
    }
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
    background-color: var(--bg, white);
    border-bottom: solid 2px currentColor;
    z-index: 1;
  }

  .search {
    font-size: 2rem;
    padding: .5rem;
    width: 100%;
    background-color: var(--tilebg, white);
    color: var(--fg, black);
    border: solid 2px currentColor;
  }

  .section {
    border-style: solid;
    border-color: var(--color);
    border-left-width: 2ch;
    border-right-width: 2ch;
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

  .tile.rare::after {
    content: '⭐ RARE ⭐';
    width: 100%;
    text-align-last: justify;
    display: block;
    font-size: .8em;
  }

  .prize {
    padding-bottom: 1rem;
    padding-top: 1rem;
    width: 100%;

    p {
      margin: 0;
      text-align: center;

      &:first-child {
        font-weight: bold;
      }
    }

  }

  .about {
    width: 70ch;
    padding-left: calc(2ch + 2rem);
  }

  .darkModeToggle {
    position: fixed;
    bottom: .5rem;
    right: .5rem;
  }

  .showAbout {
    padding-left: calc(2ch + 2rem);
    cursor: pointer;
    width: 6rem;
    width: min-content;
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

  @media (min-width: 45rem) {
    .tiles {
      flex-grow: 1;
      justify-content: flex-start;
    }

    .prize p {
      text-align: left;
    }
  }

  @media (min-width: 79rem) {
    .section {
      justify-content: space-between;
    }

    .prize {
      width: auto;

      p {
        text-align: right;
      }
    }
  }

  body.darkMode {
    --bg: #222222;
    --fg: rgb(245, 245, 245);
    --tilebg: #333333;
    --link: #5b8ccc;

    ::placeholder {
      color: #c5c5c5;
    }

    .section {
      filter: saturate(.8) brightness(.9);
    }
  }
</style>
