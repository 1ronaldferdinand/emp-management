<template>
    <div>
        <b-modal id="modal-search-alert" title="Search Employee" size="sm" hide-footer no-close-on-backdrop no-close-on-esc>
            <p v-if="status == 1" class="my-2">Employee Found!</p>
            <p v-else-if="status == 2" class="my-2">Unable to process employeee hierarchy! <b>{{ employeeNameFormatted }}</b> not having hierarchy.</p>
            <p v-else-if="status == 3" class="my-2">Unable to process employee tree. <b>{{ employeeNameFormatted }}</b> has multiple managers: <b>{{ managers }}</b>.</p>
            <p v-else-if="status == 4" class="my-2">Please write the employee's name in the search column.</p>
            <p v-else class="my-2">Employee Not Found!</p>
        </b-modal>
    </div>
</template>

<script>
export default {
    name: "SearchAlert",
    props: {
        employeeName: String,
        managers: String,
        status: Number,
    },
    computed: {
        employeeNameFormatted() {
            // Format first character to uppercase
            return this.employeeName.charAt(0).toUpperCase() + this.employeeName.slice(1).toLowerCase();
        }
    }
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