# Cysteine Ligandability

Using code with small modifications from https://github.com/Brian-hongyan/DeepCoSI from the paper by Du et al. "Proteome-Wide Profiling of the Covalent-Druggable Cysteines with a Structure-Based Deep Graph Learning Network" doi: 10.34133/2022/9873564

Human Proteome AF dataset used for creating the binding pockets (environments of cysteines): https://ftp.ebi.ac.uk/pub/databases/alphafold/latest/UP000005640_9606_HUMAN_v4.tar

The file hct116_iwthout_CR.csv can be made from the last cell in analysis_of_compounds.ipynb and comes from the study by Kuljanin et al. https://www.nature.com/articles/s41587-020-00778-3  -> it is just modified to suit the objective of classification https://static-content.springer.com/esm/art%3A10.1038%2Fs41587-020-00778-3/MediaObjects/41587_2020_778_MOESM8_ESM.xlsx

And for the cheminformatics analysis of compounds thea again the data by Kuljanin et al. https://static-content.springer.com/esm/art%3A10.1038%2Fs41587-020-00778-3/MediaObjects/41587_2020_778_MOESM7_ESM.xlsx is used. The data can be accessed even if you don't have access to the article.


## Introduction & Background
Cysteine is one of the most common amino acids in nucleophilic enzymatic catalysis. Therefore, it can be a target for covalently binding (electrophilic) drugs. However, for the sulfur on cysteine to be nucleophilic, it needs to be in a specific environment that will promote the production of the nucleophilic S- intermediate that would carry out the nucleophilic attack. Hence, it would be valuable to have a model that predicts whether a cysteine is nucleophilic and can represent a target for a drug-development campaign (target validation).

As a simple, primary approach a model that looks at the X neighbouring amino acids by residue index. I have implemented two such random forest models (model_benchmark.ipynb) that look at the 9 amino acids before and after the cysteine of interest (one-hot encoding the amino acids in sklearn).

A more sophisticated method would care about the environment of the cysteine because an amino acid residue hundreds of amino acids later in the chain can be close to and affecting the nucleophilicity of cysteine. Du et al. (https://doi.org/10.34133/2022/9873564) have developed a deep graph learning model Deep Covalent Site Identification (DeepCoSI) which aims to solve that problem. 

