<template>
        <div id="board">
            <md-list v-for="(result, id) in results.data" v-bind:key="id" class="md-triple-line">
                <md-subheader> {{result.value}}
                    <div :class="{'count-color' : result.tasks.length >= 4 && result.id ===2}">
                        {{ "("+ result.tasks.length+ ")"}}
                    </div>
                </md-subheader>
                <md-list-item v-for="(task, key) in result.tasks" v-bind:key="key" @click.prevent.stop="handleClick($event, task)">
                    <div :class="{'completed' : task.accepted}">{{task.text}}</div>
                </md-list-item>
                <md-field md-inline v-if="result.value === 'Todos'">
                    <label>Input task</label>
                    <md-input v-model="text" @change="createTask()"></md-input>
                </md-field>
            </md-list>
            <vue-simple-context-menu
                    :elementId="'context-board-menu'"
                    :options="options"
                    :ref="'vueSimpleContextMenu'"
                    @option-clicked="optionClicked"/>
            <md-progress-spinner md-mode="indeterminate" v-show="barDisplay"></md-progress-spinner>
        </div>
</template>

<script>
  import * as axios from "axios";
  import Vue from 'vue'
  import VueMaterial from 'vue-material'
  import 'vue-material/dist/vue-material.min.css'
  import 'vue-material/dist/theme/default.css'
  import 'vue-simple-context-menu/dist/vue-simple-context-menu.css'
  import VueSimpleContextMenu from 'vue-simple-context-menu'

  
  Vue.component('vue-simple-context-menu', VueSimpleContextMenu)

  Vue.use(VueMaterial)

  export default {
    name: 'Board',
    data() {
      return {
        results: [],
        barDisplay: false,
        options: [
          {
            name: 'Done',
            do: 'done'
          },
          {
            name: 'Remove',
            do: 'remove'
          },
          {
            name: 'Move to urgent',
            do: 'move'
          }],
        text: ''
      };
    },
    mounted: function () {
      this.init();
    },

    methods: {
      init() {
        axios.get(process.env.VUE_APP_API_URL+'/boards').then(response => {
          this.results = response.data
        }). catch(response => {
          console.log(response);
        })
      },
      createTask() {
        let formData = new FormData();
        formData.append('text', this.text);
        formData.append('board_id', 1);
        this.barDisplay = true;
        axios.post(process.env.VUE_APP_API_URL+'/tasks', formData)
          .then(response => {
            if (response.status == 200) {
              this.text = '';
              this.init();
              this.barDisplay = false;
            }
          })
          .catch(e => {
            this.errors.push(e)
          })
      },
      handleClick(event, item) {
        this.$refs.vueSimpleContextMenu.showMenu(event, item)
      },

      optionClicked(event) {
        switch (event.option.do) {
          case "done":
            this.modifyTask({accepted: 1}, event.item);
            break;
          case "remove":
            this.deleteTask(event.item);
            break;
          case "move":

            this.modifyTask({board_id: 2 == event.item.board_id ? 1 : 2}, event.item);
            break;
        }
      },

      modifyTask(values, element) {
        let formData = new FormData();
        for (let item in values)
        {
          formData.set(item, values[item]);
        }
        formData.append("_method", "put");
        this.barDisplay = true;
        axios.post(process.env.VUE_APP_API_URL+'/tasks/' + element.id, formData,
          {headers:  {'Content-Type': 'application/json' }} )
          .then(response => {
            if (response.status == 200) {
              this.init();
              this.barDisplay = false;
            }
          })
          .catch(e => {
            this.errors.push(e)
          })
      },

      deleteTask(element)
      {
        let formData = new FormData();
        formData.append("_method", "delete");
        this.barDisplay = true;
        axios.post(process.env.VUE_APP_API_URL+'/tasks/' + element.id, formData,
          {headers:  {'Content-Type': 'application/json' }} )
          .then(response => {
            if (response.status == 200) {
              this.init();
              this.barDisplay = false;
            }
          })
          .catch(e => {
            this.errors.push(e)
          })
      },
    }
  }
</script>

<style lang="scss" scoped>
    .md-list {
        width: 320px;
        max-width: 100%;
        display: inline-block;
        vertical-align: top;
        border: 1px solid rgba(#000, .12);
    }

    .md-progress-spinner {
        position: fixed;
        left: 50%;
        top: 50%;
    }
    .completed {
        text-decoration: line-through;
    }
    .count-color {
        color: red;
    }

    .container {
        display: flex;
    }

    .column {
        display: flex;
        flex-direction: column;
    }

</style>