<script>
	let alfabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';

	let metode = $state('caesar');
	let tekst = $state('');
	let nokkel = $state('');
	let resultat = $state('');
	let feilmelding = $state('');
	let genNokkel = $state(''); // nøkkelen som lages automatisk i One-Time Pad

	// Caesar: flytter hver bokstav "nokkel" plasser fram i alfabetet
	function caesar(tekst, nokkel) {
		tekst = tekst.toUpperCase();
		let svar = '';
		for (let i = 0; i < tekst.length; i++) {
			let bokstav = tekst[i];
			if (bokstav == ' ') {
				svar += ' '; // mellomrom beholdes
			} else {
				let tall = alfabet.indexOf(bokstav); // A=0, B=1 osv
				let nyttTall = (tall + nokkel) % 26; // % 26 gjør at Z går rundt til A
				svar += alfabet[nyttTall];
			}
		}
		return svar;
	}

	// Vigenère: samme som Caesar, men hver bokstav har sin egen nøkkelbokstav.
	// Hvis teksten er lengre enn nøkkelen gjentas nøkkelen.
	function vigenere(tekst, nokkel, skalKryptere) {
		tekst = tekst.toUpperCase();
		nokkel = nokkel.toUpperCase();
		let svar = '';
		let nokkelPlass = 0; // teller bare bokstaver, ikke mellomrom
		for (let i = 0; i < tekst.length; i++) {
			let bokstav = tekst[i];
			if (bokstav == ' ') {
				svar += ' '; // mellomrom beholdes og bruker ikke opp en nøkkelbokstav
			} else {
				let tekstTall = alfabet.indexOf(bokstav);
				// % nokkel.length gjør at nøkkelen starter på nytt når den er brukt opp
				let nokkelBokstav = nokkel[nokkelPlass % nokkel.length];
				let nokkelTall = alfabet.indexOf(nokkelBokstav);
				let nyttTall;
				if (skalKryptere) {
					nyttTall = (tekstTall + nokkelTall) % 26;
				} else {
					nyttTall = (tekstTall - nokkelTall + 26) % 26; // + 26 så det ikke blir negativt
				}
				svar += alfabet[nyttTall];
				nokkelPlass++;
			}
		}
		return svar;
	}

	// teller bokstaver (mellomrom telles ikke)
	function tellBokstaver(tekst) {
		return tekst.replaceAll(' ', '').length;
	}

	// Lager en tilfeldig nøkkel, én bokstav for hver bokstav i teksten.
	// OBS: Math.random() er ikke trygt nok til ekte kryptografi, det brukes bare for å vise hvordan det virker.
	function lagTilfeldigNokkel(tekst) {
		let nokkel = '';
		for (let i = 0; i < tellBokstaver(tekst); i++) {
			let tilfeldigTall = Math.floor(Math.random() * 26); // 0 til 25
			nokkel += alfabet[tilfeldigTall];
		}
		return nokkel;
	}

	// returnerer feilmelding, eller tom tekst hvis alt er ok
	function sjekkTekst() {
		if (tekst.trim() == '') {
			return 'Du må skrive inn en tekst.';
		}
		for (let i = 0; i < tekst.length; i++) {
			let tegn = tekst[i].toUpperCase();
			if (tegn != ' ' && !alfabet.includes(tegn)) {
				return 'Teksten kan bare ha bokstaver (A-Z) og mellomrom. Tall og spesialtegn er ikke lov.';
			}
		}
		return '';
	}

	function sjekkNokkel() {
		if (nokkel.trim() == '') {
			return 'Du må skrive inn en nøkkel.';
		}
		if (metode == 'caesar') {
			// bare siffer, så vi stopper f.eks. 1.5 og -3
			if (!/^[0-9]+$/.test(nokkel.trim()) || Number(nokkel) > 25) {
				return 'Nøkkelen må være et heltall fra 0 til 25.';
			}
		} else {
			// Vigenère og One-Time Pad: bare bokstaver, ingen tall, mellomrom eller spesialtegn
			if (!/^[A-Za-z]+$/.test(nokkel)) {
				return 'Nøkkelen kan bare ha bokstaver fra A til Z.';
			}
		}
		// One-Time Pad: nøkkelen må være like lang som teksten (uten mellomrom)
		if (metode == 'otp' && nokkel.length != tellBokstaver(tekst)) {
			return (
				'Nøkkelen har feil lengde. Den har ' + nokkel.length +
				' bokstaver, men teksten har ' + tellBokstaver(tekst) +
				' bokstaver (mellomrom telles ikke).'
			);
		}
		return '';
	}

	// tømmer feilmelding og resultat når man bytter metode
	function byttMetode() {
		resultat = '';
		feilmelding = '';
		genNokkel = '';
	}

	function krypter() {
		resultat = '';
		genNokkel = '';
		feilmelding = sjekkTekst();
		if (feilmelding != '') return;

		// One-Time Pad lager nøkkelen selv, så her trengs ikke sjekkNokkel
		if (metode == 'otp') {
			genNokkel = lagTilfeldigNokkel(tekst);
			resultat = vigenere(tekst, genNokkel, true);
			return;
		}

		feilmelding = sjekkNokkel();
		if (feilmelding != '') return;

		if (metode == 'caesar') {
			resultat = caesar(tekst, Number(nokkel));
		} else {
			resultat = vigenere(tekst, nokkel, true);
		}
	}

	function dekrypter() {
		resultat = '';
		genNokkel = '';
		feilmelding = sjekkTekst();
		if (feilmelding != '') return;
		feilmelding = sjekkNokkel();
		if (feilmelding != '') return;

		// dekryptering = å flytte like langt, men andre veien (26 - nøkkel)
		if (metode == 'caesar') {
			resultat = caesar(tekst, 26 - Number(nokkel));
		} else {
			// vigenere og otp bruker samme utregning
			resultat = vigenere(tekst, nokkel, false);
		}
	}
