<template>
    <div>
        <b-modal id="modal-employee-detail" title="Detail Employee" size="xl" centered hide-footer no-close-on-backdrop no-close-on-esc>
            <div class="mb-3">
                <span><b>Detail</b></span>
                <div class="col-12 d-flex align-items-center">
                    <div class="col-6 col-sm-4 col-lg-2">Employee Name</div>
                    <div class="col-1">:</div>
                    <div class="col-5 col-sm-7 col-lg-9">{{ foundEmployee && foundEmployee.name ? foundEmployee.name.charAt(0).toUpperCase() + foundEmployee.name.slice(1).toLowerCase() : '-' }}</div>
                </div>
                <div class="col-12 d-flex align-items-center">
                    <div class="col-6 col-sm-4 col-lg-2">Direct Reports</div>
                    <div class="col-1">:</div>
                    <div class="col-5 col-sm-7 col-lg-9">{{ foundEmployee && foundEmployee.countDirectReports ? foundEmployee.countDirectReports : 0 }}</div>
                </div>
                <div class="col-12 d-flex align-items-center">
                    <div class="col-6 col-sm-4 col-lg-2">Indirect Reports</div>
                    <div class="col-1">:</div>
                    <div class="col-5 col-sm-7 col-lg-9">{{ foundEmployee && foundEmployee.countIndirectReports ? foundEmployee.countIndirectReports : 0 }}</div>
                </div>
            </div>

            <div>
                <span><b>Hierarchies</b></span>
                <div v-for="(hierarchy, index) in hierarchies" :key="'hierarchy-' + index">
                    <span v-for="index of (3*index)" :key="'spacer-' + index">-</span>
                    {{hierarchy && hierarchy.name ? hierarchy.name.charAt(0).toUpperCase() + hierarchy.name.slice(1).toLowerCase() : '-' }} 
                    <b>
                        <span v-if="index == 0 && hierarchies.length == 2">( Root Manager & Direct Manager )</span>
                        <span v-else-if="index == 0">( Root Manager )</span>
                        <span v-else-if="index == hierarchies.length - 2">( Direct Manager )</span>
                        <span v-else-if="index == hierarchies.length - 1">( Employee )</span>
                    </b>
                </div>
            </div>
        </b-modal>
    </div>
</template>

<script>
export default {
    name: 'EmployeeDetail',
    props: {
        foundEmployee: {
            type: Object,
            default: () => {},
        },
        foundManagers: {
            type: Array,
            default: () => [],
        }
    },
    computed: {
        hierarchies() {
            // Merge managers data with the employee for easier to display
            let hierarchies = this.foundManagers;
            hierarchies.push(this.foundEmployee); 
            return hierarchies;
        },
    },
}
</script>

<style>
.modal-header {
    justify-content: space-between;
}

.close {
    border: none;
    background: none;
}
</style>