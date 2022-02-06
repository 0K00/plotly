<template lang="html">
  <div
    :class='{"tabs__light": mode === "light", "tabs__dark": mode === "dark"}'
  >
    <ul class="tabs__header">
      <li
        v-for="(tab, index) in tabs"
        :key="tab.title"
        @click="selectTab(index)"
        :class='{"tab__selected": (index == selectedIndex)}'
      >
        {{ tab.title }}
      </li>
    </ul>
    <slot></slot>
  </div>
</template>

<script>
  export default {
    props: {
      mode: {
        type: String,
        default: 'light'
      }
    },
    data () {
      return {
        selectedIndex: 0, // the index of the selected tab,
        tabs: []         // all of the tabs
      }
    },
    created () {
      this.tabs = this.$children
    },
    mounted () {
      this.selectTab(0)
    },
    methods: {
      selectTab (i) {
        this.selectedIndex = i

        // loop over all the tabs
        this.tabs.forEach((tab, index) => {
          tab.isActive = (index === i)
        })
        this.$parent.updateDate(this.$children[i].title)
      }
    }
  }
</script>

<style lang="css">

  ul.tabs__header {
    display: flex;
    gap: 4px;
    justify-content: space-between;
    list-style: none;
    padding: 8px;
    border-radius: 12px;
    background-color: #f8f9fa;
  }

  ul.tabs__header > li {
    padding: 8px 16px;
    border-radius: 8px;
    margin: 0;
    width: 100%;
    display: inline-block;
    cursor: pointer;
  }

  .tab {
    display: inline-block;
    color: black;
    min-width: 800px;
    border-radius: 10px;
  }

  .tabs__light .tab{
    width: 100%;
    background-color: #fff;
  }

  .tabs__light li {
    color: #747779;
    transition: color .15s ease-in-out,background-color .15s ease-in-out
  }

  .tabs__light li:hover {
    background-color: #e2e6ea;
  }

  .tabs__light li.tab__selected {
    background-color: #fff;
    color: #212529;
  }

  .tabs__dark .tab{
    background-color: #555;
    color: #212529;
  }

  .tabs__dark li {
    background-color: #ddd;
    color: #212529;
  }

  .tabs__dark li.tab__selected {
    background-color: #555;
    color: white;

  }
</style>
