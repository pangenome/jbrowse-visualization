# JBrowse Pangenome Visualization Guide

This guide outlines the process for visualizing pangenome graphs using JBrowse, specifically tailored for bioinformaticians interested in genomic research. The workflow leverages several tools and plugins, facilitating an interactive exploration of pangenomes.

## Prerequisites

Ensure the following tools and resources are available:

- **JBrowse**: [JBrowse components](https://github.com/GMOD/jbrowse-components/)
- **JBrowse MAF Plugin**: [MAFViewer](https://github.com/cmdcolin/jbrowse-plugin-mafviewer)
- **WGA Tools**: [wgatools](https://github.com/wjwei-handsome/wgatools)
- **MAFChunk**: [mafchunk](https://github.com/pangenome/mafchunk)
- **maf2bed**: Install via Cargo with `cargo install maf2bed`

## Workflow

### 1. Building the Graph with `pggb`

Generate the graph using `pggb` within a Docker container:

```bash
docker run -it -v .:/data ghcr.io/pangenome/pggb:latest pggb -i data/<SEQUENCES.fa> -M -o data/OUTPUTDIR -n 2 -t 15 -p 90 -s 5000
```

- Replace `<SEQUENCES.fa>` with your assembly file names, formatted as `ASSEMBLYNAME.CHROMOSOME` (e.g., `hg38.chr1`, `chm13.chr12`).

### 2. Generating MAF Files with `wgatools`

Convert PAF files to MAF format for visualization:

```bash
wgatools pafpseudo -f <FASTA_FILE> -g <TARGET> -o <OUTPUT.maf> <INPUT.paf>
```

- `<FASTA_FILE>`: File containing sequences from the pggb graph build.
- `<TARGET>`: Reference assembly name (leave empty for multiple .maf outputs).
- `<OUTPUT.maf>`: Resulting MAF file.
- `<INPUT.paf>`: PAF file from pggb graph construction.

### 3. Chunking MAF Files

Split the MAF file into manageable chunks for visualization:

```bash
mafchunk <OUTPUT.maf> 100000 > <CHUNKED.maf>
```

- `100000` is recommended for interactive visualization, adjustable based on chromosome size.

### 4. Creating Pseudo-BED for MAFViewer

Generate a pseudo-BED file from the chunked MAF:

```bash
cat <CHUNKED.maf> | maf2bed <TARGET> | bgzip > out.bed.gz
```

### 5. Indexing the Pseudo-BED

Index the generated BED file:

```bash
tabix -p bed out.bed.gz
```

## Launching JBrowse

1. **Start JBrowse** (either desktop or web version):

   For the desktop version, initiate the webpack dev server and electron window:

   ```bash
   cd products/jbrowse-desktop
   yarn start
   yarn electron
   ```

2. **Install the MAF Plugin** from the Plugin Store within JBrowse if not already done.

3. **Add a MAF Track** using the `out.bed.gz` and `out.bed.gz.tbi` files, along with the assembly names from your `<FASTA_FILE>`.

You can now add additional tracks for your reference assembly, facilitating a comprehensive pangenome analysis.

## Result Preview

The visualization process culminates in an interactive pangenome view, akin to this example:

![Initial pangenome view](pg1.png)

# Synteny View Visualization

Visualizing synteny becomes straightforward with JBrowse's capability to directly use the `.paf` files produced by `pggb`. Follow these steps for an insightful synteny analysis:

## 1. Adding Assemblies to JBrowse

Navigate to File -> Open Assembly in JBrowse and add the assemblies involved in the alignment.

## 2. Creating a Dotplot

To initiate, open a dotplot comparing two assemblies. This involves creating a new synteny track and incorporating the `.paf` file from `pggb`. The resulting dotplot provides a visual representation of synteny between the assemblies:

![Dotplot view](pg3.png)

### Troubleshooting Dotplots

- An empty dotplot usually indicates a mismatch in assembly naming conventions (e.g., using `hg38.chr21` versus simply `chr21`). Ensure consistency in naming within the FASTA files.
- The order of assembly comparison might affect the output. For optimal results, arrange the dotplot with the assembly listed first in the `.paf` file as the primary assembly.

## 3. Generating Linear Synteny Views

Following the dotplot visualization, you can select specific regions to create linear synteny views. Additionally, you may add tracks to enhance the comparison between the assemblies. Such linear views can reveal structural variations like insertions:

![Insertion view](pg2.png)

This step-by-step approach simplifies the visualization of genomic synteny, enabling a deeper understanding of structural variations and alignments.



## Todo

There are several limitations, and tools like
[Waragraph](https://github.com/chfi/waragraph) are better suited for
interactive visualization of genome structure, but not necessarily of
function.

### Linear view

Waragraph and `odgi layout` provides 1D visualizations that look like
this:
![Waragraph plot](pg4.png "Waragraph plot")

The main issue with this visualization is that it's hard to project
functional information on functional information would relate mainly
to a single track. The MAFViewer mainly replaces this
functionality.
It would be nice to have:
- ability to add copy numbers as colors
- easily switch between the references
- add features to all the assemblies in the pangenome; for JBrowse,
  they need to be put in a merged gff3
- visualize the [PanGene](https://github.com/lh3/pangene) annotations
  of GFA like in ![Pangene](p5.png)
  - for this, it would be great to write a GFA Adapter for JBrowse to
    directly interact with the GFA (or a derived format)
  - we can then visualize the paths corresponding to the genes or
    other functional features directly
- 2D view of genomes as from Waragraph: ![2D View](pg5.png)
  - but how to relate this view to the rest of the information in
    JBrowse is still an open question
