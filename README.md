# Microbiome_2021
A pipeline for reproducing sequence processing and analysis of metagenomics data produced for Berihu et al ("A framework for the targeted recruitment of crop-beneficial soil taxa based on network analysis of metagenomics data")

Scripts are found in the Scripts folder (https://github.com/ot483/Microbiome_2021/tree/main/Scripts). A single script carrying out steps 1-6 together is described in step 8. Outputs of early steps are used as inputs for steps 9 and 10.
* For each step, example input and output files are available in https://volcanicenter-my.sharepoint.com/:f:/g/personal/shmedina_volcani_agri_gov_il/EicqyHpmSmBJmwgHO0oqJ8MBcxNtvXw0KUs6hYyyQ4wvzA?e=khRLmH


| Step | Description | Script name /<br /> Example Input Files* / <br />Example Output Files* | Example command line |
| :--- | :--- | :--- | :--- |
| 1 | The script determines the taxonomic classification of each contig according to the annotations of its respective genes. The script takes as input a Megan output file with taxonomic annotation for each gene and determines the taxonomic annotation of the corresponding contigs according to most frequent taxonomy of the associated genes. | get_contig_taxonomy.py / get_contig_taxonomy_ranked_NTC_G210.txt NTC_G210.count_tab.matrix / count_table_taxonomy_NTC_G210.contig.genus.txt | python get_contig_taxonomy.py -i NTC_G210-ReadName_to_TaxonPath.txt -t temp_1 -o get_contig_taxonomy_ranked_NTC_G210.txt --get_rank |
| 2 | The script merges the count table (constructed for contigs, Methods) and the taxonomic annotations of the contigs (generated in step 1) and creates a count table based on the selected taxonomic level by merging respective contigs. | prepare _taxonomy_count_table.py / get_contig_taxonomy_ranked_NTC_G210.txt NTC_G210.count_tab.matrix / count_table_taxonomy_NTC_G210.contig.genus.txt | python prepare _taxonomy_count_table.py -o count_table_taxonomy_NTC_G210.contig.genus.txt -c NTC_G210.count_tab.matrix -t get_contig_taxonomy_ranked_NTC_G210.txt -l genus<br /> (-l : specifying taxonomic rank according to ncbi taxonomy  etc species,genus,order,phylum ) |
| 3 | The script takes as input a Megan output file with KEGG functional annotations and filters only entities with EC/ KEGGs KO identifiers. | Filter_enzymatic_functions.pl / NTC_G210-ReadName_to_KeggName_Ultimate.txt / protein_EC_codes_total_NTC_G210.txt | Perl Filter_enzymatic_functions.pl<br /> Change parameters: my $file ="NTC_G210-ReadName_to_KeggName_Ultimate"<br /> open IN ,$file<br /> open (OUT,">protein_EC_codes_total_NTC_G210.txt")<br /> |
| 4 | The script merges the count table with functional annotations retrieved from MEGAN and filtered in Step 3, creating a count table based on the functional keys (EC/KEGGs KO). | prepare_function_count_table.py / protein_EC_codes_total_NTC_G210.txt NTC_G210.count_tab.matrix NTC_G210-ReadName_to_TaxonPath.txt / count_table_ function_NTC_G210.contig.txt | python prepare_function_count_table.py -d protein_EC_codes_total_NTC_G210.txt -t temp_1 -o count_table_ function _NTC_G210.contig.txt -c NTC_G210.count_tab.matrix -p NTC_G210-ReadName_to_TaxonPath_.txt |
| 5 | The script merges the count table according to a taxonomic (Step 2) and functional (Step 4) keys into a double-key (taxonomic/functional) count table. The output file details the taxonomic distribution of reads assigned to each enzymes at a selected taxonomic level. | prepare_function_taxonomy_ctable_contigwise.py/ protein_EC_codes_total_NTC_G210.txt NTC_G210.count_tab.matrix NTC_G210-ReadName_to_TaxonPath.txt get_contig_taxonomy_NTC_G210.txt / function_taxonomy_countable_NTC_G210.contig.genus.txt | python  prepare_function_taxonomy_ctable_contigwise.py -d protein_EC_codes_total_NTC_G210.txt -r temp_1 –o function_taxonomy _countable_NTC_G210.contig.genus.txt -c NTC_G210.count_tab.matrix -p NTC_G210-ReadName_to_TaxonPath.txt -t get_contig_taxonomy_NTC_G210.txt -l genus<br /> (-l : specifying taxonomic rank according to ncbi taxonomy etc species,genus,order,phylum ) |
| 6 | The script takes as input the enzyme – diversity table generated at Step 5 and calculates a diversity score for each enzyme, indicating whether it is dominantly present in a specific taxonomic group (at a selected taxonomic level) or widely distributed across microbial community. | calculate_enzyme_diversity_score.py/ function_taxonomy _countable_BjSa_NTC.contig.genus.txt / ECs_with_dominant_taxon.txt, ECs_with_dominant_taxon_knockout.txt, RscriptOutput.csv, RscriptOutput_filter_IdentMostFreq.csv, RscriptOutput_filter_IdentMostFreqbyEnzyme.csv, RscriptOutput_filter_dominance.csv | python calculate_enzyme_diveristy_score.py  function_taxonomy_countable_BjSa_NTC.contig.genus.txt /path/to/input/file/ |
| 7 | A pipeline carrying out together steps 1-6 | Removal_Pipeline.py / NTC_G210-ReadName_to_TaxonPath.txt, NTC_G210.count_tab.matrix, NTC_G210-ReadName_to_KeggName_Ultimate.txt / output of steps 1-6 | Three input files are required - ReadName_to_TaxonPath.txt, count_tab.matrix and ReadName_to_KeggName_Ultimate.txt" <br /> There are 4 arguments that have to be stated by the following order: ReadName_to_TaxonPath.txt<br /> count_tab.matrix<br /> ReadName_to_KeggName_Ultimate<br /> BaseFolder - where the script and input files are. python Removal_Pipeline.py ReadName_to_TaxonPath.txt count_tab.matrix ReadName_to_KeggName_Ultimate "/Path/To/InputFiles/" |
| 8 | The script determines differential abundance between entities whose relative abundance is described in a count table. The script will produce, for each treatment, a table file with the results of the differential analysis and a plot showing the distribution of differentially represented elements. | using_edgeR_generic.R/ count_table_taxonomy_genus_ BjSaVSNTC.txt sample_metadata.txt (Metadata file, with at least four columns: samples ID, name of the rootstock, name of treatment and name of rootstock treatment) contrasts_used_BjSa_NTC.txt / Perfix_for_output_files | Rscript using_edgeR_generic.R. count_table_taxonomy_genus_ BjSaVSNTC.txt sample_metadata.txt Perfix_for_output_files Treatment_Rootstock contrasts_used_BjSa_NTC.txt 0.05<br /> Parameters: count table, metadata file, prefix for output files, compared treatments as specified by column names in metadata file, text file with the treatments being compared (e.g., NTC vs BjSa), FDR threshold.<br /> 0.05-FDR threshold used to determine significance. |
| 9 | The script tool for predicting metabolic activities in microbial communities based on network-based interpretation of assembled and annotated metagenomics data. The algorithm takes as input an EdgeR output file that provides information on the differential abundance of enzymatic reactions in two different treatments (Step 8). Enzymes are classified as associated with Treatment_1, Treatment_2 or not associated. The algorithm generates as output: (i) Lists of differentially abundant enzymes and their pathway association. (ii) Prediction of environmental resources that are unique to each treatment and their pathway association. (iii) Prediction of environmental compounds that are produced by the microbial community and pathway association of compounds that are treatment-specific. (iv) Network visualization of enzymes, environmental resources and produced compounds that are treatment specific (2 & 3D). | Code and instructions are available at https://github.com/ot483/NetCom |  |
| 10 | The script conducts community 'knockouts' simulations in which selected taxonomic groups are removed from the community network by eliminating enzymes associated with the group (according to the scores in step 6). The impact of the removal group is estimated according to differences in the number of metabolites between the network expanded from the truncated enzyme set, and the reference meta-network that includes the full set of enzymatic functions. | Sim4RemovalNet.py / Knock_out_file_step_6_update_name env.txt dictionary files / Perfix_for_output_files | Python Sim4RemovalNet.py<br /> Input/output files names can be updated from the script.<br /> Knock_out_file_step_6_update_name is the output of step 6.<br /> Env.txt is one of the outputs of step 9 – product (ii) Prediction of environmental resources that are unique to each treatment and their pathway association. The script requires dictionary files provided in the folder: compounds_lables_jun_1.txt, ec_reac_mapping_jun.txt full_enzymes_labels_jun.txt, reactions_3_balanced.txt |
| 11 | Visualization. The script takes as input lists of enzymes, predicted environmental resources and unique compounds and produces a network representation. | Visualize_network.py / allCompounds_BjSa.txt allCompounds_NTC.txt Compounds_BjSa_Order.txt EC_ALL.txt ECs_BjSa.txt ECs_NTC.txt enzymes_BjSa_order.txt pathways_BjSa_order.txt seeds_BjSa.txt seeds_NTC.txt compounds_lables_jun_1.txt ec_reac_mapping_jun.txt full_enzymes_labels_jun.txt reactions_3_balanced.txt / BjSa_order_removal_network.pdf BjSa_oder_removal_network.png |  |