</script>

<h1>Kryptografiverktøy</h1>

<p>Velg metode:</p>
<label><input type="radio" bind:group={metode} value="caesar" onchange={byttMetode} /> Caesar</label>
<label><input type="radio" bind:group={metode} value="vigenere" onchange={byttMetode} /> Vigenère</label>
<label><input type="radio" bind:group={metode} value="otp" onchange={byttMetode} /> One-Time Pad</label>

<p>Tekst:</p>
<textarea bind:value={tekst} rows="4" cols="50"></textarea>

{#if metode == 'caesar'}
	<p>Nøkkel (tall 0-25):</p>
{:else if metode == 'vigenere'}
	<p>Nøkkel (bokstaver A-Z):</p>
{:else}
	<p>Nøkkel (bare ved dekryptering, like mange bokstaver som teksten):</p>
{/if}
<input type="text" bind:value={nokkel} />

<br /><br />
<button onclick={krypter}>Krypter</button>
<button onclick={dekrypter}>Dekrypter</button>

{#if feilmelding != ''}
	<p class="feil">{feilmelding}</p>
{/if}

{#if resultat != ''}
	<p>Resultat:</p>
	<p class="resultat">{resultat}</p>
{/if}

{#if genNokkel != ''}
	<p>Nøkkel som ble laget (ta vare på denne, du trenger den for å dekryptere):</p>
	<p class="resultat">{genNokkel}</p>
	<p><small>Math.random() brukes bare for å vise hvordan det virker, det er ikke trygt nok til ekte kryptografi.</small></p>
{/if}

<style>
	:global(body) {
		font-family: Arial, sans-serif;
		max-width: 600px;
		margin: 20px auto;
		padding: 0 10px;
	}
	.feil {
		color: red;
	}
	.resultat {
		font-family: monospace;
		font-size: 18px;
		background-color: #eee;
		padding: 8px;
	}
</style>
