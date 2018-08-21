<template>
  <div class="tree-menu">
    <div>{{label}}</div>
    <tree-menu 
      v-for="(item, key) in children" 
      :children="item.children" 
      :label="key"
      :style="{width: item.width + 'px',height: item.height + 'px',left: item.left + 'px',top: item.top + 'px'}" 
      v-if="item.children && item.visible">
      
    </tree-menu>
  </div>
</template>
<script>
  export default { 
    props: [ 'label', 'children' ],
    name: 'tree-menu'
  }
</script>