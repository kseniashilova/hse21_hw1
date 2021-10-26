# hse21_hw1
Создаем символические ссылки на файлы в домашней директории
```
 ls /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
```
Выбираем 5 миллионов чтений типа paired-end и 1,5 миллиона типа mate-pairs
```
seqtk sample -s1031 oil_R1.fastq 5000000 > oil_R1_sub.fastq
seqtk sample -s1031 oil_R2.fastq 5000000 > oil_R2_sub.fastq
seqtk sample -s1031 oilMP_S4_L001_R1_001.fastq 1500000 > oilMP_S4_L001_R1_001_sub.fastq
seqtk sample -s1031 oilMP_S4_L001_R2_001.fastq 1500000 > oilMP_S4_L001_R2_001_sub.fastq
```
Оцениваем качество исходных чтений и получаем общую статистику
```
mkdir fastqc
ls *.fastq | xargs -P 4 -tI{} fastqc -o fastqc {}
```
```
mkdir multiqc
multiqc -o multiqc fastqc
```
Загружаем отчет на локальный компьютер из обычного терминала (вместе с putty скачался файл pscp)
```
pscp -r -P 5222  -i "private_key.ppk" kashilova@92.242.58.92:/home/kashilova/sub/fastqc C:\Users\Пользователь\Desktop\биоинформатика
pscp -r -P 5222  -i "private_key.ppk" kashilova@92.242.58.92:/home/kashilova/sub/multiqc C:\Users\Пользователь\Desktop\биоинформатика
```
Используем platanus
```
platanus_trim oil_R1_sub.fastq  oil_R2_sub.fastq
platanus_internal_trim oilMP_S4_L001_R1_001_sub.fastq oilMP_S4_L001_R2_001_sub.fastq
```
Распределяем файлы по двум папкам - fastq и trimmed_fastq (как на лекции)
```
 mv -v *trimmed trimmed_fastq
 mv -v *.fastq fastq
```
Снова составляем отчет
```
mkdir trimmed_fastqc
trimmed_fastq/* | xargs -P 4 -tI{} fastqc -o trimmed_fastqc {}
```
```
mkdir trimmed_multiqc
multiqc -o trimmed_multiqc trimmed_fastqc
```
Снова скачиваем отчеты в локальную директорию
```
pscp -r -P 5222  -i "private_key.ppk" kashilova@92.242.58.92:/home/kashilova/sub/ftrimmed_astqc C:\Users\Пользователь\Desktop\биоинформатика
pscp -r -P 5222  -i "private_key.ppk" kashilova@92.242.58.92:/home/kashilova/sub/trimmed_multiqc C:\Users\Пользователь\Desktop\биоинформатика
```
С помощью программы “platanus assemble” собираем контиги из подрезанных чтений
```
time platanus assemble -o Poil -t 1 -m 8 -f trimmed_fastq/oil_R1_sub.fastq.trimmed  trimmed_fastq/oil_R2_sub.fastq.trimmed 2>assemble.log
```
Вывод: 
```
real    8m15,576s
user    7m53,083s
sys     0m22,471s
```

Проанализируем контиги с помозью программы в гугл коллаб
