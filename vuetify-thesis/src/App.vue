<template>
<div id="app">
  <div id="trees">
    <button class="accordion opened" @click="openAccordion($event)">Keywords</button>
    <div class="displayPanel" style="max-height: 67vh">
      <KeywordTree v-bind:keywords="keywords" :fetchedSpecialKeywords="fetchedSpecialKeywords"/>
    </div>
    <button class="accordion" @click="openAccordion($event)">Defined Actions</button>
    <div class="displayPanel">
      <KeywordTree v-bind:keywords="definedActions" :dA="true" :fetchedSpecialKeywords="fetchedSpecialKeywords"/>
    </div>
  </div>
  <KeywordDropzone v-bind:keywords="keywords" v-bind:fetchedSpecialKeywords="fetchedSpecialKeywords"/>
</div>
</template>

<script>
import KeywordTree from './components/KeywordTree.vue'
import KeywordDropzone from './components/KeywordDropzone.vue'
export default {
  name: 'App',
  data: () => ({
    keywords: undefined,
    definedActions: undefined,
    fetchedSpecialKeywords: undefined,
    pageContent: [],
    splittedSyntax: [],
  }),
  components: {
    KeywordTree,
    KeywordDropzone
  },
    created(){
        //fetch(window.location.protocol+"//"+window.location.host+":"+window.location.port+"?jsonKeywords&testJson=true")
        //fetch(window.location.protocol+"//"+window.location.host+"?jsonKeywords&testJson=true")
        fetch('https://my-json-server.typicode.com/Silask98/ThesisJson/db')
        //fetch('http://kdta-3.netcare.local:28080/?jsonKeywords&testJson=true')
         //fetch('http://localhost:28080/?jsonKeywords&testJson=true')
        .then(response => response.json())
        //.then(json => console.log(json))
        .then((json) => {
            console.log(json)
            this.keywords=json.keywords
            this.fetchedSpecialKeywords = json.specialKeywords
            this.definedActions = json.definedActions
        })
    },methods:{
      rebuildSyntax: function(enteredValues){
            this.splittedSyntax = []
            console.log(enteredValues)
            for(let i=0;i<enteredValues.length;i++){
              if(enteredValues[i].content){
                 this.splittedSyntax.push(enteredValues[i].fitNesseContent)
              }
              else if(enteredValues[i].attachedKeyword){
                let parentkeywordSyntax = enteredValues[i].syntax.split("|").filter(function(el) {return el.length != 0})
                for(let x=0;x<parentkeywordSyntax.length;x++){
                  if(parentkeywordSyntax[x].includes("%")){
                    if(!(parentkeywordSyntax[x] in enteredValues[i])){
                      parentkeywordSyntax[x] = "attachedKeyword"
                    }
                  }         
                }
                this.splittedSyntax.push(parentkeywordSyntax)            
              }
              else if(enteredValues[i].specialKeyword){
                this.splittedSyntax.push(enteredValues[i].syntax.split("|").concat("specialKeyword").filter(function(el) {return el.length != 0}))
              }
              else if(enteredValues[i].tableItems){
                this.splittedSyntax.push(enteredValues[i].syntax.split("|").concat("tableKeyword").filter(function(el) {return el.length != 0}))
              }
              else{
                this.splittedSyntax.push(enteredValues[i].syntax.split("|").filter(function(el) {return el.length != 0}))
              }
            }
            for(let i=0;i<this.splittedSyntax.length;i++){
              for(let x=0;x<this.splittedSyntax[i].length;x++){
                if(this.splittedSyntax[i][x].includes("%")){
                  this.splittedSyntax[i][x] = enteredValues[i][this.splittedSyntax[i][x]]
                }
                else if(this.splittedSyntax[i][x].includes("attachedKeyword")){
                  let innerAttachedSyntax = enteredValues[i].attachedKeyword.syntax.split("|").filter(function(el) {return el.length != 0})
                  for(let a=0;a<innerAttachedSyntax.length;a++){
                    if(innerAttachedSyntax[a].includes("%")){
                      innerAttachedSyntax[a] = enteredValues[i].attachedKeyword[innerAttachedSyntax[a]]
                    }
                  }
                  innerAttachedSyntax = innerAttachedSyntax.join(" | ")
                  this.splittedSyntax[i][x] = innerAttachedSyntax
                }
                else if(this.splittedSyntax[i][x].includes("specialKeyword")){
                  let innerSpecialSyntax = enteredValues[i].specialKeyword.syntax.split("|").filter(function(el) {return el.length != 0}).slice(1)
                  for(let a=0;a<innerSpecialSyntax.length;a++){
                    if(innerSpecialSyntax[a].includes("%")){
                      innerSpecialSyntax[a] = enteredValues[i].specialKeyword[innerSpecialSyntax[a]]
                    }
                  }
                  innerSpecialSyntax = innerSpecialSyntax.join(" | ")
                  this.splittedSyntax[i][x] = innerSpecialSyntax
                }
                //WIP0501
                else if(this.splittedSyntax[i][x].includes("tableKeyword")){
                  for(let a=0;a<enteredValues[i].usedTableItems.length;a++){
                    console.log(enteredValues[i].usedTableItems[a].syntax)
                    this.splittedSyntax[i][x] += "\n"+enteredValues[i].usedTableItems[a].syntax
                  }
                }
              }
            } 
            for(let i=0;i<this.splittedSyntax.length;i++){
              if(Array.isArray(this.splittedSyntax[i])){
                this.pageContent += " | "+this.splittedSyntax[i].join(" | ")+" | \n"
              }else{
                this.pageContent += this.splittedSyntax[i]
              }
            }
            console.log(this.pageContent)
        this.postPageContent(enteredValues) 
      },
      postPageContent: function(enteredValues){
        fetch('https://jsonplaceholder.typicode.com/posts', {
        method: 'POST',
        body: JSON.stringify({
          pageContent: this.pageContent,
          json: enteredValues
        }),
        /*
        headers: {
          'Content-type': 'application/json; charset=UTF-8',
        },*/
      })
      .then((response) => response.json())
      this.pageContent = []
      },
      openAccordion: function($event){
        $event.target.classList.toggle("opened")
        var panel = $event.target.nextElementSibling
        var otherPanel = ""
        var otherButton = ""
        if($event.target.innerHTML == "Keywords"){
         otherPanel = $event.target.nextElementSibling.nextElementSibling.nextElementSibling
         otherButton = $event.target.nextElementSibling.nextElementSibling
        }else{
         otherPanel = $event.target.previousElementSibling
         otherButton = $event.target.previousElementSibling.previousElementSibling
        }
        otherButton.classList.remove("opened")
        if(panel.style.maxHeight){
          panel.style.maxHeight = null
        } else {
          panel.style.maxHeight = "67vh"
          if(otherPanel.style.maxHeight){
            otherPanel.style.maxHeight = null
          }
        } 
      }
    },
}
</script>

<style scoped>
#app{
  margin-top: 2%;
}
#trees{
  width: 380px;
  float: left;
  height: fit-content;
}

.accordion {
  background-color: rgb(136, 136, 136);
  color: white;
  cursor: pointer;
  padding: 18px;
  width: 100%;
  border: none;
  text-align: left;
  outline: none;
  transition: 0.4s;
  border: 1px solid white;
  font-size: 15pt;
  font-family: sans-serif;
  padding-left: 28px;
  border-radius: 5px;
}

.opened, .accordion:hover {
  background-color: rgb(66, 66, 66); 
}

.displayPanel {
  margin-bottom: 5px !important;
  padding: 0 2px;
  background-color: white;
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.3s ease-out;
}
</style>

<style>
/*overwrite FitNesse Styles*/
.keywordTree button{
  padding: 0 ! important;
}
.keywordTree label{
  font-size: inherit !important;
}

.editorInsertedText p{
  margin: 0 !important;
}

.theme--light.v-application{
  background-color: none !important;
}

.vCardItem select, input[type="text"]{
  border: none !important;
  border-radius: none !important;
}

.vCardItem label{
  font-size: inherit !important;
}

.vCardItem button{
  white-space: inherit!important;
}
</style>