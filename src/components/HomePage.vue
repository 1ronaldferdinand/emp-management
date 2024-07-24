<template>
    <div id="home" class="h-100">
        <NavbarPage />
        
        <div v-if="loading" class="d-flex align-items-center justify-content-center" style="height: calc(100% - 56px);">
            <b-spinner></b-spinner>
        </div>
        <div v-else-if="employees.length == 0" class="d-flex flex-column align-items-center justify-content-center gap-3" style="height: calc(100% - 56px); overflow-y: auto;">
            <span><b>Upload JSON File</b></span>
            <input type="file" @change="handleFileUpload" />
        </div>
        <div v-else class="d-flex flex-column gap-3 align-items-center justify-content-center" style="height: calc(100% - 56px); overflow-y: auto;">
            <div class="d-flex align-items-center">
                <div class="input-search">
                    <b-icon icon="search"></b-icon>
                    <input ref="searchInput" v-model="searchKeyword" placeholder="Search employee...">
                </div>

                <b-button variant="outline-info" class="search-button" @click="searchEmployee()">Search</b-button>
            </div>

            <div class="d-flex gap-3 align-items-center justify-content-center">
                <b-button variant="warning" @click="reupload()" style="font-size: 12px; color: white;">Reupload</b-button>
                <b-button variant="info" @click="$bvModal.show('modal-upload-employees')" style="font-size: 12px; color: white;">Show Employee List</b-button>
            </div>
        </div>

        <SearchAlert :employeeName="searchKeyword" :managers="searchManagers" :status="searchStatus" />
        <EmployeeDetail :foundEmployee="foundEmployee" :foundManagers="foundManagers" />
        <EmployeeUploadDetail :invalidEmployees="invalidEmployees" :validEmployees="employees"/>

        <b-modal id="modal-upload-alert" size="sm" hide-header hide-footer no-close-on-backdrop no-close-on-esc>
            <p class="my-2">JSON Upload Success!</p>
        </b-modal>
    </div>
</template>
  
