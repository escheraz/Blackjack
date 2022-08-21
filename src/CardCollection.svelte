<script lang="ts">
	export let cardsLeft:Array<string> = loadCards();
	export let playerCards:Array<string> = [];
	export let dealerCards:Array<string> = [];
	export let gameState:string = 'IDLE';
	let playerTotal:number = 0;
	let dealerTotal:number = 0;
	let dealerHiddenCard:string = undefined;
	let surrendered:Boolean = false;
	let doubled:Boolean = false;
	let cardLimit:number = 21;
	let cardLimitDealer:number = 17;
	let startingFunds:number = 100;
	let funds:number = 100;
	let betsize:number = 1;
	let blackjackMultiplier:number = 1.5;
	let dealDelay:number = 0;
	let handsPlayed:number = 0;
	let cardsRemaining:number = 0;
	const numDecks:number = 6;

	$: dealerTotal = getCardSum(dealerCards);
	$: playerTotal = getCardSum(playerCards);
	$: cardsRemaining = cardsLeft.length

	function loadCards():Array<string> {
		const cards:Array<string> = [];
		const suits:Array<string> = ['clubs', 'diamonds', 'spades', 'hearts'];
		const ranks:Array<string> = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'ace', 'jack', 'king', 'queen'];
		for (let i = 0; i < numDecks; i++) {
			for (let suit of suits) {
				for (let rank of ranks) {
					cards.push(rank + '_of_' + suit);
				}
			}
			//cards.push('black_joker');
			//cards.push('red_joker');
		}
		return cards;
	}
	function getRandomCard(cards:Array<string>):string {
		const index:number = Math.floor(Math.random() * cards.length);
		const card:string = cards[index];
		cards.splice(index, 1); // remove from cards list
		return card;
	}
	function getCardColor(card:string):string {
		if (card === 'black_joker') return 'black';
		if (card === 'red_joker') return 'red';
		const suit = card.substring(card.lastIndexOf('_') + 1);
		if (suit === 'diamonds' || suit === 'hearts') return 'red';
		if (suit === 'spades' || suit === 'clubs') return 'black';
		return getCardColor(dealerCards[0]);
	}
	function clearCards():void {
		if (Math.random() < 0.05) cardsLeft = loadCards();
	    cardsLeft = cardsLeft;
		playerCards = [];
		dealerCards = [];
		gameState = 'IDLE';
		playerTotal = 0;
		dealerHiddenCard = undefined;
	    surrendered = false;
	    doubled = false;
	}
	function dealCards():void {
		clearCards();
		funds -= betsize;
		playerCards.push(getRandomCard(cardsLeft));
		playerCards.push(getRandomCard(cardsLeft));
		dealerCards.push(getRandomCard(cardsLeft));
		dealerCards.push('card_back');
		dealerHiddenCard = getRandomCard(cardsLeft);
		playerCards = playerCards;
		dealerCards = dealerCards;
		gameState = 'PLAYING';
		if (getCardSum([dealerCards[0], dealerHiddenCard]) == cardLimit)
			flipDealer();
		if (getCardSum(playerCards) == cardLimit)
			flipDealer();
	}
	function dealCardToPlayer():void {
		playerCards.push(getRandomCard(cardsLeft));
		playerCards = playerCards;
		if (getCardSum(playerCards) > cardLimit)
			endGame();
		if (getCardSum(playerCards) == cardLimit && !doubled)
			flipDealer();
	}
	function getCardSum(cards:Array<string>):number {
		let total = cards.map(card => card.substring(0, card.indexOf('_')))
		.map(rank => rank === 'ace' ? 11 : rank === 'jack' || rank === 'queen' || rank === 'king' ? 10 : rank == 'card' ? 0 : Number(rank))
		.reduce((partialSum, a) => partialSum + a, 0);
		let numAces = cards.filter(card => card.substring(0, card.indexOf('_')) === 'ace').length;
		while (total > cardLimit && numAces > 0)
		{
			total -= 10;
			numAces -= 1;
		}
		return total;
	}
	function surrender():void {
		surrendered = true;
		endGame();
	}
	function getWinner():number {
		// 2 = player; 1 = push, 0 = dealer
		if (playerTotal > 21 || (dealerTotal > playerTotal && dealerTotal <= 21))
			return 0;
		else if (dealerTotal > 21 || (dealerTotal < playerTotal && !surrendered))
			return betsize * 2;
		else return betsize;
	}
	function endGame():void {
		dealerTotal = getCardSum(dealerCards);
		playerTotal = getCardSum(playerCards);
		gameState = 'END';
		handsPlayed += 1;
		if (playerTotal == 21 && playerCards.length == 2 && dealerTotal != 21)funds += (1 + blackjackMultiplier) * betsize;
		else if (surrendered) funds += betsize * 0.5;
		else funds += getWinner() * (doubled ? 2 : 1);
	}
	async function flipDealer():Promise<void> {
		gameState = 'GOING_END'
		await new Promise(r => setTimeout(r, dealDelay));
		dealerCards[1] = dealerHiddenCard;
		const player_blackjack = (playerCards.length == 2 && getCardSum(playerCards) == 21)
		while (getCardSum(dealerCards) < cardLimitDealer && !player_blackjack) {
			await new Promise(r => setTimeout(r, dealDelay));
			dealerCards.push(getRandomCard(cardsLeft));
			dealerCards = dealerCards;
		}
		endGame();
	}
