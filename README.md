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
I got the human proteome from AlphaFold database (https://ftp.ebi.ac.uk/pub/databases/alphafold/latest/UP000005640_9606_HUMAN_v4.tar) and used that to create the cysteine environments (including all amino acid residues within 10 Å of the cysteine S). Unlike Du et al. who use Chimera UCSF, I used MDAnalysis (https://www.mdanalysis.org) to extract the pocket environment pdb files (I can provide a compressed copy of these on request, ~1.2 GB). An interesting question here would be whether to include only the atoms within these 10 Å or the whole amino acid residues. I opted for the latter as otherwise there would be some atoms disconnect from all of the rest and hence would not contribute to the message passing. This is also what Du et al. seem to have done. This is in preparing_pdb_pockets.ipynb.  Note: there is inconsistency between the indexing of the AF protein residues and the ones provided by Kuljanin in the HCT116 dataset for some of the structures, hence ~4700 (out of all ~23000) cysteines by Kuljanin didn't match with thos in the AF dataset and were marked as negative. Hence 18361 positive (8%) and 209 913 (92%) negative. This leads to the introduction of more errors. However, since as you will see further down, locally I can only train a toy-model, hence the extra error is not very significant.

Then, I went on to create the graphs (and the dataset). For that, I used mostly code from Du et al. for the featurisation of nodes (atoms) and edges (bonds). I added a check for whether the cysteine sulfurs are in a disulfide bridge, and if they are, I just exclude them from the dataset because they are obviously negative (already bound, not nucleophilic, don't need a model to tell me that). This also decreased the class imbalance slightly from about 8% positive to about 10.5% positive (this is in create_graphs.ipynb and then create_dataset_and_train_model.ipynb). The code went through 6517 pockets, of which 4135 train, 431 validation, 498 test (1453 in disulfide bridges) (530 positive in total out of 6517, indeed 8% so representative of the whole set), after a stratified split to account for the class imbalance (create_dataset_and_train_model.ipynb conatains a lot of draft work and shows that the random split actually underperforms compared to the stratified, for example the validation loss was lower than the training loss and the loss function was weighted in slight favour of the validation set). The create_dataset_and_train_model.ipynb is uploaded only to document the way in which the trained model was produced, I am not claiming that this is the good scalable way to approach this, but I found training the model taking very long (about 5 min/epoch) and so I had already spent days trying to get some meaningful model predictions out of this small dataset (~3% of the whole data).

The in models_benchmarking I went on to compare two RF, one random prediction, one only negative prediction and the trained DeepCoSI model on the same dataset splitting. Even though all the models don't perform well, it seems that DeepCoSI has learnt more than the others based on most of the metrics.
