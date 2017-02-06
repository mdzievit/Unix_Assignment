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
4. Next I wanted to subset the data for the maize groups:
	```
	$ awk -F "\t" -v OFS="\t" 'NR==1 || $3 == "ZMMIL" || $3 == "ZMMLR" || $3 == "ZMMMR" ' file
	```
	- I sent the standard output to a new file: **maize_genotypes.txt**
	- I counted the number of lines in the new file to make sure it matched the expected number of groups
	```
	$ wc maize_genotypes.txt
	```
5. Next, I wanted to subset the data for the teosinte groups:
	- I sent the standard output to a new file: **teosinte_genotypes.txt**
	- I counted the number of lines in the new file to make sure it matched the expected number of groups (975 vs 975) which includes the header