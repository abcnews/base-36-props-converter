<script>
  import { decode, encode } from "@abcnews/base-36-props";
  import isPlainObject from "is-plain-object";
  import stringifyObject from "stringify-object";
  import { onMount } from "svelte";

  window.decode = decode;

  let propsEl;
  let base36El;
  let onChangeProps;
  let onChangeBase36;
  let onCopyProps;
  let onCopyBase36;
  let errorMessage;

  let val = "";
  let timer;

  const debounce = v => {
    clearTimeout(timer);
    timer = setTimeout(() => {
      val = v;
    }, 750);
  };

  function createConversionHandler(converter, outputEl) {
    function asynchronouslyHandleEvent(event) {
      errorMessage = null;

      const inputValue = event.target.value;

      try {
        outputEl.value = inputValue ? converter(inputValue) : "";
      } catch (err) {
        outputEl.value = "";
        console.error(err);

        if (err.name === "SyntaxError") {
          errorMessage = "Base36 cannot be decoded";
        } else {
          errorMessage = err.message;
        }
      }
    }

    let timeout;

    return event => {
      clearTimeout(timer);
      timer = setTimeout(asynchronouslyHandleEvent, 500, event);
    };
  }

  function encodeParsedObject(value) {
    let parsed;

    try {
      parsed = eval(`(${value || 0})`);
    } catch (err) {
      throw new Error("Props cannot be parsed");
    }

    return encode(parsed);
  }

  function decodeInspectedObject(value) {
    return stringifyObject(decode(value));
  }

  function createCopyHandler(sourceEl) {
    return () => navigator.clipboard.writeText(sourceEl.value);
  }

  onMount(() => {
    onChangeProps = createConversionHandler(encodeParsedObject, base36El);
    onChangeBase36 = createConversionHandler(decodeInspectedObject, propsEl);
    onCopyProps = createCopyHandler(propsEl);
    onCopyBase36 = createCopyHandler(base36El);
  });
</script>

<style lang="scss">
  div {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 1rem;
    width: 100%;
    height: 100%;
    min-height: 100vh;
    background-color: #ffc100;
  }

  h1 {
    margin: 0;
    text-align: center;
  }

  div > * {
    display: block;
    margin: 0;
    width: 100%;
    max-width: 24rem;
  }

  label {
    margin-top: 2rem;
    padding: 0 0 0 0.125rem;
    font-size: 1.125rem;
    line-height: 1.5;
  }

  span {
    font-weight: bold;
  }

  button {
    float: right;
    margin: 0.125rem 0 0 0;
    border: 0;
    padding: 0 0.5rem;
    background-color: #000;
    color: #fff;
    font-size: 0.75rem;
    line-height: 1.75;
    letter-spacing: 0.05em;
    text-transform: uppercase;
    cursor: pointer;
  }

  textarea {
    border: 0;
    padding: 0.5rem;
    min-height: 16rem;
    background-color: #fff;
    font-size: 1rem;
    font-family: monospace;
    tab-size: 2;
    resize: vertical;
  }

  textarea ~ textarea {
    min-height: 8rem;
  }

  p {
    padding: 0.125rem;
    min-height: 3rem;
    color: red;
    line-height: 1.5;
  }
</style>

<div>
  <h1>Base36 ↔️ Props</h1>
  <label>
    <span>Props</span>
    <button on:click={onCopyProps}>Copy</button>
  </label>
  <textarea bind:this={propsEl} on:input={onChangeProps} />
  <label>
    <span>Base36</span>
    <button on:click={onCopyBase36}>Copy</button>
  </label>
  <textarea bind:this={base36El} on:input={onChangeBase36} />
  <p>{errorMessage || ''}</p>
</div>
