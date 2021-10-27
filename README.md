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

Скриншоты из отчета:
![](https://github.com/kseniashilova/hse21_hw1/blob/main/images/1.PNG) 
![](https://github.com/kseniashilova/hse21_hw1/blob/main/images/2.PNG)
![](https://github.com/kseniashilova/hse21_hw1/blob/main/images/3.PNG)
  
    
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
pscp -r -P 5222  -i "private_key.ppk" kashilova@92.242.58.92:/home/kashilova/sub/trimmed_fastqc C:\Users\Пользователь\Desktop\биоинформатика
pscp -r -P 5222  -i "private_key.ppk" kashilova@92.242.58.92:/home/kashilova/sub/trimmed_multiqc C:\Users\Пользователь\Desktop\биоинформатика
```  
  
Скриншоты из отчета:
![](https://github.com/kseniashilova/hse21_hw1/blob/main/images/4.PNG) 
![](https://github.com/kseniashilova/hse21_hw1/blob/main/images/5.PNG)
![](https://github.com/kseniashilova/hse21_hw1/blob/main/images/6.PNG)
  
  
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

Проанализируем контиги с помощью программы в гугл коллаб  

https://github.com/kseniashilova/hse21_hw1/blob/main/src/Shilova_contigs.ipynb


Выпишем ответы на вопросы:  
**1) Общее количество контигов = 607**   
**2) Их общая длина = 3923847**  
**3) Длина самого длинного контига = 179307**  
**4) N50 = 47611**  

С помощью программы “platanus scaffold” собираем скаффолды из контигов, а также из подрезанных чтений
```
time platanus scaffold -o Poil -t 1 -c Poil_contig.fa -IP1 trimmed_fastq/oil_R1_sub.fastq.trimmed trimmed_fastq/oil_R2_sub.fastq.trimmed -OP2 trimmed_fastq/oilMP_S4_L001_R1_001_sub.fastq.int_trimmed trimmed_fastq/oilMP_S4_L001_R2_001_sub.fastq.int_trimmed 2> scaffold.log
```
Код в гугл коллабе анализирует результирующий файл Poil_scaffold.fa

https://github.com/kseniashilova/hse21_hw1/blob/main/src/Shilova_contigs.ipynb

Выпишу ответы на вопросы  
**1) Общее количество = 70**   
**2) Общая длина = 3872834**  
**3) Длина самого длинного контига = 3833234**  
**4) N50 = 3833234** 

Пусть самая длинная последовательность лежит в отдельном файле
```
echo scaffold1_len3833234_cov231 > tmp.txt
seqtk subseq Poil_scaffold.fa tmp.txt > scaffold1_len3833234_cov231.fasta
```
Посчитаем количество гэпов (участков, состоящих из букв N) и их общую длину  
Количество участков (где 1 или более буква N)
```
 grep -o "N\+" scaffold1_len3833234_cov231.fasta | wc -l
```
Получилось **63** участка

Количество символов N:
```
 grep -o "N" scaffold1_len3833234_cov231.fasta | wc -l
```
Получилось **6163** символа

Запустим platanus gap close
```
time platanus gap_close -o Poil -t 1 -c Poil_scaffold.fa -IP1 trimmed_fastq/oil_R1_sub.fastq.trimmed trimmed_fastq/oil_R2_sub.fastq.trimmed -OP2 trimmed_fastq/oilMP_S4_L001_R1_001_sub.fastq.int_trimmed trimmed_fastq/oilMP_S4_L001_R2_001_sub.fastq.int_trimmed 2> gapclose.log
```

Новая последовательность в новом файле
```
echo scaffold1_cov231 > tmp2.txt
seqtk subseq Poil_gapClosed.fa tmp2.txt > scaffold2_len3833234_cov231.fasta
```
Участки: 
```
grep -o "N\+" scaffold2_len3833234_cov231.fasta | wc -l
```
Получилось **9** участков

Символы:
```
grep -o "N" scaffold2_len3833234_cov231.fasta | wc -l
```
Получилось **1847** символа


### Необязательная часть
Посмотрим, как количество чтений, которые мы берем изначально случайным образом, влияют на качество (космотрим на количество скаффолдов и гэпов)    
Запустим код, который выводит в файл file_extra.txt по три числа за один запуск (количество скаффолдов, количество участков гэпов, общая длина гэпов)  
Запустим код для n1 = {1000000, 2000000, 3000000, 4000000, 5000000} и для n2 = {300000, 600000, 900000, 1200000, 1500000} соответственно 
```
seqtk sample -s1031 oil_R1.fastq n1 > oil_R1_sub.fastq
seqtk sample -s1031 oil_R2.fastq n1 > oil_R2_sub.fastq
seqtk sample -s1031 oilMP_S4_L001_R1_001.fastq n2 > oilMP_S4_L001_R1_001_sub.fastq
seqtk sample -s1031 oilMP_S4_L001_R2_001.fastq n2 > oilMP_S4_L001_R2_001_sub.fastq

mv oilMP_S4_L001_R1_001_sub.fastq sub
mv oilMP_S4_L001_R2_001_sub.fastq sub
mv oil_R1_sub.fastq sub
mv oil_R2_sub.fastq sub 

cd sub 

platanus_trim oil_R1_sub.fastq  oil_R2_sub.fastq
platanus_internal_trim oilMP_S4_L001_R1_001_sub.fastq oilMP_S4_L001_R2_001_sub.fastq

 mv -v *trimmed trimmed_fastq
 mv -v *.fastq fastq
 
 time platanus assemble -o Poil -t 1 -m 8 -f trimmed_fastq/oil_R1_sub.fastq.trimmed  trimmed_fastq/oil_R2_sub.fastq.trimmed 2>assemble.log
 
 time platanus scaffold -o Poil -t 1 -c Poil_contig.fa -IP1 trimmed_fastq/oil_R1_sub.fastq.trimmed trimmed_fastq/oil_R2_sub.fastq.trimmed -OP2 trimmed_fastq/oilMP_S4_L001_R1_001_sub.fastq.int_trimmed trimmed_fastq/oilMP_S4_L001_R2_001_sub.fastq.int_trimmed 2> scaffold.log
 
 grep -c '>' Poil_scaffold.fa >> file_extra.txt
 
 time platanus gap_close -o Poil -t 1 -c Poil_scaffold.fa -IP1 trimmed_fastq/oil_R1_sub.fastq.trimmed trimmed_fastq/oil_R2_sub.fastq.trimmed -OP2 trimmed_fastq/oilMP_S4_L001_R1_001_sub.fastq.int_trimmed trimmed_fastq/oilMP_S4_L001_R2_001_sub.fastq.int_trimmed 2> gapclose.log
 
 grep -o "N\+" Poil_gapClosed.fa | wc -l >> file_extra.txt
 grep -o "N" Poil_gapClosed.fa | wc -l >> file_extra.txt
 
 rm fastq/*
 rm trimmed_fastq/*
 
```

https://github.com/kseniashilova/hse21_hw1/blob/main/src/Graphics_amount_of_reads.ipynb


Построим график, где по оси X будет число чтений, а по оси Y - количество скаффолдов
![](https://github.com/kseniashilova/hse21_hw1/blob/main/images/gr1.PNG) 

Видно, что с увеличением числа чтений уменьшается количество скаффолдов

Построим график, где количество чтений по оси X, а по оси Y - количество пропусков (участков), нормированное на количество скаффолдов (мне кажется, этот критерий лучше, чем смотреть просто количество участков)

![](https://github.com/kseniashilova/hse21_hw1/blob/main/images/gr2.PNG) 

И построим график, где теперь по оси Y - общая длина пропусков нормированная на количество скаффолдов
![](https://github.com/kseniashilova/hse21_hw1/blob/main/images/gr3.PNG) 

Видно, что с увеличением числа чтений улучшается и качество, но при большом количестве чтений качество улучшается медленнее, чем на небольших значениях.
