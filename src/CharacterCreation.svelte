<script>
  import {playbooks} from "./Playbooks";
  import {Character} from "./character";
  import {Stat} from "./stat";
  import * as shortid from 'shortid/lib/index';
  import {storeCharacter} from "./database";
  import router from "page";

  const statNames = ["strength", "dexterity", "constitution", "intelligence", "wisdom", "charisma"];
  const statValues = [16, 15, 13, 12, 9, 8];

  let form = {
    raceMove: null,
    optionalStartingMove: null
  };
  statNames.forEach((it, i) => form[it] = null);

  $: playbook = form.characterClass == null ? null : playbooks.find(it => it.name === form.characterClass);
  $: playbookDescription = playbook == null ? null : playbook.description || "<p>No description</p>";

  const moveDescription = (name) => {
    return playbook == null ? "" : playbook.moves.find(it => it.name === name).description;
  };

  const randomizeStats = () => {
    let values = statValues.slice();

    for (let i = values.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [values[i], values[j]] = [values[j], values[i]];
    }

    statNames.forEach((it, i) => {
      form[it] = values[i];
    });

    setTimeout(statChange);
  };

  const statChange = () => {
    const selectElements = document.querySelectorAll('.stats select');
    let usedValues = [];
    selectElements.forEach(it => {
      if (it.value !== "null") {
        usedValues.push(it.value);
      }
    });
    selectElements.forEach(it => {
      [...it.options].forEach(option => {
        option.disabled = option.value !== it.value && usedValues.indexOf(option.value) >= 0;
      });
    });
  };

  const create = () => {
    let c = new Character(
      form.characterClass,
      form.name,
      form.strength,
      form.dexterity,
      form.constitution,
      form.intelligence,
      form.wisdom,
      form.charisma
    );
    c.moves.push({name: form.raceMove});
    if (form.optionalStartingMove !== null) {
      c.moves.push({name: form.optionalStartingMove});
    }
    playbook.startingMoves.allOf.forEach(name => {
      c.moves.push({name: name});
    });
    c.id = shortid.generate();

    storeCharacter(c);

    router.redirect(`/character/${c.id}`);
  };

  $: valid = form.name != null && form.name.length > 0 &&
    statNames.every(it => form[it] != null) &&
    form.characterClass != null &&
    form.raceMove != null &&
    (playbook.startingMoves.oneOf === undefined || form.optionalStartingMove != null);

  const capitalize = (string) => string.charAt(0).toUpperCase() + string.slice(1);
</script>

