<template>
    <span v-if="item.tableItems">
        <p class="headTableKeyword">{{item.syntax.replace(/\|/g,"")}}</p>
        <draggable
            v-if="item.usedTableItems"
            :list="item.usedTableItems"
            @change="updatePosition()"
            v-bind="dragOptions"
            handle=".addedKeywordHeader"
        >   
        <div v-for="(tObj, tObjIndex) in item.usedTableItems" :key="tObjIndex.index" class="tableBodyKeyword">
        <v-expansion-panels  >
            <v-expansion-panel>
                <v-expansion-panel-header class="addedKeywordHeader" :id="item.id">
                    <!--<v-icon v-if="Object.keys(item.attachedKeyword.parameters).every(r => Object.keys(item.attachedKeyword).includes(r))" size="17"  color="teal" class="statusIcon">mdi-check</v-icon>
                    <v-icon v-else color="error" size="17"  class="statusIcon">mdi-alert-circle</v-icon>-->
                    {{tObj.name}}
                    <v-btn @click.native.stop="delTableKeyword(tObjIndex);callSetRs()" class="innerDelBtn" :id="item.id" title="innerDelBtn" dark x-small height="24" color="red" elevation="0">
                        <v-icon dark title="innerDelBtn" :id="item.id">mdi-minus</v-icon>
                    </v-btn>     
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                    <br>
                    <div v-for="(addValue, addIndex) in tObj.parameters" :key="addIndex.index">        
                        <span v-if="addValue.type == 'text' || !addValue.type">    
                            <label><b>{{addValue.label}}:</b></label>
                            <v-text-field class="detailValue" @keyup="callSetRs()" v-model="tObj[addIndex]"></v-text-field>
                        </span>
                        <span v-if="addValue.type == 'options'">
                            <label><b>{{addValue.label}}:</b></label>
                            <v-select class="detailOption" @keyup="callSetRs()" v-model="tObj[addIndex]" :items="addValue.options"></v-select>
                        </span>
                    </div>
                </v-expansion-panel-content>
            </v-expansion-panel>
        </v-expansion-panels>
        </div>
        </draggable>
        <div  class="selectTableBodyKeywordWrapper">
            <v-select 
                @change="searchTableBodyKeyword($event, item);callSetRs()"
                ref="selectedTableItem"
                :items="tableItemNames"
                label="Add Keyword"         
                solo
                prepend-icon="mdi-table-row-plus-after"
                class="selectTableBodyKeyword"
            ></v-select>  
        </div>  
    </span>
</template>
-
<script>
import draggable from "vuedraggable"
export default {
    data: () => ({
        bodyKeywordsCounter: 0,
        selectedTableItems: [],
        tableItemNames: []
    }),
    components:{
        draggable
    },
    created(){
        this.$forceUpdate()
        if(this.item.tableItems){
            for(let i=0;i<this.item.tableItems.length;i++){
                this.tableItemNames.push(this.item.tableItems[i].name)
            }
        }
    },
    props:["item","index", "keywords", "enteredValues"],
    methods:{
        searchTableBodyKeyword: function($event, usedItem){
           usedItem.tableItems.forEach(uItem =>{
                if($event == uItem.name){       
                    this.selectedTableItems[this.bodyKeywordsCounter] = {}
                    this.selectedTableItems[this.bodyKeywordsCounter]["name"] = uItem.name
                    this.selectedTableItems[this.bodyKeywordsCounter]["parameters"] = uItem.parameters
                    this.selectedTableItems[this.bodyKeywordsCounter]["syntax"] = uItem.syntax
                }   
            })
            this.bodyKeywordsCounter++
            usedItem["usedTableItems"] = {}
            usedItem["usedTableItems"] = this.selectedTableItems
            this.$refs["selectedTableItem"].reset()
            this.$forceUpdate()
        },
        updatePosition: function(){
            this.$forceUpdate()
            this.$parent.$parent.$parent.$parent.$parent.$parent.$parent.$parent.$parent.$refs.KeywordDropzone.setReadableSyntax(this.item)
        },
        delTableKeyword: function(index){
            this.item.usedTableItems.splice(index,1)
            this.bodyKeywordsCounter--
            this.$forceUpdate()
        },
        callSetRs: function(){
          this.$parent.$parent.$parent.$parent.$parent.$parent.$parent.$parent.$parent.$refs.KeywordDropzone.setReadableSyntax(this.item)
        }
    },
    computed: {
        dragOptions() {
            return {
                animation: 280,
            }
        }
    },
}
</script>


<style scoped>
.headTableKeyword{
    margin-top: 1%;
    font-size: 12pt;
    margin-bottom: 1%;
    margin-top: 0;
}
.addedKeywordHeader{
    font-size:10pt;
    min-height: 0 !important;
    padding: 0px 12px;
    background: #ababab;
}
.tableBodyKeyword{
    margin-bottom: 0.5%;
}

.selectTableBodyKeywordWrapper{
    margin-top: 4%;
    width: 95%;
}
</style>
