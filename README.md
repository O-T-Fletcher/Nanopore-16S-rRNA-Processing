# Nanopore-16S-rRNA-Processing-and-Taxonomy
This is a tutorial style workflow for people who have raw 16S rRNA Nanopore fastq sequence files and want to process those files and assign taxonomy. It uses pre-built packages for simple, reproducible sequence processing. 

This was designed for people with little to no coding experience, or who are new to Nanopore, in mind to make Nanopore 16S rRNA processing and taxonomy assignment easy and accessible for anyone. Each step will attempt to be explained in a way a beginner will understand. If you are at a higher level you may skip to the actual coding steps and ignore the explanations. 

This was created in a Linux terminal. It will likely work just the same with MacOS. Though if you have a Windows, I recommend installing Ubuntu. 

This work was done under the supervision of Dr. Vanessa Fernandes in the Microbial Ecology lab at Florida Atlantic University. Lab: https://moreirafernandeslab.weebly.com/

  ## If you use the following workflow to process your 16S rRNA Nanopore sequences, please use the following citation:
  Fletcher, O. T. (2025). A user-friendly workflow for 16S rRNA Nanopore sequence processing. Zenodo. https://doi.org/10.5281/zenodo.17109600

## Packages you will need to install: miniconda, python >3.6, pip, osfclient, bioconda, conda-forge, Emu, Dorado, cutadapt, seqkit
<br> Always check to make sure you have the most recent version!

