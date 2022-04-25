# 1. Скачивание данных ChIP-seq.

## 1.1 Ресурсы NCBI GEO и NCBI SRA

Исследователи используют для получения данных ChIP-seq ресурсы NCBI Gene Expression Omnibus (GEO) и NCBI Sequence Read Archive (SRA). В GEO депонированы описания экспериментов ChIP-seq вместе с обработанными данными. В SRA депонированы сырые риды непосредственно. Каждому эксперименту, расположенному в GEO, соответствует эксперимент, расположенный в SRA. 

![image](https://user-images.githubusercontent.com/83860672/165027139-2438de62-a2bc-47c2-a24d-563ebfbec4e2.png)


Эксперименты в GEO имеют иерархию идентификаторов вида **GSEXXXXXX** → **GSMXXXXXX**, где **GSEXXXXXX** - идентификатор Series (серия образцов; в случае данных ChIP-seq обычно соответствует целому эксперименту), а **GSMXXXXXX** - идентификатор Samples (образцы внутри серии; в случае данных ChIP-seq обычно - опытные и контрольные образцы). Эксперименты в SRA имеют аналогичную иерархию идентификаторов вида **SRPXXXXXX** → **SRXXXXXXX** → **SRRXXXXXX**, где **SRPXXXXXX** - идентификатор Project (в случае данных ChIP-seq обычно соответствует целому эксперименту), **SRXXXXXXX** - идентификатор Experiment (в случае данных ChIP-seq соответствует опытным и контрольным образцам), а **SRRXXXXXX** - идентификатор Run (соответствует техническим репликам одного образца). У каждого образца может быть несколько технических реплик (а, соответственно, несколько **SRRXXXXXX**), которые объединяются при анализе.

## 1.2 Получение сырых данных ChIP-seq на примере эксперимента по связыванию ТФ ARF6 в геноме *Arabidopsis thaliana*.

В качестве примера мы будем использовать ChIP-seq эксперимент по связыванию ТФ ARF6 в геноме *Arabidopsis thaliana*. GEO-идентификатор данного эксперимента **GSE51770**, он доступен по [ссылке](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE51770). Описание данного эксперимента выглядит следующим образом:

![image](https://user-images.githubusercontent.com/83860672/165031288-ff785117-b379-41e8-823c-de245c75a5d6.png)

Образец **ARF6 ChIP-seq** соответствует исследуемому образцу, а **GFP ChIP-seq** - контрольному.

Страница эксперимента в SRA выглядит следующим образом:

![image](https://user-images.githubusercontent.com/83860672/165033263-dc202304-6388-40c4-92fd-815d6e3c717a.png)

Необходимая для скачивания информация об идентификаторах образцов находится в таблице **Runs**. Таким образом, сводная информация о каждом из образцов имеет следующий вид:

|Название образца|Тип образца|Идентификатор образца в GEO|Идентификатор образца в SRA|
|---|---|---|---|
|**ARF6 ChIP-seq**|**Immunoprecipitated**|GSM1252254|SRR1019434|
|**GFP ChIP-seq**|**Control**|GSM1252255|SRR1019435|

Скачивание образцов осуществляется при помощи модуля **_fastq-dump_** пакета **_sra-tools_**. Данный инструмент доступен на вычислительном кластере ЦКП "Биоинформатика". Для скачивания данного эксперимента на кластер необходимо активировать окружение **students** при помощи следующей команды:

`conda activate students`

Затем необходимо переместиться в свою директорию:

`cd ../../hpcws/dolgikh-students/*username*`

В данной директории создать скрипт для скачивания **download.sh**, который должен иметь следующий вид:

```shell
#!/bin/bash
#SBATCH --mem 5GB
#SBATCH -p  common
#SBATCH --cpus-per-task=4
#SBATCH -n 1
#SBATCH -t 48:00:00

for i in SRR1019434 SRR1019435
do
fastq-dump ${i}
done
mv SRR1019434.fastq ARF6_ChIP-seq.fastq
mv SRR1019435.fastq GFP_ChIP-seq.fastq
```

В результате работы данного скрипта в директории появятся необходимые файлы. Если одному образцу соответствует более одного **Run**, то их необходимо объединить при помощи следующей команды:

`cat SRR00001.fastq SRR00002.fastq > merged.fastq`

## 1.3 Контроль качества и предобработка данных ChIP-seq

Контроль качества осуществляется при помощи пакета **_FastQC_**. Для запуска программы FastQC с исселедуемыми нами образцами необходимо создать аналогичный скрипт **quality_control.sh**, имеющий следующий вид:

```shell
#!/bin/bash
#SBATCH --mem 5GB
#SBATCH -p  common
#SBATCH --cpus-per-task=4
#SBATCH -n 1
#SBATCH -t 48:00:00

for i in ARF6_ChIP-seq GFP_ChIP-seq
do
fastqc ${i}.fastq
done
```

В результате работы данного скрипта в директории появятся **html-файлы**, содержащие стандартный отчет о качестве образцов.

По итогам осуществления процедуры контроля качества необходимо принять решение, нужна ли дальнейшая предобработка данных. Если таковая требуется, необходимо использовать пакет **_Trimmomatic_**. Пример скрипта **trimming.sh** для осуществления процедуры **SLIDINGWINDOW** с исследуемыми нами образцами выглядит следующим образом:

```shell
#!/bin/bash
#SBATCH --mem 5GB
#SBATCH -p  common
#SBATCH --cpus-per-task=4
#SBATCH -n 1
#SBATCH -t 48:00:00

for i in ARF6_ChIP-seq GFP_ChIP-seq
do
trimmomatic SE -threads 4 -phred33 -trimlog ${i}.log ${i}.fastq ${i}_trimmed.fastq SLIDINGWINDOW:4:20 
done
```
Здесь параметр `SE` указывает на то, что протокол подготовки библиотеки - single-end (информацию о протоколе подготовки библиотеки можно посмотреть в GEO в графе *Library* → *Layout*). Параметр `-threads` указывает на число ядер процессора, используемых при вычислении. Параметр `-phred33` указывает, что Phred quality score закодирован при помощи кодировки Phred33 (стандартная для современных секвенаторов Illumina) Параметр `-trimlog` принимает на вход название файла, в который будут записываться логи. Далее указываются названия входных и выходных файлов. Подробнее ознакомиться с синтаксисом программы можно в соответствующем [Tutorial] (http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf).

## 1.4 Задание.

Используя материалы лекций и данного вспомогательного материала:

- Выберете один ChIP-seq эксперимент по связыванию любого ТФ в геноме интересующего вас вида. Используйте ресурсы GEO и SRA.
- Оформите таблицу, отражающую структуру эксперимента по образцу из п. **1.2**.
- Скачайте сырые данные в формате **fastq**, используя **_fastq-dump_**.
- Произведите контроль качества данных, используя **_FastQC_**.
- Произведите предобработку данных при помощи **_Trimmomatic_**, если таковая потребуется.
 
