<template>
<!--  
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.

Authors: Shamal Faily 
-->

  <b-modal ref="templateAssetDialog" ok-only :title="dialogTitle">
    <b-container v-if="objt != undefined">
      <b-form-group label="Type" label-class="font-weight-bold text-sm-left" label-cols="3" label-for="theType" >
        <b-form-input readonly id="theType" v-model="objt.theType"></b-form-input>
      </b-form-group>
      <b-form-group label="Description" label-class="font-weight-bold text-sm-left" label-for="theDescription" >
        <b-form-textarea id="theDescription" v-model="objt.theDescription" type="text" :rows=2 :max-rows="4" readonly>
        </b-form-textarea>
      </b-form-group>
      <b-form-group label="Significance" label-class="font-weight-bold text-sm-left" label-for="theSignificance" >
        <b-form-textarea id="theSignificance" v-model="objt.theSignificance" type="text" :rows=2 :max-rows="4" readonly>
        </b-form-textarea>
      </b-form-group>
      <b-table bordered small :items="notNone" :fields="propTableFields">
      </b-table>
    </b-container>
  </b-modal>
</template>

<script>

  export default {
    name: 'template-asset-modal',
    props : {
      asset : Object
    },
    data() {
      return {
        objt : this.asset,
        propTableFields : {
          name : {label : 'Property'},
          value : {label : 'Value'},
          rationale : {label : 'Rationale'} 
        }
      }
    },
    watch : {
      asset: 'updateData'
    },
    computed : {
      dialogTitle() {
        return (this.objt != undefined ? this.objt.theName : '') + ' Template Asset';
      },
      notNone() {
        return this.objt != undefined ? this.objt.theProperties.filter(prop => prop.value != 'None') : [];
      }
    },
    methods : {
      show() {
        this.$refs.templateAssetDialog.show();
      },
      updateData() {
        this.objt = this.asset;
      }
    }
  };
</script>