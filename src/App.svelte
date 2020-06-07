<script>

// -----------------------------------------------------------------------------
// Globals
// -----------------------------------------------------------------------------

let busy = false;
let files = [];


// -----------------------------------------------------------------------------
// Reactive statements
// -----------------------------------------------------------------------------


// -----------------------------------------------------------------------------
// 
// -----------------------------------------------------------------------------

function run()
{

}


// -----------------------------------------------------------------------------
// HTML
// -----------------------------------------------------------------------------
</script>

<nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
	<a class="navbar-brand" href="/">Oct4th</a>
	<div class="collapse navbar-collapse">
		<ul class="navbar-nav mr-auto"></ul>
		<ul class="navbar-nav">
			<li class="nav-item active">
				<a class="nav-link" href="https://github.com/robertaboukhalil/oct4th">Code</a>
			</li>
		</ul>
	</div>
</nav>

<main role="main">
	<div class="jumbotron mt-2 pb-1">
		<div class="container">
			<p class="lead">Convert text files to Excel spreadsheets the safe way</p>
			<p>
				Importing CSV/TSV files into Excel can lead to mangled gene names, e.g. the gene <code>OCT4</code> becomes October 4th, while <code>DEC1</code> becomes December 1st.
			</p>
		</div>
	</div>

	<div class="container">
		<div class="row">
			<div class="col-md-4">
				<h4 class="mb-4">Choose Files</h4>

				<p>Choose comma-separated or tab-separated text files from your computer</p>

				<div class="custom-file mb-2">
					<input type="file" class="custom-file-input" id="customFile" bind:files={files} multiple>
					<label class="custom-file-label" for="customFile">Click here to select files</label>
				</div>

				<hr />

				<button type="button" class="btn btn-primary btn-lg" on:click={run} disabled={busy}>
					Convert to XLSX
					{#if busy}
						<span class="spinner-grow spinner-grow-sm mb-1" role="status" aria-hidden="true"></span>
					{/if}
				</button>

				<br /><br />
				<p>🔒 Your files never leave your browser.</p>
			</div>

			<div class="col-md-8">
				<h4 class="mb-4">Output</h4>

				{#each files as file}
					<div class="card">
						<div class="card-header">
							{file.name}
						</div>
						<div class="card-body">
							<h5 class="card-title">Special title treatment</h5>
							<p class="card-text">With supporting text below as a natural lead-in to additional content.</p>
							<a href="#" class="btn btn-primary">Go somewhere</a>
						</div>
					</div>
				{/each}
			</div>
		</div>
	</div>
</main>
