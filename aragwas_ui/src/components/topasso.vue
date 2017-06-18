<template>
    <v-layout row wrap justify-space-around>
        <v-flex xs3 wrap v-if="showControls.length>0 && view.controlPosition !== 'right'" class="associations-control-container">
            <div v-if="showControls.indexOf('maf')>-1">
                <h6 class="mt-4">MAF</h6>
                <v-checkbox v-model="filters.maf" primary label="<1% ( % of SNPs)" value="1" class="mb-0"></v-checkbox>
                <v-checkbox v-model="filters.maf" primary label="1-5% ( % of SNPs)" value="1-5" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.maf" primary label="5-10% ( % of SNPs)" value="5-10" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.maf" primary label=">10% ( % of SNPs)" value="10" class="mt-0"></v-checkbox>
            </div>
            <div v-if="showControls.indexOf('chr')>-1">
                <h6 class="mt-4">Chromosomes</h6>
                <v-checkbox v-model="filters.chr" primary label="1 ( % of SNPs)" value="1" class="mb-0"></v-checkbox>
                <v-checkbox v-model="filters.chr" primary label="2 ( % of SNPs)" value="2" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.chr" primary label="3 ( % of SNPs)" value="3" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.chr" primary label="4 ( % of SNPs)" value="4" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.chr" primary label="5 ( % of SNPs)" value="5" class="mt-0"></v-checkbox>
            </div>
            <div v-if="showControls.indexOf('annotation')>-1">
                <h6 class="mt-4">Annotation</h6>
                <v-checkbox v-model="filters.annotation" primary label="Non-synonymous coding ( % of SNPs)" value="ns" class="mb-0"></v-checkbox>
                <v-checkbox v-model="filters.annotation" primary label="Synonymous coding ( % of SNPs)" value="s" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.annotation" primary label="Intron ( % of SNPs)" value="in" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.annotation" primary label="Intergenic ( % of SNPs)" value="i" class="mt-0 mb-0"></v-checkbox>
            </div>
            <div v-if="showControls.indexOf('type')>-1">
                <h6 class="mt-4">Type</h6>
                <v-checkbox v-model="filters.type" primary label="Genic ( % of SNPs)" value="genic" class="mb-0"></v-checkbox>
                <v-checkbox v-model="filters.type" primary label="Non-genic ( % of SNPs)" value="non-genic" class="mt-0 mb-0"></v-checkbox>
            </div>
        </v-flex>
        <v-flex xs9 wrap fill-height class="association-table-container">
            <v-data-table
                    v-bind:headers="headers"
                    v-bind:items="associations"
                    v-bind:pagination.sync="pagination"
                    hide-actions
                    :loading="loading"
                    class="elevation-1 mt-2 asso-table"
            >
                <template slot="headers" scope="props">
                    <span>
                      {{ props.item.text | capitalize }}
                    </span>
                </template>
                <template slot="items" scope="props">
                    <td v-if="hideFields.indexOf('name') == -1"><div v-if="'snp' in props.item">{{ props.item.snp.chr | capitalize }}:{{ props.item.snp.position }}</div><div v-else>Missing SNP info</div></td>
                    <td v-if="hideFields.indexOf('score') == -1" class="text-xs-right">{{ props.item.score | round }}</td>
                    <td v-if="hideFields.indexOf('phenotype') == -1" class="text-xs-right"><router-link :to="{name: 'phenotypeDetail', params: { id: props.item.study.phenotype.id }}">{{ props.item.study.phenotype.name }}</router-link></td>
                    <td v-if="hideFields.indexOf('gene') == -1" class="text-xs-right"><router-link v-if="'snp' in props.item" :to="{name: 'geneDetail', params: { geneId: props.item.snp.geneName }}">{{ props.item.snp.geneName }}</router-link><div v-else class="text-xs-right">Missing SNP info</div></td>
                    <td v-if="hideFields.indexOf('maf') == -1" class="text-xs-right">{{ props.item.maf | round }}</td>
                    <td v-if="hideFields.indexOf('study') == -1" class="text-xs-right"><router-link :to="{name: 'studyDetail', params: { id: props.item.study.id }}">{{ props.item.study.name }}</router-link></td>
                </template>
            </v-data-table>
            <div class="page-container mt-5 mb-3">
                <v-pagination :length.number="pageCount" v-model="currentPage" />
            </div>
        </v-flex>
        <v-flex xs1 wrap v-show="!showSwitch" class="text-xs-right">
            <v-switch v-model="showSwitch" primary label="Controls" class="mb-0 switch"></v-switch>
        </v-flex>
        <v-flex xs3 wrap v-if="showControls.length>0 && view.controlPosition === 'right' && showSwitch" class="associations-control-container">
            <v-switch v-model="showSwitch" primary label="Controls" class="text-xs-right mb-0 " ></v-switch>
            <div v-if="showControls.indexOf('maf')>-1">
                <h6 class="mt-4">MAF</h6>
                <v-checkbox v-model="filters.maf" primary label="<1% ( % of SNPs)" value="1" class="mb-0"></v-checkbox>
                <v-checkbox v-model="filters.maf" primary label="1-5% ( % of SNPs)" value="1-5" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.maf" primary label="5-10% ( % of SNPs)" value="5-10" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.maf" primary label=">10% ( % of SNPs)" value="10" class="mt-0"></v-checkbox>
            </div>
            <div v-if="showControls.indexOf('chr')>-1">
                <h6 class="mt-4">Chromosomes</h6>
                <v-checkbox v-model="filters.chr" primary label="1 ( % of SNPs)" value="1" class="mb-0"></v-checkbox>
                <v-checkbox v-model="filters.chr" primary label="2 ( % of SNPs)" value="2" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.chr" primary label="3 ( % of SNPs)" value="3" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.chr" primary label="4 ( % of SNPs)" value="4" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.chr" primary label="5 ( % of SNPs)" value="5" class="mt-0"></v-checkbox>
            </div>
            <div v-if="showControls.indexOf('annotation')>-1">
                <h6 class="mt-4">Annotation</h6>
                <v-checkbox v-model="filters.annotation" primary label="Non-synonymous coding ( % of SNPs)" value="ns" class="mb-0"></v-checkbox>
                <v-checkbox v-model="filters.annotation" primary label="Synonymous coding ( % of SNPs)" value="s" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.annotation" primary label="Intron ( % of SNPs)" value="in" class="mt-0 mb-0"></v-checkbox>
                <v-checkbox v-model="filters.annotation" primary label="Intergenic ( % of SNPs)" value="i" class="mt-0 mb-0"></v-checkbox>
            </div>
            <div v-if="showControls.indexOf('type')>-1">
                <h6 class="mt-4">Type</h6>
                <v-checkbox v-model="filters.type" primary label="Genic ( % of SNPs)" value="genic" class="mb-0"></v-checkbox>
                <v-checkbox v-model="filters.type" primary label="Non-genic ( % of SNPs)" value="non-genic" class="mt-0 mb-0"></v-checkbox>
            </div>
        </v-flex>
    </v-layout>
