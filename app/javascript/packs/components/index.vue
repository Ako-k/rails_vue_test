<template>
  <div>
    <!-- 新規作成フォーム -->
    <div class="row">
      <div class="col s10 m11">
        <input v-model="newTask" class="from-control" placeholder="新規タスクを追加">
      </div>
      <div class="col s2 m1">
        <div v-on:click="createTask" class="btn-floating waves-effect waves-light red">
          <i class="material-icons">add</i>
        </div>
      </div>
    </div>

    <!-- リスト表示 -->
    <div>
      <ul class="collection">
         <li v-for="task in tasks" v-if="!task.is_done" v-bind:id="'row_task_' + task.id" class="collection-item" :key='task'>
          <input type="button" v-on:click="doneTask(task.id)" v-bind:id="'task_' + task.id" />
          <label v-bind:for="'task_' + task.id">{{task.name}}</label>
        </li>
      </ul>
    </div>

    <!-- 完了済タスク表示ボタン -->
    <div class="btn" v-on:click="displayFinishedTasks"> 完了済タスクを表示 </div>

    <!-- 完了済タスク -->
    <div id="finished-tasks" class="display_none">
      <ul class="collection">
        <li v-for="task in tasks" v-if="task.is_done" v-bind:id="'row_task_' + task.id" class="collection-item" :key='task'>
          <input type="checkbox" v-bind:id="'task_' + task.id" checked="checked" />
          <label v-bind:for="'task_' + task.id"  class="line-through">{{ task.name }}</label>
        </li>
      </ul>
    </div>

  </div>
</template>

<script>
  import axios from 'axios';

  export default {
    data: function() {
      return {
        tasks: [],
        newTask: ''
      }
    },
    mounted: function() {
      this.fetchTasks();
    },
    methods: {
      fetchTasks: function() {
        axios.get('/api/tasks').then((response) => {
          for (var i=0; i < response.data.tasks.length; i++){
            this.tasks.push(response.data.tasks[i]);
          }
        }, (error) => {
          console.log(error);
        });
      },
      displayFinishedTasks: function() {
        document.querySelector('#finished-tasks').classList.toggle('display_none');
      },
      createTask: function() {
        if (!this.newTask) return;

        axios.post('/api/tasks', { task: { name: this.newTask } }).then((response) => {
          this.tasks.unshift(response.data.task);
          this.newTask = "";
        }, (error) => {
          console.log(error);
        });
      },
      doneTask: function (task_id) {
        axios.put('/api/tasks/' + task_id, { task: { is_done: 1 }}).then((response) => {
          // this.moveFinishedTask(task_id);
          // タスク完了のupdate実行後画面リロード
          this.$router.go({path: this.$router.currentRoute.path, force: true})
        }, (error) => {
          console.log(error);
        });
      },
    }
  }

</script>

<style scoped>
  [v-cloak] {
    display: none;
  }
  .display_none {
    display:none;
  }

  .line-through {
    text-decoration: line-through;
  }
</style>