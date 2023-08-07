<script lang="ts">
	import { onMount } from 'svelte';
	import { Todo } from '$lib/models/todo.model';
	import { elasticIn } from 'svelte/easing';
	import moment from 'moment';
	import { DateTime } from 'luxon';

	var todos: Todo[] = [];
	var model: Todo = new Todo();
	const endpoint = 'http://localhost:3000/todos';

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

	async function postTodo() {
		model.startDate = new Date();
		model.endDate = undefined;
		model.isDeleted = false;
		model.isDone = false;
		const response = await fetch(endpoint, {
			method: 'POST',
			body: JSON.stringify(model),
			headers: {
				'Content-Type': 'application/json'
			}
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
			headers: {
				'Content-Type': 'application/json'
			}
		});
		const data = await response.json();
		console.log(data);
		getTodosPromise = getTodos();
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
						debugger;
						const input = e.currentTarget;
						const task = input.value;
						if (e.key == 'Enter' && !!task && task != '') {
							await postTodo();
						}
					}}
				/>
				<button class="btn btn-outline-secondary" type="button" id="" on:click={postTodo}
					>Add</button
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
									<td
										><button class="btn btn-sm btn-danger" on:click={() => deleteTodo(todo)}>
											<i class="fa fa-trash" />
										</button></td
									>
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
