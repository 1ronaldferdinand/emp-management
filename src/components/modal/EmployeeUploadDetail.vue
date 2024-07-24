<template>
    <div>
        <b-modal id="modal-upload-employees" title="Employees" size="lg" centered no-close-on-backdrop no-close-on-esc>
            <template v-slot:modal-header>
                <h5 class="modal-title">Uploaded Employees</h5>
            </template>
            <b-tabs>
                <b-tab title="Valid Employees">
                    <employee-tree :employees="validEmployees" />
                </b-tab>
                <b-tab title="Invalid Employees">
                    <table class="table">
                        <thead>
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Reason</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-if="invalidEmployees.length == 0">
                                <td colspan="3" class="text-center">There is no invalid data</td>
                            </tr>
                            <tr v-for="employee in invalidEmployees" :key="employee.id">
                                <td>{{ employee.id }}</td>
                                <td>{{ employee.name }}</td>
                                <td>{{ employee.reason }}</td>
                            </tr>
                        </tbody>
                    </table>
                </b-tab>
            </b-tabs>
        </b-modal>
    </div>
</template>

<script>
import EmployeeTree from './EmployeeTree.vue';

export default {
    name: "EmployeeDetailModal",
    components: {
        EmployeeTree
    },
    props: {
        invalidEmployees: {
            type: Array,
            default: () => []
        },
        validEmployees: {
            type: Array,
            default: () => []
        }
    }
}
</script>