</script>

<div>

{#if gameState === "IDLE"}
<h3>Funds</h3> <input type=range bind:value={startingFunds} min=0 max=1000000> 	<input type=number bind:value={startingFunds} min=0 max=1000000>
<h3>Bet Size</h3> <input type=range bind:value={betsize} min=1 max=1000000> 	<input type=number bind:value={betsize} min=0 max=1000000>
<br>
<div class="card">
	<button class="card_btn" on:click={() => {funds = startingFunds; dealCards();}}>
	Start new game!
</button>
</div>
{:else if gameState === "PLAYING"}
<button class="card_btn" on:click={dealCardToPlayer}>
	Hit
</button>
<button class={playerCards.length !== 2 ? "disabled_card_btn" : "card_btn"}
        on:click={() => {doubled = true; funds -= betsize; dealCardToPlayer(); if (gameState !== 'END') flipDealer();}} 
        disabled={playerCards.length !== 2}>
	Double
</button>
<button class="card_btn" on:click={flipDealer}>
	Stand
</button>
<button class="card_btn" on:click={surrender}>
	Surrender
</button>

{:else}
<button class={gameState === "GOING_END" ? "disabled_card_btn" : "card_btn"} on:click={dealCards} disabled={gameState === "GOING_END"}>
	New Deal!
</button>
{/if}
</div>

{#if gameState !== "IDLE"}
	<h2>Total Funds: {funds}</h2>
	{#if handsPlayed > 0}
	<h4>Hands Played: {handsPlayed}; Return per hand: {Math.round((funds - startingFunds) / handsPlayed * 1000 / betsize) / 10 + '% ± ' + Math.round(Math.sqrt(2 / handsPlayed) * 1000) / 10 + '%'}</h4>
	<h4>Cards Remaining: {cardsRemaining}</h4>
	{/if}
	{#if gameState === "END"}
		{#if surrendered}
			<h2>Surrendered!</h2>
		{:else if getWinner() === 0}
			<h2>Dealer Win!</h2>
		{:else if getWinner() === betsize}
			<h2>Push!</h2>
		{:else}
			<h2>Player Win!</h2>
		{/if}
	{/if}
<div class="row">
	<h2>Player Total: {playerTotal} </h2>
	<div class="card-row">
		{#each playerCards as card}
		<img src={'./' + card + '.svg'} alt={card} class=playing_card>
		{/each}

	</div>
	<h2>Dealer Total: {dealerTotal} </h2>
	<div class="card-row">
		{#each dealerCards as card}
		<img src={'./' + card + '.svg'} alt={card} class=playing_card>
		{/each}
	</div>
</div>
{/if}

<style>
	.card_btn {
		border: 2px solid black;
		background: white;
		color: black;
	}
	.disabled_card_btn {
		border: 2px solid black;
		background: white;
		color: grey;
	}
    .playing_card {
    	width: 8em;
    	height: auto;
    	padding: 0.2em 0.2em 0.2em 0.2em;
  	}
</style>
