
<template>
    <div id="dropzone">
        <v-app>
            <v-card id="vCardDropzone"
                class="mx-auto"
                max-width="1500"
                min-width="95%"
                min-height="80vh"
                max-height="0"
                style="padding:3%;overflow:auto;"
                outlined
                elevation="6" 
                
            >
                <draggable
                    v-if="enteredValues.length > 0"
                    :key="enteredValues.id"
                    :list="enteredValues"
                    v-bind="dragOptions"
                    :move="onItemMove"
                    handle=".draggableElement"
                >   
                <!-- :multi-drag="true"
                    multiDragKey="ctrlKey"
                    selected-class="multi-drag"-->
                <div v-for="(item,index) in enteredValues" :key="index.index" class="vCardItem">
                    <div v-if="index==0 && enteredValues[0].name != 'editorItem'  && showPaste == false && selectedKeywords.length == 0" 
                    class="textEditorPlaceholder" @click="setTextEditor('TextEditor', index, $event, item)"></div>
                    <div v-html="item.content" v-if="item.name == 'editorItem' && !editorIndex.includes(index)" :id="item.id" 
                        @click="setTextEditor('TextEditor', index, $event, item)" :class="{ disabledTextEditor: item.disabled }" class="editorInsertedText draggableElement"></div>
                        <span v-if="enteredValues[index-1]">
                            <div v-if="item.name != 'editorItem' && enteredValues[index-1].name != 'editorItem' && showPaste == false && selectedKeywords.length == 0" 
                                class="textEditorPlaceholder" @click="setTextEditor('TextEditor', index, $event, item)">
                            </div>
                        </span>
                    <component v-if="editorIndex.includes(index)" v-bind:item="item" :enteredValues="enteredValues"  :lastEditor="false" :editorIndex="editorIndex" :is="component"/>
                    <v-expansion-panels class="exPanel" v-if="item.name != 'editorItem'" :readonly="setReadonly" :value="expansionStatus" :class="{ disabled: item.disabled }">
                        <v-expansion-panel active-class="highlightedHeader" @click.native="selectMultiKeywords($event,item, index)"> 
                            <v-expansion-panel-header class="expansionHeader draggableElement" :id="item.id" :class="{ disabledHeader: item.disabled }">
                                <span class="preInfo" v-html="item.readableSyntax"></span>
                                <span class="keywordName">{{item.name}}</span>                              
                                <span class="statusIconSpan">
                                    <v-icon v-if="checkStatusIcon(item)" size="17" class="statusIcon" color="teal">mdi-check</v-icon>
                                    <v-icon v-else color="error" size="17" class="statusIcon">mdi-alert-circle</v-icon> 
                                </span>
                            </v-expansion-panel-header>
                            <v-expansion-panel-content class="expansionContent">
                                <br>
                                <span v-for="(value, index) in item.parameters" :key="index.index">
                                    <span v-if="value.type == 'text' || !value.type">
                                        <b>{{value.label}}:</b>
                                        <v-text-field class="detailValue" @keyup="setReadableSyntax(item)" v-model="item[index]" :id="index"></v-text-field>
                                    </span>
                                    <span v-if="value.type == 'options'">
                                        <b>{{value.label}}</b>
                                        <v-select class="detailOption" @change="setReadableSyntax(item)" v-model="item[index]" :id="index" :items="value.options"></v-select>
                                    </span>
                                    <AttachedKeyword v-bind:value="value" :item="item" :index="index" :keywords="keywords" :enteredValues="enteredValues"></AttachedKeyword>
                                </span>
                                <SpecialKeyword v-bind:item="item" :index="index" :keywords="keywords" :enteredValues="enteredValues" :fetchedSpecialKeywords="fetchedSpecialKeywords"></SpecialKeyword>
                                <TableKeyword v-bind:item="item" :index="index" :keywords="keywords" :enteredValues="enteredValues"></TableKeyword>
                            </v-expansion-panel-content>
                        </v-expansion-panel>
                    </v-expansion-panels> 
                        <!--TextEditor at the End-->
                    <div v-if="enteredValues[index+1] == null && enteredValues[index].name != 'editorItem' && showPaste == false && selectedKeywords.length == 0"
                        class="textEditorPlaceholder" @click="setTextEditor('TextEditor', index+1, $event, item)"></div>

                    <component v-if="index+1 == enteredValues.length && editorIndex.includes(index+1)"
                        v-bind:item="item" :enteredValues="enteredValues" :editorIndex="editorIndex" :lastEditor="true" :is="component"/>

                    <v-speed-dial :open-on-hover="true" class="openMenu" direction="right" transition="scale-transition">
                        <template v-slot:activator>
                            <v-icon >mdi-dots-vertical</v-icon>
                        </template>
                        <v-btn class="menuBtn" fab dark color="green" @click="copyKeyword(item)">
                            <v-icon size="13">mdi-content-copy</v-icon>
                        </v-btn>
                        <v-btn class="menuBtn" fab dark color="blue" @click="duplicateKeyword(item, index)">
                            <v-icon  size="13">mdi-content-duplicate</v-icon>
                        </v-btn>
                            <v-btn class="menuBtn" fab dark color="red" @click="delKeyword(index)">
                            <v-icon size="13" >mdi-delete</v-icon>
                        </v-btn>
                    </v-speed-dial> 
                    <v-icon size="16" class="pasteArrow" v-if="showPaste == true" @click="pasteKeyword(index)">mdi-arrow-right-drop-circle</v-icon>
                </div>
                <!--<div class="textEditorPlaceholder" @click="setTextEditor('TextEditor', 3, $event, 0)"></div>-->
                </draggable>
            </v-card>             
        </v-app>
        <div class="MultiBtnWrapper">
            <v-btn fab dark id="cancelSel" @click="cancleMulti()"><v-icon size="27">mdi-close</v-icon></v-btn>
            <v-btn fab dark id="copySelBtn" @click="copyMultiple()"><v-icon size="23">mdi-content-copy</v-icon></v-btn>
            <v-btn fab dark id="cutPasteBtn" @click="cutPasteMultiple()"><v-icon size="23">mdi-content-cut</v-icon></v-btn>
            <v-btn fab dark id="dupSelBtn" @click="duplicateMultiple();cancleMulti()"><v-icon  size="23">mdi-content-duplicate</v-icon></v-btn>
            <v-btn fab dark id="delSelBtn" @click="deleteMultiple();cancleMulti()"><v-icon size="27" >mdi-delete</v-icon></v-btn>
            <v-btn fab dark id="disableSelBtn" @click="disableMultiple();cancleMulti()"><v-icon size="27" >mdi-crosshairs-off</v-icon></v-btn>
            <v-btn id="saveAllBtn" @click="transferPostData()">save</v-btn> 
        </div>
    </div>
