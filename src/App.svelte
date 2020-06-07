<script>
let files = [];
let results = {};
let sampleFile = `
GeneName	SomeValue
TP53	123
OCT4	234
DEC1	345
MARCH1	456
MARCH2	567
MARC2	678
`;


// -----------------------------------------------------------------------------
// Reactive statements
// -----------------------------------------------------------------------------

$: for(let file of files)
{
	file.out = `${file.name.replace(/\.[^/.]+$/, "")}.xlsx`;
	results[file.name] = {};
	results[file.name].message = `Parsing <code>${file.name}</code>...`;

	// Parse file in a WebWorker
	let data = file instanceof File ? file : file.data;
	Papa.parse(data, {
		worker: true,
		fastMode: false,  // quotes in CSV files properly parsed
		dynamicTyping: true,
		skipEmptyLines: true,
		complete: d => {
			// Check for errors
			if((d.errors && d.errors.length > 0) || d.length == 0) {
				results[file.name].error = true;
				results[file.name].message = "Could not parse this file. Make sure it's a comma- or tab-separated file.";
				return;
			}

			results[file.name].message = "Converting to <code>.xlsx</code>...";
			setTimeout(() => generateXLSX(file, d.data), 100);
		}
	});
}


// -----------------------------------------------------------------------------
// Utility functions
// -----------------------------------------------------------------------------

function generateXLSX(file, data)
{
	// Create XLSX file
	let wb = XLSX.utils.book_new();
	let ws = XLSX.utils.aoa_to_sheet(data);
	XLSX.utils.book_append_sheet(wb, ws, "Sheet1");

	results[file.name].message = "Writing to <code>.xlsx</code>... This may take a few minutes...";
	setTimeout(() => writeXLSX(file, wb), 100);
}

function writeXLSX(file, wb)
{
	// Create Blob from XLSX output, then create URL from Blob
	let out = XLSX.write(wb, { compression: true, bookType: "xlsx", type: "binary" });
	let blob = new Blob([ s2ab(out)], { type: "application/octet-stream" });
	let url = URL.createObjectURL(blob);
	results[file.name].url = url;
	results[file.name].ready = true;
}

// Source: https://github.com/SheetJS/js-xlsx/blob/master/README.md
function downloadXLSX(file)
{
	let a = document.createElement("a");
	a.download = file.out;
	a.href = results[file.name].url;
	document.body.appendChild(a);
	a.click();
}

// Source: https://github.com/SheetJS/js-xlsx/blob/master/README.md
function s2ab(s) {
	var buf = new ArrayBuffer(s.length);
	var view = new Uint8Array(buf);
	for (var i=0; i!=s.length; ++i) view[i] = s.charCodeAt(i) & 0xFF;
	return buf;
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

				<p class="text-center mt-2">
					or use
					<button 
						type="button" class="btn btn-link p-0" style="vertical-align: baseline"
						on:click={() => files = [
							{ name: "genes.csv", out: "genes.xlsx", data: sampleFile },
						]}
					>
						<strong>a sample CSV file</strong>
					</button>
				</p>


				<p>🔒 Your files never leave your browser.</p>
			</div>

			<div class="col-md-8">
				<h4 class="mb-4">Output</h4>

				{#each files as file, fileID}
					<div class="card">
						<h5 class="card-header">
							{file.name}
							{#if !results[file.name].ready && !results[file.name].error}
								<span class="spinner-grow spinner-grow-sm text-primary mb-1" role="status" aria-hidden="true"></span>
							{/if}
						</h5>
						<div class="card-body">
							{#if results[file.name].ready}
								<button type="button" class="btn btn-primary" on:click={() => downloadXLSX(file)}>Download {file.out}</button>
							{:else}
								<p class="card-text">{@html results[file.name].message}</p>
							{/if}
						</div>
					</div>
				{/each}
			</div>
		</div>
	</div>
</main>
