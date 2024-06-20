<!-- EmployeeForm.svelte -->

<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import { onMount } from 'svelte';
	import axios from 'axios';

	export let editMode: boolean;
	export let selectedEmployee: Employee | null;

	// Define Employee interface
	interface Employee {
		id?: number; // Optional because ID will be assigned by the backend
		name: string;
		email: string;
		phone: string;
		address: string;
		joining_date: string;
	}

	let employee: Employee = {
		name: '',
		email: '',
		phone: '',
		address: '',
		joining_date: ''
	};

	let errorMessage: string = '';

	const dispatch = createEventDispatcher();

	async function handleSubmit() {
		if (!validateForm()) {
			return;
		}

		try {
			if (editMode && selectedEmployee) {
				// Update existing employee
				await updateEmployee(selectedEmployee.id!, employee);
			} else {
				// Add new employee
				await addEmployee(employee);
			}
		} catch (error) {
			console.error('Error:', error);
			errorMessage = 'An error occurred. Please try again later.'; // Set error message for display
		}
	}

	async function addEmployee(newEmployee: Employee) {
		try {
			const response = await axios.post<Employee>('http://localhost:3000/employees', newEmployee);
			console.log('Add Employee Response:', response.data); // Example: Log response data
			dispatch('employeeAdded', newEmployee); // Dispatch event to parent component
		} catch (error) {
			console.error('Error adding employee:', error);
			// Handle error, set error message, etc.
		}
	}

	async function updateEmployee(id: number, updatedEmployee: Employee) {
		try {
			const response = await axios.put<Employee>(`http://localhost:3000/employees/${id}`, updatedEmployee);
			console.log('Update Employee Response:', response.data); // Example: Log response data
			dispatch('employeeUpdated', updatedEmployee); // Dispatch event to parent component
		} catch (error) {
			console.error('Error updating employee:', error);
			// Handle error, set error message, etc.
		}
	}

	function validateForm() {
		// Basic form validation example (you can expand this as needed)
		if (!employee.name.trim() || !employee.email.trim() || !employee.phone.trim()) {
			errorMessage = 'Please fill out all required fields.';
			return false;
		}
		return true;
	}
	function closeForm() {
		dispatch('closeModal'); // Dispatch event to parent component to close the modal
	}
	// If in edit mode, populate form fields with selectedEmployee data
	onMount(() => {
		if (editMode && selectedEmployee) {
			employee = { ...selectedEmployee };
		}
	});
</script>
<div>

</div>
<form on:submit|preventDefault={handleSubmit} class="space-y-4 ">
	<!-- Display error message if there's an error -->
	{#if errorMessage}
		<p class="text-red-500">{errorMessage}</p>
	{/if}
	<div class="bg-white p-6 rounded-lg shadow-md">
		<h2 class="text-xl font-bold text-gray-800 mb-4">{editMode ? 'Update Employee' : 'Create Employee'}</h2>

		<div class="mb-4">
			<label for="name" class="block text-sm font-medium text-gray-700">Name</label>
			<input type="text" id="name" name="name" autocomplete="name" required
						 class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
						 bind:value={employee.name}>
		</div>

		<div class="mb-4">
			<label for="email" class="block text-sm font-medium text-gray-700">Email</label>
			<input type="email" id="email" name="email" autocomplete="email" required
						 class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
						 bind:value={employee.email}>
		</div>

		<div class="mb-4">
			<label for="phone" class="block text-sm font-medium text-gray-700">Phone</label>
			<input type="tel" id="phone" name="phone" autocomplete="tel" required
						 class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
						 bind:value={employee.phone}>
		</div>

		<div class="mb-4">
			<label for="address" class="block text-sm font-medium text-gray-700">Address</label>
			<input type="text" id="address" name="address" autocomplete="street-address"
						 class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
						 bind:value={employee.address}>
		</div>

		<div class="mb-6">
			<label for="joining_date" class="block text-sm font-medium text-gray-700">Joining Date</label>
			<input type="date" id="joining_date" name="joining_date" autocomplete="bday" required
						 class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
						 bind:value={employee.joining_date}>
		</div>

		<div class="flex justify-end">
			<button type="submit"
							class="inline-flex items-center px-4 py-2 bg-indigo-600 border border-transparent rounded-md font-semibold text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 mr-2">
				<!--{#if editMode}-->
				<!--	Update Employee-->
				<!--{:else}-->
				<!--	Add Employee-->
				<!--{/if}-->

				Save
			</button>

			<button type="button"
							class="inline-flex items-center px-4 py-2 bg-gray-300 border border-transparent rounded-md font-semibold text-gray-800 hover:bg-gray-400 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500" on:click={closeForm}>
				Cancel
			</button>
		</div>
	</div>

</form>