</template>

<script>
import draggable from "vuedraggable"
import TextEditor from "./TextEditor.vue"
import AttachedKeyword from "./AttachedKeyword.vue"
import SpecialKeyword from "./SpecialKeyword.vue"
import TableKeyword from "./TableKeyword.vue"
import { uuid } from 'vue-uuid'; 

export default {
    data: () => ({
       enteredValues: [],
       readableSyntax: [],
       expansionStatus: undefined,
       setReadonly: false,
       component: false,
       editorIndex: [],
       copiedItem: undefined,
       showPaste: false,
       cutPaste: false,
       selectedKeywords: [],
       lastIndexMulti: undefined,
       copiedMulti: []
    }),
    props:["keywords","fetchedSpecialKeywords"],
    created() {
        //register component to access it from KeywordTree
        this.$root.$refs.KeywordDropzone = this
    },
    components:{
        draggable,
        TextEditor,
        AttachedKeyword,
        SpecialKeyword,
        TableKeyword
    },
    computed: {
        dragOptions() {
            return {
                animation: 280
            }
        }
    },
    methods:{
        getInputs: function(item){
            this.enteredValues.push(item)
            console.log(item)
            console.log(this.enteredValues)
        },
        checkStatusIcon: function(item){
            if(item.attachedKeyword){
                return Object.keys(item.attachedKeyword.parameters).every(r => Object.keys(item.attachedKeyword).includes(r))
            }else if(item.tableItems){
                if(item.usedTableItems){
                    var loopCounter = item.usedTableItems.length-1
                    for(let i=0;i<item.usedTableItems.length;i++){
                        if(Object.keys(item.usedTableItems[loopCounter].parameters).every(r => Object.keys(item.usedTableItems[loopCounter]).includes(r))){
                            return true
                        }
                    }
                }else{
                    return false
                }
            }else{
                return Object.keys(item.parameters).every(r => Object.keys(item).includes(r))
            }
        },
        onItemMove: function(){
            this.$forceUpdate()
            console.log(this.enteredValues)
        },
        preventExpansion: function(){
            this.setReadonly = true
            this.expansionStatus = 1
        },
        delKeyword: function(index){
           this.enteredValues.splice(index,1)
        },
        duplicateKeyword: function(item, index){
            let dupItem = Object.assign({}, item)
            if(item.attachedKeyword){
                var dupAttachedItem = Object.assign({}, item.attachedKeyword)
                dupItem.attachedKeyword = dupAttachedItem
            }
            if(item.specialKeyword){
                var dupSpecialItem = Object.assign({}, item.specialKeyword)
                dupItem.specialKeyword = dupSpecialItem
            }
            /*
            if(item.usedTableItems){
                var dupUsedTableItems = Object.assign([{}], item.usedTableItems)
                dupItem.usedTableItems = dupUsedTableItems
            }*/
            dupItem.id = uuid.v4()
            this.enteredValues.splice(index, 0, dupItem)
        },
        copyKeyword: function(item){
            this.copiedItem = Object.assign({}, item)
            if(item.attachedKeyword){
                var copiedAttachedItem = Object.assign({}, item.attachedKeyword)
                this.copiedItem.attachedKeyword = copiedAttachedItem
            }
            if(item.specialKeyword){
                var copiedSpecialItem = Object.assign({}, item.specialKeyword)
                this.copiedItem.specialKeyword = copiedSpecialItem
            }
            /*
            if(item.usedTableItems){
                var copiedUsedTableItems = Object.assign({}, item.usedTableItems)
                this.copiedItem.usedTableItems = copiedUsedTableItems
            }*/
            this.copiedItem.id = uuid.v4()
            this.showPaste = true
            var allItems = document.querySelectorAll(".vCardItem")
            for(let i=0;i<allItems.length;i++){
                allItems[i].style.marginTop = "8px"
                allItems[i].style.marginBottom= "8px"
            }
        },
        pasteKeyword: function(index){
            let pasteIndex = index+1
            if(this.copiedItem != undefined){
               this.enteredValues.splice(pasteIndex, 0, this.copiedItem) 
               this.copiedItem = undefined
               var allItems = document.querySelectorAll(".vCardItem")
                for(let i=0;i<allItems.length;i++){
                    allItems[i].style.marginTop = "0"
                    allItems[i].style.marginBottom= "0"
                }
            }else{
                for(let i=0;i<this.copiedMulti.length;i++){
                    this.enteredValues.splice(pasteIndex+i, 0, this.copiedMulti[i])
                }
                if(this.cutPaste == true){
                    for(let i=0;i<this.selectedKeywords.length;i++){
                        let delIndex = this.enteredValues.map(function(e) { return e.id; }).indexOf(this.selectedKeywords[i].id)
                        this.enteredValues.splice(delIndex,1)
                    }
                }
                this.cutPaste = false  
                this.copiedMulti = []
                this.selectedKeywords = []
            }
            this.showPaste = false
        },
        selectMultiKeywords: function($event,item, index){
            if($event.ctrlKey){
                this.dragOptions.disabled=true
                var valObj = this.selectedKeywords.filter(function(elem){
                    return (elem.id == item.id)
                })
                if(valObj.length > 0){
                    document.getElementById(item.id).style.backgroundColor = "#FFFFFF"
                    let indexDel = this.selectedKeywords.findIndex(x => x.id === valObj[0].id)
                    this.selectedKeywords.splice(indexDel,1)
                }else{
                    document.getElementById(item.id).style.backgroundColor = "#c8ebfb"
                    this.selectedKeywords.push(item)
                }    
            }else{
                //reset the expansion prevention
                this.setReadonly = false
                this.expansionStatus = undefined
            }
            this.lastIndexMulti = index
            console.log(this.setReadonly)
            this.$forceUpdate()
        },
        cancleMulti: function(){
            this.selectedKeywords = []
            this.copiedItem = undefined
            this.copiedMulti = []
            this.showPaste = false
            this.cutPaste = false
        },
        duplicateMultiple: function(){
            for(let i=0;i<this.selectedKeywords.length;i++){
                let dupItem = Object.assign({}, this.selectedKeywords[i])
                if(this.selectedKeywords[i].attachedKeyword){
                    var dupAttachedItem = Object.assign({}, this.selectedKeywords[i].attachedKeyword)
                    dupItem.attachedKeyword = dupAttachedItem
                }
                if(this.selectedKeywords[i].specialKeyword){
                    var dupSpecialItem = Object.assign({}, this.selectedKeywords[i].specialKeyword)
                    dupItem.specialKeyword = dupSpecialItem
                }
                /*
                if(this.selectedKeywords[i].usedTableItems){
                    var dupUsedTableItems = Object.assign({}, this.selectedKeywords[i].usedTableItems)
                    dupItem.usedTableItems = dupUsedTableItems
                }*/
                dupItem.id = uuid.v4()
                this.enteredValues.splice(this.lastIndexMulti, 0, dupItem)
            }
        },
        copyMultiple: function(){
            for(let i=0;i<this.selectedKeywords.length;i++){
                this.copiedMulti.push(Object.assign({}, this.selectedKeywords[i]))
                if(this.selectedKeywords[i].attachedKeyword){
                    var copiedAttachedItem = Object.assign({}, this.selectedKeywords[i].attachedKeyword)
                    this.copiedMulti[i].attachedKeyword = copiedAttachedItem
                }
                if(this.selectedKeywords[i].specialKeyword){
                    var copiedSpecialItem = Object.assign({}, this.selectedKeywords[i].specialKeyword)
                    this.copiedMulti[i].specialKeyword = copiedSpecialItem
                }
                /*
                if(this.selectedKeywords[i].usedTableItems){
                    var copiedUsedTableItems = Object.assign({}, this.selectedKeywords[i].usedTableItems)
                    this.copiedMulti[i].usedTableItems = copiedUsedTableItems
                }*/
                this.copiedMulti[i].id = uuid.v4()
            }
            this.showPaste = true
        },
        cutPasteMultiple: function(){
            for(let i=0;i<this.selectedKeywords.length;i++){
                this.copiedMulti.push(Object.assign({}, this.selectedKeywords[i]))
                if(this.selectedKeywords[i].attachedKeyword){
                    var copiedAttachedItem = Object.assign({}, this.selectedKeywords[i].attachedKeyword)
                    this.copiedMulti[i].attachedKeyword = copiedAttachedItem
                }
                if(this.selectedKeywords[i].specialKeyword){
                    var copiedSpecialItem = Object.assign({}, this.selectedKeywords[i].specialKeyword)
                    this.copiedMulti[i].specialKeyword = copiedSpecialItem
                }
                this.copiedMulti[i].id = uuid.v4()
            }
            this.showPaste = true
            this.cutPaste = true
        },
        deleteMultiple: function(){
           for(let i=0;i<this.selectedKeywords.length;i++){
              let delIndex = this.enteredValues.map(function(e) { return e.id; }).indexOf(this.selectedKeywords[i].id)
              this.enteredValues.splice(delIndex,1)
           }
        },
        disableMultiple: function(){
            for(let i=0;i<this.selectedKeywords.length;i++){
                let disableIndex = this.enteredValues.map(function(e) { return e.id; }).indexOf(this.selectedKeywords[i].id)
                if(!this.enteredValues[disableIndex].disabled){
                   this.enteredValues[disableIndex]["disabled"] = true 
                }else{
                    delete this.enteredValues[disableIndex]["disabled"]
                }
                
            }
        },
        setTextEditor: function(val, index, $event, item){
            if($event.ctrlKey){
                this.selectMultiKeywords($event,item, index)
            }else{
               this.editorIndex.push(index)
               this.component = val 
            }
        },
        setReadableSyntax: function(item){
            //Handling attached Keyword
             if(item.attachedKeyword){
                var attachedSplittedSyntax = []
                attachedSplittedSyntax = item.attachedKeyword.syntax.split("|").filter(function(el) {return el.length != 0})
                for(let i=0;i<attachedSplittedSyntax.length;i++){
                    if(attachedSplittedSyntax[i].includes("%")){
                        if(item.attachedKeyword[attachedSplittedSyntax[i]] != undefined){
                            if(item.attachedKeyword[attachedSplittedSyntax[i]].length > 15){
                                attachedSplittedSyntax[i] = "<span class='rsParameter'>"+item.attachedKeyword[attachedSplittedSyntax[i]].slice(0,15)+"...</span>"
                            }else{
                                attachedSplittedSyntax[i] = "<span class='rsParameter'>"+item.attachedKeyword[attachedSplittedSyntax[i]]+"</span>"
                            }
                        }else{
                            attachedSplittedSyntax[i] = "<span class='rsParameter'></span>"
                        }
                    }
                }
                var attachedSytnax = attachedSplittedSyntax.join(" ")
            }
            //handling the normal fields/syntax
            var splittedSyntax = []
            splittedSyntax = item.syntax.split("|").filter(function(el) {return el.length != 0})
            for(let i=0;i<splittedSyntax.length;i++){
                if(splittedSyntax[i].includes("%")){
                    var paramObj = item.parameters[splittedSyntax[i]]
                    if(item[splittedSyntax[i]] != undefined){
                        if(item[splittedSyntax[i]].length > 30){
                            splittedSyntax[i] = "<span class='rsParameter'>"+item[splittedSyntax[i]].slice(0,30)+"...</span>"
                        }else{
                            splittedSyntax[i] = "<span class='rsParameter'>"+item[splittedSyntax[i]]+"</span>"
                        }
                    }else if(paramObj.type == "keyword"){
                        //insert attached Keyword to correct position
                        if(attachedSytnax != undefined){
                            splittedSyntax[i] = "<span class='rsAttachedKeyword'>"+attachedSytnax+"</span>"
                        }else{
                            splittedSyntax[i] = "<span class='rsAttachedKeyword'> </span>"
                        }
                    }
                    else{
                        splittedSyntax[i] = "<span class='rsParameter'></span>"
                    }
                }
            }
            let readableSyntax = splittedSyntax.join(" ")
            /*special Keyword and tableItems are always at the end of the readableSyntax 
            handling special Keywords*/
            if(item.specialKeyword){
                if(item.specialKeyword["%2"] == undefined){
                    item.specialKeyword["%2"] = ""
                }
                let specialKeywordFixture = item.specialKeyword.syntax.split("|").filter(function(el) {return el.length != 0})
                readableSyntax = readableSyntax.concat("  <span class='rsSpecialKeyword'>"+specialKeywordFixture[1]+"<span class='rsInnerSpecialKeyword'> "+item.specialKeyword["%2"]+"</span></span>")
            }
            //handling table keywords
            if(item.usedTableItems){
                var tableSplittedSyntax = []
                for(let a=0;a<item.usedTableItems.length;a++){
                    tableSplittedSyntax = item.usedTableItems[a].syntax.split("|").filter(function(el) {return el.length != 0})
                    for(let i=0;i<tableSplittedSyntax.length;i++){
                        if(tableSplittedSyntax[i].includes("%")){
                            if(item.usedTableItems[a][tableSplittedSyntax[i]] != undefined){
                                if(item.usedTableItems[a][tableSplittedSyntax[i]].length > 15){
                                    tableSplittedSyntax[i] = "<span class='rsParameter'>"+item.usedTableItems[a][tableSplittedSyntax[i]].slice(0,15)+"...</span>"
                                }else{
                                    tableSplittedSyntax[i] = "<span class='rsParameter'>"+item.usedTableItems[a][tableSplittedSyntax[i]]+"</span>"
                                }
                            }else{
                                tableSplittedSyntax[i] = "<span class='rsParameter'></span>"
                            }
                        }
                    }
                    tableSplittedSyntax = tableSplittedSyntax.join(" ")
                    //need to check this
                   // readableSyntax = readableSyntax.concat("<br><br>"+tableSplittedSyntax)
                    readableSyntax = readableSyntax.concat("<br><br><span class='rsTableKeyword'>"+tableSplittedSyntax+"</span>")
                }    
                readableSyntax = readableSyntax.concat("<br><br>")
            }
            item["readableSyntax"] = readableSyntax
            this.$forceUpdate()
        },
        transferPostData: function(){
            this.$parent.rebuildSyntax(this.enteredValues)
        }
    },
    watch: {
        selectedKeywords: function(val){
            var allDotMenus = document.querySelectorAll(".openMenu")
            var allItems = document.querySelectorAll(".vCardItem")
            var exHeader = document.querySelectorAll(".draggableElement")
            console.log(val)
            if(val.length > 0){
                document.getElementById("copySelBtn").style.display = "inline"
                document.getElementById("cutPasteBtn").style.display = "inline"
                document.getElementById("dupSelBtn").style.display = "inline"
                document.getElementById("delSelBtn").style.display = "inline"
                document.getElementById("cancelSel").style.display = "inline"
                document.getElementById("disableSelBtn").style.display = "inline"
                for(let i=0;i<allDotMenus.length;i++){
                    allDotMenus[i].style.display = "none"
                }
                
                for(let i=0;i<allItems.length;i++){
                        allItems[i].style.marginTop = "8px"
                        allItems[i].style.marginBottom= "8px"
                }
            }else{
                document.getElementById("copySelBtn").style.display = "none"
                document.getElementById("cutPasteBtn").style.display = "none"
                document.getElementById("dupSelBtn").style.display = "none"
                document.getElementById("delSelBtn").style.display = "none"
                document.getElementById("cancelSel").style.display = "none"
                document.getElementById("disableSelBtn").style.display = "none"
                this.setReadonly = false
                this.expansionStatus = undefined
                for(let i=0;i<allDotMenus.length;i++){
                    allDotMenus[i].style.display = "inline"
                }
                for(let i=0;i<allItems.length;i++){
                    allItems[i].style.marginTop = "0"
                    allItems[i].style.marginBottom= "0"
                }
                for(let i=0;i<exHeader.length;i++){
                    exHeader[i].style.backgroundColor = "#FFFFFF"
                }
                this.dragOptions.disabled=false
            }
        },
        showPaste: function(val){
            if(val == true){
                document.getElementById("cancelSel").style.display = "inline"
                document.getElementById("copySelBtn").style.display = "none"
                document.getElementById("cutPasteBtn").style.display = "none"
                document.getElementById("dupSelBtn").style.display = "none"
                document.getElementById("delSelBtn").style.display = "none"
                document.getElementById("disableSelBtn").style.display = "none"
            }else{
                document.getElementById("cancelSel").style.display = "none"
            }
        }
    },
    mounted(){
        window.addEventListener('keydown', (e) => {
            if (e.ctrlKey) {
                this.preventExpansion()
            }
        })
    },
}
</script>



