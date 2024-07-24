<template>
    <div>
        <ul>
            <li v-for="employee in employees" :key="employee.id">
                <div>
                    <strong>{{ employee.name }}</strong>
                    <employee-tree v-if="employee.directReports.length > 0" :employees="employee.directReports" />
                </div>
            </li>
        </ul>
    </div>
</template>
  
<script>
import EmployeeTree from './EmployeeTree.vue';
  
export default {
    name: 'EmployeeTree',
    components: {
        EmployeeTree
    },
    props: {
        employees: {
            type: Array,
            default: () => []
        }
    }
}
</script>
  