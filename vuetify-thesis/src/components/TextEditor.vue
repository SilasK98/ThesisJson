<template>
  <div class="editorWrapper">
    <div id="toolbar" @mousedown="$event.preventDefault()">
      <select class="ql-size">
        <option selected></option> 
        <option value="huge">Header1</option>
        <option value="large">Header2</option>
        <option value="small">Header3</option>   
      </select>
      <button class="ql-bold"></button>
      <button class="ql-italic"></button>
      <button class="ql-strike"></button>
      <button class="ql-escape"><img src="../assets/editorLogos/escape.svg"></button>
      <button class="ql-clean"></button>
      <button class="ql-align" value="center"></button>
      <button class="ql-image"></button>
      <button class="ql-hashTable"><img src="../assets/editorLogos/hashTable.svg"></button>
      <button class="ql-link"></button>
      <span class="ql-formats">
        <button class="ql-list" value="bullet"></button>
        <button class="ql-indent" value="+1"></button>
        <button class="ql-indent" value="-1"></button>
        <button class="ql-rule"><img src="../assets/editorLogos/horizontalRule.svg"></button>
        <button class="ql-table"><img src="../assets/editorLogos/table.svg"></button>
      </span>
      <span class="ql-placeholder">
        <button class="ql-csClosed"><img src="../assets/editorLogos/collapsibleSectionDefaultClosed.svg"></button>
        <button class="ql-csOpen"><img src="../assets/editorLogos/collapsibleSectionDefaultOpen.svg"></button>
        <button class="ql-csHidden"><img src="../assets/editorLogos/collapsibleSectionHidden.svg"></button>
      </span>

      <span class="ql-formats">
        <select class="ql-size" @change="setWidget($event)"> 
          <option value="leer">-Widgets-</option>
          <option value="!include ">Include</option>
          <option value="!see ">Cross Reference</option>
          <option value="!heading ">Headings</option>
          <option value="!note ">Notes</option>  
          <option value="!anchor .#ANCHOR-NAME ">Links Withhin Pages</option>  
          <option value="[[displayed link name][focused page]] ">Alias links</option>  
          <option :value="'{{{ \n\n }}}'">Preformatted text</option>  
          <option value="# ">Comments</option>
          <option value="!define VAR-NAME VAR-VALUE">Variables</option>  
          <option value="!path ">Classpaths</option>  
          <option value="!contents ">Table of Contents</option>
          <option value="!help ">Help</option>   
          <option value="!lastmodified ">Last Modification Time</option> 
          <option value="!today ">Today's Date</option>    
        </select>
      </span>
      <span class="ql-formats">
        <select class="ql-size" @change="setWidget($event)"> 
          <option value="leer">-Style Widgets-</option>
          <option value="!style_error()">Error</option>
          <option value="!style_pass()">Pass</option>
          <option value="!style_fail()">Fail</option>
          <option value="!style_ignore()">Ignore</option>  
          <option value="!style_caps{}">Caps</option>  
          <option value="!style_code[]">Code</option>  
          <option value="!style_red[]">Color</option>  
          <option value="!style_code(!style_red[])">Code & Color</option>
          <option value="!style_note[]">Note</option>  
          <option value="!style_right[]">Align Right</option>  
          <option value="!style_meta[]">Meta</option>
          <option value="!style_included[] ">Included</option>   
          <option value="!style_setup[]">Setup</option> 
          <option value="!style_collapse_rim[]">Collapse Rim</option>    
          <option value="!style_pageHelp[]">Page help</option>   
          <option value="x !style_hidden[hidden] x">Hidden</option>   
          <option value="!style_test_output_name[]">Test output name</option>   
          <option value="!style_fit_label[]">Fit label</option>   
          <option value="!style_fit_stacktrace[]">Fit Stacktrace</option>   
          <option value="!style_fit_grey[]">Fit Grey</option>   
        </select>
      </span>

      <span class="hideEditor">
        <!-- @click="closeEditor(item.id)"-->
        <v-btn dark x-small color="red" @click="closeEditor()">
          <v-icon size="17">
            mdi-eye-off
          </v-icon>
        </v-btn>
      </span>
    </div>
    <!--  @blur="closeEditor(item.id)" -->
    <vue-editor v-model="editorContent" class="editor" ref="textEditor" :editorOptions="editorSettings"></vue-editor>
  </div>
</template>

<script>
import { VueEditor} from "vue2-editor";
import { uuid } from 'vue-uuid'; 

