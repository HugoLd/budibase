<script>
  import { onMount } from "svelte"
  import { fade } from "svelte/transition"

  export let _bb
  export let model
  export let title
  export let buttonText

  const TYPE_MAP = {
    string: "text",
    boolean: "checkbox",
    number: "number",
  }

  let record
  let store = _bb.store
  let schema = {}
  let modelDef = {}
  let saved = false
  let saving = false
  let recordId
  let isNew = true

  let inputElements = {}

  $: if (model && model.length !== 0) {
    fetchModel()
  }

  $: fields = Object.keys(schema)

  $: Object.values(inputElements).length && setForm(record)

  async function fetchModel() {
    const FETCH_MODEL_URL = `/api/models/${model}`
    const response = await _bb.api.get(FETCH_MODEL_URL)
    modelDef = await response.json()
    schema = modelDef.schema
  }

  async function save() {
    // prevent double clicking firing multiple requests
    if (saving) return
    saving = true
    const SAVE_RECORD_URL = `/api/${model}/records`
    const response = await _bb.api.post(SAVE_RECORD_URL, record)

    const json = await response.json()

    if (response.status === 200) {
      store.update(state => {
        state[model] = state[model] ? [...state[model], json] : [json]
        return state
      })

      // wipe form, if new record, otherwise update
      // model to get new _rev
      if (isNew) {
        resetForm()
      } else {
        record = json
      }

      // set saved, and unset after 1 second
      // i.e. make the success notifier appear, then disappear again after time
      saved = true
      setTimeout(() => {
        saved = false
      }, 1000)
    }
    saving = false
  }

  // we cannot use svelte bind on these inputs, as it does not allow
  // bind, when the input type is dynamic
  const resetForm = () => {
    for (let el of Object.values(inputElements)) {
      el.value = ""
      if (el.checked) {
        el.checked = false
      }
    }
    record = {
      modelId: model,
    }
  }

  const setForm = rec => {
    if (isNew || !rec) return
    for (let fieldName in inputElements) {
      if (typeof rec[fieldName] === "boolean") {
        inputElements[fieldName].checked = rec[fieldName]
      } else {
        inputElements[fieldName].value = rec[fieldName]
      }
    }
  }

  const handleInput = field => event => {
    let value

    if (event.target.type === "checkbox") {
      value = event.target.checked
      record[field] = value
      return
    }

    if (event.target.type === "number") {
      value = parseInt(event.target.value)
      record[field] = value
      return
    }

    value = event.target.value
    record[field] = value
  }

  onMount(() => {
    const routeParams = _bb.routeParams()
    recordId =
      Object.keys(routeParams).length > 0 && (routeParams.id || routeParams[0])
    isNew = !recordId || recordId === "new"

    if (isNew) {
      record = { modelId: model }
    } else {
      const GET_RECORD_URL = `/api/${model}/records/${recordId}`
      _bb.api
        .get(GET_RECORD_URL)
        .then(response => response.json())
        .then(rec => {
          record = rec
          setForm(rec)
        })
    }
  })
</script>

<form class="form" on:submit|preventDefault>
  {#if title}
    <h1>{title}</h1>
  {/if}
  <hr />
  <div class="form-content">
    {#each fields as field}
      <div class="form-item">
        <label class="form-label" for="form-stacked-text">{field}</label>
        {#if schema[field].type === 'string' && schema[field].constraints.inclusion}
          <select on:blur={handleInput(field)} bind:this={inputElements[field]}>
            {#each schema[field].constraints.inclusion as opt}
              <option>{opt}</option>
            {/each}
          </select>
        {:else}
          <input
            bind:this={inputElements[field]}
            class="input"
            type={TYPE_MAP[schema[field].type]}
            on:change={handleInput(field)} />
        {/if}
      </div>
      <hr />
    {/each}
    <div class="button-block">
      <button on:click={save} class:saved>
        {#if saved}
          <div in:fade>
            <span class:saved>Success</span>
          </div>
        {:else}
          <div>{buttonText || 'Submit Form'}</div>
        {/if}
      </button>
    </div>
  </div>
</form>

<style>
  .form {
    display: flex;
    flex-direction: column;
    justify-content: center;
    margin: auto;
    padding: 40px;
  }
  .form-content {
    margin-bottom: 20px;
  }
  .input {
    border-radius: 5px;
    border: 1px solid #e6e6e6;
    padding: 1em;
    font-size: 16px;
  }

  .form-item {
    display: grid;
    grid-template-columns: 30% 1fr;
    margin-bottom: 16px;
    align-items: center;
    gap: 1em;
  }

  hr {
    border: 1px solid #f5f5f5;
    margin: 40px 0px;
  }
  hr:nth-last-child(2) {
    border: 1px solid #f5f5f5;
    margin: 40px 0px;
  }
  .button-block {
    display: flex;
    justify-content: flex-end;
  }
  button {
    font-size: 16px;
    padding: 8px 16px;
    box-sizing: border-box;
    border-radius: 4px;
    color: white;
    background-color: black;
    outline: none;
    height: 40px;
    cursor: pointer;
    transition: all 0.2s ease 0s;
    overflow: hidden;
    outline: none;
    user-select: none;
    white-space: nowrap;
    text-align: center;
  }

  button.saved {
    background-color: #84c991;
    border: none;
  }

  button:hover {
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1),
      0 4px 6px -2px rgba(0, 0, 0, 0.05);
  }

  input[type="checkbox"] {
    transform: scale(2);
    cursor: pointer;
  }

  select::-ms-expand {
    display: none;
  }
  select {
    cursor: pointer;
    display: inline-block;
    align-items: baseline;
    box-sizing: border-box;
    padding: 1em 1em;
    border: 1px solid #eaeaea;
    border-radius: 5px;
    font: inherit;
    line-height: inherit;
    -webkit-appearance: none;
    -moz-appearance: none;
    -ms-appearance: none;
    appearance: none;
    background-repeat: no-repeat;
    background-image: linear-gradient(45deg, transparent 50%, currentColor 50%),
      linear-gradient(135deg, currentColor 50%, transparent 50%);
    background-position: right 17px top 1.5em, right 10px top 1.5em;
    background-size: 7px 7px, 7px 7px;
  }
</style>