<br> If you don't already have these packages / databases, install in this order... 
  <br> - install miniconda: https://www.anaconda.com/docs/getting-started/miniconda/install#linux-2
  <br> - install python >3.6: ```sudo apt install python3.8``` (python github: https://github.com/python)
  <br> - install pip: ```sudo apt-get install python3-pip``` (pip github: https://github.com/pypa/pip)
  <br> - install osfclient: ```pip install osfclient``` (osfclient github: https://github.com/osfclient/osfclient)
  
  <br> - Taxonomy database installation: Please visit the Emu repository (https://github.com/treangenlab/emu) for details and explanation. The section you want is the first chunk of code in the **Installation** section. This workflow will use the SILVA database. Other databases are available, or you can build your own (instructions are on their github). For detailed explanation for beginners:
      <br> * Use the ```cd``` command to go to the location on your computer where you want to install your database. 
      <br> * Use the ```mkdir``` command to create the directory where you want to store the database. Example: ```mkdir silva```. I recommend staying in this directory during the rest of the intallation process. Then...
      <br> ```export EMU_DATABASE_DIR='silva'``` '**silva**' in this case is the directory name I chose in the previous step. Make sure to **adjust this** based on your chosen directory name.
      <br> ```cd ${EMU_DATABASE_DIR}``` this step is likely redundant if you are already there. 
      <br> ```osf -p 56uf7 fetch osfstorage/emu-prebuilt/emu.tar```
      <br> ```tar -xvf emu.tar```

  <br> - Install Emu (and bioconda and conda-forge) (this is also on the Emu github repository): 
  <br> ```conda config --add channels defaults```
  <br> ```conda config --add channels bioconda``` (bioconda github: https://github.com/bioconda)
  <br> ```conda config --add channels conda-forge``` (conda-forge github: https://github.com/conda-forge)
  <br> ```conda create -n emu_env python=3.8``` if you have a later version of Python, make sure to adjust the **python=3.8** line accordingly. 
  <br> ```conda activate emu_env```
  <br> ```conda install emu```

  <br> - Install Dorado.
  <br> First you will need to go to the Dorado github (https://github.com/nanoporetech/dorado) and download the file that is the correct file for your computer type. I recommend putting this file in the same location as your Emu database. If you don't do this, and you're not currently in the directory where you have put this file, the following code will not work and you will need to add the directory location. See Dorado github **Installation** section for details.
  <br> ```tar -xvzf dorado-0.9.6-linux-x64.tar.gz``` **dorado-0.9.6-linux-x64.tar.gz** will need to be changed based on your operating system type / what file you downloaded. 
  
  <br> - install cutadapt: ```conda install -c bioconda cutadapt``` (cutadapt github: https://github.com/marcelm/cutadapt)
  <br> - install seqkit: ```conda install -c bioconda seqkit``` (seqkit github: https://github.com/shenwei356/seqkit)
  <br> conda install -c bioconda nanofilt (https://github.com/wdecoster/nanofilt)

  Restart your terminal when this process is completed.


## What you will start with: 
<br> Demultiplexed raw fastq.gz files from your sequence run. 
<br> You will most likely need to change the names of the files from the associated barcode to your sample name. You should create a csv file with the barcode name in the first column and the associated sample name in the second column. Do not label the columns (ex., cell A1 should be Barcode01, cell B1 should be Sample01). If you created the CSV on a Windows system, you will need to install the following package: ```sudo apt install dos2unix```. This is necessary because Windows creates strange characters in their CSV files that are not compatible with Linux or Linux systems, and it may create strange characters in the renaming process. 
<br> ```cd``` into your folder that contains the barcode files. Put your CSV file in this file.
<br> You will then run ```dos2unix filename.csv```. This makes the CSV compatible with Linux. Remember to replace filename with the name of your CSV file. This step is not needed if you made the CSV with Linux's "LibreOffice"
<br> Run ```while IFS=, read -r old new; do
 mv "$old" "$new";
done < filename.csv``` to change the barcode names to the associated sample names that you entered into your CSV. Remember to replace filename with the name of your CSV file. 

Transfer those demultiplexed files onto the computer and put them inside a folder with a relevant name (recommended). ```cd``` into this directory (folder) in your command line (terminal).
<br> Example: open terminal
<br> ```cd Desktop```
<br> ```cd Nanopore-16S```
<br> ```cd 072025_grass``` This is what is referred to as the "project directory" in the following code. You should remain in your project directory while running this code. 
<br> A folder named *samples* is where all of the fastq.gz files are for the example code. 
<br> **Tip**: if you can't find a directory, use the ```ls``` function to list all of the folders in the location where you are now. If need to know where you are, enter ```pwd```
<br> **Tip**: if you want to leave the directory, use the ```cd..``` command to go back.

### Please note that some of the following steps may take hours, depending in your computers computational power and the amount of data you have. _You cannot let your computer sleep during these steps_. The taxonomy step may take multiple days.

## Concatenate your fastq.gz files
<br> There will be multiple fastq.gz files within each sample/ barcode file. These files need to be concatenated. 
<br> ```for dir in samples/*; do
  if [ -d "$dir" ]; then
    sample=$(basename "$dir");
    gunzip -c "$dir"/*.fastq.gz > "${sample}.merged.fastq";
  fi;
done```
<br> This is saying "for files in the sample folder (```for dir in samples```), unzip (```gunzip```) and concatenate (```-c```) the files", and it will keep each sample seperate. If you're in the "samples" folder (or whatever you chose to name ypour file) you can just put ```for dir in */```... Unzipping is necessary for this step. The other code in this block loops the code, so you don't have to perform this function for each individual barcode / sample. For this to work, you will need to be inside of the folder / directory that has the folder with all of the samples inside, in this case it's labeled **samples**. This is what you will have to change based on what you named the folder. The output will be in your specific project directory, in my case “072025_grass” with the naming convention *SAMPLENAME*.merged.fastq. 
<br> **Note**: You don't have to name the resulting files *SAMPLENAME*.merged.fastq, but if you don't you will need to watch for that in the next step and update the code based on what you named it. **This applies for all of the steps and the input / output names.** 

## Filter based on Quality Score (optional)
<br> NanoFilt is a Nanopore tool that allows you to filter your reads based on the quality score. You can do this during your sequencing run, however you would risk losing a majority of your samples if they are below the quality threshold you applied.
<br> '''for file in *merged.fastq; do sample=$(basename "$file" .merged.fastq); NanoFilt -q 15 < "$file" > "${sample}.q15.fastq"; done'''

## Remove Barcodes Sequences
<br> Dorado is a Nanopore tool that allows you to reference the specific barcoding kit that you used in order to remove the barcpde sequences from your sample sequences. In this case, the barcoding kit used is **SQK-NBD114.96**. You may need to replace this based on your barcoding kit. Not all kits are supported, check that yours is by running ```dorado --help```.
<br> ```for file in *merged.fastq; do
  base_name=$(basename "$file" .merged.fastq);
  /home/fernandeslab/Desktop/Nanopore-16S/dorado-0.9.6-linux-x64/bin/dorado trim
    --sequencing-kit SQK-NBD114.96
    --emit-fastq
    "$file" > "${base_name}_trimmed.fastq";
done```
<br> The output will be in your specific project directory, in this case “072025_grass” with the naming convention *SAMPLENAME*_trimmed.fastq. This code is basically saying "for files that end in **.merged.fastq**, run the Dorado function". It again is written as a loop so you don't have to do it for each individual samples. The **/home/fernandeslab/Desktop/Nanopore-16S/dorado-0.9.6-linux-x64/bin/dorado** will need to be the location of your Dorado file that you downloaded. 

## Remove Primer Sequences
<br> The ```cutadapt``` function removes the primer sequences. If you did not do PCR you can skip this step. This is again looped so you don't have to run it for each sample. **Reminder**: if you are not using the same file naming convention (in this case the input files are named **_trimmed.fastq**) you will need to change where those file names are referenced in the code.
<br> ```for file in *_trimmed.fastq; do base=$(basename "$file" _trimmed.fastq); cutadapt -g AGAGTTTGATCMTGGCTCAG -g CGGTTACCTTGTTACGACTT -o "${base}_cut.fastq" "$file"; done```
<br> **AGAGTTTGATCMTGGCTCAG** (Forward) and **CGGTTACCTTGTTACGACTT** (Reverse) are the primers that I used. You may need to change this part of the code based on which primers you used. The output will be in your specific project directory, in this case “072025_grass” with the naming convention *SAMPLENAME*_cut.fastq. 

## Filter for 16S rRNA full sequence  
<br> The ```seqkit``` function allows you to extract sequences based on a minimum and maximum length. Since the 16S gene is about 1500 base pairs long, it is common to filter for a minimum sequence length of 1300 (```-m 1300```) and a maximum sequence length of 1700 (```-M 1700```). 
<br> ```for file in *cut.fastq; do base_name=$(basename "$file" .fastq); seqkit seq -m 1300 -M 1700 -g "$file" > "${base_name}_filtered.fastq"; done```

## Assign Taxonomy
<br> This step uses Emu to assign taxonomy. 
<br> Emu was chosen because the creators tested their algorithm against other common tools minimap2, kraken2, bracken, nanoclust, centrifuge, metamaps, and qiime2. As of the release in 2022, Emu stands from these other tools because it reduces the number of false positives, a common road block for long-read sequences, and the algorithm is able to tell the difference between genetically similar species. You can find more information on their github or in the creators publication. 

<br> First, you must activate emu:
<br> ```conda activate emu_env```
<br> ```emu_env``` was created earlier in the installation section.

<br> To actually run Emu (on a loop):
<br> ```for file in *_cut_filtered.fastq; do emu abundance --db /home/fernandeslab/Desktop/Nanopore-16S/silva "$file"; done```
<br> The input will be files that end in **_cut_filtered.fastq** and ```--db``` is the location of your database. So **/home/fernandeslab/Desktop/Nanopore-16S/silva** will need to be changed based on where you installed your database and what you named it. 

<br> The output from the above step will be a file for each sample that has the relative abundance of each taxa found. To combine all of the files into one file that will have all of your samples as columns, the taxa as rows, with the data being relative abundance, run:
<br> ```emu combine-outputs /home/fernandeslab/Desktop/Nanopore-16S/grass/072025_grass/results tax_id```
<br> You will need to change **/home/fernandeslab/Desktop/Nanopore-16S/grass/072025_grass/results** to the location of the project directory you have been working in / where all of your files from the last step are. The results folder was automatically created. ```tax_id``` is an option to have the classification get all of the taxonomy information. You can replace tax_id with species, genus, all the way to superkingdom depending on your needs. 

**Note**: When you open your file, it may ask you if you wan't to convert the numbers based on scientific notation. You *must* click yes if you want the relative abundance to add up to 0.1 (100%- which is what it should always add up to for every sample). 


## Licenses and citations used in creation of this workflow 
<br>**Emu**: Kristen D. Curry et al., “Emu: Species-Level Microbial Community Profiling of Full-Length 16S RRNA Oxford Nanopore Sequencing Data,” Nature Methods, June 30, 2022, 1–9, https://doi.org/10.1038/s41592-022-01520-4. 
<br>**Dorado**: Oxford Nanopore Technologies. Dorado (v1.1.1), GitHub. https://github.com/nanoporetech/dorado RRID: SCR_025883
<br>**Cutadapt**: Martin, M. (2011). Cutadapt removes adapter sequences from high-throughput sequencing reads. EMBnet.journal, 17(1), 10–12. https://doi.org/10.14806/ej.17.1.200
<br>**Seqkit**: Wei Shen*, Botond Sipos, and Liuyang Zhao. 2024. SeqKit2: A Swiss Army Knife for Sequence and Alignment Processing. iMeta e191. doi:10.1002/imt2.191.

Dependencies such as Miniconda, pip, conda-forge, bioconda, and osfclient are used for installation and environment management.

