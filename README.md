###Matt Dzievit
###EEOB546X
###Spring 2017
###Unix Assignment

_______


###**Data Inspection**
1. Inspected both files for the number of columns and rows
	- Fang_genotypes file
		- 2782 lines (used wc -l) 
		- 986 columns (used ```$ awk -F "\t" '{print NF; exit}'```
		- 11MB in size (used ```$du -h```)
	- snp_positions file
		- 984 lines
		- 15 columns
		- 84k
2. Inspected both files for their column headers in order to determine how to join the files
	- There were different number of column headers and no common column from which to join
	- Determined that the fang_genotypes file needs to be transformed in order to join
	- Before transforming, subsetting needs to be done on the maize and teosinte groups (transforming loses the ability to subset)
3. Number of different groups within the file
	- I wanted to know how many different groups were in the file and if I was going to be using all of the since we needed to subset
	```
	$ awk '{print $3}' fang_et_al_genotypes.txt | uniq | wc
	```
	- Now that I know how many groups, I wanted to see how many lines were in each group. This way when I subset, I can count the number of lines and make sure I have the correct number of lines.
	```
	$ cut -f 3 fang_et_al_genotypes.txt | uniq -c
	```
	- This counted the occurrence of each group, and since I know which ones I want to subset, I can add up the numbers. Counting the three groups + the header = 1574

###Data Transformation

1. Next I wanted to subset the data for the maize groups:
	```
	$ awk -F "\t" -v OFS="\t" 'NR==1 || $3 == "ZMMIL" || $3 == "ZMMLR" || $3 == "ZMMMR" ' file
	```
	- I sent the standard output to a new file: **Maize_genotypes.txt**
	- I counted the number of lines in the new file to make sure it matched the expected number of groups
	```
	$ wc maize_genotypes.txt
	```
2. Next, I wanted to subset the data for the teosinte groups:
	- I sent the standard output to a new file: **teosinte_genotypes.txt**
	- I counted the number of lines in the new file to make sure it matched the expected number of groups (975 vs 975) which includes the header
3. Data files are ready to be transformed using the supplied program
	- Standard output sent to **t\_Maize_genotypes.txt**
	- Standard output sent to **t\_teosinte_genotypes.txt**
4. Analyze the files for header information and determine how to join with the SNP positions file
	- **t\_maize_genotypes.txt**
		- This file has 1574 columns, and 986 rows. From out previous analysis, we know that the snp_positions file has 984 lines, so there are 2 extra rows in our transformed file.
		- Looking at the head command, we can see that there are rows for Samp_ID, JG_OTU, Group_iD, then what looks like the SNP_ID. I know this is the SNP_ID because row 4 first entry matches the first entry for SNP_ID on the snp_position file
	- Using the code below, I skipped the first 3 header lines and checked the 2 files to make sure they were sorted by SNP_ID
	```
	$ awk 'NR == 1 || NR == 2 || NR == 3; NR > 3 {print $0 | "sort -c"}' file
	```
	- This confirmed the files are sorted by SNP_ID
	- Next, I checked the **snp\_positions.txt** file to make sure it was sorted by SNP_ID, which it was.
	```
	$ awk 'NR == 1 ; NR > 1 {print | "sort"}' snp_position.txt | head -n 3
	```
	- Since the header files of the transformed files is going to get in the way, I am going to remove them, and put it into a seperate header file to rejoin later. Created 2 new files: 
	**noHead\_t\_teosinte\_genotypes.txt** and **noHead\_t\_Maize\_genotypes.txt**
	```
	$ tail -n +4 file > file
	```
	- Did this with the **snp\_position.txt** file to remove the header, and removed excess columns. created the file **nohead\_reduced\_snp\_position.txt**
	```
	$ cut -f 1,3,4 snp_position.txt | tail -n +2 > noHead_reduced_snp_position.txt
	```
	- Compared the lines and both files had the same number of lines: 983
	- I am not going to create a new header file, because I can just take it from the original later.
###Improved Organization
1. Created new folders **Maize** and **Tesointe** to improve organization
	```
	$ mv *Maize* toDest
	```
###Joining Files
1. Combine the files with **snp\_position.txt** file
	- remove the CR from the end of the reduced snp position file
	```
	$ sed 's/\r//' noHead_reduced_snp_position.txt > noCR_noHead_reduced_snp_position.txt
	```
	- Used the join file to combine the 2 files and piped that to sort them by chr and pos, and also piped that to remove snps that are unknown or multiple (non-numeric): created **sorted\_joined\_Maize\_genotypes.txt** and the corresponding teosinte file
	```
	$ join -t $'\t' -1 1 -1 1 noCR_noHead_reduced_snp_position.txt noHead_t_teosinte_genotypes.txt | sort -k2,2n -k3,3n | awk ' function isnum(x){return(x==x+0)} { if (isnum($2) && isnum($3)) print $0 > "rmMS_sorted_joined_teosinte_genotypes.txt";}'
	```
	- Decided to create the reverse file here. Also, replaced the ? character with - here as well	
	```
	$ join -t $'\t' -1 1 -1 1 noCR_noHead_reduced_snp_position.txt noHead_t_teosinte_genotypes.txt | sort -k2,2n -k3,3nr | sed 's/?/-/g' | awk ' function isnum(x){return(x==x+0)} { if (isnum($2) && isnum($3)) print $0 > "rmMS_rev_sorted_joined_teosinte_genotypes.txt";}' 
	```	
###Subsetting Files
1. Subset each file
	- Created 10 files with the following prefix: **teosinte\_genotypes\_chr0 # .txt**
	```
	$ awk '{for (i=1; i<=10; i++) if ($2==i && i != 10) print $0 > "teosinte_genotypes_chr0"i".txt"; else if ($2==i && i == 10) print $0 > "teosinte_genotypes_chr"i".txt";}' rmMS_sorted_joined_teosinte_genotypes.txt
	```
	- Used the same code to create the same files but for maize: **maize\_genotypes\_chr # .txt**
	- Used the column extract/uniq code from data processing part 3 to spot check each file and make sure that only that chr was present
	- Changed file names to subset the reverse folder. Again spot checked to make sure it was in the right order and the characters change **teosinte\_rev\_genotypes\_chr0 # .txt**
	```
	$ awk ' {for (i=1; i<=10; i++) if ($2==i && i != 10) print $0 > "teosinte_rev_genotypes_chr0"i".txt"; else if ($2==i && i == 10) print $0 > "teosinte_rev_genotypes_chr"i".txt";}' rmMS_rev_sorted_joined_teosinte_genotypes.txt
	```
###Final Location of Files
1. Two folders exits **Maize** and **Teosinte**. Within these folders are 20 files, 10 with **rev** in the same are in reverse order, 10 without are in forward order.
2. Included in the folder is the header information as that was removed for analysis. 
	- Header for the SNP Information: **teosinte\_combinedHeader.txt**. Did the same for maize.
	```
	$ echo -e "SNP_ID\nChrom\nPos" > teosinte_combinedHeader.txt | awk '{print $1}' teosinte_genotypes.txt | tail -n +2 >> teosinte_combinedHeader.txt
	```
	- Created the genotype information file that we removed from transposed file: **teosinte\_genotype\_info.txt** Did the same thing for maize
	```
	$ awk '{print $1 "\t" $2 "\t" $3}' teosinte_genotypes.txt > teosinte_genotype_info.txt
	```