export default {
    data: function(){
      return{
        editorContent: undefined,
        editorFitNesseContent: undefined,
        editorObj: {},
        closeIndex: undefined,
        editorSettings: {
          modules: {            
            toolbar: {
              container: '#toolbar',
              handlers: {               
                escape: this.setEscape,
                rule: this.setRule,
                table: this.setTable,
                hashTable: this.setHashTable,
                csClosed: this.setCsClosed,
                csOpen: this.setCsOpen,
                csHidden: this.setCsHidden
              }
            }
          }
        }
      }
    },
    components: {
        VueEditor
    },
    created(){
      if(this.item.name == "editorItem"){
        this.editorContent = this.item.content
      }
      Object.entries(this.enteredValues).forEach(([key, value]) => {
        if(this.item.id === value.id){
          this.closeIndex = key
        }
      })
    },
    props:["enteredValues", "item", "editorIndex", "lastEditor"],
    methods:{
      closeEditor: function(){
        for(let i=0;i<this.editorIndex.length;i++){
          let indexDel = this.editorIndex.findIndex(x => x == this.closeIndex)
          this.editorIndex.splice(indexDel,1)
        }
        this.createFitNesseSyntax()
      },
      setEscape: function(){ 
        let r = window.getSelection().baseNode.parentNode
        let selIndex = r.innerHTML.indexOf(window.getSelection().toString())
        let selLength =  window.getSelection().toString().length
        r.innerHTML = r.innerHTML.slice(0,selIndex)+"!-"+window.getSelection().toString()+"-!" + r.innerHTML.slice(selIndex+selLength)
      },
      setRule: function(){
        let quill = this.$refs.textEditor.quill
        const cursorPosition = quill.getSelection().index
        quill.pasteHTML(cursorPosition,"<hr>")
      },
      setTable: function(){
        let quill = this.$refs.textEditor.quill
        let r = window.getSelection().baseNode.parentNode
        const cursorPosition = quill.getSelection().index
        const selLength = quill.getSelection().length
        let tableSyntax = quill.getText(cursorPosition,selLength).split(" ").join("|")
        let lines = quill.getText(cursorPosition,selLength).split("\n").length
        console.log(lines)
        console.log(tableSyntax)
        if(lines <= 1){
          if(tableSyntax == ""){
            quill.insertText(cursorPosition,"|")
          }else{
            r.innerHTML ="|"+tableSyntax+"|"
          }
        }else{
          //##WIP
          console.log("multipleRows")
        }
        
      },
      setHashTable: function(){
        console.log("hashTable")
      },
      setCsClosed: function(){
        let quill = this.$refs.textEditor.quill
        const selLength = quill.getSelection().length
        if(selLength == 0){
          const cursorPosition = quill.getSelection().index
          quill.insertText(cursorPosition,"!***> section title\n\n*!\n")
        }else{
          let r = window.getSelection().baseNode.parentNode
          let selIndex = r.innerHTML.indexOf(window.getSelection().toString())
          r.innerHTML = r.innerHTML.slice(0,selIndex)+"!***> "+window.getSelection().toString()+"\n\n*!"
        }
      },
      setCsOpen: function(){
        let quill = this.$refs.textEditor.quill
        const selLength = quill.getSelection().length
        if(selLength == 0){
          const cursorPosition = quill.getSelection().index
          quill.insertText(cursorPosition,"!*** section title\n\n*!\n")
        }else{
          let r = window.getSelection().baseNode.parentNode
          let selIndex = r.innerHTML.indexOf(window.getSelection().toString())
          r.innerHTML = r.innerHTML.slice(0,selIndex)+"!*** "+window.getSelection().toString()+"\n\n*!"
        }
      },
      setCsHidden: function(){
        let quill = this.$refs.textEditor.quill
        const selLength = quill.getSelection().length
        if(selLength == 0){
          const cursorPosition = quill.getSelection().index
          quill.insertText(cursorPosition,"!***< section title\n\n*!\n")
        }else{
          let r = window.getSelection().baseNode.parentNode
          let selIndex = r.innerHTML.indexOf(window.getSelection().toString())
          r.innerHTML = r.innerHTML.slice(0,selIndex)+"!***< "+window.getSelection().toString()+"\n\n*!"
        }
      },
      setWidget: function($event){       
        let selValue = $event.target.value
        if(selValue != "leer"){
          let quill = this.$refs.textEditor.quill
          quill.focus()
          const cursorPosition = quill.getSelection().index
          if(selValue == "# "){
            let commentPosition = cursorPosition;
            console.log(quill.getSelection())
            let lines = quill.getText().split("\n").filter(item => item)
            //handle multiple rows/only one row
            console.log(lines)
            //needs Rework #####
            for(let i=0;i<lines.length;i++){
              if(cursorPosition==0){
                console.log(commentPosition)
                quill.insertText(commentPosition, selValue)
                commentPosition = commentPosition+lines[i].length+3
              }else{
                console.log("mid Comment")
              }
            }
          }else{
             quill.insertText(cursorPosition, selValue)
          }
          quill.setSelection(cursorPosition + selValue.length)
        }
      },
      createFitNesseSyntax: function(){
        if(this.editorContent == undefined || this.editorContent == "" ){
          console.log("Nothing to parse")
          if(this.enteredValues[this.closeIndex].name == "editorItem"){
            this.enteredValues.splice(this.closeIndex, 1)
          }
        }else{
          //Wrapper <p>
          this.editorFitNesseContent = this.editorContent.replace(/<p>/g,"").replace(/<\/p>/g,"\n")
          //<br>
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<br>/g,"")
          //bold
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<strong>/g,"'''").replace(/<\/strong>/g,"'''")
          //italics
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<em>/g,"''").replace(/<\/em>/g,"''")
          //strike
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<s>/g,"--").replace(/<\/s>/g,"--")
          //Header1
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<span class="ql-size-huge">/g,"!1 ").replace(/<\/span>/g,"")
          //Header2
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<span class="ql-size-large">/g,"!2 ").replace(/<\/span>/g,"")
          //Header3
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<span class="ql-size-small">/g,"!3 ").replace(/<\/span>/g,"")
          //list
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<ul><li>/g," * ").replace(/<\/li><\/ul>/g,"\n")
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<\/li><li>/g,"\n * ")
          //hr
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<hr>/g,"----\n")
          //align center
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<p class="ql-align-center">/g,"!c ")
          //indent
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/<p class="ql-indent-1">/g,"")
           for(let i=0;i<=8;i++){
            let replace = '<p class="ql-indent-'+i+'">'
            let re = new RegExp(replace,"g");
            this.editorFitNesseContent = this.editorFitNesseContent.replace(re,"")
           }
          //link
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/target="_blank">/g,"")
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/ target="_blank"/g,"")
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/class="ql-size-huge">/g,"!1 ")
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/class="ql-size-large">/g,"!2 ")
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/class="ql-size-small">/g,"!3 ")
          this.editorFitNesseContent = this.editorFitNesseContent.replace(/rel="noopener noreferrer" /g,"")

          let urlMatches = this.editorFitNesseContent.match(/\bhttps?:\/\/\S+/gi)
          if(urlMatches != null){
            for(let i=0;i<urlMatches.length;i++){
              let replace = 'href="'+urlMatches[i]+''
              let re = new RegExp(replace,"g");
              this.editorFitNesseContent = this.editorFitNesseContent.replace(re,"")
              this.editorFitNesseContent = this.editorFitNesseContent.replace(/<a /g,"[[").replace("</a>","]["+urlMatches[i]+"]]")
            }
          }
          if(this.item.name == "editorItem"){
            this.item.content = this.editorContent
            this.item.fitNesseContent = this.editorFitNesseContent 
          }else{
            this.editorObj["id"] = uuid.v4()
            this.editorObj["name"] = "editorItem"
            this.editorObj["content"] = this.editorContent
            this.editorObj["fitNesseContent"] = this.editorFitNesseContent 
            if(this.lastEditor == false){
              this.enteredValues.splice(this.closeIndex, 0, this.editorObj)
            }else{
              // if lastEditor add 2 to the Index, to get it inserted at the bottom (index is modified before in KeywordDropzone, so the normal logic is broken here)
              this.enteredValues.splice(this.closeIndex+2, 0, this.editorObj)
            }
          }
        }
      }
    }
}

</script>


<style scoped>
  .hideEditor{
      float:right;
  }
  .ql-picker{
    min-width: 150px !important;
  }
  .editorWrapper{
    max-width: 95%;
    margin-top: 7px;
    margin-bottom: 7px;
  }
  .ql-formats{
    margin-left: 15px;
  }
</style>
<style>
@import "../assets/styles/vue2-editor.scss";
@import '../assets/styles/quill.core.css';
@import '../assets/styles/quill.bubble.css';
@import '../assets/styles/quill.snow.css';

  .ql-picker-options{
    z-index: 4 !important;
    max-height: 130px !important;
    overflow-y: scroll !important;
    z-index: 2 !important;
  }

.ql-editor{
      min-height: 130px !important;
      max-height: 131px !important;
  }
.ql-tooltip{
  z-index: 2 !important;
  margin-left: 20% !important;
}
</style>