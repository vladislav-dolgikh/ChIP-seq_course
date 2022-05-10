# 2. Выравнивание сырых прочтений на референсный геном.

Выравнивание сырых прочтений осуществляется при помощи программ **_Bowtie_** (короткие прочтения длиной до 50 п.н.) или **_Bowtie2_** (длинные прочтения длиной до 1000 п.н.). Процедура выравнивания включает в себя два этапа:

1. Построение геномного индекса (строится один раз);
2. Выравнивание прочтений с использованием геномного индекса.

## 2.1 Построение геномного индекса

Геномный индекс строится при помощи подпрограмм **bowtie-build**/**bowtie2-build**, которые получают на вход файл референсного генома в формате **fasta**. Для построения индекса можно использовать скрипт **index.sh**, который может иметь следующий вид (здесь и далее в теле скрипта будут присутствовать примеры команд для запуска **_Bowtie_** и **_Bowtie2_**):

```shell
#!/bin/bash
#SBATCH --mem 5GB
#SBATCH -p common
#SBATCH --cpus-per-task=4
#SBATCH -n 1
#SBATCH -t 48:00:00

bowtie-build reference_genome.fasta reference_index ##if you use Bowtie1
bowtie2-build reference_genome.fasta reference_index ##if you use Bowtie2
```

## 2.2 Выравнивание прочтений на референсный геном.

Для выравнивания прочтений на референсный геном можно использовать скрипт **align.sh**, содержащий цикл, внутри которого поочередно запускается выравнивание двух образцов, скачанных ранее:

```shell
#!/bin/bash
#SBATCH --mem 5GB
#SBATCH -p common
#SBATCH --cpus-per-task=4
#SBATCH -n 1
#SBATCH -t 48:00:00

#Bowtie
for i in ARF6_ChIP-seq GFP_ChIP-seq
do
	bowtie –S reference_index ${i}.fastq > ${i}.sam 	
done

#Bowtie2
for i in ARF6_ChIP-seq GFP_ChIP-seq
do
	bowtie2 [options] -x reference_index -U ${i}.fastq -S ${i}.sam 
done
```
## 2.3 Преобразование текстовых SAM-файлов в бинарный формат BAM.

Разработчики программы **_MACS3_**, осуществляющей поиск пиков, рекомендуют использовать в качестве входных данных выровненные прочтения в бинарном формате **bam**. Для преобразования **sam** в **bam** используется инструмент **_Samtools_**. Пример скрипта **sam_to_bam.sh**, осуществляющего подобное преобразование:

```shell
#!/bin/bash
#SBATCH --mem 5GB
#SBATCH -p common
#SBATCH --cpus-per-task=4
#SBATCH -n 1
#SBATCH -t 48:00:00

for i in ARF6_ChIP-seq GFP_ChIP-seq
do 
	samtools view -Sb ${i}.sam > ${i}.bam
done
```
# 3. Получение пиков ChIP-seq

Получение пиков осуществляется при помощи программы **_MACS3_**. Процедура включает в себя два этапа:

1. Определение среднего размера фрагмента исследуемого образца;
2. Определение пиков.

Для запуска **_MACS3_** можно использовать скрипт **peak_calling.sh**.

```shell
#!/bin/bash
#SBATCH --mem 5GB
#SBATCH -p common
#SBATCH --cpus-per-task=4
#SBATCH -n 1
#SBATCH -t 48:00:00

macs3 ARF6_ChIP-seq.bam
# результат – средний размер фрагмента (d), число

macs3 callpeak –t ARF6_ChIP-seq.bam –c GFP_ChIP-seq.bam --nomodel --extsize <d> –g <genome_size>
# -g значение эффективного размера генома 
# (g=2.7e9 для H. sapiens
# g= 1.87e9 для M. musculus
# g=1.19e8 для A. thaliana)

```
## 3.1 Задание

1. Проведите выравнивание и поиск пиков в подобранных вами образцах.
