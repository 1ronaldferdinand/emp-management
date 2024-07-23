<template>
    <div id="home" class="h-100">
        <NavbarPage />
        
        <div v-if="loading" class="d-flex align-items-center justify-content-center" style="height: calc(100% - 56px);">
            <b-spinner></b-spinner>
        </div>
        <div v-else class="d-flex align-items-center justify-content-center" style="height: calc(100% - 56px); overflow-y: auto;">
            <div class="d-flex align-items-center">
                <div class="input-search">
                    <b-icon icon="search"></b-icon>
                    <input ref="searchInput" v-model="searchKeyword" placeholder="Search employee...">
                </div>

                <b-button variant="outline-info" class="search-button" @click="searchEmployee()">Search</b-button>
            </div>
        </div>

        <SearchAlert :employeeName="searchKeyword" :managers="searchManagers" :status="searchStatus" />
        <EmployeeDetail :foundEmployee="foundEmployee" :foundManagers="foundManagers" />
    </div>
</template>
  
<script>
    import EmployeeDetail from './modal/EmployeeDetail.vue';
    import SearchAlert from './modal/SearchAlert.vue';
    import NavbarPage from './NavbarPage.vue';
    import axios from "axios";

    export default {
        name: 'HomePage',
        components: {
            NavbarPage,
            SearchAlert,
            EmployeeDetail
        },
        mounted() {
            this.getEmployees();
        },
        data() {
            return {
                loading: true,

                employees: [],
                
                foundEmployee: {},
                foundManagers: [],
                
                searchKeyword: '',
                searchManagers: '',
                searchStatus: null,
                
                found: false,
            }
        },
        methods: {
            async getEmployees() {
                try {
                    // asynchronously to get data from json-server using axios
                    const res = await axios.get(process.env.VUE_APP_JSON_SERVER_URL + "employees");
                    this.employees = res.data; // store result data to employees
                    this.employees = this.employees.map(employee => { 
                        return {
                            ...employee,
                            // adding new params to store number of direct reports & indirect reports of employees
                            countDirectReports: this.countDirectReports(employee.id),
                            countIndirectReports: this.countIndirectReports(employee.id)
                        };
                    });

                    setTimeout(() => {
                        this.loading = false; // show input field after 800ms
                        this.$nextTick(() => {
                            if (this.$refs.searchInput) {
                                // focus on input when input is showed
                                this.$refs.searchInput.focus();
                            }
                        });
                    }, 800);
                } catch (error) {
                    console.error(error);
                }
            },
            countDirectReports(id){
                // count how many employee's direct reports
                return this.getDirectReports(id).length;
            },
            countIndirectReports(id){
                // count how many employee's indirect reports
                return this.getIndirectReports(id).length;
            },
            getDirectReports(id){
                // get data of employee's direct reports, output in array
                return this.employees.filter(employee => employee.managerId === Number(id));
            },
            getIndirectReports(id){
                // get direct reports
                let directReports = this.getDirectReports(id);
                let indirectReports = [];

                // loop over direct reports data then take data from direct reports' direct reports
                for (let report of directReports) {
                    indirectReports = indirectReports.concat(this.getDirectReports(report.id));
                    
                    // perform this function continuously until the direct reports have no more direct reports
                    indirectReports = indirectReports.concat(this.getIndirectReports(report.id));
                }   

                return indirectReports;
            },
            getDirectManagers(id){
                // get data of employee's direct manager, output in array
                return this.employees.filter(employee => employee.id === Number(id));
            },
            getIndirectManagers(id) {
                let managers = [];
                let currentId = id; // define first id to get direct manager

                // Loop until get manager with managerId == null
                while (currentId !== null) {
                    let manager = this.getDirectManagers(currentId);
                    
                    managers.push(manager[0]); // The obtained manager data is stored in a temporary variable
                    currentId = manager[0].managerId; // Update the currentId data which previously stored the managerId to be searched
                }

                // Reverse the data hierarchy so that the root manager is at the front
                return managers.reverse();
            },
            async showModal(){
                if(this.found){
                    // If employee found, has hierarchy, not has more than one manager. It will show detail's modal
                    this.$bvModal.show('modal-employee-detail');
                } else {
                    // If employee not found, not has hierarchy, has more than one manager. It will show error's modal
                    this.$bvModal.show('modal-search-alert');
                    
                    setTimeout(() => {
                        this.$bvModal.hide('modal-search-alert');
                    }, 2000);
                }
            },
            searchEmployee(){
                this.searchStatus = 0;
                this.searchManagers = '';
                this.found = false;

                if(!this.searchKeyword){
                    // Show modal with message if search's input is empty
                    this.searchStatus = 4;
                    this.showModal();
                    return;
                }

                if(this.employees.length != 0){
                    // Look for employees with names that match keywords
                    let findEmployee = this.employees.filter(employee => employee.name.toLowerCase() === this.searchKeyword.toLowerCase());
                    
                    if(findEmployee.length == 1){
                        // If only one employee found
                        this.searchStatus = 2;

                        let hasReports = this.getDirectReports(findEmployee[0].id); // get his/her direct reports
                        let hasManagers = this.getDirectManagers(findEmployee[0].managerId); // get his/her direct manager

                        if(hasReports.length > 0 || hasManagers.length > 0){
                            // set found status is true when employee has direct report or has direct manager
                            this.searchStatus = 1;
                            this.foundEmployee = findEmployee[0];
                            this.foundManagers = this.getIndirectManagers(findEmployee[0].managerId);
                            this.found = true;
                        }
                    } else if (findEmployee.length > 1){
                        // If found more than one employee with same name
                        let managers = [];

                        // get the managers data of that employee
                        findEmployee.forEach(element => {
                            let manager = this.getDirectManagers(element.managerId);
                            managers.push(manager[0].name.charAt(0).toUpperCase() + manager[0].name.slice(1));
                        });

                        // combine the manager data into one string as an additional error message
                        this.searchManagers = managers.join(', ');
                        this.searchStatus = 3;
                    }
                } 

                this.showModal();
                return;
            },
        }
    }
</script>

<style scoped>
.show-detail-button {
    cursor: pointer;
}

.show-detail-button:hover {
    color: #052c65 !important;
}

.search-button {
    padding: 5.5px 12px;
    font-size: 12px;
    color: #0dcaf0;
}

.search-button:hover {
    color: white;
    background-color: #0dcaf0;
}

.input-search{
    margin: 8px;
    width: fit-content;
    border-radius: 8px;
    padding: 3px 12px;
    border: 1px solid;
    font-size: 14px;
}

.input-search input{
    border: none !important;
    margin-left: 0.5rem;
}

.input-search input:focus{
    border: none !important;
    outline: none !important;
}
</style>