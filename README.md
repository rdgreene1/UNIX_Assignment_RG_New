# UNIX Assignment

## Data Inspection

### Attributes of `fang_et_al_genotypes`

```
less fang_et_al_genotypes.txt
```
and

```
$ less snp_positions
```
* I took quite a bit of time trying to understand the files, but I noticed similarties between the sample IDs of the fang_et_al_genotypes file and the SNP IDs of the snp_postition files.
* I know we only needed columns 1, 3, and 4 from the snp_positions file so I did:

```
$ cut -f1,3,4 snp_position.txt > column134_snp_position.txt
```

* This gave me a file with only the 3 columns (1, 3, and 4) I wanted to join with the genotype data.
* Before I transposed the genotype data, I wanted to only extract the data that was either teosinte or maize. To do this I used this:

```
$ awk -F'\t' 'NR==1 || ($3 == "ZMPBA") || ($3 == "ZMPIL") || ($3 == "ZMPJA") { print }' fang_et_al_genotypes.txt > teosinte_data.txt
```

and 

```
$ awk -F'\t' 'NR==1 || ($3 == "ZMMIL") || ($3 == "ZMMLR") || ($3 == "ZMMMR") { print }' fang_et_al_genotypes.txt > maize_data.txt
```
* These codes use awk to search the tab delimited file (fang_et_al_genotypes) from in column 3 for the three codes that were given to us for teosinte and maize and returns a file that contains data only for teosinte and maize.
* I wanted to include the column headers in the new files, so I specified the first row of the file is the header row. 
* Now that I had a separate file for both organisms, I transposed both data files using the given code:

```
$ awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
```
* I tried joining the `column134_snp_position.txt` and the `transposed_teosinte_genotypes.txt` several times based on the matching IDs in the first column of each file before I realized I needed to sort the data first.
* I sorted both files but still received an error, which I finally realized was due to the headers.
* I sorted the data without the headers using:

```
$ tail -n +2 transposed_teosinte_data.txt | sort -k1,1 transposed_teosinte_data.txt > sorted_transposed_teosinte_data.txt
```

* This got rid of the header using `tail` and sorted the data based on the contents of the first column.
* I did this for all three files that I needed to join.
* I then joined the files using:

```
$ join -t $'\t' -1 1 -2 1 new_sorted_column134_snp_position.txt sorted_transposed_teosinte_data.txt > joined_teosinte_data.txt

```
* This joined the data based on the first column of both file, again specifying it's a tab delimited file.
* Now that I had joined SNP and genomic data for both orgainsms, I extracted the data for each chromosome.
* Instead of doing each chromosome individually, I figured I could set up a loop that would do all 10 chromosomes in one code, so I did:

```
$ for i in {1..10}; do 
> awk -F'\t' -v chr="$i" '$2 == chr {print > "teosinte_chr_"chr".txt}' joined_teosinte_data.txt 
> done

```
* This code loops from 1 to 10 (each chromosome) based on the value in column 2 and prints the data for those chromosomes in new files that are named with the number of that chromosome.
* Once I had chromosomal data for both organisms, I just needed to sort them and format them properly.
* To do so, I used the following codes:

```
$ for file in teosinte_chr_*.txt; do   sed 's#?/?#?#g' "$file" | sort -t$'\t' -k3 -n > "${file%.txt}_replaced_increasing.txt"; 
> done

```
and 

```
$ for file in teosinte_chr_*.txt; do   sed 's#?/?#-#g' "$file" | sort -t$'\t' -k3 -nr > "${file%.txt}_replaced_decreasing.txt"; 
> done

```
* These two codes were also looped that search for any file that contains the name `teosinte_chr` and replaces the ?/? with either ? or -, and also sorts the file based on the numerical value of column 3 in either the increasing or decreasing `-r` direction.
* Finally, I need the unknown and multiple files for both organisms, so I used the awk command:

```

$ awk -F'\t' '$3 == "unknown" {print}' joined_teosinte_data.txt > teosinte_chr_unknown.txt

```

and 

```

$ awk -F'\t' '$3 == "multiple" {print}' joined_teosinte_data.txt > teosinte_chr_multiple.txt

```


* This final code searchs the 3rd column of the `joined_organism_data.txt` file for the word unknown and multiple and prints new files with those data.
