# hse21_hw1
Сборка генома бактерии, выделенной из воды с нефтью, на основании парно-концевых (paired-end, PE) и чтений типа mate-pairs (MP).

0. Создание папки и символических ссылок

Изначально находимся в домашней дирректории: ~ <=> /home/vavasileva_3
```javascript
mkdir HW1_assembly
cd ~/HW1_assembly/
```
Создаем символические ссылки на используемые файлы:
```javascript
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```
1. Случайный выбор 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs:
random seed = 621
```javascript
seqtk sample -s621 oil_R1.fastq 5000000 > sub1_oil_R1.fastq
seqtk sample -s621 oil_R2.fastq 5000000 > sub2_oil_R2.fastq  
seqtk sample -s621 oilMP_S4_L001_R1_001.fastq 1500000 > sub3_oilMP_S4_L001_R1_001.fastq
seqtk sample -s621 oilMP_S4_L001_R2_001.fastq 1500000 > sub4_oilMP_S4_L001_R2_001.fastq
```
2. Оценка качества наших подвыборок исходных чтений.

запуск программы fastqc последовательно для четырёх наборов чтений:
```javascript
mkdir fastqc
ls sub* | xargs -P 1 -tI{} fastqc -o fastqc {}
```
объединение четырёх отчётов из папки fastqc в один отчёт:
```javascript
mkdir multiqc
multiqc -o multiqc fastqc
```
Анализируя отчёт, понимаем, что среди парных чтений наблюдаются нуклеотиды недосточного качества, а также остались адаптеры => необходимо выполнить следующий пункт.

3. Подрезание чтений по качеству и удаление праймеров.
```javascript
platanus_trim sub1_oil_R1.fastq sub2_oil_R2.fastq
platanus_internal_trim sub3_oilMP_S4_L001_R1_001.fastq sub4_oilMP_S4_L001_R2_001.fastq
```
4. Перемещаем подрезанные чтений в отдельную папку:
```javascript
mkdir trimmed_fasta
mv -v fastq/*trimmed trimmed_fasta
```
5. Оценка качества подвыборок подрезанных чтений.
запуск программы fastqc последовательно для четырёх наборов подрезанных чтений:
```javascript
mkdir trimmed_fastqc 
ls trimmed_fasta/* | xargs -P 1 -tI{} fastqc -o trimmed_fastqc {}
```
объединение четырёх отчётов из папки fastqc в один отчёт:
```javascript
mkdir trimmed_multiqc
multiqc -o trimmed_multiqc trimmed_fastqc
```
6. Используем platanus assemble, чтобы собрать контиги из подрезанных чтений: 
```javascript
time platanus assemble -o Poil -t 1 -m 8 -f trimmed_fasta/sub1_oil_R1.fastq.trimmed trimmed_fasta/sub2_oil_R2.fastq.trimmed 2> assemble.log
```
Подсчитать сколько всего фрагментов(контигов)
```javascript
grep '^>'Poil_config.fa | wc -l
```
7. Запускаем platanus scaffold - собираем контиги в скафолды.
```javascript
time platanus scaffold -o Poil -t 1 -c Poil_contig.fa -IP1 trimmed_fasta/sub1_oil_R1.fastq.trimmed trimmed_fasta/sub2_oil_R2.fastq.trimmed -OP2 trimmed_fasta/sub3_oilMP_S4_L001_R1_001.fastq.int_trimmed trimmed_fasta/sub4_oilMP_S4_L001_R2_001.fastq.int_trimmed
```
Подсчитаем число скаффолдов:
```javascript
grep '^>' Poil_scaffold.fa | wc -l
```
8. Запускает программу platanus gap_close, чтобы закрыть gaps.
```javascript
time platanus gap_close -o Poil -t 1 -c Poil_scaffold.fa -IP1 trimmed_fasta/sub1_oil_R1.fastq.trimmed trimmed_fasta/sub2_oil_R2.fastq.trimmed -OP2 trimmed_fasta/sub3_oilMP_S4_L001_R1_001.fastq.int_trimmed trimmed_fasta/sub4_oilMP_S4_L001_R2_001.fastq.int_trimmed 2> gapclose.log
```
10. Вытащим самый длинный контиг из файла с уже закрытыми неопределённостями.
```javascript
echo scaffold1_cov231 > _tmp.txt
seqtk subseq Poil_gapClosed.fa _tmp.txt > oil_genome.fna
```
