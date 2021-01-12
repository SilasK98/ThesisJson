<template>
    <span v-if="value.type == 'keyword'">
        <b>{{value.label}}</b>
        <v-expansion-panels  v-if="item.attachedKeyword" class="attachedKeyword">
            <v-expansion-panel>
                <v-expansion-panel-header class="addedKeywordHeader" :id="item.id">
                    <!--<v-icon v-if="Object.keys(item.attachedKeyword.parameters).every(r => Object.keys(item.attachedKeyword).includes(r))" size="17"  color="teal" class="statusIcon">mdi-check</v-icon>
                    <v-icon v-else color="error" size="17"  class="statusIcon">mdi-alert-circle</v-icon>-->
                    {{item.attachedKeyword.name}}
                    <v-btn @click="delAttachedKeyword($event);callSetRs()" class="innerDelBtn" :id="item.id" title="innerDelBtn" dark x-small height="24" color="red" elevation="0">
                        <v-icon dark title="innerDelBtn" :id="item.id">mdi-minus</v-icon>
                    </v-btn>     
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                    <br>
                    <div v-for="(addValue, addIndex) in item.attachedKeyword.parameters" :key="addIndex.index"  class="attachedKeywordInputs">             
                        <span v-if="addValue.type == 'text' || !addValue.type">    
                            <label><b>{{addValue.label}}:</b></label>
                            <v-text-field class="detailValue" @change="callSetRs();$forceUpdate()" v-model="item.attachedKeyword[addIndex]"></v-text-field>
                        </span>
                        <span v-if="addValue.type == 'options'">
                            <label><b>{{addValue.label}}:</b></label>
                            <v-select class="detailOption" @change="callSetRs();$forceUpdate()"  v-model="item.attachedKeyword[addIndex]" :items="addValue.options"></v-select>
                        </span>
                    </div>
                </v-expansion-panel-content>
            </v-expansion-panel>
        </v-expansion-panels>
        <v-select v-else 
            max-height="200px;"
            @change="searchAttachedKeyword($event, item.id);callSetRs()"
            :items="item.possibleAttachedKeywords"
        ></v-select>             
    </span>
</template>

<script>
export default {
    props:["value","item","index", "keywords", "enteredValues"],
    methods: {
        delAttachedKeyword: function($event){
           let delId = $event.target.id
            Object.entries(this.enteredValues).forEach(([key, value]) => {
                if(delId === value.id){
                    delete this.enteredValues[key].attachedKeyword
                }
            })
            this.$forceUpdate()
        },
        searchAttachedKeyword: function($event, id){
            this.attachedKeywordId = id
            this.keywords.forEach(item =>{
                this.searchObject(item, $event)
            })
            this.$forceUpdate()
        },
        searchObject: function(item, aK){
            Object.keys(item).forEach(key =>{
                //keep on searching if is object (loop through all nested levels)
                if(typeof item[key] === "object"){
                    this.searchObject(item[key],aK)
                }
                //searched word has to be string and return item
                if(typeof item[key] === "string" && item.name == aK){
                    let searchAsRegEx = new RegExp(aK, "gi");
                    if (item[key].match(searchAsRegEx)){
                        Object.entries(this.enteredValues).forEach(([key, value]) => {
                            if(this.attachedKeywordId === value.id){
                                this.enteredValues[key]["attachedKeyword"] = {}
                                //this.enteredValues[key]["attachedKeyword"]["id"] = item.id
                                this.enteredValues[key]["attachedKeyword"]["name"] = item.name
                                if(item.parameters != undefined){
                                   this.enteredValues[key]["attachedKeyword"]["parameters"] = item.parameters 
                                }else{
                                    this.enteredValues[key]["attachedKeyword"]["parameters"] = {}
                                }
                                this.enteredValues[key]["attachedKeyword"]["syntax"] = item.syntax
                                this.enteredValues[key]["attachedKeyword"]["returnType"] = item.returnType
                            }
                        })       
                    }
                }      
            })
            this.$forceUpdate()
        },
        callSetRs: function(){
            // idk ^^ irgendwie unsauber
          this.$parent.$parent.$parent.$parent.$parent.$parent.$parent.$parent.$parent.$refs.KeywordDropzone.setReadableSyntax(this.item)
        }
    }
}
</script>

<style scoped>
.addedKeywordHeader{
    font-size:12pt;
    min-height: 0 !important;
    padding: 3px 12px;
    background: #ffdca8;
}
.attachedKeyword{
    margin-bottom: 3%;
}
</style>