</template>


<script lang="ts">
    import Vue from "vue";
    import {Component, Watch, Prop} from "vue-property-decorator";

    import {loadTopAssociations, loadAssociationsOfPhenotype, loadAssociationsOfStudy, loadAssociationsOfGene} from "../api";

    @Component({
        filters: {
            capitalize(str) {
                return str.charAt(0).toUpperCase() + str.slice(1);
            },
            round(number) {
                return Math.round( number * 1000) / 1000;
            }
        },
        name: "topAssociations",
        props: ["showControls", "hideFields", "filters", "view"],
    })
    export default class TopAssociationsComponent extends Vue {
        @Prop()
        showControls: string[];
        @Prop()
        hideFields: string[];
        @Prop()
        view: {name: "top-associations", phenotypeId: 0, studyId: 0, geneId: "1", zoom: 0, controlPosition: "left"};
        @Prop()
        filters: {chr: string[], annotation: string[], maf: string[], type: string[]};
        localfilters : {};
        loading: boolean = false;
        headers = [{text: "SNP", value: "snp.chr", name: "name"},{text: "score", value: "score", name: "score"},
            {text: "phenotype",value: "study.phenotype.name", name: "phenotype", sortable: false},{text: "gene",value: "snp.geneName", name: "gene", sortable: false},
            {text: "maf",value: "maf", name: "maf", sortable: false},{text: "study", value: "study.name", name: "study", sortable: false}];
        associations = [];
        currentPage = 1;
        pageCount = 5;
        totalCount = 0;
        breadcrumbs = [{text: "Home", href: "/"}, {text: "Top Associations", href: "#/top-associations", disabled: true}];
        maf = ["1", "1-5", "5-10", "10"];
        chr = ["1", "2", "3", "4", "5"];
        annotation = ["ns", "s", "in", "i"];
        type = ["genic", "non-genic"];
        pagination = {rowsPerPage: 25, totalItems: 0, page: 1, ordering: name, sortBy: "score", descending: true};
        showSwitch = false;

        @Watch("currentPage")
        onCurrentPageChanged(val: number, oldVal: number) {
            this.loadData(this.currentPage);
        }
        @Watch("filters.maf")
        onMafChanged(val: number, oldVal: number) {
            this.loadData(this.currentPage);
        }
        @Watch("filters.chr")
        onChrChanged(val: number, oldVal: number) {
            this.loadData(this.currentPage);
        }
        @Watch("filters.annotation")
        onAnnotationChanged(val: number, oldVal: number) {
            this.loadData(this.currentPage);
        }
        @Watch("filters.type")
        onTypeChanged(val: number, oldVal: number) {
            this.loadData(this.currentPage);
        }
        mounted(): void {
            this.hideHeaders(this.hideFields);
            this.loadData(this.currentPage);
        }
        loadData(pageToLoad): void {
            this.loading = true;
            if (this.view.name == "top-associations") {
                loadTopAssociations(this.filters, pageToLoad).then(this._displayData);
            } else if (this.view.name == "phenotype") {
                loadAssociationsOfPhenotype(this.view.phenotypeId, this.filters, pageToLoad).then(this._displayData)
            } else if (this.view.name == "study") {
                loadAssociationsOfStudy(this.view.studyId, this.filters, pageToLoad).then(this._displayData)
            } else if (this.view.name == "gene") {
                loadAssociationsOfGene(this.view.geneId, this.view.zoom, this.filters, pageToLoad).then(this._displayData)
            }
        }
        _displayData(data): void {
            this.associations = data.results;
            this.pagination.totalItems = data.count;
            this.pageCount = Math.ceil(data.count/this.pagination.rowsPerPage);
            this.loading = false;
        }
        hideHeaders(fields): void {
            for(let i = this.headers.length-1; i>= 0; i--) {
                if (fields.indexOf(this.headers[i].name)>-1){
                    this.headers.splice(i,1)
                }
            }
        }


    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .page-container {
        display:flex;
        justify-content:center;
    }
    .asso-table th {
        text-align: left !important;
    }
    .switch {
        text-align: right;
    }
</style>