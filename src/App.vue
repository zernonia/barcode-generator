<template>
  <div id="app">
    <div class="sidebar">
      <div v-for="(item, index) in bulk" :key="index">
        <p id="item" @click="select(index)">{{ item.Product }}</p>
      </div>
    </div>
    <div style="opacity:0; position: absolute; z-index:-10;">
      <svg ref="svg" id="barcode" />
    </div>
    <div id="container">
      <h2>{{ header }}</h2>
      <canvas ref="canvas" id="canvas"></canvas>
      <div id="input">
        <label>Value</label>
        <input type="text" v-model="value" placeholder="56612345">
        
        <label>Label</label>
        <textarea v-model="label" rows="3"></textarea>

        <div style="display: flex; margin: 1.5rem 0rem;">
          <button @click.prevent="render(value, label)">Generate</button>
          <button @click.prevent="save()">Save Image</button>
          <button @click.prevent="saveAll()">Save Bulk</button>
        </div>

        <button @click.prevent="upload()">Upload</button>
      </div>
    </div>
  </div>
</template>

<script>
  import JsBarCode from './plugins/JsBarcode.all.min.js'
  import {fabric} from 'fabric'
  import { mkdir } from 'fs'
import { setTimeout } from 'timers'
import { resolve } from 'dns'
  var XLSX = require('xlsx')
  var fs = require('fs')
  const { dialog } = require('electron').remote

  var buffer,base64Data,canvas

  export default {
    data: () => ({
      type: '',
      value: '123456789',
      label: 'ORBS3025',
      test: '',
      url: '',
      bulk: [],
      header:''
    }),
    methods:{
      async render(value, label){
        let promise = new Promise( async(resolve) =>{

        if(canvas._objects.length){
          canvas._objects.forEach( (obj, key) => {
            canvas.remove(obj)
          })
        }
        console.log(value)
        JsBarcode("#barcode", value);
        var width = this.$refs.svg.width.baseVal.value
        var height = this.$refs.svg.height.baseVal.value
        var ctx = this.$refs.canvas.getContext('2d')
        var data = this.$refs.svg
        var xml = new XMLSerializer().serializeToString(data);

        // make it base64
        var svg64 = btoa(xml);
        var b64Start = 'data:image/svg+xml;base64,';

        // prepend a "header"
        var image64 = b64Start + svg64;
        var offsetX = (420 - width) / 2
        var offsetY = (300 - height) 

        // set it as the source of the img element
        var img = new Image()
        img.onload = function() {
            // draw the image onto the canvas
            ctx.drawImage(img, offsetX, offsetY);
        }
        img.src = image64;

        var text = new fabric.Text( label , { left: 100, top: 30 });
        canvas.add(text);

        await this.timeout(500)
        try{
          console.log('ok')
          resolve(canvas._objects[0].text)
        }
        catch{
          console.log('error')
        }

        })
        const done = await promise
        console.log('render done')
        return done
      },
      save(){
        console.log(this.$refs.canvas)
        buffer = this.$refs.canvas.toDataURL()

        var options = {
          title: "Save Image",
          defaultPath : "image",
          buttonLabel : "Save",

          filters :[
            {name: 'Image Files', extensions: ['png','jpg','jpeg']},
            {name: 'All Files', extensions: ['*']}
           ]
        }
        base64Data = buffer.replace(/^data:image\/png;base64,/, "");
        dialog.showSaveDialog( options ).then((filename) => {
          console.log(filename)
          fs.writeFile(filename.filePath, base64Data, 'base64', function(err) {
            console.log(err);
          });
        })
      },
      saveAll(){
          var options = {
          title: "Select Folder",
          buttonLabel : "Select Folder",
          properties: ['openDirectory']
        }
        dialog.showOpenDialog( options ).then( async (filename) => {
          if (!filename.canceled) { 
              var select = filename.filePaths[0]
              var dir = select.slice(-7)
              if(dir != "Barcode"){ 
                var concat = select + '\\Barcode'
                if( !concat ){
                  mkdir(select + '\\Barcode' , err => { console.log(err) })
                }
                select = select + '\\Barcode' 
              }
              for(var i = 0; i < this.bulk.length ; i++){
              // for(var item of this.bulk){
                var item = this.bulk[i]
                await new Promise( next => {
                  
                  this.render(item.Barcode, item.Label).then( async(res) => {     
                                
                  buffer = this.$refs.canvas.toDataURL()
                  base64Data = buffer.replace(/^data:image\/png;base64,/, "")
                  fs.writeFileSync(select+`\\${ item.Product }.png`, base64Data, 'base64')                   
                  if(i == this.bulk.length -1){ alert('Successful')}
                  next()
                });
                })
                }
              }
        })
      },
      upload(){
        var options = {
          title: "Upload files",
          buttonLabel: 'Upload',
          properties: ['openFile'],

          filters :[
            {name: 'Excel file', extensions: ['xlsx']},
            {name: 'All Files', extensions: ['*']}
           ]
        }

        dialog.showOpenDialog( options ).then((filename) => {
          if (!filename.canceled) { 
              var upload = filename.filePaths[0].toString(); 
              var workbook = XLSX.readFile(upload);
              var ws = workbook.SheetNames[0]
              var data = XLSX.utils.sheet_to_json(workbook.Sheets[`${ws}`])
              data.forEach( item => {
                this.bulk.push(item)
              })
              console.log(this.bulk)
        }})
      },
      async select(index){
        var selected = this.bulk[index]

        this.render(selected.Barcode, selected.Label).then( res =>{
          console.log(res)

        })
        // return done
      },
      timeout(ms){
        return new Promise(resolve => setTimeout(resolve, ms))
      },
    },
    mounted(){

      canvas = new fabric.Canvas('canvas',{
        backgroundColor: 'rgb(255,255,255)',
        width: 420,
        height: 300
      });
      console.log(canvas)
      this.render(this.value, this.label)
      
    }
  }
</script>

<style>
body{
  margin: 0;
  padding: 0;
  background: #222222;
  font-family: Arial, Helvetica, sans-serif;
}
#container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  width: 100%;
}
#input {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  margin-top: 1.5rem;
}
label{
  color: white;
  margin: 5px;
}
h2 {
  color: white;
}
.sidebar{
  height: 100vh; 
  width: auto;
  flex:none;
  max-width: 200px; 
  padding: 0 1rem;
  background-color: #111111;
  color: white;
  overflow-y: scroll;
}
#app{
  display: flex;
}
#item {
  padding: 1rem;
  cursor: pointer;
  background-color: #222222;
}
button{
  min-width: 150px;
  background-color: #111111;
  padding: 0.8rem;
  outline: none;
  border: none;
  margin: 0rem 0.8rem;
  color:white;
  cursor: pointer;
}
button:hover{
  background-color: #333333;
}
</style>
