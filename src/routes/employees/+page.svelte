
<!-- EmployeeList.svelte -->

<script lang="ts">
    import axios from 'axios';
    import { onMount } from 'svelte';
    import EmployeeForm from './../EmployeeForm/+page.svelte'; // Import the EmployeeForm component

    interface Employee {
        id: number;
        name: string;
        email: string;
        phone: string;
        address: string;
        joining_date: string;
    }

    let employees: Employee[] = [];
    let error: Error | null = null;
    let loading: boolean = true;

    onMount(async () => {
        try {
            const response = await axios.get<Employee[]>('http://localhost:3000/employees');
            employees = response.data;
            console.log('Fetched Employees:', employees);
        } catch (err) {
            error = err as Error;
            console.log('Error fetching employees:', error);
        } finally {
            loading = false;
            console.log('Loading state:', loading);
        }
    });

    let showModal = false;
    let editMode = false;
    let isDeleteModalOpen = false;
    let selectedEmployee: Employee | null = null;
    let employeeToDelete: Employee | null = null;


    function openAddModal() {
        editMode = false;
        selectedEmployee = null;
        showModal = true;
        console.log('Add Employee modal opened');
    }

    function openEditModal(employee: Employee) {
        editMode = true;
        selectedEmployee = employee;
        showModal = true;
        console.log('Edit Employee modal opened for employee:', employee);
    }


    function openDeleteModal(employee: Employee) {
        employeeToDelete = employee;
        isDeleteModalOpen = true;
    }

    function closeModal() {
        showModal = false;
        employeeToDelete = null;
        console.log('Modal closed');
    }

    function handleEmployeeAdded() {
        closeModal();
        console.log('Employee added successfully');
        // Reload employees or update state after adding a new employee if needed
    }

    function handleEmployeeUpdated() {
        closeModal();
        console.log('Employee updated successfully');
        // Reload employees or update state after updating an employee if needed
    }
    function closeDeleteModal() {
        employeeToDelete = null;
        isDeleteModalOpen = false;
        console.log('Delete modal closed');
    }

    async function deleteEmployee() {
        if (!employeeToDelete) return;
        try {
            const response = await axios.delete(`http://localhost:3000/employees/${employeeToDelete.id}`);
            console.log('Delete Employee Response:', response.data); // Log response data if needed
            employees = employees.filter(e => e.id !== employeeToDelete!.id); // Update local employee list
            console.log('Employee deleted successfully:', employeeToDelete);
        } catch (error) {
            console.error('Error deleting employee:', error);
            // Handle error, display message, etc.
        } finally {
            closeDeleteModal(); // Close modal regardless of success or failure
        }
    }

</script>

<div class=" p-4 h-screen">
    <!-- Employee List -->
    <!-- Add Employee Button -->
    <div class="sm:items-center sm:justify-end justify-between flex flex-row sm:flex-row mt-4  p-4 rounded-md bg-sky-100">
        <div class="sm:flex-auto">
            <h1 class="text-2xl font-extrabold leading-6 text-gray-900">Employees</h1>
        </div>
        <div class="">
            <button type="button" class="block rounded-md bg-sky-400 px-3 py-2 text-center text-sm font-semibold text-white shadow-sm hover:bg-sky-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-sky-600" on:click={openAddModal}>Add Employee</button>
        </div>
    </div>

    <div class="mt-2 flow-root p-4 rounded-md bg-sky-100">
        <!-- Table showing employees -->

        <table class="min-w-full table-fixed divide-y divide-gray-300">
            <thead>
            <tr>
                <th scope="col" class="relative px-7 sm:w-12 sm:px-6">
                    <input type="checkbox" class="absolute left-4 top-1/2 -mt-2 h-4 w-4 rounded border-gray-300 text-sky-600 focus:ring-sky-600">
                </th>
                <th scope="col" class="sticky min-w-[12rem] py-3.5 pr-3 text-left text-sm font-semibold text-gray-900">ID</th>
                <th scope="col" class="sticky px-3 py-3.5 text-left text-sm font-semibold text-gray-900">Name</th>
                <th scope="col" class="sticky px-3 py-3.5 text-left text-sm font-semibold text-gray-900">Email</th>
                <th scope="col" class="sticky px-3 py-3.5 text-left text-sm font-semibold text-gray-900">Phone</th>
                <th scope="col" class="sticky px-3 py-3.5 text-left text-sm font-semibold text-gray-900">Address</th>
                <th scope="col" class="sticky px-3 py-3.5 text-left text-sm font-semibold text-gray-900">Joining Date</th>
                <th scope="col" class="relative py-3.5 pl-3 pr-4 sm:pr-3">
                    <span class="sr-only">Actions</span>
                </th>
            </tr>
            </thead>
            <tbody class="divide-y divide-gray-200 bg-white">
            {#each employees as employee}
                <tr>
                    <td class="relative px-7 sm:w-12 sm:px-6">
                        <input type="checkbox" class="absolute left-4 top-1/2 -mt-2 h-4 w-4 rounded border-gray-300 text-sky-600 focus:ring-sky-600">
                    </td>
                    <td class="whitespace-nowrap py-4 pr-3 text-sm font-medium text-gray-900">{employee.id}</td>
                    <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">{employee.name}</td>
                    <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">{employee.email}</td>
                    <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">{employee.phone}</td>
                    <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">{employee.address}</td>
                    <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500">{employee.joining_date}</td>
                    <td class="whitespace-nowrap py-4 pl-3 pr-4 text-right text-sm font-medium sm:pr-3 space-x-3">
                        <!-- Edit button -->
                        <button class="text-sky-400 hover:text-sky-900" on:click={() => openEditModal(employee)}>Edit</button>
                        <button class="text-red-400 hover:text-red-900" on:click={() => openDeleteModal(employee)}>Delete</button>
                    </td>
                </tr>
            {/each}
            </tbody>
        </table>
    </div>
    {#if isDeleteModalOpen}
        <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div class="bg-white rounded-lg overflow-hidden shadow-xl transform transition-transform sm:max-w-md sm:w-full">
                <div class="p-8">
                    <p class="text-lg font-semibold text-gray-800">Are you sure you want to delete this employee?</p>
                    <div class="mt-6 flex justify-end space-x-4">
                        <button type="button" class="px-4 py-2 bg-red-600 text-white rounded-md font-semibold hover:bg-red-700 focus:outline-none" on:click={deleteEmployee}>Yes</button>
                        <button type="button" class="px-4 py-2 bg-gray-300 text-gray-800 rounded-md font-semibold hover:bg-gray-400 focus:outline-none" on:click={closeDeleteModal}>No</button>
                    </div>
                </div>
            </div>
        </div>
    {/if}

    <!-- Modal for Add/Edit Employee -->
    {#if showModal}
        <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div class="bg-white rounded-lg overflow-hidden shadow-xl transform transition-transform sm:max-w-md sm:w-full">
                <!-- Include EmployeeForm component for add/edit -->

                <EmployeeForm {editMode} {selectedEmployee} on:employeeAdded={handleEmployeeAdded} on:employeeUpdated={handleEmployeeUpdated} on:closeModal={closeModal} />
            </div>
        </div>
    {/if}
</div>