<style>
.editorInsertedText p{
    margin-bottom: 0;
}
.disabled{
    opacity: 0.5;
}
.disabledHeader .keywordName{
    opacity: 1 !important;
    color: #da4e02 !important;
}
.disabledTextEditor{
    color: #da4e02 !important;
    opacity: 0.5;
}
.rsParameter{
    display: inline-block;
    min-height: 10px;
    min-width: 35px;
    font-size: 10pt;
    padding: 2px 12px;
    border-radius: 10px;
    background-color: #dedede;
    font-weight: normal !important;
}

.multi-drag{
    background-color: blue;
}
.rsSpecialKeyword{
    display: inline-block;
    font-size: 10pt;
    padding: 3px 17px;
    border-radius: 10px;
    background-color: #9cbf9b;
    font-weight: bold !important;
}
.rsAttachedKeyword{
    display: inline-block;
    font-size: 10pt;
    padding: 3px 17px;
    border-radius: 10px;
    background-color: #ffdca8;
    min-height: 10px;
    font-weight: bold !important;
}

.rsTableKeyword{
    font-size: 10pt;
    border-radius: 4px;
    min-height: 10px;
    margin-bottom: 1%;
    font-weight: bold !important;
}


.rsInnerSpecialKeyword{
    display: inline-block;
    min-height: 10px;
    min-width: 35px;
    font-size: 10pt;
    padding: 1px 10px;
    border-radius: 10px;
    margin-left: 5px;
    background-color: #dedede;
    font-weight: normal !important;
}
.editorInsertedText{
    cursor: text;
    padding: 2px 0 2px 0;
    max-width: 95%;
    border-radius: 4px;
}
.pasteArrow{
    margin-left: -23px;
    float: left;
    max-height: 7px;
}
.textEditorPlaceholder{
    width: 95%;
    float: left;
    padding-bottom: 8px;
    transition: 0.4s;
}
.textEditorPlaceholder:hover{
    border: 1px dashed black;
    height: 10px !important;
    padding-top: 0.5%;
    cursor: text;
}
.exPanel{
    border: 1px solid rgb(190, 190, 190);
}
.openMenu{
    float:right;
    top: -28px;
    margin-right: 2%;
    max-height: 0;
}
.menuBtn{
    max-width: 20px;
    max-height: 20px;
}
.v-item-group{
    max-width:95%;
}
.v-speed-dial--direction-right .v-speed-dial__list{
    left: 5px !important;
    height: inherit !important;
    z-index: 2;
}
.v-speed-dial__list .v-btn{
    margin:1px !important;
}
.ghost{
  opacity: 0.5;
  background: #c8ebfb;
  cursor: grabbing;
}
.addedKeywordHeader div{
    margin-left: 0 !important;
}
.v-expansion-panel-header__icon{
    margin-left: 0 !important;
}
.v-application--wrap{
    min-height: 0;
    margin-bottom: 2%;
}
.keywordName{
    max-width: 120px;
    font-size: 10pt;
    opacity: 0.7;
}

