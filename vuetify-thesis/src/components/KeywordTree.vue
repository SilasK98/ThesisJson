<template>
<div class="keywordTree">

  <v-app>
  <v-card
    elevation="6"
  >  
    <v-sheet :class="searchBarClass">
      <v-text-field
        v-model="search"
        :label="searchTitle"
        dark
        flat
        solo-inverted
        hide-details
        clearable
        clear-icon="mdi-close-circle-outline"
      ></v-text-field>
    </v-sheet>
    <v-card-text>
      <v-treeview
        :items="keywords"
        :search="search"
        open-on-click
        dense
        :open.sync="open"
        transition
      >
        <template slot="append" slot-scope="{ item }">
          <v-icon size="18" style="z-index:202" v-if="!item.children" @mouseover="showHelpText(item)" @mouseleave="closeHelpText()">mdi-help-circle</v-icon>
        </template>
        <template slot="label" slot-scope="{ item }">
            <span v-if="!item.children" @click="useKeyword(item)" @mouseleave="closeHelpText()" :class="{ deprecated: item.deprecated, generated: item.generated}">{{ item.name }}</span>
            <span v-else>{{ item.name }}</span>
        </template>   
      </v-treeview>
    </v-card-text>
  </v-card>
  <v-dialog
        v-model="dialog"
        max-width="29%"
        v-if="dialog != false"
  >
    <v-card>
      <v-card-title class="headline" >
        <span v-if="!keywordHelpItem.deprecated">
          {{keywordHelpItem.name}}
        </span>
        <span v-else>
          <span class="deprecated">{{keywordHelpItem.name}}</span> <br><span class="deprecatedInfo">[Keyword is deprecated]</span>
        </span>
      </v-card-title>
      <v-card-text>
        <span><b>Description:</b></span><br>
        <span>{{keywordHelpItem.helpText}}</span>
      </v-card-text>
    </v-card>
  </v-dialog>
  </v-app>
  </div>
</template>

<script>


