<template>
  <div class="objectview">
    <b-container fluid>
        <b-form-row>
            <b-col align-self="start">
                <b-button type="submit" variant="secondary" @click="reload(prevButton)">{{prevButton}}</b-button>
            </b-col>
            <b-col cols=10>
                <template v-if="this.dim !== 'Risk'">
                    <select class="form-control" v-model="selectedOption" @change="onSelect()">
                        <option v-bind:value="undefined" selected disabled>{{this.dim}} List</option>
                        <template v-if="this.dim !== 'Threat'">
                            <option v-for="concept in concepts" :key="concept.theName" v-bind:value="concept">{{ concept.theName }}</option>
                        </template>
                        <template v-if="this.dim == 'Threat'">
                            <option v-for="concept in concepts" :key="concept.theThreatName" v-bind:value="concept">{{ concept.theThreatName }}</option>
                        </template>
                    </select>
                </template>
            </b-col>
            <b-col align-self="end">
                <b-button class="float-right" variant="primary" type="submit" @click="reload(nextButton)">{{nextButton}}</b-button>
            </b-col>
        </b-form-row>
    </b-container> 

    <component :is="componentLoader" :object="this.objt" :label="this.commitLabel" v-on:object-commit="commitObjt" />
  </div>
</template>

<script>

import axiosMixin from '../mixins/axiosMixin';
import axios from 'axios';
import store from '../store';
import EventBus from '../utils/event-bus';
import defaultObjectFactory from '../utils/defaultObjectFactory';
import objectViewParametersFactory from '../utils/objectViewParametersFactory';

export default {
    props: {
        dim: {
            type: String,
            default: 'Asset',
        },
    },
    mixins : [
        axiosMixin
    ],
    computed : {
        nextButton() {
            return this.dim == 'Role' ? 'Asset' : (
                    this.dim == 'Asset' ? 'Vulnerability' : (
                        this.dim == 'Vulnerability' ? 'Threat' : (
                            this.dim == 'Threat' ? 'Risk' : 'Risk')))
        },
        prevButton() {
            return this.dim == 'Risk' ? 'Threat' : (
                this.dim == 'Threat' ? 'Vulnerability' : (
                    this.dim == 'Vulnerability' ? 'Asset' : (
                        this.dim == 'Asset' ? 'Role' : 'Role')))
        },
        componentLoader() {
            this.refresh++;
            return () => import(`@/components/${this.dim}.vue`);
        }
    },
    data() {
        return {
            objt : undefined,
            selectedOption: undefined,
            commitLabel : 'Create',
            concepts : null,
            refresh : 0,
        }
    },
    beforeRouteEnter (to, from, next) {
        next(vm => {
            vm.objt = JSON.parse(JSON.stringify(defaultObjectFactory[vm.dim.toLowerCase()]));
        });
    },
    methods: {
        reload(newDim) {

            this.dim = newDim;
            this.setData();
        },
        onSelect() {
            this.commitLabel = 'Update';
            this.objt = this.selectedOption;
            this.refresh++;
        },
        getList() {
            if (this.dim !== "Risk"){
                axios.get(this.objtParams.createPath, {
                    baseURL : store.state.url,
                    params : {'session_id' : store.state.session}
                })
                .then(response => {
                    this.concepts = response.data
                })
                .catch((error) => {
                    EventBus.$emit('operation-failure', error)
                })
            }
        },
        commitObjt(objt) {
            this.objt = objt;

            if (this.dim == "Threat") {
                var objtName = encodeURIComponent(this.objt.theThreatName)
            } else {
                var objtName = encodeURIComponent(this.objt.theName)
            }

            var updateUrl = this.$store.state.url + this.objtParams.updatePath + objtName + "?session_id=" + this.$store.state.session;
            var createUrl = this.$store.state.url + this.objtParams.createPath;
            this.fastEditCommit(updateUrl, createUrl);

            this.setData();
        },
        setData() {
            this.selectedOption = undefined;
            this.commitLabel = 'Create';
            this.objt = JSON.parse(JSON.stringify(defaultObjectFactory[this.dim.toLowerCase()]));
            this.objtParams = objectViewParametersFactory[this.dim.toLowerCase()];
            this.getList();
            this.refresh = 0;
        }
    },
    mounted() {
        this.setData();
    }
}
</script>
