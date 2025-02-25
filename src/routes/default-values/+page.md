<script lang="ts">
  import Head from '$lib/Head.svelte'
</script>

# Default values

<Head title="Default values" />

When `superValidate` encounters a schema field that isn't optional, or when a `FormData` field is falsy, a default value is returned to the form, to ensure that the type is correct:

| type    | value       |
| ------- | ----------- |
| string  | `""`        |
| number  | `0`         |
| boolean | `false`     |
| Array   | `[]`        |
| object  | `{}`        |
| bigint  | `BigInt(0)` |
| symbol  | `Symbol()`  |

## Changing a default value

You can, of course, set your own default values in the schema, using the `default` method. You can even abuse the typing system a bit to handle the classic "agree to terms" checkbox:

```ts
const schema = z.object({
  age: z
    .number()
    .positive()
    .default('' as unknown as number),
  agree: z.literal(true).default(false as true)
});
```

This looks a bit strange, but will ensure that the age isn't set to 0 as default (which will hide placeholder text in the input field), and also ensure that the agree checkbox is unchecked as default while also only accepting `true` (checked) as a value.

> The type system is bypassed with this, so the default value will not correspond to the type, but this will usually not be a problem since `form.valid` will be `false` if these values are posted, and that should be the main determinant of whether the data is trustworthy.

## optional vs. nullable

Fields set to `null` will take precedence over `undefined`, so a field that is both `nullable` and `optional` will have `null` as its default value. Otherwise, it's `undefined`.

## Non-supported defaults

The Zod types `ZodEnum` and `ZodUnion` can't use the listed default values, in which case you must set a default value for them:

```ts
const schema = z.object({
  fish: z.enum(['Salmon', 'Tuna', 'Trout']).default('Salmon')
});

// If it's nullable/optional/nullish, no need for a default
// (but it can still be set)
const schema2 = z.object({
  fish: z.enum(['Salmon', 'Tuna', 'Trout']).nullable()
});
```
