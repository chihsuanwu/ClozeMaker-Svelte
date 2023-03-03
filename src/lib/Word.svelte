<script>
    export let data;
    export let answers;
    export let checkAnswer;
    export let numberStart;
    export let underlineLength;
    export let onWordClick;

    function checkAnswerCorrect(data) {
        return answers.find(item => item.id === data.id).number === data.answer;
    }
</script>


{#if data.type === 'word'}
    {#if data.selected >= 0}
        {#if data.answer !== ''}
					<span class="word{checkAnswer ? (checkAnswerCorrect(data) ? ' correct' : ' wrong' ) : ''}" on:click={onWordClick}>{
                        `(${data.selected + numberStart})_${data.answer}${'_'.repeat(underlineLength - 2)}`
                    }</span>
        {:else}
					<span class="word" on:click={onWordClick}>{
                        `(${data.selected + numberStart})${'_'.repeat(underlineLength)}`
                    }</span>
        {/if}
    {:else}
        <span class="word" on:click={onWordClick}>{data.text}</span>
    {/if}
{:else if data.type === 'not_word'}
    <span class="not_word">{data.text}</span>
{:else if data.type === 'br'}
    <br>
{/if}

<style>
    .word {
        display: inline-block;
        font-family: 'Ubuntu', sans-serif;
        font-size: 20px;
        cursor: pointer;
        white-space:pre;
    }

    .word:hover {
        color: #1565c0;
        font-weight: bold;
    }

    .not_word:hover {
        color: #660000;
        font-weight: bold;
    }

    .word.correct {
        color: blue;
    }

    .word.wrong {
        color: red;
    }
</style>