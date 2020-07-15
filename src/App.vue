<template>
  <div id="app">
    <div style="opacity:0; position: absolute; z-index:-10;">
      <svg ref="svg" id="barcode" />
    </div>
    <canvas ref="canvas" id="canvas"></canvas>
    <div id="input">
      <label>Value</label>
      <input type="text" v-model="value" placeholder="56612345">
      
      <label>Label</label>
      <textarea v-model="label" rows="3"></textarea>

      <button @click.prevent="render()">Generate</button>
      <button @click.prevent="save()">Print</button>
      <!-- <a :href="this.test" download="image/png" >Download as jpeg</a> -->
    </div>
  </div>
</template>

<script>
  import JsBarCode from './plugins/JsBarcode.all.min.js'
  import {fabric} from 'fabric'
  var fs = require('fs');
  const { dialog } = require('electron').remote
  var canvas

  export default {
    data: () => ({
      type: '',
      value: '123456789',
      label: 'ORBS3025',
      test: '',
      url: ''
    }),
    methods:{
      render(){
        JsBarcode("#barcode", `${this.value}`);
        var width = this.$refs.svg.width.baseVal.value
        var height = this.$refs.svg.height.baseVal.value
        console.log(width)
        var ctx = this.$refs.canvas.getContext('2d')
        var data = this.$refs.svg
        var xml = new XMLSerializer().serializeToString(data);

        // make it base64
        var svg64 = btoa(xml);
        var b64Start = 'data:image/svg+xml;base64,';

        // prepend a "header"
        var image64 = b64Start + svg64;
        var offsetX = (400 - width) / 2
        var offsetY = (400 - height) 

        // set it as the source of the img element
        var img = new Image()
        img.onload = function() {
            // draw the image onto the canvas
            ctx.drawImage(img, offsetX, offsetY);
        }
        img.src = image64;

        var text = new fabric.Text( this.label , { left: 100, top: 50 });
        canvas.add(text);
      },
      save(){
        console.log(this.$refs.canvas)
        var buffer = this.$refs.canvas.toDataURL()

        var options = {
          title: "Save Image",
          defaultPath : "image",
          buttonLabel : "Save",

          filters :[
            {name: 'Image Files', extensions: ['jpeg','jpg','png']},
            {name: 'All Files', extensions: ['*']}
           ]
        }
        var base64Data = buffer.replace(/^data:image\/png;base64,/, "");
        dialog.showSaveDialog( options ).then((filename) => {
          console.log(filename)
          fs.writeFile(filename.filePath, base64Data, 'base64', function(err) {
            console.log(err);
          });
        })
      },
    },
    mounted(){

      canvas = new fabric.Canvas('canvas',{
        backgroundColor: 'rgb(255,255,255)',
        width: 400,
        height: 400
      });
      console.log(canvas)
      this.render()
      
    }
  }
</script>

<style>
body{
  margin: 0;
  padding: 0;
  background: #222222;
}
#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  width: 100vw;
}
#input {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
}
label{
  color: white;
}
</style>
