<script>
  import { Router, Route, navigate } from 'svelte-routing';
  import CrosswordPage from './crossword/+page.svelte';
  import WordsearchPage from './wordsearch/+page.svelte';
  import wheeloffortune from './wheeloffortune/+page.svelte';
  import Play from './wordsearch/Play.svelte';
</script>

<div class="position">
  <button on:click={() => navigate('/puzzle-maker/crossword')}> Go to Crossword </button>
  <button on:click={() => navigate('/puzzle-maker/wordsearch')}> Go to wordsearch </button>
  <button on:click={() => navigate('/puzzle-maker/wheeloffortune')}> Go to wordsearch </button>
</div>

<Router>
  <Route path="/puzzle-maker/crossword" component={CrosswordPage} />
  <Route path="/puzzle-maker/wordsearch" component={WordsearchPage} />
  <Route path="/puzzle-maker/wordsearch/play" component={Play} />
  <Route path="/puzzle-maker/wheeloffortune" component={wheeloffortune} />
</Router>

<style>
    .position {
      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      gap: 16px;
    }
  
    button {
      padding: 12px 24px;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      transition: background-color 0.3s ease;
    }
  
    button:hover {
      background-color: #0056b3;
    }
</style>
