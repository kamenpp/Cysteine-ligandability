# Cysteine Ligandability

Using code with small modifications from https://github.com/Brian-hongyan/DeepCoSI from the paper by Du et al. "Proteome-Wide Profiling of the Covalent-Druggable Cysteines with a Structure-Based Deep Graph Learning Network" doi: 10.34133/2022/9873564

Human Proteome AF dataset used for creating the binding pockets (environments of cysteines): https://ftp.ebi.ac.uk/pub/databases/alphafold/latest/UP000005640_9606_HUMAN_v4.tar

The file hct116_iwthout_CR.csv can be made from the last cell in analysis_of_compounds.ipynb and comes from the study by Kuljanin et al. https://www.nature.com/articles/s41587-020-00778-3  -> it is just modified to suit the objective of classification https://static-content.springer.com/esm/art%3A10.1038%2Fs41587-020-00778-3/MediaObjects/41587_2020_778_MOESM8_ESM.xlsx

And for the cheminformatics analysis of compounds thea again the data by Kuljanin et al. https://static-content.springer.com/esm/art%3A10.1038%2Fs41587-020-00778-3/MediaObjects/41587_2020_778_MOESM7_ESM.xlsx is used. The data can be accessed even if you don't have access to the article.


## Introduction & Background
Cysteine is one of the most common amino acids in nucleophilic enzymatic catalysis. Therefore, it can be a target for covalently binding (electrophilic) drugs. However, for the sulfur on cysteine to be nucleophilic, it needs to be in a specific environment that will promote the production of the nucleophilic S- intermediate that would carry out the nucleophilic attack. Hence, it would be valuable to have a model that predicts whether a cysteine is nucleophilic and can represent a target for a drug-development campaign (target validation).

As a simple, primary approach a model that looks at the X neighbouring amino acids by residue index. I have implemented two such random forest models (model_benchmark.ipynb) that look at the 9 amino acids before and after the cysteine of interest (one-hot encoding the amino acids in sklearn). However, this ignore bla bla

A more sophisticated method would care about the environment of the cysteine because an amino acid residue hundreds of amino acids later in the chain can be close to and affecting the nucleophilicity of cysteine. Du et al. (https://doi.org/10.34133/2022/9873564) have developed a deep graph learning model Deep Covalent Site Identification (DeepCoSI) which aims to solve that problem. 

A study by Kuljanin et al. (doi:10.1038/s41587-020-00778-3) identifies experimentally the ligandable cysteines in the human proteome via an iodoacetamide probe. Hence, these will represent the positive sample in dataset used for training a model and all the rest of the cysteines in the human proteome will be negative samples. This introduces some false negatives because in some of the proteins in the Kuljanin study might not have been expressed in high enough quantities to be reacted with the probe and then detected by the mass spectrometer.

## What have I done?
I got the human proteome from AlphaFold database (https://ftp.ebi.ac.uk/pub/databases/alphafold/latest/UP000005640_9606_HUMAN_v4.tar) and used that to create the cysteine environments (including all amino acid residues within 10 Å of the cysteine S). Unlike Du et al. who use Chimera UCSF, I used MDAnalysis (https://www.mdanalysis.org) to extract the pocket environment pdb files (I can provide a compressed copy of these on request, ~1.2 GB). An interesting question here would be whether to include only the atoms within these 10 Å or the whole amino acid residues. I opted for the latter as otherwise there would be some atoms disconnect from all of the rest and hence would not contribute to the message passing. This is also what Du et al. seem to have done.

