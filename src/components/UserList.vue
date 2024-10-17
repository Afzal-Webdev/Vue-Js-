<template>
    <div class="user-list p-6">
        <div class="flex justify-between items-center mb-4">
            <!-- Search Box -->
            <div class="relative w-full max-w-md">
                <input
                    v-model="searchQuery"
                    @input="searchUsers"
                    type="text"
                    placeholder="Search by Username, Age, Role or Company"
                    class="border px-4 py-2 w-full pl-10 rounded-lg focus:outline-none"
                />
                <!-- Search Icon -->
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-5 w-5 text-gray-500 absolute left-3 top-1/2 transform -translate-y-1/2 pointer-events-none"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                >
                    <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M10 18a8 8 0 100-16 8 8 0 000 16zm6-2l4 4"
                    />
                </svg>
            </div>
            <!-- Filter Section -->
            <div>
                <multiselect
                    v-model="selectedDepartment"
                    :options="departments"
                    label="label"
                    track-by="value"
                    placeholder="Choose Department"
                />
            </div>
            <div>
                <multiselect
                    v-model="selectedCity"
                    :options="cities"
                    label="label"
                    track-by="value"
                    placeholder="Choose City"
                />
            </div>
            <!-- Buttons for Applying and Resetting Filters -->
            <div class="flex items-center gap-4">
                <button
                    @click="applyFilters"
                    class="px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600"
                >
                    Apply Filters
                </button>
                <button
                    @click="resetFilters"
                    class="px-4 py-2 bg-gray-500 text-white rounded hover:bg-gray-600"
                >
                    Reset Filters
                </button>
            </div>
        </div>

        <!-- User Table -->
        <table class="min-w-full bg-white">
            <thead>
                <tr class="text-left">
                    <th class="px-6 py-3">Username</th>
                    <th class="px-6 py-3">Birthdate</th>
                    <th class="px-6 py-3">Age</th>
                    <th class="px-6 py-3">Role</th>
                    <th class="px-6 py-3">Company</th>
                    <th class="px-6 py-3">Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="user in filteredUsers" :key="user.id">
                    <td class="px-6 py-3">{{ user.username }}</td>
                    <td class="px-6 py-3">{{ user.birthDate }}</td>
                    <td class="px-6 py-3">{{ user.age }}</td>
                    <td class="px-6 py-3">{{ user.role }}</td>
                    <td class="px-6 py-3">{{ user.company.name }}</td>
                    <td class="px-6 py-3">
                        <button @click="viewUser(user)">👁 View</button>
                    </td>
                </tr>
            </tbody>
        </table>

        <!-- Pagination -->
        <div class="mt-4 flex justify-between items-center">
            <button
                @click="prevPage"
                :disabled="page === 1"
                class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-700 disabled:opacity-50"
            >
                Previous
            </button>
            <span>Page {{ page }}</span>
            <button
                @click="nextPage"
                :disabled="!hasMoreUsers"
                class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-700 disabled:opacity-50"
            >
                Next
            </button>
        </div>

        <!-- Modal for Viewing Address -->
        <div
            v-if="selectedUser"
            class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center"
        >
            <div class="bg-white p-6 rounded">
                <h3 class="text-xl mb-4">Address Details</h3>
                <p>
                    {{ selectedUser.address.street }},
                    {{ selectedUser.address.city }}
                </p>
                <button
                    @click="closeModal"
                    class="mt-4 px-4 py-2 bg-red-500 text-white rounded"
                >
                    Close
                </button>
            </div>
        </div>
    </div>
</template>

<script>
import axios from "axios";
import Multiselect from "vue-multiselect";

