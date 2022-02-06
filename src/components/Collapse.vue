<template>
  <div class="list-item-container">
    <div class="list-item-head p-8" @click="user.visible = !user.visible">
      {{user.name}}
      <span class="toggle float-right"  
            v-text="toggletText[user.visible * 1]">
      </span>
    </div>
    <transition name='slide-fade'>
    <div class="list-item-body p-8" v-show="user.visible"> 
      <slot />
    </div>
    </transition>
  </div>
</template>
<script>
export default {
  props:["user"],
  data(){
    return { toggletText: ['+', '-'] }
  }
}
</script>
<style scoped>
.list-item-container{
  margin: 0 auto;
  margin-bottom: 10px;
}
.p-8{
  padding: 8px 16px;
}
.list-item-head{
  display: flex;
  justify-content: space-between;
  cursor: pointer;
  color: #212529;
  background-color: #f8f9fa;
  border-radius: 8px;
  transition: color .15s ease-in-out,background-color .15s ease-in-out
}

.list-item-head:hover{
  background-color: #e2e6ea;
}

.list-item-head:active, .list-item-head:focus {
  background-color: #dae0e5;
}

.list-item-body {
  border: #E9E9E9 1px solid;
  border-radius: 8px;
  margin-top: 8px;
}

.float-right{
  float: right;
}
.toggle{
  cursor: pointer;
}

.slide-fade-enter-active {
  transition: all .3s ease-in-out;
}
.slide-fade-leave-active {
  transition: all .3s ease-in-out;
}
.slide-fade-enter, .slide-fade-leave-to {
  transform: translateY(-10px);
  opacity: 0;
}
</style>