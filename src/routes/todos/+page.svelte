<script lang="ts">
	import { Todo } from '$lib/models/todo.model';
	import moment from 'moment';
	import { onMount } from 'svelte';

	var todos: Todo[] = [];
	var model: Todo = new Todo();
	const endpoint = 'http://localhost:3000/todos';
	const headers = {
		'Content-Type': 'application/json'
	};

	async function getTodos() {
		const response = await fetch(endpoint);
		if (response.ok) {
			const data: Todo[] = await response.json();
			todos = data;
			return todos;
		} else {
			// Sometimes the API will fail!
			throw new Error('Request failed');
		}
	}
	let getTodosPromise = getTodos();

	async function onSave() {
		if (!!model.id) {
			await putTodo();
		} else {
			await postTodo();
		}
	}

	async function postTodo() {
		model.startDate = new Date();
		model.endDate = undefined;
		model.isDeleted = false;
		model.isDone = false;
		const response = await fetch(endpoint, {
			method: 'POST',
			body: JSON.stringify(model),
			headers: headers
		});
		const data = await response.json();
		console.log(data);
		model = new Todo();
		getTodosPromise = getTodos();
	}

	async function putTodo() {
		const response = await fetch(`${endpoint}/${model.id}`, {
			method: 'Put',
			body: JSON.stringify(model),
			headers: headers
		});
		const data = await response.json();
		console.log(data);
		model = new Todo();
		getTodosPromise = getTodos();
	}

	async function deleteTodo(todo: Todo) {
		const response = await fetch(`${endpoint}/${todo.id}`, {
			method: 'DELETE',
			body: JSON.stringify(todo),
			headers: headers
		});
		const data = await response.json();
		console.log(data);
		getTodosPromise = getTodos();
	}

	async function editTodo(todo: Todo) {
		model = todo;
	}
	// kind of on init
	onMount(getTodos);
</script>

<main>
	<div class="card">
		<div class="card-body h-70">
			<h4 class="card-title">Todos</h4>
			<div class="input-group mb-3">
				<input
					type="text"
					class="form-control"
					placeholder=""
					aria-label="Button"
					aria-describedby=""
					bind:value={model.task}
					on:keydown={async (e) => {
						const task = e.currentTarget.value;
						if (e.key == 'Enter' && !!task && task != '') {
							await onSave();
						}
					}}
				/>
				<button class="btn btn-outline-secondary" type="button" id="" on:click={onSave}>Save</button
				>
			</div>

			<div class="table-responsive" style="overflow-y:auto; max-height:30rem">
				<table class="table table-default">
					<thead>
						<tr>
							<th scope="col">Task</th>
							<th scope="col">Start</th>
							<th scope="col">End</th>
							<th scope="col">Actions</th>
						</tr>
					</thead>
					<tbody>
						{#await getTodosPromise}
							<tr class="">
								<td colspan="4">waiting..</td>
							</tr>
						{:then todoRes}
							{#each todoRes as todo}
								<tr class="">
									<td>{todo.task}</td>
									<td>{moment(todo.startDate).format('DD MM YYYY')}</td>
									<td>{!!todo.endDate ? moment(todo.endDate).format('DD MM YYYY') : 'not set'}</td>
									<td>
										<button class="btn btn-sm btn-danger" on:click={async (e) => deleteTodo(todo)}>
											<i class="fa fa-trash" />
										</button>
										<button class="btn btn-sm btn-primary" on:click={async (e) => editTodo(todo)}>
											<i class="fa fa-pencil" />
										</button>
									</td>
								</tr>
							{/each}
						{:catch error}
							<p style="color: red">couldn't load data{error.message}</p>
						{/await}
					</tbody>
				</table>
			</div>
		</div>
	</div>
</main>
