<script>
	import Word from './lib/Word.svelte';

	let content = "";

	let mainContentData = [];

	let answers = [];
	let clickedAnswer = {};

	let answerMode = false;
	let checkAnswer = false;

	let numberStart = 1;
	let underlineLength = 4;

	const onTextChanged = () => {
		answerMode = false;
		clickedAnswer = {};

		let words = content.split(" ");
		let words_temp = [];

		let count = 0;
		for (let i = 0; i < words.length; ++i) {
			words_temp[i + count] = words[i];
			words_temp[i + count + 1] = " ";
			count += 1;
		}
		words = words_temp;

		count = 0; words_temp = [];
		for (let i = 0; i < words.length; ++i) {
			words_temp[i + count] = '';

			if (words[i] === ' ') {
				words_temp[i + count] = words[i];
				continue;
			}

			for (let j = 0; j < words[i].length; ++j) {
				const charCode = words[i].charCodeAt(j);

				if ((charCode >= 65 && charCode <= 90) || (charCode >= 97 && charCode <= 122)) {
					words_temp[i + count] += words[i][j];
					continue;
				}

				if (j === words[i].length - 1) {     // Char is at the end of the string.
					count += 1;
					words_temp[i + count] = words[i][j];
				} else if (words_temp[i + count] !== '' && j !== 0) {     // Char is at the middle of the string.
					count += 1;
					words_temp[i + count] = words[i][j];
					count += 1;
					words_temp[i + count] = '';
				} else {       // Char is at the beginning of the string.
					words_temp[i + count] = words[i][j];
					count += 1;
					words_temp[i + count] = '';
				}
			}
		}

		let newData = [];

		words_temp.forEach((word, index) => {
			if (word.includes("\n\n")) {
				let temp = 0

				for (let j = 0; j < word.length; ++j) {
					if (word.charAt(j) === "\n") {
						temp += 1;
					}
				}
				for (let j = 0; j < temp; ++j) {
					newData.push({type: 'br'});
				}
			} else if (word.includes(" ") ||
					word.includes(".") || word.includes(",") ||
					word.includes("!") || word.includes("?") ||
					word.includes(";") || word.includes(":") ||
					word.includes("“") || word.includes("”") ||
					word.includes("'") || word.includes('"') ||
					word.includes("—") || word.includes("-") ||
					word.includes("(") || word.includes(")") ||
					word.includes("[") || word.includes("]") ||
					word.includes("{") || word.includes('}')) {
				newData.push({type: 'not_word', id: index, text: word});
			} else if (word.includes("\n")) {
				newData.push({type: 'br'});
			} else {
				newData.push({type: 'word', id: index, text: word, selected: -1, answer: ''});
			}
		});

		newData.push({type: 'br'});
		mainContentData = newData;
	};

	const onWordClick = id => {
		if (!answerMode) {
			const data = mainContentData.find(data => data.id === id);
			if (data.selected < 0) {
				data.selected = answers.length;
			} else {
				data.selected = -1;
			}

			let selected = mainContentData.filter(data => data.selected >= 0)
			selected.forEach((data, index) => data.selected = index);
			answers = selected.sort((a, b) => a.text.localeCompare(b.text))
					.map((data, index) => {
						return {id: data.id, number: String.fromCharCode(65 + index), text: data.text};
					});

			mainContentData = mainContentData // trigger update
		} else if (clickedAnswer !== {}) {
			const data = mainContentData.find(data => data.id === id);
			if (data.selected >= 0) {
				data.answer = clickedAnswer.number;
				clickedAnswer = {};
			}
		}
	};

	const onAnswerClick = answer => {
		if (!answerMode) return;

		clickedAnswer = answer === clickedAnswer ? {} : answer;
	};

	const onCopyClick = () => {
		let copy_str = '';

		for (const data of mainContentData) {
			if (data.type === 'word') {
				if (data.selected >= 0) {
					copy_str += `(${data.selected + numberStart})${'_'.repeat(underlineLength)}`;
				} else {
					copy_str += data.text;
				}
			} else if (data.type === 'not_word') {
				copy_str += data.text;
			} else {
				copy_str += "\n";
			}
		}

		copy_str += '\n\n';

		for (const ans of answers) {
			copy_str += ans.number + '. ' + ans.text + '\n';
		}

		copyToClipboard(copy_str);
	};

	function copyToClipboard(str) {
		const element = document.createElement('textarea');
		element.value = str;
		document.body.appendChild(element);
		element.select();
		document.execCommand('copy');
		document.body.removeChild(element);
	}
</script>


<h1>CLOZE MAKER</h1>
<div id="input_container">
	<textarea id="input" placeholder="Paste paragraph here..." bind:value={content} on:input={onTextChanged} ></textarea>

	<button id="btn_clear" on:click={_ => content = ''}>Clear</button>
</div>

<div id="main_container">
	<h3>Click the word to blank it.
		<span style="padding-left: 800px; font-family: Arial">Answer Mode</span>
		<input type="checkbox" bind:checked={answerMode}>

		{#if answerMode}
			<span style="padding-left: 50px; font-family: Arial">Check Answer</span>
			<input type="checkbox" bind:checked={checkAnswer}>
		{/if}
	</h3>

	<div id="main">
		{#each mainContentData as data}
			<Word data={data}
				  answers={answers}
				  checkAnswer={checkAnswer}
				  numberStart={numberStart}
				  underlineLength={underlineLength}
				  onWordClick={_ => onWordClick(data.id)}
			/>
		{/each}
	</div>
	{#if !answerMode}
		<div id="tool" style="padding: 8px 4px 4px 4px;">
			<span>Question number start from:</span>
			<input class="guide_of_set" type="number" bind:value={numberStart} min="1" max="100">

			<span style="padding-left: 48px;">Length of question's underline:</span>
			<input class="guide_of_set" type="number" bind:value={underlineLength} min="3" max="10">

			<span style="padding-left: 48px;">Solution</span>
			<input class="guide_of_set" style="vertical-align: middle;" type="checkbox" id="show_answer" />
		</div>
	{/if}
</div>

<div id="answer_container">
	<h3>Here is the answer.</h3>
	<div id="answer">
		{#each answers as ans, i}
			<span class="answer{clickedAnswer === ans ? ' clicked' : ''}" on:click={onAnswerClick(ans)}>{
					`(${ans.number})${ans.text} `
			}</span>
		{/each}
	</div>

	<button id="btn_copy" on:click={onCopyClick}>Copy to clipboard</button>
</div>

<style>
	.answer.clicked {
		color: purple;
	}
</style>
