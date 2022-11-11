<template>
    <div class="listing-layout">
        <virtual-list
            ref="listing"
            class="listing"
            role="list"
            data-key="id"
            :offset="offset"
            :data-sources="items"
            :data-component="{}"
            @scroll="onScroll">
            <template v-slot:item="{ item }">
                <slot name="item" :item="item" :current-offset="getOffset()" />
            </template>
            <template v-slot:footer>
                <LoadingSpan v-if="loading" class="m-2" message="Loading" />
            </template>
        </virtual-list>
    </div>
</template>
<script>
import VirtualList from "vue-virtual-scroll-list";
import { throttle } from "lodash";
import LoadingSpan from "components/LoadingSpan";
import { mapCacheActions } from "vuex-cache";
import { mapGetters } from "vuex";
// import Vue from "vue"
// import JsonCSV from 'vue-json-csv'

// Vue.component('downloadCsv', JsonCSV)



export default {
    components: {
        LoadingSpan,
        VirtualList,
    },
    props: {
        offset: { type: Number, default: 0 },
        loading: { type: Boolean, default: false },
        items: { type: Array, default: null },
        queryKey: { type: String, default: null },
    },
    data() {
        return {
            throttlePeriod: 20,
            deltaMax: 20,
            accountingArray: []
        };
    },
    watch: {
        queryKey() {
            this.$refs.listing.scrollToOffset(0);
        },
    },
    created() {
        this.onScrollThrottle = throttle((event) => {
            this.onScroll(event);
        }, this.throttlePeriod);
        // console.log('Data from Listing Layout ', this.items)
        // Making a JSON file for Accounting the Jobs, This loop will fetch JobId for all Jobs in the current History and then fetch the JobMetrices for each of them 
        this.items?.length > 0 ? this.items.forEach( async (job) =>  {
            var Accounting = {}
            console.log('Job is ', job)
            if(job.id){
                Accounting.ganttBarConfig = {'label' : job.name, "id": job.hid, "jobid": job.id}
                await this.fetchJobMetricsForJobId(job.id);
                // const metrics = this.getJobMetricsByJobId(job.id);    
                // console.log('Metrics is ', metrics)
                // Accounting.myBeginDate = metrics[1].value
                // Accounting.myEndDate = metrics[2].value
                const metrics = this.$store.state?.jobMetrics?.jobMetricsByJobId[`${job.id}`]
                Accounting.myBeginDate = metrics[1].value
                Accounting.myEndDate = metrics[2].value
                this.accountingArray.push(Accounting)
                // this.$store.commit('saveAccountingData', this.accountingArray)
                // console.log('This.store ' , this.$store )
                // const final_data = JSON.stringify(this.accountingArray)
                // fileSystem.writeFile("./AccointingTestData.json", final_data, err=>{
                // if(err){
                // console.log("Error writing file" ,err)
                // } else {
                // console.log('JSON data is written to the file successfully')
                // }
                // })
            }

        } ) : console.log("For some reason the items list is empty")
        console.log("For each complete and now the final array is ", this.accountingArray)
        var stringData = JSON.stringify(this.accountingArray)
        console.log("String Data ", stringData)
        this.storeIntoAccountingArray(this.accountingArray)
    },
    methods: {
        ...mapCacheActions(["fetchJobMetricsForDatasetId", "fetchJobMetricsForJobId","storeIntoAccountingArray"]),
        ...mapGetters(["getJobMetricsByDatasetId", "getJobMetricsByJobId"]),
        onScrollHandler(event) {
            /* CURRENTLY UNUSED
            // this avoids diagonal scrolling, we either scroll left/right or top/down
            // both events are throttled and the default handler has been prevented.
            if (Math.abs(event.deltaY) > Math.abs(event.deltaX)) {
                // handle vertical scrolling with virtual scroller
                const listing = this.$refs.listing;
                const deltaMax = this.deltaMax;
                const deltaY = Math.max(Math.min(event.deltaY, deltaMax), -deltaMax);
                this.offset = Math.max(0, listing.getOffset() + deltaY);
                this.$refs.listing.scrollToOffset(this.offset);
            } else {
                // dispatch horizontal scrolling as regular event
                var wheelEvent = new WheelEvent("wheel", {
                    deltaX: event.deltaX,
                    bubbles: true,
                    cancelable: false,
                });
                event.target.dispatchEvent(wheelEvent);
            }
            */
        },
        jobMetrics: function () {
            if (this.jobId) {
                return this.getJobMetricsByJobId(this.jobId);
            } else {
                return this.getJobMetricsByDatasetId(this.datasetId, this.datasetType);
            }
        },
        onScroll() {
            const rangeStart = this.$refs.listing.range.start;
            this.$emit("scroll", rangeStart);
        },
        getOffset() {
            return this.$refs.listing?.getOffset() || 0;
        },
    },
};
</script>

<style scoped lang="scss">
@import "scss/mixins.scss";
.listing-layout {
    .listing {
        @include absfill();
        scroll-behavior: smooth;
        overflow-y: scroll;
        overflow-x: hidden;
    }
}
</style>
