<template>
    <page-base v-on:onPrev="onPrev()" v-on:onNext="onNext()" v-on:onComplete="onComplete()">
        <survey v-bind:survey="survey"></survey>
    </page-base>
</template>

<script lang="ts">
import { Component, Vue, Prop, Watch} from 'vue-property-decorator';

import * as SurveyVue from "survey-vue";
import * as surveyEnv from "@/components/survey/survey-glossary.ts"
import surveyJson from "./forms/removePerson.json";

import PageBase from "../../PageBase.vue";
import { stepInfoType, stepResultInfoType } from "@/types/Application";

import { namespace } from "vuex-class";   
import "@/store/modules/application";
const applicationState = namespace("Application");

@Component({
    components:{
        PageBase
    }
})

export default class RemovePerson extends Vue {
    
    @Prop({required: true})
    step!: stepInfoType;

    @applicationState.Action
    public UpdateGotoPrevStepPage!: () => void

    @applicationState.Action
    public UpdateGotoNextStepPage!: () => void

    @applicationState.Action
    public UpdateStepResultData!: (newStepResultData: stepResultInfoType) => void

    respondentName = ""
    survey = new SurveyVue.Model(surveyJson);
    currentStep=0;
    currentPage=0;
    
    @Watch('pageIndex')
    pageIndexChange(newVal) 
    {
        this.survey.currentPageNo = newVal;        
    }

    beforeCreate() {
        const Survey = SurveyVue;
        surveyEnv.setCss(Survey);
    }
    
    mounted(){
        this.initializeSurvey();
        this.reloadPageInformation();
    }

    public initializeSurvey(){
        this.survey = new SurveyVue.Model(surveyJson);
        this.survey.commentPrefix = "Comment";
        this.survey.showQuestionNumbers = "off";
        this.survey.showNavigationButtons = false;
        surveyEnv.setGlossaryMarkdown(this.survey);
    }


    public reloadPageInformation() {

        if (this.step.result && this.step.result['removeSurvey']){
            this.survey.data = this.step.result['removeSurvey'];
        }
        
        let progress = 50;
        if(Object.keys(this.survey.data).length)
            progress = this.survey.isCurrentPageHasErrors? 50 : 100;
        
        this.currentStep = this.$store.state.Application.currentStep;
        this.currentPage = this.$store.state.Application.steps[this.currentStep].currentPage;
        this.$store.commit("Application/setPageProgress", { currentStep: this.currentStep, currentPage:this.currentPage, progress:progress })
    
        let protectedPartyNameObject = this.$store.state.Application.protectedPartyName;
            
        //console.log(protectedPartyNameObject)

        if (protectedPartyNameObject) {
            let protectedPartyName =
                protectedPartyNameObject.first +
                " " +
                protectedPartyNameObject.middle +
                " " +
                protectedPartyNameObject.last;
                this.survey.setVariable("protectedPartyName", protectedPartyName);
        }

        let respondentNameObject = this.$store.state.Application.respondentName;
        if (respondentNameObject) {
            this.respondentName =
                respondentNameObject.first +
                " " +
                respondentNameObject.middle +
                " " +
                respondentNameObject.last;
        }        

        if (this.respondentName) {
            this.survey.setVariable("RespondentName", this.respondentName);
        }
    }
    

    public onPrev() {
        this.UpdateGotoPrevStepPage()
    }

    public onNext() {
        if(!this.survey.isCurrentPageHasErrors) {
            this.UpdateGotoNextStepPage()
        }
    }

    public onComplete() {
        this.$store.commit("Application/setAllCompleted", true);
    }  
  
    beforeDestroy() {
        
        const progress = this.survey.isCurrentPageHasErrors? 50 : 100;
        this.$store.commit("Application/setPageProgress", { currentStep: this.currentStep, currentPage:this.currentPage, progress:progress })
        const currPage = document.getElementById("step-" + this.currentStep+"-page-" + this.currentPage);
        if(currPage){
            if(this.survey.isCurrentPageHasErrors)
                currPage.style.color = "red";
            else
            {
                currPage.style.color = "";
                currPage.className="";
            }  
        } 

        this.UpdateStepResultData({step:this.step, data: {removeSurvey: this.survey.data}})
        
    }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
@import "src/styles/common";
</style>
