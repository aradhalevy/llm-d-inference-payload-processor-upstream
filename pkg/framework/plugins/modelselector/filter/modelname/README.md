# Model Name Filter

Restricts the candidate models to the model name(s) in the request body. The body's
model field is treated as a single model name, with a special case allowing the name
to be a JSON-encoded array of model names (still a string-typed field).

It is registered as type `model-name-filter` and runs as a modelselector filter.

The filter does not read the datalayer. It only intersects the requested name(s) with
the candidate models the pipeline hands it (those candidates having been sourced from
the datalayer upstream). "Available" below means "present in that candidate list".

## What it does

1. Reads the model field from the request body (`model` by default).
2. When the field holds a single available model name, that one model becomes the candidate.
3. When the field holds a string starting with `[`, it is parsed as a JSON-encoded array of model names (e.g. `"model": "[\"model-A\", \"model-B\"]"`), interpreted as "choose from the list". The filter keeps the requested names that are available and drops the rest. The scorers and picker select the best of that subset, and the model-selector plugin writes the chosen model back into the `model` field.
4. If the field is absent, an empty string, or an encoded empty array (`"[]"`), all incoming candidates are kept.
5. If no requested model is available, or the field is malformed (not a string, or a `[`-prefixed string that does not parse as a JSON array of non-empty strings), the filter returns no candidates and the pipeline rejects the request with HTTP 400.

## Inputs consumed

- The configured model field of the request body.
- The candidate model list passed in by the pipeline.

## Configuration

```json
{"requestModelField": "model"}
```

- `requestModelField` (optional): the request-body field holding the requested model name. Defaults to `model`.
