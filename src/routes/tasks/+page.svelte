<script lang="ts">
    import axios from 'axios';
    import { onMount } from 'svelte';
    import TaskForm from './../TaskForm/+page.svelte'; // Import the TaskForm component

    interface Task {
        id: number;
        name: string;
        description: string;
        assigned_to: string;
        priority: string;
        status: string;
        deadline: string;
    }

    let tasks: Task[] = [];
    let error: Error | null = null;
    let loading: boolean = true;

    let showModal = false;
    let editMode = false;
    let isDeleteModalOpen = false;
    let selectedTask: Task | null = null;
    let taskToDelete: Task | null = null;

    onMount(async () => {
        console.log('Component mounted, fetching tasks...');
        try {
            const response = await axios.get<Task[]>('http://localhost:3000/tasks');
            tasks = response.data;
            console.log('Tasks fetched successfully:', tasks);
        } catch (err) {
            error = err as Error;
            console.error('Error fetching tasks:', error);
        } finally {
            loading = false;
            console.log('Loading state:', loading);
        }
    });

    function openAddModal() {
        console.log('Opening add modal...');
        editMode = false;
        selectedTask = null;
        showModal = true;
    }

    function openEditModal(task: Task) {
        console.log('Opening edit modal for task:', task);
        editMode = true;
        selectedTask = task;
        showModal = true;
    }

    function openDeleteModal(task: Task) {
        console.log('Opening delete modal for task:', task);
        taskToDelete = task;
        isDeleteModalOpen = true;
    }

    function closeModal() {
        console.log('Closing modal...');
        showModal = false;
        taskToDelete = null;
    }

    function handleTaskAdded() {
        console.log('Task added successfully');
        closeModal();
        fetchTasks();
    }

    function handleTaskUpdated() {
        console.log('Task updated successfully');
        closeModal();
        fetchTasks();
    }

    function closeDeleteModal() {
        console.log('Closing delete modal...');
        taskToDelete = null;
        isDeleteModalOpen = false;
    }

    async function deleteTask() {
        if (!taskToDelete) return;
        console.log('Deleting task:', taskToDelete);
        try {
            await axios.delete(`http://localhost:3000/tasks/${taskToDelete.id}`);
            tasks = tasks.filter(t => t.id !== taskToDelete!.id);
            console.log('Task deleted successfully');
        } catch (error) {
            console.error('Error deleting task:', error);
        } finally {
            closeDeleteModal();
        }
    }

    async function fetchTasks() {
        console.log('Fetching tasks...');
        try {
            const response = await axios.get<Task[]>('http://localhost:3000/tasks');
            tasks = response.data;
            console.log('Tasks fetched successfully:', tasks);
        } catch (err) {
            error = err as Error;
            console.error('Error fetching tasks:', error);
        }
    }
</script>

