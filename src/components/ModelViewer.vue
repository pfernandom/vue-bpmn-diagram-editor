<template>
  <div class="hello">
    <h1>Pizza Process Viewer</h1>
    <div id="canvas"></div>
  </div>
</template>

<script>
import BpmnViewer from 'bpmn-js';
import pizzaDiagram from '../assets/pizzaDiagram.bpmn';

export default {
  name: "ModelViewer",
  data() {
    return {
      msg: "Welcome to Your Vue.js App"
    };
  },
  mounted(){
    this.$nextTick(function () {
      var viewer = new BpmnViewer({
        container: '#canvas'
      });

      var eventBus = viewer.get('eventBus');
      // you may hook into any of the following events
      var events = [
        'element.hover',
        'element.out',
        'element.click',
        'element.dblclick',
        'element.mousedown',
        'element.mouseup'
      ];

      events.forEach(function(event) {

        eventBus.on(event, function(e) {
          // e.element = the model element
          // e.gfx = the graphical element

          console.log(event, 'on', e.element.id);
        });
      });

      viewer.importXML(pizzaDiagram, function(err) {

        if (!err) {
          console.log('success!');
          viewer.get('canvas').zoom('fit-viewport');
        } else {
          console.log('something went wrong:', err);
        }
      });
    });
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

#canvas { 
  width: 100%;
  height: 100vh;
}
</style>
