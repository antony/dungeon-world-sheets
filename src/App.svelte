<script>
  import Tailwindcss from './Tailwindcss.svelte';

  import {character} from "./store";
  import CharacterCreation from "./CharacterCreation.svelte";
  import CharacterSheet from "./CharacterSheet.svelte";
  import CharacterList from "./CharacterList.svelte";
  import {loadCharacter, setupDatabase, storeCharacter} from "./database";
  import router from "page";

  // routing setup
  let page;
  router('/create', () => {
    page = CharacterCreation;
    character.set(null);
  });
  router('/character/:id', (context) => {
    const id = context.params.id;
    loadCharacter(id)
      .then((c) => {
        character.set(c);
        page = CharacterSheet;
      })
      .catch((error) => {
        // TODO: implement 404 page
        console.warn(`character ${id} not found`);
        router.redirect('/');
        character.set(null);
      });
  });
  router('/', () => {
    page = CharacterList;
    character.set(null);
  });
  router.start();

  // update the character in the database when it changes
  setupDatabase()
    .then(() => {
      character.subscribe(c => {
        if (c != null) {
          storeCharacter(c);
        }
      });
    });

</script>

<Tailwindcss/>

<svelte:component this={page}/>

<style global>
  * {
    font-family: 'IM Fell DW Pica', sans-serif;
  }

  h1 {
    font-family: 'IM Fell DW Pica SC', sans-serif;
  }

  header {
    @apply flex mb-2 items-center bg-gray-800 text-white h-12;
  }

  header > h1 {
    @apply flex-grow leading-none text-2xl;
  }

  header > button {
    @apply flex-initial text-white mx-2;
  }

  header button[disabled] {
    @apply text-gray-500;
  }

  section header > h1 {
    @apply mx-2;
  }

  main.container {
    @apply grid grid-cols-5 bg-gray-200 my-4 pb-1;
  }

  main > header {
    @apply col-span-5 mb-6;
  }

  main > header > h1 {
    @apply text-4xl mx-4;
  }

  main > section {
    @apply m-2 bg-white border-solid border-2 border-gray-800 ;
  }

  section > ul,
  section > table,
  section > footer,
  section > h2,
  section > p {
    @apply m-2;
  }

  li {
    @apply my-2;
  }

  main input {
    @apply px-1;
  }

  input[type=text],
  input[type=number] {
    @apply border-b-2 border-solid border-gray-400 bg-gray-100;
  }

  button[disabled] {
    @apply text-gray-500;
  }

  .bonus-positive {
    /* this is stupid but rollup (or something) discards the ::before rule if there's no rule for the base selector */
    cursor: inherit;
  }

  .bonus-positive::before {
    content: '+';
  }

  .playbook-description p {
    @apply mb-3;
  }

  /* move rendering */
  .move .move-options li {
    @apply list-none;
  }

  .move .move-options li::before {
    @apply text-lg leading-none;
    content: '\2610\0a';
  }

  .move .move-options-inline {
    @apply inline;
  }

  .move .move-options-inline li {
    @apply inline mr-2;
  }

  .move .move-options li.move-option-custom-value::before,
  .move .move-options li.move-option-interactive::before {
    content: none;
  }

  .move .move-options li.move-option-custom-value::after {
    content: '\0a____';
  }

  .move .move-options li.move-option-custom-value.move-option-interactive::after {
    content: none;
  }

  .move h3 {
    @apply mt-2;
    margin-left: 0 !important; /* TODO: only needed because of conflicting rule in Move.svelte */
  }
</style>