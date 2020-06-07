<script>

// -----------------------------------------------------------------------------
// Globals
// -----------------------------------------------------------------------------

let files = [];
let results = {};


// -----------------------------------------------------------------------------
// Reactive statements
// -----------------------------------------------------------------------------

$: for(let file of files)
{
	results[file.name] = {};
	results[file.name].message = `Parsing <code>${file.name}</code>...`;

	// Parse file in a WebWorker
	Papa.parse(file, {
		worker: true,
		fastMode: false,  // quotes in CSV files properly parsed
		dynamicTyping: true,
		skipEmptyLines: true,
		complete: d => {
			// TODO: check for errors
			// if((data.errors && data.errors.length > 0) || data.length == 0) {
			//     dialog.showErrorBox("Skipped File", "Skipping file " + path.basename(f) + " - invalid format");
			//     console.log("WARNING: Skipping file " + path.basename(f) + " - invalid format");
			//     convertFile(allFiles, i+1);
			//     return;
			// }

			results[file.name].message = `Converting to <code>.xlsx</code>...`;
			generateXLSX(file, d.data);
		}
	});
}


// -----------------------------------------------------------------------------
// 
// -----------------------------------------------------------------------------

function generateXLSX(file, data)
{
	let ws_name = "SheetJS";
	let wb = XLSX.utils.book_new();
	let ws = XLSX.utils.aoa_to_sheet(data);
	XLSX.utils.book_append_sheet(wb, ws, ws_name);

	results[file.name].message = "Done";
	results[file.name].wb = wb;
	results[file.name].ready = true;
}

function downloadXLSX(file)
{
	let wb = results[file.name].wb;
	XLSX.writeFile(wb, file.name + ".xlsx", { compression: true });
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
				<div class="custom-file mb-2">
					<input type="file" class="custom-file-input" id="customFile" bind:files={files} multiple>
					<label class="custom-file-label" for="customFile">Click here to select files</label>
				</div>
			
				<p>🔒 Your files never leave your browser.</p>
			</div>

			<div class="col-md-8">
				<h4 class="mb-4">Output</h4>

				{#each files as file, fileID}
					<div class="card">
						<h5 class="card-header">
							{file.name}
							<span class="spinner-grow spinner-grow-sm text-primary mb-1" role="status" aria-hidden="true"></span>
						</h5>
						<div class="card-body">
							<p class="card-text">{results[file.name].message}</p>
							{#if results[file.name].ready}
							<button type="button" on:click={() => downloadXLSX(file)}>Download</button>
							{/if}
						</div>
					</div>
				{/each}
			</div>
		</div>
	</div>
</main>
