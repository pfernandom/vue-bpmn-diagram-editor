<template>
  <div class="hello">
    <h1>To-Dos Editor</h1>
    <div class="content" id="js-drop-zone">

    <div class="message intro">
      <div class="note">
        Create a diagram and it will automatically be saved for this session. 
        You can refresh the page, but
        if you close your browser window and you don't download your diagram,
        you will lose all changes.
      </div>
      <ul class="buttons">
        <li>
          <a href v-on:click="resetDiagram">Reset Diagram</a>
        </li>
        <li>
          <a id="js-download-diagram" :href="diagramUrl" v-bind:download="diagramName" title="download BPMN diagram">
            Download as BPMN diagram
          </a>
        </li>
        <li>
          <a id="js-download-svg" :href="svgUrl" v-bind:download="svgName"  title="download as SVG image">
            Donwload as SVG image
          </a>
        </li>
      </ul>
    </div>

    <div class="canvas" id="canvas"></div>
    <div id="properties"></div>
  </div>
  </div>
</template>

<script>
import BpmnModeler from 'bpmn-js/lib/Modeler';
// import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/camunda';
// import camundaModdleDescriptor from 'camunda-bpmn-moddle/resources/camunda';
// import propertiesPanelModule from 'bpmn-js-properties-panel';
import {
  parseBPMNJson,
  getBlankDiagramDownloadUrl,
  openLocalDiagram,
  resetDiagramToBlank,
  saveDiagram
} from '../assets/diagram-utils.js';
import style from 'bpmn-js/dist/assets/diagram-js.css'; // eslint-disable-line no-unused-vars
import icons from 'bpmn-font/dist/css/bpmn-embedded.css'; // eslint-disable-line no-unused-vars
// import '../assets/styles.css';

export default {
  name: "ModelEditor",
  data() {
    return {
      msg: "Welcome to Your Vue.js App",
      modeler: null,
      diagramUrl: getBlankDiagramDownloadUrl(),
      diagramName: 'diagram.bpmn',
      svgUrl: 'data:application/bpmn20-xml;charset=UTF-8,hello',
      svgName: 'diagram.svg',
    };
  },
  methods: {
    resetDiagram(ev) {
      ev.preventDefault();
      resetDiagramToBlank(this.modeler);
    }
  },
  mounted(){
    this.$nextTick(() => {
      this.modeler = new BpmnModeler({
        x:100,
        container: '#canvas'/*,
        propertiesPanel: {
          parent: '#properties'
        },
        additionalModules: [
          propertiesPanelModule,
          propertiesProviderModule
        ],
        // needed if you'd like to maintain camunda:XXX properties in the properties panel
        moddleExtensions: {
          camunda: camundaModdleDescriptor
        }*/
      });

      let modeler = this.modeler;

      modeler.on('element.changed', function(event) {
        let element = event.element;
        if(event.gfx){
          console.log(event.gfx.textContent);
        }
        // the element got changed by the users
      });

      let self = this;
      modeler.on('commandStack.changed', () => {
        var definitions = modeler.get('canvas').getRootElement().businessObject.$parent;
        parseBPMNJson(definitions)
        // console.log(JSON.stringify(definitions));
        saveDiagram(modeler).then(url => {
          self.diagramUrl = url;
        });
        modeler.saveSVG(function(err, svg) {
          self.svgUrl = `data:application/bpmn20-xml;charset=UTF-8,${encodeURIComponent(svg)}`;
        });
      });
      openLocalDiagram(modeler);
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

.canvas {
  width: 100%;
  height: 70vh;
}

</style>