.preInfo{
    font-size: 10pt;
    font-weight: bold;
}
.expansionHeader{
    font-size: 9pt;
    color: black;
    min-height: 0 !important;
    padding: 3px 12px;
}

.statusIconSpan{
    max-width:fit-content;
    margin-left: auto;
    margin-right:2%;
    margin-left: 2%;
}
.statusIcon{
    max-width:2px;
    padding-right:2%;
    margin-bottom:3px;
}
.expansionContent{
    font-size: 10pt;
}
.detailValue, .detailOption{
    font-size: 10pt;
    padding: 0 !important;
    margin: 0 !important;
}
.editBtn{
    float: right;
    margin-right: 0.2%;
    max-width: fit-content;
}
.delBtn{
    float: right;
    max-width: fit-content;
}
.innerDelBtn{
    max-width: 3%;
    margin-left: auto;
}
#saveAllBtn{
   background-color: #337ab7 !important;
    color: #ffffff;
    float: right;
    border: 1px solid transparent;
    margin-right: 2%;
}

#dupSelBtn, #delSelBtn, #copySelBtn, #cancelSel, #disableSelBtn, #cutPasteBtn{
    background-color: #337ab7 !important;
    border: 1px solid transparent;
    color: #ffffff;
    margin-right: 1%;
    display: none;
}
.MultiBtnWrapper{
    margin-left: 425px;
}
#copySelBtn{
    background-color:  #4CAF50 !important;
}
#cutPasteBtn{
    background-color:  #358080 !important;
}
#disableSelBtn{
    background-color: #da4e02 !important;
}
#dupSelBtn{
    background-color: #2196F3 !important;
}
#delSelBtn{
    background-color: #F44336 !important;
}
.dragItem{
    cursor: grab;
}

.ql-size-huge{
  font-size: 2.5em;
}
.ql-size-large{
    font-size: 1.5em;
}
.ql-align-center{
    text-align: center;
}
.ql-indent-1{
    padding-left: 5%;
}
.ql-indent-2{
    padding-left: 10%;
}
.ql-indent-3{
    padding-left: 15%;
}
.ql-indent-4{
    padding-left: 20%;
}
.ql-indent-5{
    padding-left: 25%;
}
.ql-indent-6{
    padding-left: 30%;
}
.ql-indent-7{
    padding-left: 35%;
}
.ql-indent-8{
    padding-left: 40%;
}
</style>