<script>
    import EmployeeUploadDetail from './modal/EmployeeUploadDetail.vue';
    import EmployeeDetail from './modal/EmployeeDetail.vue';
    import SearchAlert from './modal/SearchAlert.vue';
    import NavbarPage from './NavbarPage.vue';

    export default {
        name: 'HomePage',
        components: {
            NavbarPage,
            SearchAlert,
            EmployeeDetail,
            EmployeeUploadDetail
        },
        mounted() {
            setTimeout(() => {
                this.loading = false;
            }, 800);
        },
        data() {
            return {
                loading: true,

                employees: [],
                invalidEmployees: [],
                
                foundEmployee: {},
                foundManagers: [],
                
                searchKeyword: '',
                searchManagers: '',
                searchStatus: null,
                
                found: false,
            }
        },
        methods: {
            reupload() {
                this.loading = true;
                this.employees = [];
                this.invalidEmployees = [];
                this.foundEmployee = {};
                this.foundManagers = [];
                this.searchKeyword = '';
                this.searchManagers = '';
                this.searchStatus = null;
                this.found = false;

                setTimeout(() => {
                    this.loading = false;
                }, 800);
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
            recursiveSearch(employeeList, keyword, withId = false) {
                for (const employee of employeeList) {
                    if(withId){
                        // Check if the employee's name matches the keyword
                        if (employee.id == keyword) {
                            return employee;
                        }
                    } else { 
                        // Check if the employee's name matches the keyword
                        if (employee.name.toLowerCase().includes(keyword.toLowerCase())) {
                            return employee;
                        }
                    }

                    // Recursively search in directReports if they exist
                    if (employee.directReports && employee.directReports.length > 0) {
                        let found = null;

                        if(withId){
                            found = this.recursiveSearch(employee.directReports, keyword, true);
                        } else {
                            found = this.recursiveSearch(employee.directReports, keyword);
                        }

                        if (found) return found;
                    }
                }

                return null;
            },
            getIndirectManagers(id) {
                let managers = [];
                let currentId = id; // define first id to get direct manager

                // Loop until get manager with managerId == null
                while (currentId !== null) {
                    let manager = this.recursiveSearch(this.employees, currentId, true);
                    
                    if (manager === null) {
                        // If manager is not found, break the loop
                        break;
                    }

                    managers.push(manager); // The obtained manager data is stored in a temporary variable
                    currentId = manager.managerId; // Update the currentId data which previously stored the managerId to be searched
                }

                // Reverse the data hierarchy so that the root manager is at the front
                return managers.reverse();
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
                    this.foundEmployee = this.recursiveSearch(this.employees, this.searchKeyword);
                    if (this.foundEmployee) {
                        this.foundManagers = this.getIndirectManagers(this.foundEmployee.managerId);
                        this.searchStatus = 1;
                        this.found = true;
                    }
                } 

                this.showModal();
            },
            handleFileUpload(event) {
                const file = event.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        try {
                            let inputData = JSON.parse(e.target.result);
                            const result = this.transformData(inputData);
                           
                            this.employees = result.transformedData;
                            this.invalidEmployees = result.invalidData;
                            console.log(this.employees);
                            
                            this.$bvModal.show('modal-upload-alert');

                            setTimeout(() => {
                                this.$bvModal.hide('modal-upload-alert');
                            }, 800);
                            
                            this.$bvModal.show('modal-upload-employees');
                        } catch (err) {
                            alert('Invalid JSON file');
                        }
                    };
                    reader.readAsText(file);
                }
            },
            transformData(data) {
                const dataMap = {};
                const topLevels = [];
                const invalidData = [];

                // Initialize the dataMap and counts
                data.forEach(item => {
                    dataMap[item.id] = { ...item, directReports: [], countDirectReports: 0, countIndirectReports: 0 };
                });

                // Build the directReports and topLevels
                data.forEach(item => {
                    if (item.managerId === null) {
                        topLevels.push(dataMap[item.id]);
                    } else {
                        if (dataMap[item.managerId]) {
                            dataMap[item.managerId].directReports.push(dataMap[item.id]);
                        } else {
                            invalidData.push({ ...item, reason: 'Manager not found' });
                        }
                    }

                    // Cek apakah ID karyawan muncul lebih dari sekali
                    if (Object.values(dataMap).some(entry => entry.name === item.name && entry.managerId !== item.managerId)) {
                        invalidData.push({ ...item, reason: 'Unable to process employee tree. ' + item.name + ' has multiple managers' });
                    }
                });

                // Check for invalid employees
                Object.values(dataMap).forEach(item => {
                    if (item.managerId === null && item.directReports.length === 0) {
                        invalidData.push({ ...item, reason: 'Unable to process employeee hierarchy. ' + item.name + ' not having hierarchy' });
                    }
                });

                // Function to recursively remove invalid employees from directReports
                const removeInvalidEmployees = (employee) => {
                    employee.directReports = employee.directReports
                        .filter(child => !invalidData.some(invalidItem => invalidItem.id === child.id))
                        .map(child => {
                            removeInvalidEmployees(child);
                            return child;
                        });
                };

                // Remove invalid employees from dataMap
                invalidData.forEach(invalidItem => {
                    delete dataMap[invalidItem.id];
                });

                // Apply recursive filtering to topLevels
                topLevels.forEach(employee => removeInvalidEmployees(employee));

                // Calculate counts for each valid employee
                topLevels.forEach(employee => this.calculateCounts(employee));

                // Rebuild topLevels without invalid employees
                const filteredTopLevels = topLevels.filter(employee => dataMap[employee.id]);

                return { transformedData: filteredTopLevels, invalidData };
            },
            calculateCounts(employee){
                employee.countDirectReports = employee.directReports.length;
                employee.countIndirectReports = employee.directReports.reduce((total, child) => {
                    this.calculateCounts(child);
                    return total + child.countDirectReports + child.countIndirectReports;
                }, 0);

                return employee;
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