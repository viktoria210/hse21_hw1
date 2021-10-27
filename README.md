# hse21_hw1
Сборка генома бактерии, выделенной из воды с нефтью, на основании парно-концевых (paired-end, PE) и чтений типа mate-pairs (MP).

0. Создание папки и символических ссылок

Изначально находимся в домашней дирректории: ~ <=> /home/vavasileva_3

    mkdir HW1_assembly
    cd ~/HW1_assembly/

Создаем символические ссылки на используемые файлы:

    ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq

1. Случайный выбор 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs:
random seed = 621

    seqtk sample -s621 oil_R1.fastq 5000000 > sub1_oil_R1.fastq
    seqtk sample -s621 oil_R2.fastq 5000000 > sub2_oil_R2.fastq
    seqtk sample -s621 oilMP_S4_L001_R1_001.fastq 1500000 > sub3_oilMP_S4_L001_R1_001.fastq
    seqtk sample -s621 oilMP_S4_L001_R2_001.fastq 1500000 > sub4_oilMP_S4_L001_R2_001.fastq

2. Оценка качества наших подвыборок исходных чтений.

запуск программы fastqc последовательно для четырёх наборов чтений:

    mkdir fastqc
    ls sub* | xargs -P 1 -tI{} fastqc -o fastqc {}

объединение четырёх отчётов из папки fastqc в один отчёт:

    mkdir multiqc
    multiqc -o multiqc fastqc

Анализируя отчёт, понимаем, что среди парных чтений наблюдаются нуклеотиды недосточного качества, а также остались адаптеры => необходимо выполнить следующий пункт.

3. Подрезание чтений по качеству и удаление праймеров.

    platanus_trim sub1_oil_R1.fastq sub2_oil_R2.fastq
    platanus_internal_trim sub3_oilMP_S4_L001_R1_001.fastq sub4_oilMP_S4_L001_R2_001.fastq

4. Перемещаем подрезанные чтений в отдельную папку:

    mkdir trimmed_fasta
    mv -v fastq/*trimmed trimmed_fasta

5. Оценка качества подвыборок подрезанных чтений.
запуск программы fastqc последовательно для четырёх наборов подрезанных чтений:

    mkdir trimmed_fastqc
    ls trimmed_fasta/* | xargs -P 1 -tI{} fastqc -o trimmed_fastqc {}

объединение четырёх отчётов из папки fastqc в один отчёт:

    mkdir trimmed_multiqc
    multiqc -o trimmed_multiqc trimmed_fastqc
