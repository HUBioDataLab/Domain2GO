# Mutual Annotation-Based Prediction of Protein Domain Functions with Domain2GO

<!-- omit in toc -->

[![publication](https://img.shields.io/badge/DOI-10.1002/pro.4988-b31b1b.svg)](https://doi.org/10.1002/pro.4988) [![tool](https://img.shields.io/badge/tool-x.svg?logo=streamlit&logoColor=black&color=ffd21c)](https://huggingface.co/spaces/HUBioDataLab/Domain2GO) [![license](https://img.shields.io/badge/license-GPLv3-blue.svg)](http://www.gnu.org/licenses/)

Identifying the functions of known proteins is an important task in understanding their critical roles in biological processes. Although high throughput sequencing tools seem to make this task easier, experimental function identification techniques cannot keep up with the increasing amount of sequence data due to their expensive and time-consuming nature. Continuously expanding output of the high throughput sequencing tools is stored in public databases such as the UniProt Knowledgebase (UniProtKB) and the majority of it (currently ~%99, November 2021) is not experimentally annotated yet.  
<br/>
Here, we propose a new method called Domain2GO with the aim of identifying unknown protein functions by associating domains with Gene Ontology terms, thus defining the problem as domain function prediction. Domain2GO mappings are generated using the existing domain and GO annotation data. In order to obtain highly reliable associations, we employed statistical resampling and analyzed the co-occurrence patterns of domains and GO terms on the same proteins. Furthermore, three different association probability measures were calculated by the expectation-maximization (EM) algorithm and an ablation study was performed to compare the predictive performance of the different versions of the proposed method. As a use-case study, examples selected from the finalized mappings are examined via literature review, to assess their biological relevance. We then applied the proposed method to predict protein functions, by propagating domain-associated GO terms to proteins that are annotated with those domains. For protein function prediction performance evaluation and comparison against other methods, we employed CAFA3 challenge datasets. The results demonstrated the potential of Domain2GO, especially when predicting molecular function and biological process terms, as it performed better than baseline predictors and curated associations (Fmax = 0.48 and 0.36 for MFO and BPO, respectively). The approach proposed here can be extended to other ontologies, biological entities, and their features to explore unknown relationships in complex and large-scale biological data.  
<br/>
The study is summarized in the schematic workflow below.


| ![alt text for screen readers](/figures/full_methodology.png "Schematic workflow of the Domain2GO") | 
|:--:| 
| **Schematic representation of the proposed method. (A)** The source datasets were downloaded and organized; **(B)** initial mapping between the InterPro domains and GO terms were obtained, and the mapping parameters were calculated; **(C)** generation of the randomized annotation and mapping sets were constructed; **(D)** co-occurrence similarity distributions were plotted, and thresholds were selected based on statistical resampling; **(E)** an ablation study was conducted by calculating the enrichment of top predictions ranked by different statistical measures and finalized Domain2GO mappings were generated by filtering initial mappings; **(F)** protein function predictions were generated by propagating Domain2GO mappings to target proteins. |
<br/>

## Content <!-- omit in toc -->

- [Mutual Annotation-Based Prediction of Protein Domain Functions with Domain2GO](#mutual-annotation-based-prediction-of-protein-domain-functions-with-domain2go)
  - [Installation](#installation)
  - [Descriptions of Folders and Files in the Domain2GO Repository](#descriptions-of-folders-and-files-in-the-domain2go-repository)
  - [Downloading the Finalized Domain and Protein Function Prediction Datasets](#downloading-the-finalized-domain-and-protein-function-prediction-datasets)
  - [Predicting the Functions of a New Protein Sequence Using Domain2GO Mappings](#predicting-the-functions-of-a-new-protein-sequence-using-domain2go-mappings)
    - [User Interface](#user-interface)
    - [Command Line](#command-line)
  - [How to Reproduce the Results in the Paper](#how-to-reproduce-the-results-in-the-paper)
    - [Explanation of the Parameters](#explanation-of-the-parameters)
  - [Publication](#publication)
  - [License](#license)

<br/>

## Installation

- Clone the Git repository.
- We highly recommend you to use conda platform for installing dependencies properly. After installation of appropriate conda version for your operating system, create and activate conda environment with dependencies as below:

  ```
  conda create --name your_env --file requirements.txt
  ```

- Download the datasets from [here](https://drive.google.com/drive/folders/1YF-lgvPsv5Xt_dhjr21QUCOl6z1iV9ZJ?usp=sharing). Uncompress the "input_data.zip" file and move the contents into the input_data folder of clonned repo. "output.zip" file contains all output files generated by source code. If you want to skip some steps of the process when re-producing results, we strongly recommend you to uncompress the "output.zip" file and move the contents into the output folder of clonned repo, as some steps use the previous step's output (For example enrichment analysis uses the output of the em algorithm). If you want to run the whole process with an empty output folder, please create "cafa_raw_predictions" and "em_algorithm" folders inside "output" folder.

<br/>

## Descriptions of Folders and Files in the Domain2GO Repository

- **bin** folder includes the source code of Domain2GO. To run the code, please set the location of bin folder as the current working directory.
- **input_data** folder contains various training datasets and **cafa** folder, which contains files related to CAFA evaluation step.
  - **input_data/cafa/benchmark** contains CAFA3 benchmark dataset.
  - **input_data/cafa/predictions** contains propagated and parsed versions of our raw predictions. We include raw prediction generation step in our source code but propagation step is not included since this is performed using https://github.com/CAFA-Challenge/CAFA_evaluation.
- **output** folder contains all output files generated by source code.
  - **output/cafa/benchmark** contains two files that is not created by Domain2GO source code; blast.txt and naive.txt. These predictions are generated as explained in the CAFA experiments, using the Swiss-Prot annotations that existed before the CAFA3 annotation collection period (version date: 2016/09).

<br/>

## Downloading the Finalized Domain and Protein Function Prediction Datasets
- Final Domain2GO mapping set is available at the output folder of this repository. You can also download it together with the protein function prediction set from [here](https://drive.google.com/drive/folders/1YF-lgvPsv5Xt_dhjr21QUCOl6z1iV9ZJ?usp=sharing). "output.zip" file contains all output files generated by source code:
  - "filtered_original.txt" file in this folder contains the resulting Domain2GO mapping set (26,696 associations between 4,742 InterPro domains and 11,742 GO terms). This file contains the following columns:
      - **GO**: GO term ID
      - **GO_aspect**: GO term category
      - **GO_name**: GO term name
      - **Interpro**: InterPro domain accession
      - **n**: number of co-occurrences of the domain and the GO term 
      - **n_go**: number of proteins annotated with the GO term
      - **n_ip**: number of proteins annotated with the domain
      - **s**: co-occurrence similarity score (association probability)
  - "protein_function_predictions.txt" file in this folder contains the protein function prediction set (5,046,060 GO term predictions for 291,519 proteins and 11,742 GO terms) obtained by propagating the finalized Domain2GO mapping set to proteins annotated with the domains.

<br/>

## Predicting the Functions of a New Protein Sequence Using Domain2GO Mappings

You can generate function predictions for a query protein using the final Domain2GO mapping set. Please note that the following program is designed to generate predictions for a single protein due to the extended runtime of InterProScan. If you need predictions for multiple UniProtKB/Swiss-Prot proteins, we recommend utilizing our comprehensive protein function prediction dataset available [here](https://drive.google.com/drive/folders/1YF-lgvPsv5Xt_dhjr21QUCOl6z1iV9ZJ?usp=sharing). The file "protein_function_predictions.txt" within this folder contains function predictions for a substantial collection of 291,519 UniProtKB/Swiss-Prot proteins.

To generate function predictions for a single protein sequence, you can either use user interface or command line.

### User Interface

You can access the user interface of Domain2GO from [here](https://huggingface.co/spaces/HUBioDataLab/Domain2GO). You can query a protein sequence by entering it directly or by uploading a FASTA file. Upon entering the query sequence, the program will identify domains within the sequence and generate function predictions based on the final Domain2GO mapping set. The output will be available for download as a CSV file. For more information please refer to the User Guide page of the user interface.


### Command Line

To generate function predictions for a protein sequence using command line, please use the following command:

```
python run_domain2go.py --mapping_path=../output
```

Upon executing the command, you will be prompted to provide an email and a query protein sequence or the path to a FASTA file containing the sequence. Email is required for InterProScan to notify you when your job is done. Your email will not be used for any other purpose.

You have two options for entering the query protein sequence:


**1. Direct Sequence Entry**

```
Please enter the protein sequence or fasta file location: MEYGKVIFLFLLFLKSGQGESLENYIKTEGASLSNSQKKQFVASSTEECEALCEKETEFVCRSFEHYNKEQKCVIMSENSKTSSVERKRDVVLFEKRIYLSDCKSGNGRNYRGTLSKTKSGITCQKWSDLSPHVPNYAPSKYPDAGLEKNYCRNPDDDVKGPWCYTTNPDIRYEYCDVPECEDECMHCSGENYRGTISKTESGIECQPWDSQEPHSHEYIPSKFPSKDLKENYCRNPDGEPRPWCFTSNPEKRWEFCNIPRCSSPPPPPGPMLQCLKGRGENYRGKIAVTKSGHTCQRWNKQTPHKHNRTPENFPCRGLDENYCRNPDGELEPWCYTTNPDVRQEYCAIPSCGTSSPHTDRVEQSPVIQECYEGKGENYRGTTSTTISGKKCQAWSSMTPHQHKKTPDNFPNADLIRNYCRNPDGDKSPWCYTMDPTVRWEFCNLEKCSGTGSTVLNAQTTRVPSVDTTSHPESDCMYGSGKDYRGKRSTTVTGTLCQAWTAQEPHRHTIFTPDTYPRAGLEENYCRNPDGDPNGPWCYTTNPKKLFDYCDIPQCVSPSSFDCGKPRVEPQKCPGRIVGGCYAQPHSWPWQISLRTRFGEHFCGGTLIAPQWVLTAAHCLERSQWPGAYKVILGLHREVNPESYSQEIGVSRLFKGPLAADIALLKLNRPAAINDKVIPACLPSQDFMVPDRTLCHVTGWGDTQGTSPRGLLKQASLPVIDNRVCNRHEYLNGRVKSTELCAGHLVGRGDSCQGDSGGPLICFEDDKYVLQGVTSWGLGCARPNKPGVYVRVSRYISWIEDVMKNN
```

Additionally, provide a name for the protein sequence:

```
Please enter a name for the protein sequence: sp|O18783|PLMN_NOTEU
```

**2. FASTA File Entry**

```
Please enter the protein sequence or fasta file location: ../input_data/example_protein_query.fasta
```

Once you've entered the query sequence, the program will identify domains within the sequence and generate function predictions based on the final Domain2GO mapping set. The output will be saved in the specified path.

Here is an example output:

| protein_name           | GO_ID      | GO_term                                                      | GO_category        | sequence_region                 | probability | domain_accession | domain_name |
| ---------------------- | ---------- | ------------------------------------------------------------ | ------------------ | ------------------------------- | ----------- | ---------------- | ----------- |
| sp\|O18783\|PLMN_NOTEU | GO:0042730 | fibrinolysis                                                 | biological_process | 101-264;273-354;369-450;473-557 | 0.224       | IPR000001        | Kringle     |
| sp\|O18783\|PLMN_NOTEU | GO:0004175 | endopeptidase activity                                       | molecular_function | 576-804                         | 0.293       | IPR001254        | Trypsin_dom |
| sp\|O18783\|PLMN_NOTEU | GO:0004252 | serine-type endopeptidase activity                           | molecular_function | 576-804                         | 0.598       | IPR001254        | Trypsin_dom |
| sp\|O18783\|PLMN_NOTEU | GO:0008233 | peptidase activity                                           | molecular_function | 576-804                         | 0.223       | IPR001254        | Trypsin_dom |
| sp\|O18783\|PLMN_NOTEU | GO:0008236 | serine-type peptidase activity                               | molecular_function | 576-804                         | 0.593       | IPR001254        | Trypsin_dom |
| sp\|O18783\|PLMN_NOTEU | GO:0016825 | hydrolase activity, acting on acid phosphorus-nitrogen bonds | molecular_function | 576-804                         | 0.584       | IPR001254        | Trypsin_dom |
| sp\|O18783\|PLMN_NOTEU | GO:0017171 | serine hydrolase activity                                    | molecular_function | 576-804                         | 0.584       | IPR001254        | Trypsin_dom |
| sp\|O18783\|PLMN_NOTEU | GO:0031638 | zymogen activation                                           | biological_process | 576-804                         | 0.259       | IPR001254        | Trypsin_dom |
| sp\|O18783\|PLMN_NOTEU | GO:0070011 | peptidase activity, acting on L-amino acid peptides          | molecular_function | 576-804                         | 0.223       | IPR001254        | Trypsin_dom |

<br/>

## How to Reproduce the Results in the Paper

To generate the Domain2GO mapping set, please set the location of bin folder as the current working directory, and run the main_training script with the desired settings as below:


```
python main_training.py --em skip --enrichment skip --cafa_eval skip
```

### Explanation of the Parameters
**--em** EM algorithm mode (default: skip); EM algorithm can be performed to calculate only theta (likelihood) scores by setting --em only_theta, or to calculate both theta and E (evidence) by setting --em full_mode.

**--enrichment** whether or not to perform enrichment analysis (default:skip, can be set to any other value to perform EA)

**--cafa_eval** whether or not to perform CAFA evaluation (default:skip, can be set to any other value to perform evaluation)

<br/>

## Publication

Please refer to our publication for more information. If you use the Domain2GO method or the datasets provided in this repository, please cite this paper:

Ulusoy E., Doğan T. (2024) Mutual annotation-based prediction of protein domain functions with Domain2GO. _Protein Science, 33_(6), e4988. [Link](https://doi.org/10.1002/pro.4988)

<br/>

## License
Copyright (C) 2022 HUBioDataLab

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see http://www.gnu.org/licenses/.


