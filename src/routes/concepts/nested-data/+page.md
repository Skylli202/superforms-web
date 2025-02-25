<script lang="ts">
  import Head from '$lib/Head.svelte'
  import Form from './Form.svelte'
  import Next from '$lib/Next.svelte'
	import SuperDebug from 'sveltekit-superforms/client/SuperDebug.svelte'
  import { concepts } from '$lib/navigation/sections'

	export let data;
</script>

# Nested data

<Head title="Nested data" />

Basic HTML forms can only handle string values, and the `<form>` element cannot nest other forms, so there is no native way to represent a nested data structure or more complex values like dates. Fortunately, Superforms has a few solutions for this!

## Usage

```ts
const { form, errors, constraints } = superForm(data.form, {
  dataType: 'form' | 'json' = 'form'
})
```

### dataType

By simply setting `dataType` to `'json'`, you can store any data structure allowed by [devalue](https://github.com/Rich-Harris/devalue) in the `form` store, and you don't have to worry about failed coercion, converting arrays to strings and back, etc.

You don't even have to set names for the form fields anymore, since the actual data in `$form` is now posted, not the fields in the HTML. They are now merely UI components for modifying a data model.

> When `dataType` is set to `'json'`, the `onSubmit` event contains `formData`, but it cannot be used to modify the posted data. You need to set or update the `form` store.<br><br>It also means that the `disabled` attribute, which normally prevents fields from being submitted, will not have that effect. Everything in `$form` will be posted when `dataType` is set to `'json'`.

Modifying the `form` store programmatically is simple, you can either assign `$form.field = ...` directly, which will taint the affected fields. If you don't want to taint anything, you can use `update` with an extra option:

```ts
form.update(
  ($form) => {
    $form.name = "New name";
    return $form;
  },
  { taint: false }
);
```

The `taint` options are:

```ts
{ taint: boolean | 'untaint' | 'untaint-all' }
```

Which can be used to not only prevent tainting, but also untaint the modified field(s), or the whole form.

## Requirements

The requirements for nested data to work are as follows:

1. **JavaScript is enabled in the browser.**
2. **The form has use:enhance applied.**

## Nested errors and constraints

When your schema contains arrays or objects, you can access them through `$form` as an ordinary object. But how does it work with errors and constraints?

`$errors` and `$constraints` actually mirror the `$form` data, but with every field or "leaf" in the object replaced with `string[]` and `InputConstraints` respectively.

### Example

Given the following schema, which describes an array of tag objects:

```ts
import { z } from 'zod';

export const schema = z.object({
  tags: z
    .object({
      id: z.number().int().min(1),
      name: z.string().min(2)
    })
    .array()
});

const tags = [{ id: 1, name: 'test' }];

export const load = async () => {
  const form = await superValidate({ tags }, schema);
  return { form };
};
```

You can build up a HTML form for these tags using an `{#each}` loop:

```svelte
<script lang="ts">
  const { form, errors, enhance } = superForm(data.form, {
    // This is a requirement when the schema contains nested objects:
    dataType: 'json'
  });
</script>

<form method="POST" use:enhance>
  {#each $form.tags as _, i}
    <div>
      Id
      <input
        type="number"
        data-invalid={$errors.tags?.[i]?.id}
        bind:value={$form.tags[i].id} />
      Name
      <input
        data-invalid={$errors.tags?.[i]?.name}
        bind:value={$form.tags[i].name} />
      {#if $errors.tags?.[i]?.id}
        <br />
        <span class="invalid">{$errors.tags[i].id}</span>
      {/if}
      {#if $errors.tags?.[i]?.name}
        <br />
        <span class="invalid">{$errors.tags[i].name}</span>
      {/if}
    </div>
  {/each}
  <button>Submit</button>
</form>
```

> Take extra care with the [optional chaining operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining) `?.`, it's easy to miss a question mark, which will lead to confusing errors.

Note that we're using the index of the loop, so the value can be bound directly to `$form`. This is what it looks like:

<Form {data} />

## Arrays with primitive values

Since you can post multiple HTML elements with the same name, you could, but don't have to use `dataType: 'json'` for arrays of primitive values like numbers and strings. Just add the input fields, **all with the same name as the schema field**, which can only be at the top level of the schema, of course. Superforms will handle the type coercion to array automatically (just remember the name attribute):

```ts
export const schema = z.object({
  tags: z.string().min(2).array().max(3)
});
```

```svelte
<script lang="ts">
  const { form, errors } = superForm(data.form);
</script>

<form method="POST">
  <div>Tags</div>
  {#if $errors.tags?._errors}
    <div class="invalid">{$errors.tags._errors}</div>
  {/if}

  {#each $form.tags as _, i}
    <div>
      <input name="tags" bind:value={$form.tags[i]} />
      {#if $errors.tags?.[i]}
        <span class="invalid">{$errors.tags[i]}</span>
      {/if}
    </div>
  {/each}

  <button>Submit</button>
</form>
```

To summarize, the index `i` of the `#each` loop is used to access `$form.tags`, where the current values are, and then the `name` attribute is set to the schema field `tags`, so its array will be populated when posted.

This example, having a `max(3)` limitation of the number of tags, also shows how to display array-level errors with the `$errors.tags._errors` field.

<Next section={concepts} />
