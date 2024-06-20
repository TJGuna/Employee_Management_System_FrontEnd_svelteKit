<script lang="ts">
	import { createEventDispatcher, onMount } from 'svelte';
	import axios from 'axios';

	// Define the interface for Task (if not imported)
	interface Task {
		id: number;
		name: string;
		description: string;
		assigned_to: string;
		priority: string;
		status: string;
		deadline: string;
	}

	// Exported props for edit mode and the selected task
	export let editMode: boolean;
	export let selectedTask: Task | null;

	const dispatch = createEventDispatcher();

	let id: number | null = null;
	let name: string = '';
	let description: string = '';
	let assigned_to: string = '';
	let priority: string = '';
	let status: string = '';
	let deadline: string = '';

	// Initialize form fields if editing a task
	onMount(() => {
		if (editMode && selectedTask) {
			console.log('Initializing form for edit mode with task:', selectedTask);
			id = selectedTask.id;
			name = selectedTask.name;
			description = selectedTask.description;
			assigned_to = selectedTask.assigned_to;
			priority = selectedTask.priority;
			status = selectedTask.status;
			deadline = selectedTask.deadline;
		}
	});

	async function handleSubmit() {
		console.log('Submitting form with data:', { id, name, description, assigned_to, priority, status, deadline });
		const taskData = { name, description, assigned_to, priority, status, deadline };

		try {
			if (editMode && selectedTask) {
				await axios.put(`http://localhost:3000/tasks/${id}`, taskData);
				dispatch('taskUpdated');
				console.log('Task updated successfully');
			} else {
				await axios.post('http://localhost:3000/tasks', taskData);
				dispatch('taskAdded');
				console.log('Task added successfully');
			}
		} catch (error) {
			console.error('Error saving task:', error);
		}
	}

	function close() {
		console.log('Closing modal');
		dispatch('closeModal');
	}
</script>

<!-- Task Form Modal -->
<div class="p-8 bg-white rounded-lg shadow-lg">
	<h2 class="text-xl font-semibold text-gray-800">{editMode ? 'Edit Task' : 'Add Task'}</h2>
	<form on:submit|preventDefault={handleSubmit} class="space-y-4 mt-4">
		<div>
			<label for="name" class="block text-sm font-medium text-gray-700">Name</label>
			<input type="text" id="name" class="mt-1 block w-full rounded-md border border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" bind:value={name} required />
		</div>
		<div>
			<label for="description" class="block text-sm font-medium text-gray-700">Description</label>
			<textarea id="description" class="mt-1 block w-full rounded-md border border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" bind:value={description} required></textarea>
		</div>
		<div>
			<label for="assigned_to" class="block text-sm font-medium text-gray-700">Assigned To</label>
			<input type="text" id="assigned_to" class="mt-1 block w-full rounded-md border border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" bind:value={assigned_to} required />
		</div>
		<div>
			<label for="priority" class="block text-sm font-medium text-gray-700">Priority</label>
			<input type="text" id="priority" class="mt-1 block w-full rounded-md border border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" bind:value={priority} required />
		</div>
		<div>
			<label for="status" class="block text-sm font-medium text-gray-700">Status</label>
			<input type="text" id="status" class="mt-1 block w-full rounded-md border border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" bind:value={status} required />
		</div>
		<div>
			<label for="deadline" class="block text-sm font-medium text-gray-700">Deadline</label>
			<input type="date" id="deadline" class="mt-1 block w-full rounded-md border border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" bind:value={deadline} required />
		</div>
		<div class="mt-6 flex justify-end space-x-4">
			<button type="button" class="px-4 py-2 bg-gray-300 text-gray-800 rounded-md font-semibold hover:bg-gray-400 focus:outline-none" on:click={close}>Cancel</button>
			<button type="submit" class="px-4 py-2 bg-indigo-600 text-white rounded-md font-semibold hover:bg-indigo-700 focus:outline-none">{editMode ? 'Update' : 'Add'}</button>
		</div>
	</form>
</div>
