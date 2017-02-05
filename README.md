###Matt Dzievit
###EEOB546X
###Spring 2017
###Unix Assignment

_______


###**Data Inspection**
1. Inspected both files for the number of columns and rows
	- Fang_genotypes file
		- 2782 lines (used wc -l) 
		- 986 columns (used awk -F "\t" '{print NF; exit}'
		- 11MB in size (used du -h)
	- snp_positions file
		- 984 lines
		- 15 columns
		- 84k
2. Inspected both files for their column headers in order to determine how to join the files
	- There were different number of column headers and no common column from which to join
	- Determined that the fang_genotypes file needs to be transformed in order to join
	- Before transforming, subsetting needs to be done on the maize and teosinte groups (transforming loses the ability to subset)
3.   