<script>
	import { HighlightSvelte } from 'svelte-highlight';
	import { HighlightAuto } from 'svelte-highlight';
	import Markdown from 'svelte-exmarkdown';
	import highlightStyle from 'svelte-highlight/styles/gruvbox-dark-soft';
	import dedent from 'dedent';
	// import svelte from "svelte-highlight/languages/typescript";
	// import vue from "svelte-highlight/languages/vue";

	const sections = [
		{
			title: 'Hello World (Counter)',
			description: `
        Buttons that increment a counter and displays the count. Showcases basic reactive state and event handling.
      `,
			svelte: {
				code: `
          <script>
            const count = $state(0)

            function increment() {
              count += 1
            }
          <\/script>

          <button onclick={increment}>
            Increment with function {count}
          </button>
          <button onclick={() => count += 1}>
            Increment with inline handler {count}
          </button>
        `,
				notes: `
          - uses browser-native event names:  \`onclick=\`
          - always uses the variable name directly (no \`.value\` needed)
        `
			},
			vue: {
				code: `
          <script setup>
            import {ref} from "vue"

            const count = ref(0)

            function increment() {
              count.value += 1  // NOTICE: .value needed in script
            }
          <\/script>

          <template>
            <button @click="increment">
              Increment with function {{ count }}
            <\/button>
            <button @click="count += 1"> <!-- NOTICE: no .value -->
              Increment with inline handler {{ count }}
            <\/button>
          </template>
        `,
				notes: `
          - uses special syntax for event bindings: \`@click=\`
          - use \`.value\` in script, don't use it in template. **big gotcha**
        `
			}
		},

		{
			title: 'Input Binding',
			description: `Synchronizing state with an input's value. Not much to say here, just a matter of taste.`,
			svelte: {
				code: `
          <script>
            let name = $state('Dmitry')
            let checked = $state(true)
          <\/script>

          <!-- bind:value for input, textarea, select, etc -->
          <input type="text" bind:value={name} />

          <!-- checkboxes use bind:checked -->
          <input type="checkbox" bind:checked={checked} />
        `,
				notes: `- \`bind:\` syntax`
			},
			vue: {
				code: `
          <script setup>
            import {ref} from "vue"

            let name = ref('Dmitry')
            let checked = ref(false)
          <\/script>

          <template>
            <input type="text" v-model="name">
            <input type="checkbox" v-model="checked">

            <!-- long form: -->
            <input type="text" :value="name" @input="event => name = event.target.value">
            <input type="checkbox" :value="checked" @input="event => checked = event.target.value">
          </template>
        `,
				notes: `
          - v-model attribute (personally not a fan)
        `
			}
		},

		{
			title: 'Control Flow',
			description: `Ifs and loops`,
			svelte: {
				code: `
          <!-- if statement -->
          {#if condition}
            <p>it's true</p>
          {:else if otherCondition}
            <p>it's something else</p>
          {:else}
            <p>it's nothing</p>
          {/if}

          <!-- each loop -->
          {#each items as item}
            <p>{item}</p>
          {:else}
            <p>there are no items</p>
          {/each}
        `,
				notes: `
          - special syntax for control flow
          - small learning curve but at least it's consistent: # to start, : in the middle, / to end
          - control flow can't get lost visually - it's always clear
        `
			},
			vue: {
				code: `
          <!-- if statement -->
          <p v-if="condition">it's true</p>
          <p v-else-if="otherCondition">it's something else</p>
          <p v-else="otherCondition">it's nothing</p>

          <!-- for loop -->
          <p v-for="item in items">{{ item }}</p>

          <!-- empty check. there is no "else" so have to wrap in v-if -->
          <div v-if="items.length > 0">
            <p v-for="item in items">{{ item }}</p>
          </div>
          <div v-else>
            <p>there are no items</p>
          </div>
        `,
				notes: `
          - special attributes v-if, v-for, etc
          - control flow can get lost in a sea of other attributes on the same element
        `
			}
		},

		{
			title: 'Deep State Reactivity / Object Mutability',
			description: `Reactivity with objects (or arrays)`,
			svelte: {
				code: `
          <script>
            let item = $state({id: 1, name: "foo"})
            item.name = "bar";
          <\/script>

          <p>{item.id}: {item.name}</p>
        `,
				notes: `- $state is used for both primives and objects`
			},
			vue: {
				code: `
          <script setup>
            import {reactive} from "vue"

            let item = reactive({id: 1, name: "foo"})
            item.value.name = "bar";
          <\/script>

          <template>
            <p>{{ item.id }}: {{ item.name }}</p>
          </template>
        `,
				notes: `
        - for objects, use \`reactive()\`
        - for primitive values (strings, numbers), use \`ref()\`
        - have to remember to use ref or reactive - **big gotcha**
        `
			}
		},

		{
			title: 'Derived state',
			description:
				'Deriving state from other state with calculations. Not too different between Svelte and Vue, so matter of taste.',
			svelte: {
				code: `
          <script>
            let numbers = $state([1, 2, 3, 4])
            let total = $derived(numbers.reduce((acc, n) => acc + n, 0))
            // total is 5
            numbers.push(5);
            // total is now 10
          <\/script>

          <p>The total is: {total}</p>
        `,
				notes: `
          - don't need callback function in $derived (you can if needed)
        `
			},
			vue: {
				code: `
          <script setup>
            import {reactive, computed} from "vue"
            
            const numbers = reactive([1, 2, 3, 4])
            const total = computed(() => numbers.reduce((acc, n) => acc + n, 0))
            // total is 5
            numbers.value.push(5);
            // total is now 10
          <\/script>

          <template>
            <p>The total is: {{ total }}</p>
          </template>
        `,
				notes: `
          - computed() takes a callback function
        `
			}
		},

		{
			title: 'Components Props',
			description: `Passing values to custom components.`,
			svelte: {
				code: `
          <!-- Person.svelte -->
          <script>
            let {name} = $props()
          <\/script>

          <p>My name is {name}</p>

          <!-- usage -->
          <script>
            import Person from "./Person.svelte"

            let name = "Dmitry"
          <\/script>

          <Person name={name} />
          <!-- or shorthand: -->
          <Person {name} />
        `,
				notes: `
          - declare props with \`$props()\`
          - pass in with \`name={name}\` or \`{name}\` shorthand if prop name and variable name are the same
        `
			},
			vue: {
				code: `
          <!-- Person.vue -->
          <script setup>
            const {name} = defineProps(['name'])
          <\/script>

          <template>
            <p>My name is {{ name }}</p>
          </template>

          <!-- usage: -->
          <script setup>
            import Person from "./Person.vue"

            let name = "Dmitry"
          <\/script>

          <template>
            <Person :name="name" />
            <!-- there is no shorthand -->
          </template>
        `,
				notes: `
          - declare props with \`defineProps()\`
        `
			}
		},

		{
			title: 'Async (Promises)',
			description: `Waiting for a promise to resolve, displaying loading state, and handling errors.`,
			svelte: {
				code: `
          <script>
            let promise = fetch("...")
          <\/script>

          {#await promise}
            <p>Loading...</p>
          {:then data}
            <p>{data}</p>
          {:catch error}
            <p>{error}</p>
          {/await}
        `,
				notes: `
          - built-in await block which captures data, error, and loading states
        `
			},
			vue: {
				code: `
          <script setup>
            import {ref} from "vue"

            let loading = ref(true)
            let data = ref(null)
            let error = ref(null)
            let promise = fetch("...")
              .then(d => data.value = d)
              .catch(e => error.value = e)
              .finally(() => loading.value = false)
          <\/script>

          <p v-if="loading">Loading...</p>
          <p v-if="data">{{ data }}</p>
          <p v-if="error">{{ error }}</p>
        `,
				notes: `
          - Vue has no built-in promise support in templates
          - Have to build something custom with refs
        `
			}
		}
	];