import { uuid } from 'vue-uuid'; 
export default {
  props:["keywords","fetchedSpecialKeywords","dA"],
  data: () => ({
    search: null,
    searchTitle: undefined,
    searchBarClass: undefined,
    dialog: false,
    keywordHelpItem: undefined,
    open: [],
    caseSensetive: false,
    attachedKeyword: undefined,
    keywordName: undefined,
    keywordSyntax: undefined,
    selectedSpecialKeyword: undefined,
    keywordReturnValue: undefined,
    tableItems: undefined,
    parameters: {},
    enteredValues: {},
    parameterSyntax: {},
    selectedSpecialKeywordObject: {},
    specialKeywords: [],
    tableBodyKeywords: [],
    addKeywordObjectList: [],
    enteredAddObject: {}
  }),
  computed: {
    filter () {
      return this.caseSensetive
        ? (item, search, textKey) => item[textKey].indexOf(search) > -1 
        : undefined
    },
  },
  created(){
    if(this.dA == true){
      this.searchTitle = "Suchen..."
      this.searchBarClass = "pa-4 secondary lighten-2"
    }else{
      this.searchTitle = "Suchen..."
      this.searchBarClass = "pa-4 primary lighten-2"
    }
  },
  methods: {
    useKeyword: function(item){
      //reset Objects and params
      this.parameters = {}
      this.enteredValues = {}
      this.parameterSyntax = {}
      this.addKeywordObjectList = []
      this.enteredAddObject = {}
      this.specialKeywords = []
      this.tableBodyKeywords = []
      this.selectedSpecialKeywordObject = {}
      this.selectedSpecialKeyword = undefined
      this.attachedKeyword = undefined
      this.keywordName = undefined
      this.keywordSyntax = undefined
      this.keywordReturnType = undefined
      this.tableItems = undefined
      this.keywordSyntax = item.syntax
      this.keywordName = item.name
      if(item.parameters != undefined){
        this.parameterSyntax = item.parameters
      } 
      //if the used item has returnType(String/boolean/int) it can get a SpecialKeyword 
      if(item.returnType && (item.returnType =="String" || item.returnType =="int" || item.returnType =="boolean")){
        this.keywordReturnType = item.returnType
        for(let i=0;i<this.fetchedSpecialKeywords.length;i++){
          this.specialKeywords.push(this.fetchedSpecialKeywords[i].name)
          this.specialKeywords.sort()
        }
      }
      Object.entries(this.parameterSyntax).forEach(([key, value]) => {
        this.parameters[key] = value
        if(this.parameters[key].type == "keyword"){
          this.getEachAttachableItem(this.keywords)
        }
      }) 
      if(item.tableItems){
        this.tableItems = item.tableItems
      }

      this.callGetInputs()
    },
    callGetInputs: function(){
      //maybe add other values of Json later
      this.enteredValues["id"] = uuid.v4()
      this.enteredValues["name"] = this.keywordName
      this.enteredValues["parameters"] = this.parameterSyntax
      this.enteredValues["syntax"] = this.keywordSyntax
          
      if(this.keywordReturnType != undefined){
        this.enteredValues["returnType"] = this.keywordReturnType
      }
      if(this.tableItems != undefined){
        this.enteredValues["tableItems"] = this.tableItems
      }       
      if(Object.keys(this.selectedSpecialKeywordObject).length > 0){
        this.enteredValues["specialKeyword"] = this.selectedSpecialKeywordObject
        this.selectedSpecialKeywordObject = {}
        this.selectedSpecialKeyword = undefined
      }
      if(this.specialKeywords.length > 0){
        this.enteredValues["possibleSpecialKeywords"] = this.specialKeywords
      }
      if(this.addKeywordObjectList.length > 0){
        this.enteredValues["possibleAttachedKeywords"] = this.addKeywordObjectList
      }
      if(this.attachedKeyword != undefined){
        this.enteredValues["attachedKeyword"] = this.enteredAddObject
        this.enteredAddObject = {}
        this.attachedKeyword = undefined
      }    
      this.$root.$refs.KeywordDropzone.getInputs(this.enteredValues)
      this.enteredValues = {}
    },
    getEachAttachableItem: function(object){
      object.forEach(item =>{
        this.searchItem(item)
      })
    },
    searchItem: function(item){
      Object.keys(item).forEach(key =>{
        //keep on searching if is object (loop through all nested levels)
        if(typeof item[key] === "object"){
          this.searchItem(item[key])
        }
        //searched word has to be string and has returnType and not a table item.
        if(typeof item[key] === "string" && (item.returnType == "boolean" || item.returnType == "int" || item.returnType == "String") && !item.tableItems){
          let searchAsRegEx = new RegExp(item.returnType, "gi");
          if (item[key].match(searchAsRegEx)) {
            this.addKeywordObjectList.push(item.name)
            this.addKeywordObjectList.sort()
          }    
        }
      })
    },
    showHelpText: function(item){
      setTimeout(() => {this.keywordHelpItem = item}, 300) 
      setTimeout(() => { this.dialog = true }, 300) 
    },
    closeHelpText: function(){
      setTimeout(() => {this.dialog = false }, 299)
      setTimeout(() => {this.keywordHelpItem = undefined}, 299)     
    }
  }
}
</script>

<style scoped>
.keywordTree{
  max-height: 67vh;
  overflow: auto; 
}
.showMoreBtn{
  float: right;
}
.useBtn:hover{
  color: black
}
.deprecated{
   text-decoration: line-through;
   opacity: 0.7
}
.deprecatedInfo{
  font-size: 14pt;
}
.headline{
  background-color: #5f5a5a;
  margin-bottom: 3%;
  color: white;
}
</style>

<style>
.theme--light.v-treeview{
  color: black
}
</style>