<!-- HTML template structure -->
<div class="px-4 sm:px-6 lg:px-8">
    <div class="sm:items-center sm:justify-end justify-between flex flex-row sm:flex-row">
        <div class="sm:flex-auto">
            <h1 class="font-extrabold text-xl leading-6 text-gray-900">Tasks</h1>
        </div>
        <div>
            <button type="button" class="block rounded-md bg-indigo-600 px-3 py-2 text-center text-sm font-semibold text-white shadow-sm hover:bg-indigo-500 focus:outline-none focus:ring-2 focus:ring-indigo-600" on:click={openAddModal}>
                Add Task
            </button>
        </div>
    </div>

    <div class="mt-8">
        {#if loading}
            <p>Loading...</p>
        {:else if error}
            <p>Error: {error.message}</p>
        {:else}
            <div class="overflow-x-auto">
                <table class="min-w-full border-separate border-spacing-0">
                    <thead>
                    <tr>
                        <th scope="col" class="sticky top-0 z-10 border-b border-gray-300 bg-white bg-opacity-75 py-3.5 pl-4 pr-3 text-left text-sm font-semibold text-gray-900 backdrop-blur backdrop-filter sm:pl-6 lg:pl-8">Task ID</th>
                        <th scope="col" class="sticky top-0 z-10 hidden border-b border-gray-300 bg-white bg-opacity-75 px-3 py-3.5 text-left text-sm font-semibold text-gray-900 backdrop-blur backdrop-filter sm:table-cell">Task Name</th>
                        <th scope="col" class="sticky top-0 z-10 hidden border-b border-gray-300 bg-white bg-opacity-75 px-3 py-3.5 text-left text-sm font-semibold text-gray-900 backdrop-blur backdrop-filter lg:table-cell">Description</th>
                        <th scope="col" class="sticky top-0 z-10 hidden border-b border-gray-300 bg-white bg-opacity-75 px-3 py-3.5 text-left text-sm font-semibold text-gray-900 backdrop-blur backdrop-filter lg:table-cell">Assigned To</th>
                        <th scope="col" class="sticky top-0 z-10 hidden border-b border-gray-300 bg-white bg-opacity-75 px-3 py-3.5 text-left text-sm font-semibold text-gray-900 backdrop-blur backdrop-filter lg:table-cell">Priority</th>
                        <th scope="col" class="sticky top-0 z-10 hidden border-b border-gray-300 bg-white bg-opacity-75 px-3 py-3.5 text-left text-sm font-semibold text-gray-900 backdrop-blur backdrop-filter lg:table-cell">Status</th>
                        <th scope="col" class="sticky top-0 z-10 border-b border-gray-300 bg-white bg-opacity-75 px-3 py-3.5 text-left text-sm font-semibold text-gray-900 backdrop-blur backdrop-filter">Deadline</th>
                        <th scope="col" class="sticky top-0 z-10 border-b border-gray-300 bg-white bg-opacity-75 py-3.5 pl-3 pr-4 backdrop-blur backdrop-filter sm:pr-6 lg:pr-8">
                            <span class="sr-only">Edit</span>
                        </th>
                    </tr>
                    </thead>
                    <tbody>
                    {#each tasks as task}
                        <tr>
                            <td class="whitespace-nowrap border-b border-gray-200 py-4 pl-4 pr-3 text-sm font-medium text-gray-900 sm:pl-6 lg:pl-8">{task.id}</td>
                            <td class="hidden whitespace-nowrap border-b border-gray-200 px-3 py-4 text-sm text-gray-500 sm:table-cell">{task.name}</td>
                            <td class="hidden whitespace-nowrap border-b border-gray-200 px-3 py-4 text-sm text-gray-500 lg:table-cell">{task.description}</td>
                            <td class="whitespace-nowrap border-b border-gray-200 px-3 py-4 text-sm text-gray-500">{task.assigned_to}</td>
                            <td class="whitespace-nowrap border-b border-gray-200 px-3 py-4 text-sm text-gray-500">{task.priority}</td>
                            <td class="whitespace-nowrap border-b border-gray-200 px-3 py-4 text-sm text-gray-500">{task.status}</td>
                            <td class="whitespace-nowrap border-b border-gray-200 px-3 py-4 text-sm text-gray-500">{task.deadline}</td>
                            <td class="whitespace-nowrap border-b border-gray-200 px-3 py-4 text-sm font-medium space-x-3">
                                <button class="text-indigo-600 hover:text-indigo-900" on:click={() => openEditModal(task)}>Edit</button>
                                <button class="text-red-600 hover:text-red-900" on:click={() => openDeleteModal(task)}>Delete</button>
                            </td>
                        </tr>
                    {/each}
                    </tbody>
                </table>
            </div>
        {/if}
    </div>

    <!-- Modal for Add/Edit Task -->
    {#if showModal}
        <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div class="bg-white rounded-lg overflow-hidden shadow-xl transform transition-transform sm:max-w-md sm:w-full">
                <TaskForm {editMode} {selectedTask} on:taskAdded={handleTaskAdded} on:taskUpdated={handleTaskUpdated} on:closeModal={closeModal} />
            </div>
        </div>
    {/if}

    <!-- Modal for Delete Task Confirmation -->
    {#if isDeleteModalOpen}
        <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
            <div class="bg-white rounded-lg overflow-hidden shadow-xl transform transition-transform sm:max-w-md sm:w-full">
                <div class="p-8">
                    <p class="text-lg font-semibold text-gray-800">Are you sure you want to delete this task?</p>
                    <div class="mt-6 flex justify-end space-x-4">
                        <button type="button" class="px-4 py-2 bg-red-600 text-white rounded-md font-semibold hover:bg-red-700 focus:outline-none" on:click={deleteTask}>Yes</button>
                        <button type="button" class="px-4 py-2 bg-gray-300 text-gray-800 rounded-md font-semibold hover:bg-gray-400 focus:outline-none" on:click={closeDeleteModal}>No</button>
                    </div>
                </div>
            </div>
        </div>
    {/if}
</div>
