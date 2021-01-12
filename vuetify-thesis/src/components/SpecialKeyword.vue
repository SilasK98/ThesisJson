<template>
    <span v-if="item.specialKeyword || item.possibleSpecialKeywords">
        <b>Special Keyword (optional):</b>
        <v-expansion-panels v-if="item.specialKeyword">
            <v-expansion-panel>
                <v-expansion-panel-header class="addedKeywordHeader" :id="item.id">
                    <!--<v-icon size="17"  color="teal" class="statusIcon">mdi-check</v-icon>
                    <v-icon  color="error" size="17"  class="statusIcon">mdi-alert-circle</v-icon>-->
                    {{item.specialKeyword.name}}
                    <v-btn @click="delSpecialKeyword($event);callSetRs()" class="innerDelBtn" :id="item.id" title="innerDelBtn" dark x-small height="24" color="red" elevation="0">
                        <v-icon dark title="innerDelBtn" :id="item.id">mdi-minus</v-icon>
                    </v-btn>     
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                    <br>
                    <div class="attachedKeywordInputs" v-for="(addValue, addIndex) in item.specialKeyword.parameters" :key="addIndex.index">
                        <span v-if="item.returnType == 'boolean' && addValue.type != 'keyword'">
                            <label><b>{{addValue.label}}</b></label>
                            <v-select class="detailOption" @change="$forceUpdate();callSetRs()" v-model="item.specialKeyword[addIndex]" :items="['true','false']"></v-select>
                        </span>             
                        <span v-if="(item.returnType == 'String' || item.returnType == 'int') && addValue.type != 'keyword'">    
                            <label><b>{{addValue.label}}:</b></label>
                            <v-text-field class="detailValue" @change="$forceUpdate();callSetRs()" v-model="item.specialKeyword[addIndex]"></v-text-field>
                        </span>
                    </div>
                </v-expansion-panel-content>
            </v-expansion-panel>
        </v-expansion-panels>
        <v-select v-else 
            max-height="200px;"
            @change="searchSpecialKeyword($event, item.id);callSetRs()"
            :items="item.possibleSpecialKeywords"
            :hint="'gets attached at the End of '+item.name"
            persistent-hint
        ></v-select>             
    </span>
</template>

<script>
export default {
    props:["item","index", "keywords", "enteredValues","fetchedSpecialKeywords"],
    methods:{
        delSpecialKeyword: function($event){
           let delId = $event.target.id
            Object.entries(this.enteredValues).forEach(([key, value]) => {
                if(delId === value.id){
                    delete this.enteredValues[key].specialKeyword
                }
            })
            this.$forceUpdate()
        },
        searchSpecialKeyword: function($event, id){
                for(let i=0; i<this.fetchedSpecialKeywords.length;i++){
                    if(this.fetchedSpecialKeywords[i].name == $event){
                        Object.entries(this.enteredValues).forEach(([key, value]) => {
                            if(id == value.id){
                                this.enteredValues[key]["specialKeyword"] = {}
                                this.enteredValues[key]["specialKeyword"]["name"] = this.fetchedSpecialKeywords[i].name
                                this.enteredValues[key]["specialKeyword"]["parameters"] = this.fetchedSpecialKeywords[i].parameters
                                this.enteredValues[key]["specialKeyword"]["syntax"] = this.fetchedSpecialKeywords[i].syntax
                            }
                        })
                    }
                }
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
    background: #9cbf9b;
}
</style>