<main class="container">
  <header>
    <h1>Create a Character</h1>
  </header>

  <section class="character-creation">
    <form>
      <fieldset class="playbook">
        <legend>Select a playbook</legend>
        <div class="playbook-list">
          {#each playbooks as playbook}
            <label class="playbook-{playbook.name.toLowerCase().replace(' ', '-')}"
                   class:selected={form.characterClass === playbook.name}>
              <input type="radio" bind:group={form.characterClass} value={playbook.name}>
              <span class="playbook-name">{playbook.name}</span>
            </label>
          {/each}
        </div>
        <div class="playbook-description">
          {@html playbookDescription}
        </div>
      </fieldset>

      <fieldset class="racial-moves">
        <legend>Choose a race</legend>
        {#each (playbook == null ? [] : playbook.startingMoves.raceOptions) as move}
          <div class="move move-select">
            <input type="radio" value={move} bind:group={form.raceMove} class="move-selector">
            <article>
              <h2>{move}</h2>
              {@html moveDescription(move)}
            </article>
          </div>
        {/each}
      </fieldset>

      <fieldset class="character-name">
        <legend>Name your character</legend>
        <label>
          <input type="text" bind:value={form.name}>
        </label>
      </fieldset>

      <fieldset class="stats">
        <legend>Assign their stats</legend>
        {#each statNames as statName}
          <label>
            <span class="name">{capitalize(statName)}</span>
            <select bind:value={form[statName]} on:change={statChange}>
              <option value={null}></option>
              {#each statValues as statValue}
                <option value={statValue}>{statValue}</option>
              {/each}
            </select>
            <span class="stat-bonus"
                  class:bonus-positive={form[statName] != null && form[statName] >= 9}>{form[statName] == null ? "" : new Stat(form[statName]).bonus}</span>
          </label>
        {/each}
        <footer>
          <button type="button" on:click={randomizeStats}>Randomize!</button>
        </footer>
      </fieldset>

      {#if playbook !== null && playbook.startingMoves.oneOf !== undefined}
        <fieldset class="starting-moves">
          <legend>Start with one of&hellip;</legend>
          {#each playbook.startingMoves.oneOf as move}
            <div class="move move-select">
              <input type="radio" value={move} bind:group={form.optionalStartingMove} class="move-selector">
              <article>
                <h2>{move}</h2>
                {@html moveDescription(move)}
              </article>
            </div>
          {/each}
        </fieldset>
      {/if}
      {#if playbook !== null &&  playbook.startingMoves.allOf !== undefined}
        <fieldset class="starting-moves">
          <legend>Start with all of&hellip;</legend>
          {#each playbook.startingMoves.allOf as move}
            <div class="move">
              <h2>{move}</h2>
              {@html moveDescription(move)}
            </div>
          {/each}
        </fieldset>
      {/if}

      <footer>
        <button type="button" disabled={!valid} on:click={create}>Create</button>
      </footer>

    </form>
  </section>
</main>

<style>
  .character-creation {
    @apply col-span-5;
  }

  .character-creation form {
    @apply flex flex-col pt-4;
  }

  .racial-moves {
    @apply mx-auto;
    columns: 2;
    max-width: 75%;
  }

  fieldset {
    @apply border-2 border-gray-500 border-solid mt-0 mb-6 mx-6 p-4 pt-2;
  }

  legend {
    @apply text-xl font-bold px-2;
  }

  label {
    @apply text-xl m-2 cursor-pointer;
  }

  .playbook {
    @apply grid grid-cols-5;
  }

  .playbook-list {
    @apply col-span-3 grid grid-cols-3 col-gap-2 content-start;
  }

  .playbook-list label {
    @apply flex items-center border-2 border-solid border-gray-400 h-12 m-2 px-2;
    break-inside: avoid;
  }

  .playbook-list label.selected {
    @apply border-2 bg-gray-500;
    filter: invert(100%) grayscale(100%);
  }

  .playbook-list input[type=radio] {
    display: none;
  }

  .playbook-name {
    background-size: contain;
    background-repeat: no-repeat;
    padding-left: 2.25rem;
    white-space: nowrap;
  }

  .playbook-the-barbarian .playbook-name {
    background-image: url('/icons/the-barbarian.svg');
  }

  .playbook-the-bard .playbook-name {
    background-image: url('/icons/the-bard.svg');
  }

  .playbook-the-cleric .playbook-name {
    background-image: url('/icons/the-cleric.svg');
  }

  .playbook-the-druid .playbook-name {
    background-image: url('/icons/the-druid.svg');
  }

  .playbook-the-fighter .playbook-name {
    background-image: url('/icons/the-fighter.svg');
  }

  .playbook-the-immolator .playbook-name {
    background-image: url('/icons/the-immolator.svg');
  }

  .playbook-the-mage .playbook-name {
    background-image: url('/icons/the-mage.svg');
  }

  .playbook-the-paladin .playbook-name {
    background-image: url('/icons/the-paladin.svg');
  }

  .playbook-the-priest .playbook-name {
    background-image: url('/icons/the-priest.svg');
  }

  .playbook-the-ranger .playbook-name {
    background-image: url('/icons/the-ranger.svg');
  }

  .playbook-the-templar .playbook-name {
    background-image: url('/icons/the-templar.svg');
  }

  .playbook-the-thief .playbook-name {
    background-image: url('/icons/the-thief.svg');
  }

  .playbook-the-wizard .playbook-name {
    background-image: url('/icons/the-wizard.svg');
  }

  .playbook-description {
    @apply col-span-2 text-lg ml-4;
    columns: 2;
  }

  .character-name {
    @apply mx-auto grid items-center;
  }

  .character-name label {
    @apply flex;
  }

  .character-name input[type=text] {
    @apply flex-grow text-3xl;
  }

  .stats {
    @apply mx-auto grid grid-cols-2 col-gap-2 row-gap-0;
    justify-items: start;
  }

  .stats label {
    @apply grid grid-cols-3 col-gap-2 row-gap-0 items-center text-xl;
    grid-template-columns: 7rem 3rem 1.5rem;
  }

  .stats .name {
    @apply text-right font-bold;
  }

  .stats select {
    @apply text-center;
  }

  .stats footer {
    @apply flex items-center justify-end col-span-2 w-full mt-2;
  }

  .starting-moves {
    @apply mx-auto;
    max-width: 75%;
    columns: 2;
  }

  form > footer {
    @apply flex items-center justify-end col-span-5 h-16 px-6;
  }

  form > footer button {
    @apply text-2xl;
  }

  form > footer button[disabled] {
    @apply text-gray-500;
  }
</style>
