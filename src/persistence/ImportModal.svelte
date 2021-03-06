<script lang="ts">
  import { getContext } from "svelte";
  import Button from "../buttons/Button.svelte";
  import FileInput from "../buttons/FileInput.svelte";
  import { fromMidi } from "musenet-midi";
  import { encodingToArray, encodingToString } from "../state/encoding";
  import type { MusenetEncoding } from "../state/encoding";
  import { toReadableNodeState } from "../state/trackTree";
  import type {
    NodeStore,
    NodeState,
  } from "../state/trackTree";
  import { loadMidi } from "./persistence";
  import colorLookup from "../colors";
  import examples from "./examples";
  import type { Readable } from "svelte/store";
  import type { Section } from "../state/section";
  import toCss from "react-style-object-to-css";

  const { close } = getContext("simple-modal");

  export let importUnderStore: NodeStore;

  let importUnderStoreConverted: Readable<NodeState>;
  $: importUnderStoreConverted = toReadableNodeState(importUnderStore);

  let importUnderState: NodeState;
  $: importUnderState = $importUnderStoreConverted;

  let section: Section | null;
  $: section =
    importUnderState.type === "root" ? null : importUnderState.section;

  let sectionStartsAt: number;
  $: sectionStartsAt = section === null ? 0 : section.startsAt;

  let sectionEndsAt: number;
  $: sectionEndsAt = section === null ? 0 : section.endsAt;

  let encoding: string = "";

  let encodingArray: MusenetEncoding;
  $: encodingArray = encodingToArray(encoding.trim());

  let encodingInvalid: boolean;
  $: encodingInvalid = encodingArray.some(isNaN);

  async function midiSelected(file: File) {
    const encodingArray: MusenetEncoding = await fromMidi(file);
    encoding = encodingToString(encodingArray);
  }

  async function importEncoding() {
    close();
    await loadMidi(
      encodingArray,
      sectionStartsAt,
      sectionEndsAt,
      importUnderStore
    );
  }

  const lighterStyle: JSX.CSSProperties = {
    backgroundColor: colorLookup.bgLight,
    borderColor: colorLookup.border,
    color: colorLookup.text,
  };
</script>

<style>
  h1 {
    margin-top: 0;
  }

  .row {
    display: flex;
    flex-direction: row;
    align-items: center;
  }

  .encoding {
    width: 100%;
    height: 300px;
    scrollbar-color: #c3cee3 #1f292e;
    resize: none;
    margin: 0;
  }

  .encoding::-webkit-scrollbar {
    width: 10px;
  }

  .encoding::-webkit-scrollbar-track {
    background: #1f292e;
  }

  .encoding::-webkit-scrollbar-thumb {
    background-color: #c3cee3;
  }

  .spread {
    flex-grow: 1;
  }
</style>

<!--TODO provide a guide like MrCheeze's-->
<h1 style={toCss({ color: colorLookup.text })}>Import</h1>

<div class="row">
  <span class="spread">What to import:</span>
  <div class="spread">
    <FileInput fileTypes=".mid" handleFile={midiSelected}>
      Upload Midi
    </FileInput>
  </div>
  <div class="spread row">
    <label for="example" style="margin-right: 12px;">Examples:</label>
    <select
      id="example"
      bind:value={encoding}
      style={toCss({ margin: 0, width: 200, ...lighterStyle })}>
      <option selected />
      {#each Object.entries(examples) as [name, exampleEncoding]}
        <option value={exampleEncoding}>{name}</option>
      {/each}
    </select>
  </div>
</div>

<label for="encoding" style="display: none">Encoding</label>
<textarea
  id="encoding"
  class="encoding"
  bind:value={encoding}
  on:drop|preventDefault={(event) => event.dataTransfer && midiSelected(event.dataTransfer.files[0])}
  placeholder="MuseNet Encoding"
  style={toCss({ border: '1px dotted', ...lighterStyle })} />

<Button disabled={encodingInvalid} on:click={importEncoding}>Import</Button>
