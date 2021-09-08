A template for formatting of function definitions to be added to rscripts 

```
#' <FUNCTION NAME>(): brief description 
#'
#' More in depth description in terms of the input and output 
#'
#' @param <argument> description of argument 
#' @param <argument> description of argument 
#' ...
#'
#' @return # format of the returned object
#'
#' @examples # an example of how the function is written out when used 
#'
#' @export # the function  
<FUNCTION NAME> <- function(
   ...
)
#TEST: <FUNCTION NAME>(): # expected output structure 
#TEST: <FUNCTION NAME>():# expected output structure 
#...
# gives: 
# > str(output_object)
```

Example from `read_count_functions.R` of function [GetGeneDatamatrix](https://github.com/riboviz/riboviz/blob/885bbf17b11c2e4da2190c100defd2fd85279d4c/rscripts/read_count_functions.R#L101):

```
#' GetGeneDatamatrix(): Get matrix of read counts  by length for one gene and dataset from .h5 file
#'
#' This accesses the attribute `reads/data` in .h5 file.
#'
#' @param gene gene name to pull out read counts for
#' @param dataset name of dataset stored in .h5 file
#' @param hd_file name of .h5 hdf5 file holding read data for all genes, created from BAM files for dataset samples
#'
#' @return numeric matrix of read count data for given gene in given dataset
#'
#' @examples
#' GetGeneDatamatrix(gene="YAL068C", dataset="vignette", hd_file="vignette/output/WTnone/WTnone.h5")
#'
#' @export
GetGeneDatamatrix <- function(gene, dataset, hd_file){
  rhdf5::h5read(file = hd_file, name = paste0("/", gene, "/", dataset, "/reads/data")) %>%
    return()
}
#TEST: GetGeneDatamatrix(): returns matrix TRUE
```

Example from `read_count_functions.R` of function [AllGenes3EndPositionLengthCountsTibble](https://github.com/riboviz/riboviz/blob/885bbf17b11c2e4da2190c100defd2fd85279d4c/rscripts/read_count_functions.R#L460)

```
#' AllGenes3EndPositionLengthCountsTibble(): Calculate sum of position- and read-length specific total counts over all genes at 3' end
#'
#' @param gene_names vector of gene names to pull out data for (created early in generate_stats_figs.R by line: "gene_names <- rhdf5::h5ls(hd_file, recursive = 1)$name")
#' @param dataset name of dataset stored in .h5 file
#' @param hd_file name of .h5 hdf5 file holding read data for all genes, created from BAM files for dataset samples
#' @param gff_df data.frame or tibble; riboviz-format GFF in tidy data format, as created by readGFFAsDf()
#'
#' @return tibble (tidy data frame) of metagene data (summed across all genes),
#' with three columns: "ReadLen", "Pos", "Counts"
#'
#' @examples
#'  min_read_length <- 10
#'  max_read_length <- 50
#'  n_buffer <- 25
#'  nnt_buffer <- 25
#'  nnt_gene <- 50
#'
#'  gff_df <- readGFFAsDf("vignette/input/yeast_YAL_CDS_w_250utrs.gff3")
#'  gene_names <- rhdf5::h5ls(hd_file="vignette/output/WTnone/WTnone.h5", recursive = 1)$name
#'  AllGenes3EndPositionLengthCountsTibble(
#'    gene_names=gene_names,
#'    dataset="vignette",
#'    hd_file="vignette/output/WTnone/WTnone.h5",
#'    gff_df
#'  )
#'
#' @export
AllGenes3EndPositionLengthCountsTibble <- function(gene_names, dataset, hd_file, gff_df){
  gene_poslen_counts_3end_df <-
    lapply(gene_names,
           function(gene)
             GetGeneDatamatrix3end(
               gene,
               dataset,
               hd_file,
               posn_3end = GetCDS3end(gene, gff_df),
               posn_5start = GetCDS5start(gene,gff_df),
               n_buffer = nnt_buffer,
               nnt_gene = nnt_gene
             )) %>%
    Reduce("+", .) %>% # sums the list of data matrices
    TidyDatamatrix(startpos = -nnt_gene + 1, startlen = min_read_length)

  return(gene_poslen_counts_3end_df)
}
#TEST: AllGenes3EndPositionLengthCountsTibble(): returns tidy format data frame (tibble)
#TEST: AllGenes3EndPositionLengthCountsTibble(): tibble has 3 columns
#TEST: AllGenes3EndPositionLengthCountsTibble(): number of rows of output tibble = nrow(data_mat) * ncol(data_mat)
#TEST: AllGenes3EndPositionLengthCountsTibble(): the column names are %in% c("ReadLen", "Pos", "Counts")
# # gives:
# # > str(gene_poslen_counts_3end_df)
# # Classes ‘tbl_df’, ‘tbl’ and 'data.frame':	3075 obs. of  3 variables:
# #   $ ReadLen: int  10 11 12 13 14 15 16 17 18 19 ...
# #   $ Pos    : int  -49 -49 -49 -49 -49 -49 -49 -49 -49 -49 ...
# #   $ Counts : int  0 0 0 0 0 49 62 27 219 50 ...
```
