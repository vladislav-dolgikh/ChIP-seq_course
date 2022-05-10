# Выравнивание сырых прочтений на референсный геном.

Выравнивание сырых прочтений осуществляется при помощи программ **_Bowtie_** (короткие прочтения длиной до 50 п.н.) или **_Bowtie2_** (длинные прочтения длиной до 1000 п.н.). Процедура выравнивания включает в себя два этапа:

1. Построение геномного индекса (строится один раз);
2. Выравнивание прочтений с использованием геномного индекса.

## 1.1 Построение геномного индекса

Геномный индекс строится при помощи подпрограмм **bowtie-build**/**bowtie2-build**, которые получают на вход файл референсного генома в формате **fasta**. Для построения индекса можно использовать скрипт **index.sh**, который может иметь следующий вид (здесь и далее в теле скрипта будут присутствовать примеры команд для запуска **_Bowtie_** и **_Bowtie2_**):

```shell
#!/bin/bash
#SBATCH --mem 5GB
#SBATCH -p  common
#SBATCH --cpus-per-task=4
#SBATCH -n 1
#SBATCH -t 48:00:00

bowtie-build reference_genome.fasta reference_index ##if you use Bowtie1
bowtie2-build reference_genome.fasta reference_index ##if you use Bowtie2
```

## 1.2 Выравнивание прочтений на референсный геном.

Для выравнивания прочтений на референсный геном можно использовать скрипт **align.sh**, содержащий цикл, внутри которого поочередно запускается выравнивание двух образцов, скачанных ранее:

```shell
#!/bin/bash
#SBATCH --mem 5GB
#SBATCH -p  common
#SBATCH --cpus-per-task=4
#SBATCH -n 1
#SBATCH -t 48:00:00

#Bowtie
for i in control treat
do
	bowtie –S reference_index ${i}.fastq > ${i}.sam 	
done

#Bowtie2
for i in control treat
do
	
	bowtie2 [options] -x reference_index -U ${i}.fastq -S ${i}.sam 
done
```

