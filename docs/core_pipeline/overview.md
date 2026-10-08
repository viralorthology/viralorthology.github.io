# Overview

The core pipeline is structured into four stages:

## Redundancy Reduction Stage

This stage identifies genomes that are closely related to one another. The pipeline performs a BLASTN search between genomes and selects the most divergent sequences for the first round of analysis. Redundant genomes and their corresponding proteins are later reintegrated into the analysis.

## Initial Stage

The initial stage consists of six steps:

- **Paralog search**: The annotated proteins from each genome are used in reciprocal BLASTP searches and grouped into clusters of paralog sequences. From each cluster, the longest protein is selected for further analysis, while the remaining proteins are set aside for re-evaluation during the final stage of the pipeline.

- **Proteome prediction**: ORFfinder is used to predict the proteome of each genome. When an annotated proteome is available, it is used to refine the predicted proteome by removing sequences that are already present in the annotated proteome.

- **Initial ortholog group creation**: The available annotated proteomes from the non-redundant genomes are used to generate the initial ortholog groups with ProteinOrtho. If an ortholog group contains more than one protein from the same genome, only the longest protein is retained, while the remaining proteins are set aside for re-evaluation during the final stage of the pipeline.

- **Annotation-based merge**: The most common gene annotation within each ortholog group is used to identify groups that can be merged. If two groups have the same most common annotation and are compatible (i.e., they do not contain proteins from the same genomes), they are merged into a single group.

- **HMM-based cleaning**: The profile HMM of each ortholog group is used to search against all annotated proteins. If a protein assigned to an ortholog group is not detected by the corresponding HMM-based search, it is removed from the group.

- **HMM-based merge**: The profile HMM of each ortholog group is compared with the profile HMMs of the other groups. If the HMM comparison score is above the specified threshold and the ortholog groups are compatible (i.e., they do not contain proteins from the same genomes), they are merged into a single group.

## Enrichment Stage

The enrichment stage consists of two iterative steps:

- **HMM-based search**: The profile HMM of each ortholog group is used to search against all proteins that are not yet assigned to an ortholog group. The search is performed in two phases. First, the HMMs are iteratively searched against the annotated proteins until no additional proteins can be assigned. The search then proceeds to the predicted proteins for a single iteration. Once this is completed, the process returns to the annotated proteins and starts a new iteration, continuing until no additional proteins can be assigned.

- **BlastP search**: A BLASTP search is performed using the unassigned proteins as queries and the proteins belonging to ortholog groups as subjects. The search is performed first against the annotated proteins and subsequently against the predicted proteins.

The HMM-based and BLASTP searches are combined to progressively assign previously ungrouped proteins to ortholog groups. The HMM-based search is repeated after each BLASTP search to identify additional proteins that can be assigned based on the newly enriched ortholog groups.

## Redundant Genomes Integration

If redundant genomes were previously identified, they are added to the analysis at this stage. Their paralogs are identified and their proteomes are predicted using the same procedures described in the initial stage. The enrichment stage is then run again to incorporate proteins from the newly added genomes into the ortholog groups.

## Final Stage

The final stage consists of three steps. The HMM-based merge and annotation-based merge steps described above are repeated, with an additional paralogs HMM search step to re-evaluate the proteins that were previously set aside as paralogs:

- **HMM-based merge**: Same as in the initial stage.

- **Paralog HMM-based search**: The profile HMM of each ortholog group is used to search against all previously ignored paralog sequences. If a paralog has a significant hit to an ortholog group that does not contain a protein from the same genome, it is automatically added to the group. If the group already contains a protein from the same genome, a BLASTP search is used to determine which protein has the best match to the other proteins in the group.

- **Annotation-based merge**: Same as in the initial stage.