export default {
    components: { Multiselect },
    data() {
        return {
            users: [],
            filteredUsers: [],
            searchQuery: "",
            page: 1,
            limit: 5,
            hasMoreUsers: true,
            selectedUser: null,
            selectedDepartment: null,
            selectedCity: null,
            isLoading: true,
            departments: [
                { value: "Engineering", label: "Engineering" },
                { value: "Sales", label: "Sales" },
                { value: "Product Management", label: "Product Management" },
                { value: "Accounting", label: "Accounting" },
            ],
            cities: [
                { value: "New York", label: "New York" },
                { value: "Los Angeles", label: "Los Angeles" },
                { value: "Chicago", label: "Chicago" },
                { value: "Houston", label: "Houston" },
            ],
        };
    },
    created() {
        this.loadFiltersFromLocalStorage();
        this.fetchUsers();
        // this.fetchFilteredUsers(); // Fetch filtered users on creation
    },
    mounted() {
        this.loadFiltersFromLocalStorage(); // Load filters when the page is loaded
    },
    methods: {
        async fetchUsers() {
            const skip = (this.page - 1) * this.limit;
            const response = await axios.get(
                `https://dummyjson.com/users?limit=${this.limit}&skip=${skip}`
            );
            this.users = response.data.users;
            this.filteredUsers = this.users; // Default to all users when page loads
            this.hasMoreUsers = response.data.total > skip + this.limit;
        },

        // Search Users
        searchUsers() {
            this.filteredUsers = this.users.filter((user) => {
                return (
                    user.username.includes(this.searchQuery) ||
                    user.age.toString().includes(this.searchQuery) ||
                    user.role.includes(this.searchQuery) ||
                    user.company.name.includes(this.searchQuery)
                );
            });
        },

        // Apply Filters
        applyFilters() {
            // Save the selected department and city in localStorage
            localStorage.setItem(
                "selectedDepartment",
                JSON.stringify(this.selectedDepartment)
            );
            localStorage.setItem(
                "selectedCity",
                JSON.stringify(this.selectedCity)
            );
            this.fetchFilteredUsers();

            this.filteredUsers = this.users.filter((user) => {
                const departmentMatches =
                    !this.selectedDepartment ||
                    user.company.department === this.selectedDepartment.value;
                const cityMatches =
                    !this.selectedCity ||
                    user.address.city === this.selectedCity.value;

                return departmentMatches && cityMatches;
            });
        },

        loadFiltersFromLocalStorage() {
            // Load the saved department and city from localStorage if available
            const savedDepartment = localStorage.getItem("selectedDepartment");
            const savedCity = localStorage.getItem("selectedCity");

            // Check if the saved department exists and is valid JSON
            if (savedDepartment) {
                try {
                    this.selectedDepartment = JSON.parse(savedDepartment);
                } catch (e) {
                    console.error(
                        "Error parsing savedDepartment from localStorage:",
                        e
                    );
                }
            }

            // Check if the saved city exists and is valid JSON
            if (savedCity) {
                try {
                    this.selectedCity = JSON.parse(savedCity);
                } catch (e) {
                    console.error(
                        "Error parsing savedCity from localStorage:",
                        e
                    );
                }
            }

            // Reapply the filters on page reload
            if (this.selectedDepartment || this.selectedCity) {
                this.fetchFilteredUsers();
            }
        },

        // Reset Filters
        resetFilters() {
            // Clear filters and remove them from localStorage
            this.selectedDepartment = null;
            this.selectedCity = null;
            localStorage.removeItem("selectedDepartment");
            localStorage.removeItem("selectedCity");

           
          
            this.fetchUsers();
            // this.selectedDepartment = null;
            // this.selectedCity = null;
            // this.filteredUsers = this.users;
        },
        async fetchFilteredUsers() {
            console.log(
                "Applying filters: ",
                this.selectedDepartment,
                this.selectedCity
            );

            // Reset filtered users to an empty array before fetching
            this.filteredUsers = [];

            // Prepare the filter parameters
            const department = this.selectedDepartment
                ? this.selectedDepartment.value
                : null;
            const city = this.selectedCity ? this.selectedCity.value : null;

            try {
                // Fetch all users from the API
                const response = await axios.get(
                `https://dummyjson.com/users?limit=${this.limit}&skip=${skip}`
            );
                const allUsers = response.data.users;

                // Filter users based on the selected department and city
                this.filteredUsers = allUsers.filter((user) => {
                    const departmentMatches =
                        !department || user.company.department === department;
                    const cityMatches = !city || user.address.city === city;

                    return departmentMatches && cityMatches;
                });

                // Update hasMoreUsers based on filtered results
                this.hasMoreUsers =
                    this.filteredUsers.length > this.page * this.limit;

                console.log("Filtered users:", this.filteredUsers);
            } catch (error) {
                console.error("Error fetching users:", error);
            } finally {
                // Set loading state to false once done
                this.isLoading = false; // Set loading to false
            }
        },

        // View User in Modal
        viewUser(user) {
            this.selectedUser = user;
        },

        closeModal() {
            this.selectedUser = null;
        },

        // Pagination
        nextPage() {
            this.page++;
            this.fetchUsers();
        },

        prevPage() {
            if (this.page > 1) {
                this.page--;
                this.fetchUsers();
            }
        },
    },
};
</script>

<style scoped>
/* Custom styles */
</style>