</script>

<svelte:head>
	{@html highlightStyle}
</svelte:head>

<main>
	<h1>Svelte vs Vue</h1>
	{#each sections as section}
		<section>
			<h2 class="section-header">{section.title}</h2>
			<Markdown md={dedent(section.description)} />
			<div class="side-by-side">
				<div>
					<b>Svelte</b>
					<HighlightSvelte code={dedent(section.svelte.code)} />
				</div>
				<div>
					<b>Vue</b>
					<HighlightAuto code={dedent(section.vue.code)} />
				</div>
				<div>
					{#if section.svelte.notes}
						<Markdown md={dedent(section.svelte.notes)} />
					{/if}
				</div>
				<div>
					{#if section.vue.notes}
						<Markdown md={dedent(section.vue.notes)} />
					{/if}
				</div>
			</div>
		</section>
	{/each}
</main>

<style>
	main {
		max-inline-size: 1100px;
		margin: 0 auto;
		padding-bottom: 5rem;
		padding-inline: 1rem;
	}

	section {
		min-height: 100vh;
		margin-block: 5rem;
	}

	.side-by-side {
		display: grid;
		column-gap: 1rem;
		grid-template-columns: 1fr 1fr;
	}

	.side-by-side > div {
		overflow-x: auto;
	}

	h1,
	h2 {
		text-align: center;
	}
</style>
